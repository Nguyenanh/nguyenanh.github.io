---
layout: post
title: Giảm N+1 Query khi count record dữ liệu quan hệ
excerpt: "Trong Rails đã hỗ trợ một method includes dùng để hổ trợ việc giảm N+1 query trong truy vấn cơ sở dữ liệu quan hệ. Như vậy ở đây mình đặt ra một bài toán như sau."
modified: 2016-03-28
tags: [deploy, capistrano, unicorn, nginx]
comments: true
---
Trong Rails đã hỗ trợ một method includes dùng để hổ trợ việc giảm N+1 query trong truy vấn cơ sở dữ liệu quan hệ. Như vậy ở đây mình đặt ra một bài toán như sau.
Mình có table `categories`  has_many với table `posts` và mình muốn lấy list categories và count các bài post tương ứng của category đó thì mình làm như sau:

```ruby
#  Controller
class CategoryController < ApplicationController
  def index
    categories = Category.all
  end
end
# View
<% @categories.each do |category| %>
  <h1><%= category.name %>(<%= category.posts.count %>)<h1>
<% end %>
```
Nhìn vào console bạn sẽ thấy như sau:

>Category Load (0.1ms)  SELECT `categories`.* FROM `categories`
   (0.2ms)  SELECT COUNT(*) FROM `posts` WHERE `posts`.`category_id` = 2
   (0.3ms)  SELECT COUNT(*) FROM `posts` WHERE `posts`.`category_id` = 3
   (0.2ms)  SELECT COUNT(*) FROM `posts` WHERE `posts`.`category_id` = 4

3 câu lệnh SELECT count được thực hiện.  Điều này sẽ ảnh hưởng đến tốc độ truy vấn dữ liệu khi hệ thống dữ liệu lớn. Để giải quyết vấn đề này mình xin hướng dãn 2 cách 
## 1. Tạo một câu query
Mình tạo một cái scope với dòng query như sau:

``` ruby
# Model
class Category < ActiveRecord::Base
  has_many :posts
  scope :with_count_posts, -> {joins(:posts).select("categories.* ,Count(posts.id) AS posts_count").group("categories.id")}
end

# Controller
class CategoryController < ApplicationController
  def index
    @categories = Category.with_count_posts
  end
end

# View
<% @categories.each do |category| %>
  <h1><%= category.name %>(<%= category.posts_count %>)<h1>
<% end %>
```

Nhìn vào console bạn sẽ thấy sự khác biệt
>Category Load (0.1ms)  SELECT categories.* ,Count(posts.id) AS posts_count FROM `categories` INNER JOIN `posts` ON `posts`.`category_id` = `categories`.`id` GROUP BY categories.id

Nó chỉ cần chạy một truy vấn sơ với ban đầu. Dòng trên có ý nghĩa là khi khi mình lấy hết dữ của bảng categories mình sẽ tạo thêm 1 field là `posts_count` để chứa toàn bộ tổng số những record posts tương ứng với category đó.

------------------------------------------------------------
###  Lưu ý
 `scope` ở trên sẽ không lấy ra những category không có record `posts`  nào. Nếu muốn lấy hết thì bạn có thể tạo một `scope` như dưới đây.
 
``` ruby
  scope :with_count_posts, -> {joins("LEFT JOIN posts ON categories.id = posts.category_id").select("categories.* ,Count(posts.id) AS posts_count").group("categories.id")}
```  
-------------------------------------------------------------

## 2. Dùng `Gem`
Ở đây mình xin giới thiệu 2 `gem` là [includes-count](https://github.com/manastech/includes-count) và [dase](https://github.com/vovayartsev/dase)
Các bạn có thể vào 2 link trên để tìm hiểu cách sử dụng

---------------------------
Trên đây là một bài viết chia sẽ của mình để giảm tốc độ truy vấn khi muốn lấy count record quan hệ. Chúc các bạn thành công.
*Nguyen Anh*
