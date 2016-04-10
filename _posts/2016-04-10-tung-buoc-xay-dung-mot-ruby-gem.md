---
layout: post
title: Từng bước để xây dựng một Ruby Gem
excerpt: "Bài viết này chúng sẽ không đi sâu vào lý thuyết, thay vào đó sẽ sẽ hướng dẫn các bạn bước từng bước để tạo ra một gem đơn giản nhất có thể chính là tạo ra helper in ra dòng Hello Word. Nghe thôi thấy cũng đơn giản rồi :D. Bắt đầu thôi."
modified: 2016-03-28
tags: [ruby, gem, rails]
comments: true
---
Bài viết này chúng sẽ không đi sâu vào lý thuyết, thay vào đó sẽ sẽ hướng dẫn các bạn bước từng bước để tạo ra một `gem` đơn giản nhất có thể chính là tạo ra helper in ra dòng `Hello Word`. Nghe thôi thấy cũng đơn giản rồi :D. Bắt đầu thôi.

## I: Xây dựng Gem

#### Bước 1: Tạo cấu trúc của `gem`

``` ruby
bundle gem helloword-rails
```
Chúng ta sẽ được cấu trúc như này sau khi chạy lệnh trên.
<figure>
	<img src="https://cloud.githubusercontent.com/assets/7424863/14407550/b8d384a2-fef7-11e5-82af-1bb2a7cbc944.png"></a>
</figure>

#### Bước 2: Tạo một file helper `helloword_helper.rb`

Tiếp theo tạo ra một file helper đặt trong thư mục `lib/helloword/rails` bằng cách chạy 2 dòng lệnh dưới đây.

``` ruby
cd helloword-rails/
touch lib/helloword/rails/helloword_helper.rb
```

#### Bước 3: Tạo ra một phương thức của helper để in ra dòng hello word

Mở file `helloword_helper.rb` và thêm nhưng dòng code dưới dây vào.

``` ruby
# lib/helloword/rails/helloword_helper.rb
module HellowordHelper
  def hello_word_tag
    "Hello Word!!!"
  end
end
```
#### Bước 4: Khai báo helper của chúng ta vào rails

Khao báo helper của chúng ta vào `ActionView::Helpers::AssetTagHelper` để Rails có thể biết và thực thi nó.

``` ruby
# lib/helloword/rails.rb
require "helloword/rails/version"
require "helloword/rails/helloword_helper"

module ActionView
  module Helpers
    module AssetTagHelper
      include HellowordHelper
    end
  end
end
```
#### Bước 5: Tạo một ứng dụng Rails để test gem.

Chạy lệnh sau để tạo mới ứng dụng rails.

``` ruby
# terminal
rails new helloword-rails-app
cd helloword-rails-app
rails generate controller site index
```

#### Bước 6: Thêm gem của chúng ta vào `Gemfile`

Trong file view `index.html.erb` và thêm helper của chúng ta vào.

``` ruby
#view/site/index.html.erb
<h1><%= hello_word_tag %></h1>

```
Mở `Gemfile` và thêm dòng

``` ruby
# Gemfile
gem 'helloword-rails', '0.1.0', path: '/Users/admin/Project-Ruby/helloword-rails/'
```

Trong đó `path` là đường dẫn chưa source gem của tôi và bạn sẽ thay vào đó đường dẫn của bạn.
Cuối cũng chạy lệnh `bundle install` và `rails s`, sau đó truy cập <http://localhost:3000/site/index>
<figure>
	<img src="https://cloud.githubusercontent.com/assets/7424863/14407815/663ffd2a-ff00-11e5-818c-2e6716c65e45.png"></a>
</figure>
Quá đơn giản !!!!

## II: Public gem lên trang chủ.

Thay đổi một số thông tin trong `helloword-rails.gemspec`

``` ruby
# coding: utf-8
lib = File.expand_path('../lib', __FILE__)
$LOAD_PATH.unshift(lib) unless $LOAD_PATH.include?(lib)
require 'helloword/rails/version'

Gem::Specification.new do |spec|
  spec.name          = "helloword-rails"
  spec.version       = Helloword::Rails::VERSION
  spec.authors       = "Nguyen Anh"
  spec.email         = "cauut2117610@gmail.com"

  spec.summary       = %q{Write a short summary, because Rubygems requires one.}
  spec.description   = %q{Write a longer description or delete this line.}
  spec.homepage      = "https://github.com/Nguyenanh/helloword-rails"
  spec.license       = "MIT"


  spec.files         = `git ls-files -z`.split("\x0").reject { |f| f.match(%r{^(test|spec|features)/}) }
  spec.bindir        = "exe"
  spec.executables   = spec.files.grep(%r{^exe/}) { |f| File.basename(f) }
  spec.require_paths = ["lib"]

  spec.add_development_dependency "bundler", "~> 1.11"
  spec.add_development_dependency "rake", "~> 10.0"
  spec.add_development_dependency 'rspec'
end
```
Build gem của chúng ta với lệnh.

``` ruby
#terminal
rake build
```

Sau khi build thành công chúng ta được file `pkg/helloword-rails-0.1.0.gem`.
Cuối cùng đưa gem lên trang chủ <https://rubygems.org>.

``` ruby
#terminal
gem push pkg/helloword-rails-0.1.0.gem
```

Sau đó chúng ta có thể dùng `helloword-rails` một cách bình thường.

Quá đơn giản!!!.

[helloword-rails]: https://github.com/Nguyenanh/helloword-rails
