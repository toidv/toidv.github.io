---
layout: post
title:  "Follow Law of Demeter!"
date:   2017-05-16 2:35:40 +0700
categories: ruby
tags: ruby refactor
---
Việc code bẩn code rác là khá phổ biến đối với lập trình viên, mà để thay đổi nó chúng ta cần phải có thời gian 
luyện tập, mình sẽ thông qua series các bài viết chia sẻ kinh nghiệm việc mình luyện tập và apply việc refactor
code làm cho code đẹp hơn. (Trong series này mình sẽ sử dụng ngôn ngữ Ruby để làm ví dụ)

Ở phần 1 này mình sẽ đi vào nguyên lý đầu tiên [Follow Law of Demeter](https://en.wikipedia.org/wiki/Law_of_Demeter). 

Để hiểu xem cụ thể Low of Demeter (LoD) là gì chúng ta sẽ đi vào một ví dụ cụ thể. Giả sử chúng ta có 3 class là
Address, Customer và Order

{% highlight ruby %}
class Address < ActiveRecord::Base
    belongs_to: customer
end

class Customer < ActiveRecord::Base
    has_one: address
    has_many: orders
end

class Order < ActiveRecord::Base
    belongs_to: customer
end
{% endhighlight %}


Vì `Ruby` cho phép chúng ta thông qua quan hệ của các đối tượng để truy cập đến các thuộc tính và phương thức của những đối tượng liên quan. Để hiển thị địa chỉ của khách hàng cho 1 đơn hàng, chúng ta thông thường sẽ xử lý như sau

{% highlight ERB %}
<%= @order.customer.name %>
<%= @order.customer.address.street %>
<%= @order.customer.address.city %>
<%= @order.customer.address.state %>
<%= @order.customer.address.zipcode %>
{% endhighlight %}

Tuy nhiên có một số vấn đề với đoạn code ở trên, ví dụ trong tương lai thay vì một khách hàng chỉ có 1 địa chỉ chung, chúng ta muốn phân tách rõ ràng `billing address` và `shipping address`. Khi đó chúng ta phải thay đổi lại
tất cả các đoạn code liên quan đến địa chỉ cho phù hợp với model mới.

Để tránh vấn đề này chúng ta cần phải tuân theo Law of Demeter (LoD) 
>  Each unit should have only limited knowledge about other units: only units "closely" related to the current unit. Or: Each unit should only talk to its friends; Don't talk to strangers.

Trong Ruby hoặc Java chúng ta có thể hiểu nôm na tuân theo LoD có nghĩa là chúng ta chỉ sử dụng "one dot"
Ví dụ `@order.customer.name` ko tuân theo LoD nhưng `@order.customer_name` tuân theo LoD.

Chúng ta refactor lại model theo để tuân theo LoD

{% highlight ruby %}
class Address < ActiveRecord::Base
    belongs_to: customer
end

class Customer < ActiveRecord::Base
    has_one: address
    has_many: orders

    def street
        address.street
    end

    def city
        address.city
    end

    def state
        address.state
    end

    def zipcode
        address.zipcode
    end
end

class Order < ActiveRecord::Base
    belongs_to: customer

    def customer_name
        customer.name
    end

    def customer_street
        customer.street
    end

    def customer_city
        customer.city
    end

    def customer_state
        customer.state
    end 
    
    def customer_zipcode
        customer.zipcode
    end
end
{% endhighlight %}

Đoạn code hiển thị địa chỉ của đơn hàng được refactor lại như sau
{% highlight ERB %}
<%= @order.customer_name %>
<%= @order.customer_street %>
<%= @order.customer_city %>
<%= @order.customer_state %>
<%= @order.customer_zipcode %>
{% endhighlight %}

Cách tiếp cận này làm cho model tràn ngập các wrapper method. Khi có yêu cầu thay đổi chúng ta phải maintain tất cả các wrapper đó. Mặc dù
với hướng tiếp cận này chúng ta ít phải refactor code hơn so với việc update tất cả các views thông qua `@order.customer.address.street` tuy nhiên
chúng ta nên tránh cách tiếp cận này.

Trong `Ruby` chúng ta có thể giải quyết vấn đề này thông qua `:delegate`

{% highlight ruby %}
class Address < ActiveRecord::Base
    belongs_to :customer
end

class Customer < ActiveRecord::Base
    has_one :address
    has_many: orders

    delegate :street, :city, :state, :zipcode, :to => :address
end

class Order < ActiveRecord::Base
    belongs_to :customer

    delegate :name, :street, :city, :state, :zipcode, :to => :customer, :prefix => true
end
{% endhighlight %}

Đoạn code ở view của chúng ta vẫn ko thay đổi tuy nhiên model của chúng ta gọn hơn rất nhiều
{% highlight ERB %}
<%= @order.customer_name %>
<%= @order.customer_street %>
<%= @order.customer_city %>
<%= @order.customer_state %>
<%= @order.customer_zipcode %>
{% endhighlight %}



![Refer]({{ site.url }}/assets/images/keep-calm-and-happy-coding.jpg){:class="img-responsive"}