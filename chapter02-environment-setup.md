---

title: 環境設定
comments: true
permalink: /chapters/02-environment-setup.html

---

# 環境設定

要開始用 Ruby 或 Rails 來開發網站之前，第一件事（其實可能也是最難的事？）就是要先搞定開發環境。

- [安裝 Ruby](#install-ruby)
- [用 RVM 來管理 Ruby 版本](#use-rvm)
- [安裝 Rails](#install-rails)
- [建立 Rails 專案](#build-rails-project)

## <a name="install-ruby"></a>安裝 Ruby

### Unix/Linux 作業系統

如果您使用的是 Ubuntu 之類的系統，可以直接使用 `apt-get` 來安裝 Ruby：

    $ sudo apt-get install ruby

如果是 CentOS 之類的系統，則是使用 `yum` 來安裝：

    $ sudo yum install ruby

因為使用了 `sudo` 指令，所以你應該會被提示需要輸入目前的登入密碼。

### Mac 作業系統

如果是 Mac 作業系統，比較新的版本均已內建 Ruby 2.0 版本，如果沒有的話，建議可使用 [Homebrew](http://brew.sh/) 這個套件管理工具來安裝 Ruby：

    $ brew install ruby

### Windows 作業系統

在 Windows 平台可根據您的需求，選擇安裝 [Ruby Installer](http://rubyinstaller.org/) 或 [Rails Installer](http://railsinstaller.org/en)，基本上 Rails Installer 跟 Ruby Installer 沒太大的差別，只是後者多加入了 Rails 開發的相關工具（例如 Git、Bundler 等）：

### 其它系統

更多其它平台的安裝方式，或是想要直接下載原始碼自行編譯，請參閱 [Ruby 官方網站](https://www.ruby-lang.org/)的安裝說明。

## <a name="use-rvm"></a>用 RVM 來管理 Ruby 版本

Ruby 有許多的版本（1.8/1.9/2.0/2.1/2.2/2.3/2.4）以及眾多的分支實作品（例如 JRuby/IronRuby/Rubinius/Macruby/mruby 等），算一算有不少排列組合，如果想要在自己機器上安裝不同版本會有點麻煩，而且萬一亂裝把工作環境弄壞了還得花時間重建。如果是要裝在伺服器上，如果你不是系統管理員，還不一定有足夠的權限可以安裝。使用 VirtualBox 的軟體可以來模擬作業環境，玩壞了隨時都可以很快的還原或重建一個新的，即使這樣還是有點麻煩。

如果各位跟我一樣都喜歡玩些新玩具，但又擔心環境被弄髒弄壞，推薦大家可以試試 [RVM](https://rvm.io/)（Ruby Version Manager）。有 RVM 的幫忙，你可以安心的在你的電腦裡同時安裝多個不同版本的 Ruby/Rails 而不會搞混，隨時都可以輕鬆的切換。

RVM 是把程式安裝在你的的個人帳號目錄下，不需要的時候就整個 `~/.rvm` 資料夾刪除就行了，不會影響原來系統的設定。也就是因為 RVM 是安裝在你的個人帳號底下，所以你在安裝過程中不需要管理者（root）的權限就可以安裝其它相關的套件。

### 安裝 RVM

RVM 的安裝滿簡單的，只要二行指令即可完成。安裝步驟請直接參閱 [RVM 官網](https://rvm.io/)的安裝說明。

注意如果沒有安裝過 `gpg` 的話，第一行指令會得到 `-bash: gpg: command not found` ，一樣建議可使用 [Homebrew](http://brew.sh/) 這個套件管理工具來安裝。

    $ brew install gnupg gnupg2

### 使用 RVM

接著我們來看一些在 RVM 裡常用的指令。在終端機下輸入 `rvm list known` 會列出目前有哪些可以安裝的列表：

    $ rvm list known
    # MRI Rubies
    [ruby-]1.8.6[-p420]
    [ruby-]1.8.7[-head] # security released on head
    [ruby-]1.9.1[-p431]
    [ruby-]1.9.2[-p330]
    [ruby-]1.9.3[-p551]
    [ruby-]2.0.0[-p648]
    [ruby-]2.1[.10]
    [ruby-]2.2[.6]
    [ruby-]2.3[.3]
    [ruby-]2.4[.0]
    ruby-head

    [...略...]

    macruby[-0.12]
    macruby-nightly
    macruby-head

    # IronRuby
    ironruby[-1.1.3]
    ironruby-head

幾乎目前常見的 Ruby 分支實作品都有。列表裡的中括號表示那些是可以省略的，所以如果你這樣輸入：

    $ rvm install 2.4

RVM 會自動找 `[ruby-]2.4[.0]` 這個版本的 Ruby 來安裝。前面提到可以安裝多個不同的版本，所以如果你喜歡，也可以再裝個 `1.9.3` 的版本：

    $ rvm install 1.9.3

安裝完成後，我們可以使用 `rvm list` 來查看目前電腦裡已經安裝哪些版本的 Ruby：

    $ rvm list

       ruby-1.9.3-p551 [ x86_64 ]
       ruby-2.2.1 [ x86_64 ]
       ruby-2.2.2 [ x86_64 ]
       ruby-2.3.0 [ x86_64 ]
       ruby-2.3.1 [ x86_64 ]
       ruby-2.3.3 [ x86_64 ]
    =* ruby-2.4.0 [ x86_64 ]

    # => - current
    # =* - current && default
    #  * - default

因為工作上需求，所以在我的電腦上裝了好幾個版本的 Ruby。在 2.4.0 版前面的 `=*` 符號則是表示我目前正在使用這個版本。你可以在終端機下輸入這個指令，看看目前 Ruby 的版本：

    $ ruby -v
    ruby 2.4.0p0 (2016-12-24 revision 57164) [x86_64-darwin15]

如果要切換到其它版本的 Ruby，例如想要切換到 1.9.3 版本：

    $ rvm use 1.9.3

想少打幾個字的話，`use` 也可以省略：

    $ rvm 1.9.3

再來看一下Ruby的版本：

    $ ruby -v
    ruby 1.9.3p551 (2014-11-13 revision 48407) [x86_64-darwin13.4.0]

這樣就切換到 Ruby 1.9.3 了，相當便利！不過有個小問題，就是使用 RVM 指定的 Ruby 版本會在每次開啟新的終端機視窗的時候變回預設值（也就是變回系統內建的 Ruby 版本），所以如果你希望每次開終端機視窗的時候都會自動切到 `2.4.0` 版的話：

    $ rvm 2.4.0 --default

這樣之後每次開終端機視窗就會自動幫你切換到 2.4.0 版了。如果想切回到原來系統內建的版本，只要執行這個指令：

    $ rvm system

想移除某個版本的 Ruby 的話：

    $ rvm uninstall 2.4.0

這樣就可以把 `2.4.0` 版本移除掉了。如果是整個 RVM 都不想要了，只要把個人帳號 home 資料夾底下的 `.rvm` 資料夾整個移除，就會整個清潔溜溜了，完全不會動到系統內建的 Ruby。

### 運作原理

你也許會好奇為什麼 RVM 可以這麼神奇的切換 Ruby 的環境。讓我們來把系統的 PATH 變數印出來看看：

    $ echo $PATH
    /Users/user/.rvm/gems/ruby-2.4.0/bin:/Users/user/.rvm/gems/ruby...[略]...

然後查看一下 Ruby 的位置：

    $ which ruby
    /Users/user/.rvm/rubies/ruby-2.4.0/bin/ruby

再查一下 Ruby 版本：

    $ ruby -v
    ruby 2.4.0p0 (2016-12-24 revision 57164) [x86_64-darwin15]

（以上內容是我自己電腦裡的設定，應該跟各位的環境不同）

接下來把 RVM 切換到 1.9.3 版本：

    $ rvm 1.9.3

再重複把剛剛的那些資訊印出來：

    $ echo $PATH
    /Users/user/.rvm/gems/ruby-1.9.3-p551/bin:/Users/user/.rvm/gems...[略]...

    $ which ruby
    /Users/user/.rvm/rubies/ruby-1.9.3-p551/bin/ruby

    $ ruby -v
    ruby 1.9.3p551 (2014-11-13 revision 48407) [x86_64-darwin13.4.0]

仔細看上面的輸出結果，就會發現其實 RVM 是把不同版本的 Ruby 安裝在你的個人帳號底下的 `.rvm` 目錄裡。當你切換不同版本的 Ruby 的時候，RVM 會幫你把系統預設的 PATH 的最前面加上這個 `.rvm` 的資料夾。接下來當你在終端機底下輸入 `ruby` 指令時，系統原本的 `/usr/bin/ruby` 因為在 PATH 的比較後面的位置，所以系統只會先找到 RVM 版本的 Ruby（也就是原來系統的 Ruby 被鬼摭眼了）。如果各位有興趣，也可以試著輸入 `rvm info` 指令來看看 RVM 幫你做了哪些設定。

### 除了 RVM 之外...

我自己個人習慣使用 RVM，除了 RVM 之外還有其它的選擇，例如 [rbenv](https://github.com/rbenv/rbenv) 及 [chruby](https://github.com/postmodern/chruby)，這些 Ruby 版本管理工具各有其優、缺點，還請大家自己去試用看看，然後選一套自己覺得順手的來用吧。

## <a name="install-rails"></a>安裝 Rails

完成 Ruby 安裝後，接下來就準備來安裝 Rails。在開放原始碼的圈子，有非常多的善心人士開發好了功能強大又可免費取用的套件，在 Ruby 的世界我們稱它叫 `gem`。Ruby on Rails 這個網站開發框架本身也是一個 gem（更準確的說，應該是一群 gem 的集合體），要安裝 rails 的話，只要使用 `gem install` 指令加上套件名稱即可，像這樣：

    $ gem install rails
    Fetching: i18n-0.7.0.gem (100%)
    Successfully installed i18n-0.7.0
    Fetching: thread_safe-0.3.5.gem (100%)
    Successfully installed thread_safe-0.3.5
    Fetching: tzinfo-1.2.2.gem (100%)
    Successfully installed tzinfo-1.2.2
    Fetching: concurrent-ruby-1.0.4.gem (100%)
    Successfully installed concurrent-ruby-1.0.4
    Fetching: activesupport-5.0.1.gem (100%)
    Successfully installed activesupport-5.0.1
    ...[略]...
    36 gems installed

從安裝過程的訊息可大概看到 `5.0.1` 的字樣。如果過程沒發生錯誤訊息的話，接下來確認一下是不是安裝了正確的版本：

    $ rails -v
    Rails 5.0.1

搞定！接下來，我們就要用它來建立第一個 Rails 專案了。

## <a name="build-rails-project"></a>建立 Rails 專案

Rails 安裝完成後，接下來就用它來產生一個名為 `hello_rails` 的 Rails 專案，建立新專案用的是 `new` 這個參數：

    $ rails new hello_rails
          create
          create  README.md
          create  Rakefile
          create  config.ru
          create  .gitignore
          create  Gemfile
          create  app
          ...[略]...
          create  vendor/assets/javascripts/.keep
          create  vendor/assets/stylesheets
          create  vendor/assets/stylesheets/.keep
          remove  config/initializers/cors.rb
             run  bundle install
    Fetching gem metadata from https://rubygems.org/..........
    Fetching version metadata from https://rubygems.org/..
    Fetching dependency metadata from https://rubygems.org/.
    ...[略]...
    Using rails 5.0.1
    Installing sass-rails 5.0.6
    Bundle complete! 15 Gemfile dependencies, 62 gems now installed.
    Use `bundle show [gemname]` to see where a bundled gem is installed.
             run  bundle exec spring binstub --all
    * bin/rake: spring inserted
    * bin/rails: spring inserted

`rails new hello_rails` 這個幫你產生了一個名為 `hello_rails` 的目錄，接下來請使用 `cd` 指令進到剛剛產生的這個目錄：

    $ cd hello_rails

進到這個專案之後，什麼事都不用做，直接啟動 Rails 附的 web server：

    $ rails server
    => Booting Puma
    => Rails 5.0.1 application starting in development on http://localhost:3000
    => Run `rails server -h` for more startup options
    Puma starting in single mode...
    * Version 3.6.2 (ruby 2.4.0-p0), codename: Sleepy Sunday Serenity
    * Min threads: 5, max threads: 5
    * Environment: development
    * Listening on tcp://localhost:3000
    Use Ctrl-C to stop

打開瀏覽器，連上網址 `http://localhost:3000/` ，你應該可以看到這個畫面：

![image](/images/chapter02/welcome_page.png)

恭喜你，你已經順利把 Ruby 跟 Rails 安裝完成，而且建立了一個 Rails 專案並順利跑起來了。雖然環境安裝看起來只是一小步，但這一小步對第一次接觸終端機、要打一堆指令的人來說已經是不小的一步了。

