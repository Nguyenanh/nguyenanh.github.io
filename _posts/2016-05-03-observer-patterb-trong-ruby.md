---
layout: post
title: Observer - Design Pattern trong Ruby
excerpt: "Có thể hiểu Observer thuộc nhóm pattern Behavioral là một mẫu thiết kế dành cho việc một đối tượng khi thay đổi trạng thái của bản thân nó thì các đối tượng đính kèm theo cũng sẽ được thông báo."
modified: 2016-05-03
tags: [observer, ruby, design-pattern]
comments: true
---

### Pattern Observer là gì?

Có thể hiểu `Observer` thuộc nhóm pattern `Behavioral` là một mẫu thiết kế dành cho việc một đối tượng khi thay đổi trạng thái của bản thân nó thì các đối tượng đính kèm theo cũng sẽ được thông báo.

Mô hình cơ bản của 1 mẫu Observer thường bao gồm 4 thành phần sau.

1.`Subject`: giao diện cho đối tượng dữ liệu, khai báo các phương thức chính:


  * add_observer: thêm các Observer vào danh sách đăng ký các đối tượng cần phải thông báo về sự thay đổi.

  * delete_observer: xóa Observer chỉ định ra khỏi danh sách đăng ký các đối tượng cần phải thông báo về sự thay đổi.

  * notify_observers: thông báo cho các Observer đã đăng ký về những thay đổi trên Subject.

2.`ConcreteSubject`: cài đặt giao diện Subject. Vì thường là đối tượng dữ liệu, nó lưu giữ trạng thái mà các đối tượng Observer quan tâm. Khi trạng thái này thay đổi, các Observer đăng ký với nó sẽ được thông báo.

3.`Observer`: Khai báo giao diện với phương thức chính update. Phương thức này có thể truy cập đối tượng Subject mà nó đăng ký, cập nhật Observer với trạng thái thay đổi của Subject.

4.`ConcreteSubject`: cài đặt giao diện của Observer. Constructor của nó thường yêu cầu phải đăng ký nó cho đối tượng Subject mà nó theo dõi. Khi được thông báo, nó sẽ thực thi một tác vụ gì đó, ví dụ thay đổi giao diện, cập nhật biểu đồ.


### Pattern Observer được sử dụng trong trường hợp nào?

Khi bạn muốn các đối tượng liên lạc với nhau. Khi đối tượng này gửi 1 thông điệp thì các đối tượng đăng ký lắng nghe thông điệp sẽ phản ứng lại với thông điệp đó. Đối tượng gửi thông điệp sẽ không cần biết nó sẽ gửi cho ai và đối tượng nhận thông điệp sẽ không cần biết ai gửi thông điệp đó.

### Cách cài đặt.

##### 1. Tạo `Subject` và `Observer`.


```ruby

#subject.rb
module Subject
  def initialize
    @observers = []
  end

  def add_observer(observer)
    @observers << observer
  end

  def delete_observer(observer)
    @observers.delete(observer)
  end

  def notify_observers
    @observers.each do |observer|
      observer.update(self)
    end
  end
end
```

```ruby

#observer.rb
module Observer
  def update
  end
end
```

##### 2. Tạo `ConcreteSubject`

```ruby

#employee.rb
class Employee
  include Subject

  attr_reader :name, :title
  attr_reader :salary

  def initialize(name, title, salary)
    super()
    @name = name
    @title = title
    @salary = salary
  end

  def salary=(new_salary)
    @salary = new_salary
    notify_observers
  end
end

```

##### 3. Tạo `ConcreteObserver`

Ở đây mình sẽ tạo ra 2 class là `Payroll` và `TaxMan` sẽ lắng nghe khi có thông báo từ  đối tượng của lớp `Employee`

```ruby

#payroll.rb
class Payroll
  include Observer
  def update(changed_employee)
    puts "Lương của mình là: #{changed_employee.salary}!"
  end
end

```

và

``` ruby

#tax_man.rb
class TaxMan
  include Observer
  def update(changed_employee)
    puts "Gơi #{changed_employee.name} một bản hoá đơn thuê mới!"
  end
end

```

Vậy là xong quá trình cài đặt `Pattern Observer`, demo.

``` ruby

#main.rb
require_relative 'subject'
require_relative 'observer'
require_relative 'employee'
require_relative 'payroll'
require_relative 'tax_man'

nhan_vien  = Employee.new('Nguyen Anh', 'Developer', 30_000)

bang_luong = Payroll.new
nhan_vien.add_observer(bang_luong)

thue       = TaxMan.new
nhan_vien.add_observer(thue)

#Giờ mình sẽ thay đổi Lương của mình.

nhan_vien.salary = 35_000

#Kết quả:

#Lương của mình là: 35000!
#Gơi Nguyen Anh một bản hoá đơn thuê mới!
```


Nhìn kết qủa trên, Khi thay đổi thông số lương thì sẽ gởi thông báo đến cho 2 class `Payroll` và `TaxMan` cập nhập trạng thái.

### Sử dụng Pattern Observable của Ruby

```ruby

#employee.rb
require 'observer'

class Employee
  include Observable

  attr_reader :name, :title
  attr_reader :salary

  def initialize(name, title, salary)
    super()
    @name = name
    @title = title
    @salary = salary
  end

  def salary=(new_salary)
    changed # Sets boolean flag that object has changed to true
    @salary = new_salary
    notify_observers(self) # Sets changed = false
  end
end

```


```ruby

#payroll.rb
class Payroll
  include Observer
  def update(changed_employee)
    puts "Lương của mình là: #{changed_employee.salary}!"
  end
end

```


``` ruby

#tax_man.rb
class TaxMan
  include Observer
  def update(changed_employee)
    puts "Gơi #{changed_employee.name} một bản hoá đơn thuê mới!"
  end
end

```


``` ruby

#main.rb
require_relative 'employee'
require_relative 'payroll'
require_relative 'tax_man'

nhan_vien  = Employee.new('Nguyen Anh', 'Developer', 30_000)

bang_luong = Payroll.new
nhan_vien.add_observer(bang_luong)

thue       = TaxMan.new
nhan_vien.add_observer(thue)

#Giờ mình sẽ thay đổi Lương của mình.

nhan_vien.salary = 35_000

#Kết quả:

#Lương của mình là: 35000!
#Gơi Nguyen Anh một bản hoá đơn thuê mới!
```


