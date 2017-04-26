---

title: 背景工作及工作排程
comments: true
permalink: /chapters/20-background-job.html

---

# 背景工作及工作排程

有些程式在執行的時候需要比較長的時間，例如前面章節介紹的寄發 Email，當使用外部的郵件伺服器的時候，有時候會要等幾秒甚至幾十秒才會繼續往下走，這樣不僅使用者體驗不是很好，也可能讓網站不正常運作（例如使用者在點下送出的時候，以為按下去沒效果結果多按了很多下）。

所以，通常這種比較耗時的工作，會先把要執行的工作先存起來，然後把使用者畫面轉往已完成頁面，待主機比較有空檔或稍晚再執行剛剛存下來的工作。

Rails 有內建一個叫做 `ActiveJob` 的類別可以來處理這樣的事情。

## 新增工作

要新增一個工作(Job)相當容易，跟之前 Scaffold 一樣，使用 Rails 內建的產生器，一行就搞定了：

    $ rails g job user_confirm_email
      invoke  test_unit
      create    test/jobs/user_confirm_email_job_test.rb
      create  app/jobs/user_confirm_email_job.rb

這行指令會在 `app/jobs` 目錄下新增一個檔案。讓我們看一下這個檔案的內容：

```ruby
class UserConfirmEmailJob < ApplicationJob
  queue_as :default

  def perform(*args)
    # Do something later
  end
end
```

檔案的內容其實滿簡單的，產生的檔案（扣除測試的話）也就一個而已，所以也不一定要用產生器，直接手動建立也可。

其中，`queue_as` 方法是指這件工作急不急，預設值是 `:default`，如果這件工作不急，可把 `:default` 改成 `:low_priority`，如果是急件則可設定成 `:urgent`。

然後這個 `perform` 方法就是「真正在做事情的地方」，稍微改一下 `perform` 的內容：

```ruby
class UserConfirmEmailJob < ApplicationJob
  queue_as :default

  def perform(user)
    # 在這裡寄發確認信...
  end
end
```

這裡 `perform` 方法期待會接收一個 User 物件，然後寄發信件給這個使用者（寄信方法可見前一個章節）。

## 開始工作

舉例來說，如果我想要在「成功建立使用者之後寄發確認信件」，可以這樣做：

```ruby
class UsersController < ApplicationController
  # ...[略]...

  def create
    @user = User.new(user_params)
    if @user.save
      UserConfirmEmailJob.perform_later(@user)
      redirect_to @user, notice: 'User was successfully created.'
    else
      render :new
    end
  end

  # ...[略]...
end
```

這樣就行了，其中這行：

```ruby
UserConfirmEmailJob.perform_later(@user)
```

意思是這個工作「待會做」，即使這個工作需要耗時 10 秒鐘，Rails 也不會真的在這卡 10 秒鐘才往下執行，而是先把這個工作「排隊」排到某個地方（如果是設定成 `:urgent` 的話就是可以插隊的意思），然後程式繼續往下執行，所以使用者的體驗就會感覺好像馬上完成了，事實上是待會才會做這件事。

至於這個待會是多待會，就是看稍會系統比較不忙的時候做。

另外，如果想要指定什麼時候做的話，可以這樣寫：

```ruby
# 這樣是 5 秒之後做
UserConfirmEmailJob.set(wait: 5.seconds).perform_later(@user)

# 這樣是「明天下午有空再做」
UserConfirmEmailJob.set(wait_until: Date.tomorrow.noon).perform_later(@user)
```

## 「排隊」是在哪邊排隊？

前面說到的「排隊」（Queue）其實可以在好幾種地方排隊，預設是會把排程放在記憶體裡，但如果萬一伺服器當機或重開機，這個排隊的資料就不見了。在實務上常會另外設置可以排隊的地方，常見的有 `Sidekiq` 跟 `Delayed Job`，我們來試一下比較容易設定的 `delayed_job`

Delayed::Job 網址：https://github.com/collectiveidea/delayed_job

請照頁面說明，安裝 `delayed_job_active_record`：

```ruby
gem 'delayed_job_active_record'
```

> 注意：別忘了執行 `bundle install` 確認安裝套件，同時也可能需要重新啟動 `rails server`

這個 gem 會建立一個叫做 `delayed_jobs` 的資料表專門存放工作的資料表：

    $ rails generate delayed_job:active_record

記得執行 `rails db:migrate` 指令，把 Migration 轉換成資料表。接下來，要改一下 `ActiveJob` 內建存放工作的地方，請編輯 `app/configs/application.rb`：

```ruby
module MyBlog
  class Application < Rails::Application
    config.active_job.queue_adapter = :delayed_job
  end
end
```

加上的這行：

```ruby
config.active_job.queue_adapter = :delayed_job
```

意思就是把工作透過 `delayed_job` 存到資料表，然後就可以稍候再執行了。

