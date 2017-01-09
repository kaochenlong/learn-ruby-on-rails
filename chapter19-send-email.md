# 寄發信件

## <a name="send-mail">寄發信件

在 Rails 要寄發信件其實滿容易的，可以使用內建的 ActionMailer 來做這件事。首先，讓我們先用 mailer 產生器來產生需要的檔案：

    $ rails g mailer Contact
        create  app/mailers/contact_mailer.rb
        invoke  erb
        create    app/views/contact_mailer
     identical    app/views/layouts/mailer.text.erb
     identical    app/views/layouts/mailer.html.erb
        invoke  test_unit
        create    test/mailers/contact_mailer_test.rb
        create    test/mailers/previews/contact_mailer_preview.rb

這裡透過產生器，產生了一個 `ContactMailer` 類別。先讓我們看一下 `app/mailers` 目錄，現在裡面應該有 2 個檔案，分別是 `application_mailer.rb` 跟 `contact_mailer.rb`。先看一下 `application_mail.erb`：

```ruby
class ApplicationMailer < ActionMailer::Base
  default from: 'from@example.com'
  layout 'mailer'
end
```

這裡的 `default` 方法後面接著 `from: 'from@example.com'`，意思就是如果沒有特別指定的話，預設信件的寄件者就是 `from@example.com`，所以這裡可以改成我們自己的信箱。下一行的 `layout` 是指會去找 `app/views/layouts/mailer` 這個樣版。

再讓我們看一下 `contact_mailer.rb`

```ruby
class ContactMailer < ApplicationMailer
end
```

有沒發現這好像有點熟悉? 其實 Mailer 的檔案結構，跟 Controller 有點像：

|          | Controller                | Mailer                          |
|----------|:--------------------------|:--------------------------------|
| 類別     | UsersController           | ContactMailer                   |
| 繼承類別 | ApplicationController     | ApplicationMailer               |
| Layout   | app/views/layouts/application.html.erb  | /app/views/layouts/mailer.html.erb               |
| View     | app/views/users/*.erb     | /app/views/contact_mailer/*.erb |

### mailer 上的 action?

即然說 Mailer 跟 Controller 有點像，在 Mailer 也有類似像 action 之類的角色，寫起來大概像這樣：

```ruby
class ContactMailer < ApplicationMailer
  def say_hello_to(user)
    @user = user
    mail to:@user.email, subject:"你好!!"
  end
end
```

在這邊我定義了一個叫做 `say_hello_to` 的方法，並期待會接一個 user 物件做為參數。其中真正進行寄信的是 `mail` 那行。

咦? 等等，那信件內容呢? 其實跟 Controller 相比較起來，信件內容大概就是跟 View 差不多的角色。讓我們在 `app/views/contact_mailer` 目錄裡新增一個跟剛剛這個方法「同名」的檔案(檔案名稱：say_hello_to.html.erb)，並且加上以下內容：

```erb
<%= @user.name %>，你好：

今天你也有過得開心嗎!

謝謝你的來信，再見!
```

寫起來的手感是不是真的跟一般的 View 很像呢? 它一樣也可以從 Mailer 取得實體變數，並輸出在這個檔案裡。

### 準備寄發!

準備來寄信吧! 我希望在新增 User 的時候，如果新增成功，就寄一封通知信給這位使用者：

```ruby
class UsersController < ApplicationController
  # ...[略]
  def create
    @user = User.new(user_params)
    if @user.save
      ContactMailer.say_hello_to(@user).deliver
      redirect_to @user, notice: 'User was successfully created.'
    else
      render :new
    end
  end
  # ...[略]
end
```

其中這行：

```ruby
ContactMailer.say_hello_to(@user).deliver
```

就是寄信的地方了。

有發現不對勁的地方嗎? 我們在 `ContactMailer` 裡定義 `say_hello_to` 方法的時候，明明是實體方法，為什麼這邊用起來像類別方法? 其實這算是 Mailer 幫你做的魔術，定義在 Mailer 裡的方法，都會被轉換成類別方法，使用起來更方便。

試一下：

![image](images/chapter19/sendmail-1.png)

按下送出之後，過沒多久就收到信了：

![image](images/chapter19/sendmail-2.png)

