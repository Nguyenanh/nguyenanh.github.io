
---
layout: post
title: Singleton - Design Pattern trong Ruby
excerpt: "Mẫu thiết kế Singleton đảm bảo rằng một lớp chỉ có một thể hiện (instance) duy nhất. Do thể hiện này có tiềm năng sử dụng trong suốt chương trình, nên mẫu thiết kế Singleton cũng cung cấp một điểm truy cập toàn cục đến nó."
modified: 2016-03-28
tags: [singleton, ruby, design pattern]
comments: true
---
### Pattern Singleton là gì?

Mẫu thiết kế Singleton đảm bảo rằng một lớp chỉ có một thể hiện (instance) duy nhất. Do thể hiện này có tiềm năng sử dụng trong suốt chương trình, nên mẫu thiết kế Singleton cũng cung cấp một điểm truy cập toàn cục đến nó.
### Pattern Singleton được sử dụng trong trường hợp nào?

Khi bạn tạo ra một class mà bạn chỉ muốn chỉ có duy nhất một thực thể là thể hiện của class đó và bạn có thể truy cập đến nó ở bất kỳ nơi đâu khi bạn muốn. Ví dụ:

* Làm việc với log file.
* Config cho toàn bộ dự án.
* Kết nối với Database.
* ...

### Cách cài đặt.

``` ruby

#singleton/singleton_setup.rb
class Database
  attr_accessor :connecting

  @@instance = Database.new

  def self.instance
    @@instance
  end

  private_class_method :new
end
```

``` ruby

#irb
2.2.3 :001 > load 'singleton/singleton_setup.rb'
 => true 
2.2.3 :002 > Database.instance.connecting = 'mysql'
 => "mysql" 
2.2.3 :003 > first = Database.instance
 => #<Database:0x007feed305a680 @connecting="mysql"> 
2.2.3 :004 > first.connecting = 'mongodb'
 => "mongodb" 
2.2.3 :005 > Database.instance.connecting
 => "mongodb" 
2.2.3 :006 > 
```
Qua việc chạy trên cho thấy chỉ một instance được tao ra thông qua việc gọi class method instance và được sử dụng global.

### Sử dụng Singleton module của Ruby.

Trong Ruby đã hỗ trợ cho chúng ta Singleton Patttern đó là Singeleton module. Để tối giản công việc trên chúng ta chỉ cần sử dụng nó.

 ``` ruby
 #singleton/singleton_moudle.rb
 require 'singleton'

class Database
  include Singleton
  
  attr_accessor :connecting
end

```

``` ruby
2.2.3 :002 > load 'singleton/singleton_module.rb'
 => true 
2.2.3 :003 > Database.instance.connecting = 'mysql'
 => "mysql" 
2.2.3 :004 > second = Database.instance
 => #<Database:0x007fac3b09f788 @connecting="mysql"> 
2.2.3 :005 > second = Database.instance.connecting = 'postgresql'
 => "postgresql" 
2.2.3 :006 > Database.instance.connecting
 => "postgresql" 
2.2.3 :007 > 
```
