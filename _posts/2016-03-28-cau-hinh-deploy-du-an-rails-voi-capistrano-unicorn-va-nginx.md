---
layout: post
title: Cấu hình deploy một dự án Rails với capistrano, unicorn và nginx
excerpt: "`Deploy` là quá trình đưa source code bạn lên server sau khi bạn đã phát triển xong và chuẩn bị giao cho khách hàng. Đây là một công đoạn có phải nói các developer nào cũng phát biết trong con đường sự ngiệp của mình."
modified: 2016-03-28
tags: [deploy, capistrano, unicorn, nginx]
comments: true
---

`Deploy` là quá trình đưa source code bạn lên server sau khi bạn đã phát triển xong và chuẩn bị giao cho khách hàng. Đây là một công đoạn có phải nói các developer nào cũng phát biết trong con đường sự ngiệp của mình.

Tuy nhiên công việc deploy không phải lúc nào cũng diễn ra xuôi sẻ và không phải lúc nào cũng giống nhau. Nó cũng phụ thuộc vào nhiều yếu tố khác nhau

Bài viết này mình sẽ hưỡng dẫn cấu hình để deploy một dự án Rails dùng capistrano, unicorn và nginx lên server Ubuntu 14.04.

 Trước khi bắt đầu, hãy chắc chắn là server của bạn đã cài đặt một số thứ như sau: 
 
 * `Ruby`
 * `Git`
 * `Nginx`
 * `Một dự án ROR ở local`

## Chiến thôi !!
1.Add thêm một số `gem` vào Gemfile và chạy `bundle íntall` 

``` ruby
#Gemfile
gem 'capistrano'
gem 'capistrano-rails'
gem 'capistrano-bundler'
gem 'capistrano-unicorn-nginx'
gem 'unicorn'
gem 'capistrano-rvm'
```

2.Sau khi đã cài một số `gem` xong tiếp tục tạo ra những file config capistrano chạy lệnh sau:

`$ bundle exec cap install`
Lệnh trên sẽ tạo cấu trúc file như sau.

├── Capfile
├── config
│   ├── deploy
│   │   ├── production.rb
│   │   └── staging.rb
│   └── deploy.rb
└── lib
    └── capistrano
            └── tasks
3. Require thêm một số gói vào `Capfile` và `Capfile` của bạn sẽ trông như thế này.

``` ruby
#Capfile
# Load DSL and set up stages
require 'capistrano/setup'

# Include default deployment tasks
require 'capistrano/deploy'

require 'capistrano/rvm'
require 'capistrano/bundler'
require 'capistrano/rails'
require 'capistrano/unicorn_nginx'
# Load custom tasks from `lib/capistrano/tasks' if you have any defined

Dir.glob('lib/capistrano/tasks/*.rake').each { |r| import r }
```

4.Tiếp theo cấu hình cho `deploy.rb`.

``` ruby
#config/deploy.rb
#Tên ứng dụng của bạn ở đây mình đặt tên Appdeploy
set :application, 'Appdeploy'
#Repository của bạn
set :repo_url, 'https://github.com/Nguyenanh/Blog.git'
#Đường dãn chứa source trên server sau khi deploy
set :deploy_to, '/home/anhn/Appdeploy'
set :scm, :git

```
5.Vậy là ta thiết lập những cấu hình chung để `deploy`. Giờ tùy vào từng môi trường ta sẽ thiết lập riêng. Bài viết này mình sẽ thiết lập trên môi trường `production`. Mở file `production.rb`.

```ruby
#config/deploy/production.rb
#Set user và và server_name của bạn
set :user, 'anhn'
set :server_name, '1.2.3.4'
#Set branch trên Git mà bạn muốn deploy
set :branch, 'master'
#Set môi trường, ở đây mình thiết lâp cho môi trường production
set :rails_env, 'production'
set :bundle_flags, "--no-deployment"

role :app, ["#{fetch(deploy_user)}@#{fetch(server_name)}"]
role :web, ["#{fetch(deploy_user)}@#{fetch(server_name)}"]
role :db,  ["#{fetch(deploy_user)}@#{fetch(server_name)}"]

server fetch(server_name), user: fetch(deploy_user), roles: %w{web app db}, primary: true

```
Trường hợp server của bạn bắt authentication bằng `ssh` thì bạn thêm tùy chọn `ssh_option`

``` ruby
#config/deploy/production.rb
#home/anhn/.ssh/appdeploy.pem là đường dẫn mà mình dùng `.pem` để xác thực
set :ssh_options, {
  keys: %w(/home/anhn/.ssh/appdeploy.pem),
  forward_agent: false,
 }
```
6.Vậy là xong quá trình config cho quá trình `deploy`. Việc cuối cùng là kêu thằng capistrano đưa source lên server.
``` ruby
$ bundle exec cap setup
```
Lệnh này sẽ bảo `cap` khỏi tạo cấu trúc đường dẫn cũng như file config `nginx` cho quá trình deploy.
``` ruby 
$ bundle exec cap production deploy
```
Tiếp theo kêu cap deploy code lên server với môi trường production. Việc còn lại là ngồi rung đùi. Khi quay trở lại quá trình deploy hoàn tất.


### Kết luận
Bài viết này xin nhấn mạnh thiết lập các cấu hình để deploy một dự án Rails lên server. Và đây chỉ là single server. Ở những bài viết sau mình sẽ hướng dẫn deploy trên HDH CentOS hoặc deploy với cấu trúc server NAT.
