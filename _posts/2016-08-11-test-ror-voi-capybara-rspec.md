---
layout: post
title: Test ROR app vowis Capybara + Rspec
excerpt: "Chắc hẳn ai lập trình ROR(ruby on rails) đều biết đến rspec một công cụ viết test trên ROR. Rspec dẽ dàng giúp chúng ta viêt test cho các controller, các mode.. hơn thế nữa khi đi cùng simplecov chúng ta còn dẽ dàng biết được số lương “Code Coverage”"
modified: 2016-08-11
tags: [Capybara, Rspec, ruby, ruby on rails, test]
comments: true
---

Chắc hẳn ai lập trình ROR(ruby on rails) đều biết đến rspec một công cụ viết test trên ROR. Rspec dẽ dàng giúp chúng ta viêt test cho các controller, các mode.. hơn thế nữa khi đi cùng simplecov chúng ta còn dẽ dàng biết được số lương “Code Coverage”. Nhưng như thế liệu đã đủ hay chưa? chắc chắn câu trả lời sẽ là “chưa”. Khi viết test với controller, mode chúng ta chỉ có thể kiểm tra được sự đúng sai của từng function, Nhưng sự hoạt động đúng đắn của sản phẩm chúng ta vẫn chưa thể dám chắc. Bạn có chắc chắn được khi click vào nút “Login” khi đã nhập mật khẩu và tài khoản đúng nó sẽ dẫn tới trang ta mong muốn? hay nói cách khác bạn có chắc chắn giao diện đăng nhập của bạn hoạt động đúng khi chỉ có mỗi controller login đúng? Function login đã hoạt động chính sác chưa?
Capybara sẽ giúp chúng ta giải quết vẫn đề trên. Để hiểu dõ hơn khả năng của Capybara chúng ta sẽ cùng tìm hiểu sâu hơn về nó.

#### 1 Cài Đặt.
  B1: Capybara chạy trên ruby 1.9.3 trở lên. Ta cài đặt nó bằng câu lệnh:
  `gem install capybara`
  B2: Trong Rails app ta thêm dòng sau vào file `rspec_helper.rb`
  `require 'capybara/rails'`
  B3: Tạo thư mục features trong thư mục spec.
#### 2 Ví dụ.
  Tại đây ta sẽ lấy 1 ví dụ về test màn hình login.

  ``` ruby

describe SessionsController do
  it "signs super ok" do
    visit '/signin'
    within("//form[@id='sessions']") do
      fill_in 'email', with: 'super@gmail.com'
      fill_in 'Password', with: '123456789'
    end
    find('#btn_submit').click
    print page.html
    page.has_css?('//div[@data-pagetitle="Test Capybara | Client List"]')
  end
end
  ```


visit: Khai báo đừng link trang web. ở đây là trang login

within : tìm kiếm 1 phần tử trên trang web: tại đây là tìm tới form có id là “sessions”

fill_in : tìm kiếm các phần tử trên trang web. tại đây ta tìm kiếm tới id là email và Passoword và gán các giá trị tương ứng tại key with

find : được dùng để tìm kiếm các phần tử trên trang. nó giống như $(”) trong javascrip. find(‘#btn_submit’).click : thực hiện hành đông click bào button login. ta có thể dùng cách submit khác như là:click_button, hay click_on. Với các link ta có thể dùng click_link

page : Là giá trị mặc đinh được trả về sai khi submit nút login.

Dưới đây là các thuộc tính được dùng trong capybara

* Clicking links and buttons

```ruby

click_link('id-of-link')
click_link('Link Text')
click_button('Save')
click_on('Link Text')  # リンクとボタンどちらかをクリック
click_on('Button Value')
```

* Interacting with forms

```ruby

fill_in('First Name', with: 'John')
fill_in('Password', with: 'Seekrit')
fill_in('Description', with: 'Really Long Text...')
choose('A Radio Button')
check('A Checkbox')
uncheck('A Checkbox')
attach_file('Image', '/path/to/image.jpg')
select('Option', from: 'Select Box')
```

* Querying

```ruby
page.has_selector?('table tr')
page.has_selector?(:xpath, '//table/tr')
page.has_no_selector?(:content)

page.has_xpath?('//table/tr')
page.has_css?('table tr.foo')
page.has_content?('foo')
```


```ruby

page.should have_selector('table tr')
page.should have_selector(:xpath, '//table/tr')
page.should have_no_selector(:content)

page.should have_xpath('//table/tr')
page.should have_css('table tr.foo')
page.should have_content('foo')
page.should have_no_content('foo')

```


* Finding

```ruby

find_field('First Name').value
find_link('Hello').visible?
find_button('Send').click

find(:xpath, "//table/tr").click
find("#overlay").find("h1").click
all('a').each { |a| a[:href] }
```

```ruby

find('#navigation').click_link('Home')
find('#navigation').should have_button('Sign out')

```


* Scoping

```ruby

within("li#employee") do
  fill_in 'Name', with: 'Jimmy'
end

within(:xpath, "//li[@id='employee']") do
  fill_in 'Name', with: 'Jimmy'
end
```

```ruby

within_fieldset('Employee') do
  fill_in 'Name', with: 'Jimmy'
end

within_table('Employee') do
  fill_in 'Name', with: 'Jimmy'
end
```

* Scripting

```ruby

page.execute_script("$('body').empty()")
result = page.evaluate_script('4 + 4');
```


* Debugging

```ruby

save_and_open_page
```

#### Mở rộng.

Không chỉ dừng lại ở đó. capybara còn có thể kết hợp với selenium để tạo test tự động. Ta cũng có thể chụp lại màn hình kết quả sau khi chạy test.

ta cấu hình như sau: cài đặt các gem sau.

```ruby

gem "capybara-webkit"
gem "selenium-webdriver"
gem "capybara-screenshot"
```

Để chạy được selenium ta chỉ cần tạo dirver chạy selenium. Chúng ta sẽ xem code dưới đây để dễ hiểu hơn

```ruby
describe SessionsController do
  include Capybara::DSL
  it "signs super ok" do
    session = Capybara::Session.new(:selenium)

    session.visit 'http://testSelenium/signin'

    session.fill_in 'email', with: 'super@gmail.com'
    session.fill_in 'Password', with: '123456789'

    session.find('#btn_submit').click

    Capybara.default_wait_time = 3

    session.save_screenshot("file.png")

    session.find('.navbar-nav .active a').should have_text("Client List")
  end
end
```

Chúng ta dễ dàng thấy chỉ cần khai báo Capybara::Session.new(:selenium). Khi chạy qua dòng này chương trình sẽ tự gọi trình duyệt web mặc đinh lên và thực hiện những câu lện được cài đặt ở dưới. Chúng ta sẽ dễ dàng nhìn thấy sự hoạt động của nó trên màn hình. Điêu cần chú ý ở đây khắc với chạy thông thường là ta phải chỉ định đường link thực sự của web ( ‘http://testSelenium/signin’).


Nguồn: http://labs.septeni-technology.jp/technote/ror-test-capybara-rspec-bo-doi-tuyet-voi/




