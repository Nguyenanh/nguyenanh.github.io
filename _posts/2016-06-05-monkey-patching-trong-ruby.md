---
layout: post
title: Monkey Patching trong Ruby
excerpt: "`Ruby` được biết đến với ngôn ngữ có cú pháp ngắn ngọn, dễ hiểu. Ngoài ra nó còn là một ngôn ngữ rất linh hoạt, điểu này được thể hiện rõ qua Monkey patching."
modified: 2016-06-05
tags: [monkey-patching, ruby, patching, monkey]
comments: true
---

### Monkey Patching là gì?

`Monkey Patching` theo wikipedia: **"A monkey patch is a way for a program to extend or modify supporting system software locally (affecting only the running instance of the program)."**
Hiểu nôn na là một phương pháp cho phép mở rộng hay chỉnh sửa một hệ thống phần mềm 1 cách tạm thời và cục bộ.

### Monkey patching trong ruby

`Ruby` được biết đến với ngôn ngữ có cú pháp ngắn ngọn, dễ hiểu. Ngoài ra nó còn là một ngôn ngữ rất linh hoạt, điểu này được thể hiện rõ qua Monkey patching.

Tất cả các class trong ruby đều là `Open Class`. Có nghĩa là chúng ta có thể thay đổi class bất cứ lúc nào và bất cứ nơi đâu. Chúng ta có thể thêm một method, override một method đã tồn tại. Đây là một trong những sức mạnh của `ruby`. Thậm chí chúng ta có thể thay đổi một số class chuẩn của Ruby như: `String`, `Array` hoặc `Hash`.

Điều này nghe có vẻ nguy hiểm và chính vì sự mềm dẻo làm cho Ruby trở thành con dao 2 lưỡi nếu chũng ta quá lạm dụng việc sự dụng `Open Class` hoặc không có cách tổ chức code khi sử dụng Monkey patching.

### Một ví dụ của Monkey Patching.

Bạn là 1 Ruby developer và cũng là 1 Rails devloper, thì chắc hẳn bạn đã và đang một lần sử dụng hoặc nhìn thấy method `blank?`. Đây là một method rất phổ biến trong Rails.

`blank?` method được dùng hầu hết cho tất cả các đối tượng của class như `String`, `Hash`, `NilClass`, `TrueClass`, `FalseClass`, `Integer`, `Array`. Và bạn đã bao giờ sự dụng method này ngoài dự án Rails bao giờ chưa ?. 

``` ruby
#Inside Rails
"anh".blank? # => true
```

Điều này thì quá rõ ràng !!!. Và nếu chúng ta sự dụng method này bên ngoài Rails, thì có vẻ chúng ta sẽ gặp một lỗi trông như thế này.

``` ruby
#Outside Rails
"anh".blank? #NoMethodError: undefined method `blank?' for "anh":String
```

Rõ ràng là chúng ta đã có 1 lỗi `#NoMethodError: undefined method 'blank?' for "anh":String`. Vậy tại sao cùng một đối tượng `String` lại có sự khác nhau đến như vậy, thì câu trả lời là đây.

`blank?` không phải là một method của của lớp `String` trong `core` của Ruby, và Rails đã sử dụng phương pháp Monkey Patching để tạo ra một method `blank?`. Đó là lý do tại sao lại có sự khác biệt ở 2 ví dụ trên.

## Sử dụng Monkey Patching để thay đổi class core của Ruby.

Giả sử chúng ta có một đối tượng `String` `20160605` và giờ tôi muốn chuyển sang định dạng ngày tháng `2016/06/05`. Và giờ tôi sẽ viết thêm một method của class `String` để làm công việc này. :D

#### I. Thêm một method vào `String`

```ruby
require 'date'
class String
  def to_s_date
    Date.parse(self).strftime('%Y/%m/%d')
  end
end

p '20160605'.to_s_date # "2016/06/05"
```


Trong ví dụ trên, chúng ta đã define một method `to_s_date` của class `String` để định dạng chuỗi string của chúng ta. Rõ rãng `to_s_date` chỉ có nghĩa trong phạm vị ví dụ của tôi. Cung giống như method `bliank?` trong dự án Rails. Và tất cả các đối tượng của class `String` đều sử dụng được method này. 

#### II. Override một method của class `String`

Ngoài việc add thêm một method mới cho class chuẩn của Ruby, chúng ta có thể override một method đã tồn tại theo nguyên tắc "last def wins".


```ruby
p 'Nguyen Anh'.upcase # => "NGUYEN ANH"

class String
  def upcase
    self.reverse
  end  
end

p 'Nguyen Anh'.upcase # => "hnA neyugN"
```


`upcase` là một method của Ruby core có chức năng chuyển đổi chữ thường thành hoa. Ở ví dụ trên tôi đã override lại method này và bảo nó in ra thành xâu đảo ngược. Đúng là quá ảo diệu, do vậy mới nói tính năng này của Ruby là một con dao 2 lưỡi =)).

#### III. Tổ chức Monkey Patching một cách có tâm :).

Việc lạm dụng Monkey Patching sẽ làm code của chúng ta khó debug khi gặp lỗi và cũng là một cơn ác mộng cho new member khi đọc source của bạn. Để giải quyết việc này thì chúng ta nên đặt tất cả cả chúng vào chung một module, vậy chúng ta cần có một `monkey-patching convention`.

Ở đây tôi sử dụng `Rails’ monkey-patching convention`. Có nghĩa là tôi sẽ sử dụng cấu trúc của Rails, đơn giản vì Rails quá phổ biến rồi =)).

Một số `Rails’ monkey-patching convention`

* `rails/activesupport/lib/active_support/core_ext/object/blank.rb`
* `rails/activesupport/lib/active_support/core_ext/object/inclusion.rb`

Patches chung của Rails sẽ là `lib/core_extensions/class_name/group.rb`. Như vậy tôi sẽ tái cấu trúc ví dụ của tôi ở trên. 

``` ruby
require 'date'

module CoreExtensions
  module String
    module ConvertingDate
      def to_s_date
        Date.parse(self).strftime('%Y/%m/%d')
      end
    end
  end
end

String.include CoreExtensions::String::ConvertingDate

p '20160605'.to_s_date # =>"2016/06/05"
```

Nếu như chúng ta có một chuẩn chung thì công việc sẽ trở nên đơn giản hơn, giống như tiếng nói chung.







