---

title: 程式碼整理
comments: true
permalink: /chapters/24-organize-your-code

---

# 程式碼整理

當 Rails 專案成長到一定程度後，如果沒有好好的整理程式碼，很有可能發生重複的程式碼到處散落的情況。接下來這個章節是要介紹如何使用 Ruby 跟 Rails 內建的方法或設計來整理重複的程式碼。

## <a name="duplicated-logic-in-view"></a>在 View 出現有點複雜或重複的邏輯

先看一下這個畫面：

![image](/images/chapter24/user-list-1.png)

因為某些因素，在設計使用者性別（Gender）欄位的時候，可能會用數字 `1` 表示男生，用數字 `0` 表示女生。如果我想直接印出「男」、「女」字樣，可能會這樣寫：

```ruby
<tbody>
  <% @users.each do |user| %>
    <tr>
      <td><%= user.name %></td>
      <td><%= user.email %></td>
      <td>
        <% if user.gender == 1 %>
          男
        <% else %>
          女
        <% end %>
      </td>
      <td><%= link_to 'Show', user %></td>
      <td><%= link_to 'Edit', edit_user_path(user) %></td>
      <td><%= link_to 'Destroy', user, method: :delete, data: { confirm: 'Are you sure?' } %></td>
    </tr>
  <% end %>
</tbody>
```

這裡使用 `if...else...` 判斷 `user.gender` 的值然後印出字樣，以結果來看是沒問題，但在開發 Rails 專案的時候，以 MVC 的結構來說，儘量不要讓 View 有邏輯判斷，View 的工作，就是乖乖的輸出資料就好。

### 1. 使用 View Helper

在前面第 14 章有介紹到如何使用 View Helper 來把這段邏輯藏起來：

```ruby
# 檔案：app/helpers/users_helper.rb

module UsersHelper
  def print_gender(user)
    if user.gender == 1
      "男"
    else
      "女"
    end
  end
end
```

這樣一來，原來那段 View 的寫法就可改成：

```erb
<% @users.each do |user| %>
  <tr>
    <td><%= user.name %></td>
    <td><%= user.email %></td>
    <td><%= print_gender(user) %></td>
    <td><%= link_to 'Show', user %></td>
    <td><%= link_to 'Edit', edit_user_path(user) %></td>
    <td><%= link_to 'Destroy', user, method: :delete, data: { confirm: 'Are you sure?' } %></td>
  </tr>
<% end %>
```

這樣一來，原來的 `if..else..` 邏輯就可以被包到 View Helper 裡，而且其它頁面要用也可以用得上。

### 2. 在 Model 上新增實體方法

除了使用 View Helper，以上面這個例子來說，也可在 User Model 裡直接新增一個實體方法：

```ruby
class User < ApplicationRecord
  validates :name, presence: true

  def show_gender
    if gender == 1
      "男"
    else
      "女"
    end
  end
end
```

然後 View 就可改寫成：

```erb
<% @users.each do |user| %>
  <tr>
    <td><%= user.name %></td>
    <td><%= user.email %></td>
    <td><%= user.show_gender %></td>
    <td><%= link_to 'Show', user %></td>
    <td><%= link_to 'Edit', edit_user_path(user) %></td>
    <td><%= link_to 'Destroy', user, method: :delete, data: { confirm: 'Are you sure?' } %></td>
  </tr>
<% end %>
```

哪種做法比較好？

如果這個邏輯可能跟其它同一個 View 的變數有關，我會選擇第 1 種做法；如果就只是像這個例子一樣，資料的呈現僅與自身 Model 有關，我個人會比較偏好第 2 種寫法。

## <a name="using-controller-callback"></a>在 Controller 好幾個 Action 都看到在做一樣的事

舉個例子來說：

```ruby
class UsersController < ApplicationController
  def show
    @user = User.find(params[:id])
  end

  def edit
    @user = User.find(params[:id])
  end

  def update
    @user = User.find(params[:id])
    respond_to do |format|
      #...[略]...
    end
  end

  def destroy
    @user = User.find(params[:id])
    @user.destroy
    respond_to do |format|
      #...[略]...
    end
  end
end
```

在這個 Controller 裡，`show`、`edit`、`update` 以及 `destroy` 都有用 `User.find(params[:id])` 的方法在查詢使用者，像這種在同一個 Controller 裡有好幾個 Action 都在做類似的事，可以使用 Controller 內建的 Callback，例如：

```rubbby
class UsersController < ApplicationController
  before_action :set_user, only: [:show, :edit, :update, :destroy]

  #...[略]...

  private
  def set_user
    @user = User.find(params[:id])
  end
end
```

