---
layout: post
title: Variable trong Ruby
excerpt: "Một khái niệm cơ bản trong ngôn ngữ lập trình đó  là Biến. Bạn có thể nghĩ biến như là một từ hoặc một tên mà nó có thể lưu giữ một giá trị."
modified: 2016-05-01
tags: [Variable, bien, ruby]
comments: true
---
### Biến là gì.

Một khái niệm cơ bản trong ngôn ngữ lập trình đó  là Biến. Bạn có thể nghĩ biến như là một từ hoặc một tên mà nó có thể lưu giữ một giá trị.
### Các loại biến trong Ruby.

Theo như mình được biết thì trong Ruby có tổng cộng 5 loại biến.

*  Biến cục bộ (local variable)
*  Biến toàn cục (global variable)
*  Biến đối tượng (instance variable)
*  Biến lớp (class variable)
*  Hằng (constant)

#### Biến cục bộ (local variable)

Phạm vi sử dụng của biến cục bộ phụ thuộc vào vị trí của biến khi khai báo.

``` ruby

#irb 
local_variable = 'Toi nam ngoai method.'
def variable_scope
    puts local_variable = 'Toi nam trong method.'
end

variable_scope #Toi nam trong method
puts local_variable #Toi nam ngoai method
```

Như vậy ta thấy, mặc dù có cùng tên nhưng giá trị in ra là khác nhau, vì biến local_variable ở trong phương thức variable_scope không có liên quan gì với biến local_variable ở ngoài phương thức. Mặc dù chúng có cùng tên biến.

#### Biến toàn cục (global variable)

Khác với biến cục bộ, biến toàn cục được khai báo với tên bắt đầu bằng ký tự $ và biến toàn cục có phạm vi sử dụng trong toàn bộ chương trình.

``` ruby
$global_variable = 'Toi chua duoc thay doi.'
def variable_scope
    puts $global_variable = 'Toi da bi thay doi.'
end
variable_scope #Toi da bi thay doi.
$global_variable #Toi da bi thay doi.
```

Ta thấy, khi giá trị của biến toàn cục thay đổi ở trong phương thức, thì sự thay đổi này ảnh hưởng ra tới bên ngoài phương thức. Như vậy, phạm vi của biến toàn cục là toàn bộ chương trình.

#### Biến đối tượng (instance variable)

Biến được bắt đầu bằng ký tự @ được gọi là "Biến Đối Tượng", có nghĩa là nó chỉ thuộc về một đối tượng riêng lẻ hoặc một đối tượng của một lớp.

``` ruby

class Nguoi

  def initialize(ten)
    @ten = ten
  end

  def show
    puts @ten
  end
end

first = Nguoi.new('Nguyen')
first.show # Nguyen

second = Nguoi.new('Anh')
second.show # Anh

```


Ở ví dụ trên ta thấy instance variable chỉ thuộc riêng lẻ cho mỗi đối tượng và phạm vi sử dụng của nó là toàn bộ trong một lớp.

#### Biến lớp (class variable)

Để định nghĩa biến thuộc class, chúng ta sử dụng  ký tự `@@` trước tên biến. Khác với `instance variable`, `class variable` sẽ được dùng chung cho tất cả các đối tượng của lớp đó.

```ruby

class Dog

  def initialize(leg)
    @@leg = leg
  end

  def show_leg
    puts @@leg
  end

end

first = Dog.new(4)
first.show_leg # 4

second = Dog.new(10)
second.show_leg # 10

first.show_leg # 10
```


Như ví dụ trên ta thấy biến `@@leg` đã bị thay đổi sau khi đối tượng second được tạo.

#### Hằng (constant)

Một Hằng trong Ruby cũng giống như một biến, ngoại trừ một điều là giá  trị của hằng không đổi trong quá trình chương trình chạy. Trình thông dịch của Ruby không bắt buộc về sự cố định giá trị của Hằng, nhưng nếu có sự thay đổi giá trị của Hằng thì trình thông dịch sẽ có thông báo về sự thay đổi đó.

``` ruby
A_CONST = 10
A_CONST = 20

"warning: already initialized constant A_CONST"
```


    





