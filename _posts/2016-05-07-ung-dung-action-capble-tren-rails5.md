---
layout: post
title: Ứng dụng ActionCable trong Rails 5
excerpt: "Một tính năng đáng chú ý của phiên bản Rails 5 đó là tích hợp ActionCable. Nhiệm vụ của ActionCable là cho phép chúng ta có thể tạo chức năng real-time trong các ứng dụng Rails trở nên đơn giản hơn rất nhiều."
modified: 2016-05-07
tags: [actioncable, rails5, realtime]
comments: true
---

Một tính năng đáng chú ý của phiên bản Rails 5 đó là tích hợp ActionCable. Nhiệm vụ của ActionCable là cho phép chúng ta có thể tạo chức năng real-time trong các ứng dụng Rails trở nên đơn giản hơn rất nhiều.

ActionCable sử dụng giao thức Websocket để hỗ trợ giao tiếp 2 chiều giữa client và server.

Bài viết này để sẽ hưỡng dẫn step-by-step để tạo 1 ứng dụng web chat real-time sử dụng tính năng Action-Cable có săn trong Rails 5.

Yêu cầu Có sẵn:
    * Ruby 2.2.2 trở lên.
    * Rails 5.

#### Step 1: Tạo một ứng dụng với Rails 5.

```sh
rails new chat-action-cable --skip-spring
cd chat-action-cable
```

#### Step 2: Tạo trang để chưa nội dung messages.

```sh
rails g controller rooms index
```

Set mặc định hệ thống sẽ truy cập đến `rooms#index`.

```ruby

#router.rb
root to: "rooms#index"
```

#### Step 3: Tạo model để kết nối DB chứa dữ liệu.

```sh

rails g model message content:text
rake db:migrate
```

Tiếp theo lấy tất cả data messages trong rooms_controller

```ruby 

#rooms_controller
def index
    @messages = Message.all
end
```

Tạo một `_message.html.erb` trong thư mục `app/views/messages`

```sh
mkdir app/views/messages
touch app/views/messages/_message.html.erb
```


```ruby

#app/views/messages/_message.html.erb
<div class='message'>
  <p><%= message.content %></p>
</div>
```


```ruby

#app/views/messages/_message.html.erb
<h1>Rooms</h1>
<div id="messages">
  <%= render @messages %>
</div>
<%= text_field_tag :body, "", id: "chat-speak" %>

```

#### step 4: Tạo một channel để giao tiếp giữa server và client.

```sh

rails g channel room speak
```

Tiếp theo ta cấu hình cho channel vưa tạo.
Cấu hình cho client.

```coffee

# app/assets/javascripts/channels/room.coffee
App.room = App.cable.subscriptions.create "RoomChannel",
  connected: ->
    # Method này sẽ được gọi khi đã kết nối với RoomChannel.

  disconnected: ->
    # Method này sẽ được gọi khi đã kết thúc kết nối với RoomChannel.

  received: (data) ->
    $('#messages').append data['message']
    #Show messges khi nhận dữ liệu từ server trả về.

  speak: (message) ->
    @perform 'speak', message: message
    # Dùng để thông báo với server là có dữ liệu truyền lên.

  $(document).on 'keypress', '#chat-speak', (event) ->
    if event.keyCode is 13 # return = send
      App.room.speak event.target.value, 
      event.target.value = ""
      event.preventDefault()

```


Cấu hình cho server.

```ruby

# app/channels/room_channel.rb
class RoomChannel < ApplicationCable::Channel
  def subscribed
    stream_from "room_channel"
  end

  def unsubscribed
  end

  def speak(data)
    Message.create! content: data['message']
  end
end
```

#### Step 5: Tạo 1 con job để thực hiện việc response cho client.

``` sh

rails g job MessageBroadcast
```

Ta sẽ sử dụng Callbacks của Active Record để gọi Jobs thực hiện response cho client.

```ruby
#models/message.rb
class Message < ApplicationRecord
  after_create_commit { MessageBroadcastJob.perform_later self }
end
```


```ruby
#jobs/message_broadcast_job.rb
class MessageBroadcastJob < ApplicationJob
  queue_as :default

  def perform(message)
    #Response về cho client.
    ActionCable.server.broadcast 'room_channel', message: render_message(message)
  end

  private
    def render_message(message)
      ApplicationController.renderer.render(partial: 'messages/message', locals: { message: message })
    end
end
```

Như vậy là đã xong phần code một trang web chat real-time. Giờ bạn có thể chạy ứng dụng `rails s`, và mở nhiều browser khác nhau để xem kết quả.



