定義一個 `set_user` 方法（通常會掛在 `private` 區塊），然後掛在 `before_action` 這個 Callback 上，並且僅在 `show`、`edit`、`update` 以及 `destroy` 這 4 個 Action 執行前先執行。

其它可以用的 Callback 還有 `after_action` 跟 `around_action` 等方法，更多詳細內容可參考 http://api.rubyonrails.org/classes/AbstractController/Callbacks/ClassMethods.html

## <a name="organize-model-logic-with-scope"></a>在 Controller 看到有點長的連續技

不知道大家有沒有在 Controller 看過類似這樣的程式碼：

```ruby
class UsersController < ApplicationController
  def index
    @users = User.where(gender: 0, city: 'Taipei').where("age >= 18")
  end
end
```

雖然看得出來大概是要查「住台北的成年女性」的使用者，但這樣寫等於是把這個查詢的「邏輯」寫在 Controller 裡了，如果在別的 Controller 要查一樣的資料，就又得再複製、貼上一次。

Rails 的 Model 有提供 `Scope` 或類別方法可以把這個邏輯包起來：

```ruby
class User < ApplicationRecord
  validates :name, presence: true

  scope :adult_female_live_in_taipei, -> { where(gender: 0, city: 'Taipei').where("age >= 18") }
end
```

這樣一來原來那段就可簡化成：

```ruby
class UsersController < ApplicationController
  def index
    @users = User.adult_female_live_in_taipei
  end
end
```

不僅在每個地方都可以使用，而且光看方法名字就大概可以猜得出來是要查什麼資料。

## <a name="using-inheritance"></a>好幾個 Controller 或 Model 都有一樣的功能

如果我們做了後台管理系統，應該會希望「所有後台管理系統的 Controller 在 `before_action` 的地方都要先檢查有沒有登入」。當然，你可以在每個後台 Controller 都加上權限控管，但也可考慮使用物件導向程式設計的「繼承」來解決這件事。

在 Rails 的 Controller，如果沒有特別改過，預設應該是繼承自 `ApplicationController` 這個類別，大概像這樣：

![image](/images/chapter24/inheritance-1.png)

但如果想讓每個後台管理系統都會在 `before_action` 做某件事，可以額外新增一個 `Admin::BaseController` 類別：

```ruby
class Admin::BaseController < ApplicationController
  before_action :do_something

  private
  def do_something
    #....
  end
end
```

然後讓所有後台的 Controller 都改繼承這個 `Admin::BaseController`：

```ruby
class Admin::UsersController < Admin::BaseController
  #...[略]...
end
```

原來的關係圖就會變成像這樣：

![image](/images/chapter24/inheritance-2.png)

利用物件導向的繼承功能，可以把共同的程式碼集中在上層類別。

## <a name="using-concern"></a>繼承雖然容易用，但不是每個 Controller 或 Model 都需要這個功能...

雖然繼承可以「把重複的程式碼寫在上層類別」，但很多時候並不是每個 Controller 或 Model 都想要有這個功能。就跟在第 8 章物件導向程式設計章節的「模組」一樣，有需要這個功能才引進來。

Rails 有提供 `Concern` 的功能，可以把「共同的行為」集中起來，有需要的再「引入」，而不使用繼承。就是「不要為了想要會飛就去當鳥的小孩」的概念：

舉個例子，我有 User 跟 AdminUser 這兩個 Model，我希望這兩個 Model 都：

1. 都有 `has_one :profile` 設定
2. 都有 `show_gender` 方法可以顯示性別字串
2. 在在新增帳號的時候都可以對輸入的密碼加密

```ruby
module Profileable
  extend ActiveSupport::Concern

  included do
    has_one :profile
    before_create :encrypt_user_password
  end

  module ClassMethods
  end

  def show_gender
    if gender == 1
      "男"
    else
      "女 "
    end
  end

  private
  def encrypt_user_password
    # 對密碼加密...
  end
end
```

其實 Concern 就是 Ruby 裡 Module 的概念，說明如下：

1. `included do ... end` 裡面放的是當這個 Module 被 include 的時候會做的事
2. `ClassMethod` 這個 Module 裡面可以定義方法，但定義的方法會直接變成類別方法
3. `show_gender` 方法被 include 之後就會變成該類別的實體方法

接下來，在 `User` Model 加上一行：

```ruby
class User < ApplicationRecord
  include Profileable
end
```

另外 `AdminUser` Model 也可以加上這行：

```ruby
class AdminUser < ApplicationRecord
  include Profileable
end
```

這樣一來，這兩個 Model 就都有 `Profileable` 這個 Module 所提供的功能了。

