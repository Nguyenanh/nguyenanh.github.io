---
layout: post
title: Lý do không tạo được record khi khai báo belongs_to trong Rails 5
excerpt: "Trong Rails 5. Bất cứ khi nào chúng ta định nghĩa một quan hệ belongs_to. Thì nó bắt buộc chúng ta cung cấp một đối tượng quan hệ với nó.

Rails sẽ tiến hành validation lỗi nếu đối tượng quan hệ không tồn tại."
modified: 2016-02-20
tags: [rails 5, belongs-to, relationship]
comments: true
---
Trong Rails 5. Bất cứ khi nào chúng ta định nghĩa một quan hệ `belongs_to`. Thì nó bắt buộc chúng ta cung cấp một đối tượng quan hệ với nó.

Rails sẽ tiến hành validation lỗi nếu đối tượng quan hệ không tồn tại

``` ruby
class User < ApplicationRecord
end
class Post < ApplicationRecord
    belongs_to :user
end

post = Post.create(title: 'Hi')
 => <Post id: nil, title: "Hi", user_id: nil, created_at: nil, updated_at: nil>

post.errors.full_messages.to_sentence
 => "User must exist"
```

Như chúng ta có thể thấy, chúng ta không thể tạo bất kỳ một `post` nào nếu không có một quan hệ `user` nào

### Làm thế nào để giải quyết được điều này trong Rails 5
Trong Rails 4.x để thêm validation cho `belongs_to` chúng ta chỉ cần thêm tùy chọn `required: true`.

``` ruby
class User < ApplicationRecord
end

class Post < ApplicationRecord
  belongs_to :user, required: true
end

post = Post.create(title: 'Hi')
=> <Post id: nil, title: "Hi", user_id: nil, created_at: nil, updated_at: nil>

post.errors.full_messages.to_sentence
=> "User must exist"
```
Mặc định trong Rails 4.x tùy chọn `required` là `false`.

Để bỏ qua validation `belongs_to` trong Rails 5. Chúng ta chỉ cần thêm tùy chọn `optional: true`.

``` ruby
class Post < ApplicationRecord
  belongs_to :user, optional: true
end

post = Post.create(title: 'Hi')
=> <Post id: 2, title: "Hi", user_id: nil>
```

Với thêm tùy chọn ở trên thì validation sẽ bỏ qua với class  `Post` và tất cả model còn lại sẽ vẫn bị validation.
### Bỏ qua validation cho toàn bộ model trong Rails 5.
Mặc định Rails 5 setting validation cho toàn bộ model

``` ruby
Rails.application.config.active_record.belongs_to_required_by_default = true.
```

Để bỏ validation chúng ta cần seting lại giá trị sang `false`.

``` ruby
Rails.application.config.active_record.belongs_to_required_by_default = false

class Post < ApplicationRecord
  belongs_to :user
end

post = Post.create(title: 'Hi')
=> <Post id: 3, title: "Hi", user_id: nil, created_at: "2016-02-11 12:36:05", updated_at: "2016-02-11 12:36:05">
``` 

Nguồn: http://blog.bigbinary.com/2016/02/15/rails-5-makes-belong-to-association-required-by-default.html