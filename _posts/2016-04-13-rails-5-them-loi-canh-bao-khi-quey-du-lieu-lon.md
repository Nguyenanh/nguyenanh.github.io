---
layout: post
title: Thêm lời cảnh báo khi query dữ liêụ lớn trong rails 5
excerpt: "Với dữ liệu data lớn khi chúng ta query thì sẽ gập một số vấn đề như tốn bộ nhớ nhiều. Dưới đây là một ví dụ."
modified: 2016-03-28
tags: [ruby, query, rails, warning]
comments: true
---
Với dữ liệu data lớn khi chúng ta query thì sẽ gập một số vấn đề như tốn bộ nhớ nhiều. Dưới đây là một ví dụ:

``` ruby
>> Post.published.count
=> 25000

>> Post.where(published: true).each do |post|
     post.archive!
   end

# Loads 25000 posts in memory
```

Để giảm bớt vấn để ở trên thì từ `Rails 5` đã hỗ thêm đã thêm một tính năng

`config.active_record.warn_on_records_fetched_greater_than` 

nhằm tạo lời cảnh báo.

Khi chúng ta config với một giá trị nguyên, bất kỳ một câu query nào trả về số record lớn hơn số gới hạn được set, thì log ra một lời cảnh báo.

```ruby
config.active_record.warn_on_records_fetched_greater_than = 1500

>> Post.where(published: true).each do |post|
     post.archive!
   end

=> Query fetched 25000 Post records: SELECT "posts".* FROM "posts" WHERE "posts"."published" = ? [["published", true]]
   [#<Post id: 1, title: 'Rails', user_id: 1, created_at: "2016-02-11 11:32:32", updated_at: "2016-02-11 11:32:32", published: true>, #<Post id: 2, title: 'Ruby', user_id: 2, created_at: "2016-02-11 11:36:05", updated_at: "2016-02-11 11:36:05", published: true>,....]
   
```

Với tính năng này sẽ giúp chúng ta tìm ra những những câu query không hiệu quả và tìm ra một giải pháp nào đó tốt hơn.

``` ruby
config.active_record.warn_on_records_fetched_greater_than = 1500

>> Post.where(published: true).find_each do |post|
     post.archive!
   end

# No warning is logged
```

Nguồn: <http://blog.bigbinary.com/2016/04/13/rails-5-adds-option-to-log-warning-when-fetching-big-result-sets.html>
