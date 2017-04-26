---

title: API 模式
comments: true
permalink: /chapters/21-api-mode.html

---

# API 模式

- [輸出成 JSON 格式 - 使用 render](#output-json-with-render)
- [輸出成 JSON 格式 - 使用 Jbuilder](#output-json-with-jbuilder)
- [API-Only 模式](#api-only)

在開發手機應用程式或是一些前端比較吃重的網站應用程式時，常會需要跟後端伺服器交換資料，交換資料的格式常見的有 JSON 或 XML 等格式，這個交換介面又稱之 API（Application Programming Interface）。雖然 Rails 的強項之一是資料的 CRUD（新增、修改、刪除）功能，但其實要拿來做 API（Application Programming Interface）也是相當合適的。

## <a name="output-json-with-render"></a>輸出成 JSON 格式 - 使用 render

舉個 User scaffold 的例子：

```ruby
class UsersController < ApplicationController
  # ...[略]...

  def index
    @users = User.all
  end

  # ...[略]...
end
```

這邊的 `index` Action 如果沒特別聲明 `render` 方法，它就會去 `app/views/users/` 目錄下找同名的 `index.html.erb` 來呈現畫面，這個我們前面都介紹過。但如果不需要畫面，可以把 `index` Action 改成這樣：

```ruby
def index
  @users = User.all
  render json: @users
end
```

原本的樣子長這樣：

![image](/images/chapter21/index-html.png)

這樣就會直接在 Controller 階段就直接以 JSON 格式輸出，輸出結果如下：

![image](/images/chapter21/index-json.png)

排版整理後結果如下：

```json
[
   {
      "id":1,
      "name":"孫悟空",
      "email":"eddie@5xruby.tw",
      "created_at":"2017-01-06T08:24:00.045Z",
      "updated_at":"2017-01-06T08:24:00.045Z"
   },
   {
      "id":2,
      "name":"貝曲達",
      "email":"hi@5xruby.tw",
      "created_at":"2017-01-06T08:24:08.318Z",
      "updated_at":"2017-01-06T08:24:08.318Z"
   },
   {
      "id":3,
      "name":"丁小雨",
      "email":"rain@5xruby.tw",
      "created_at":"2017-01-06T08:24:27.352Z",
      "updated_at":"2017-01-06T08:24:27.352Z"
   }
]
```

如果你原本是開發手機應用程式的工程師或前端工程師，不需要拜託別人，簡單幾行就可以取得你所需要的 JSON 格式資料了。

但上面這樣的寫法，原本 `/users` 的頁面就不見了，也就是說如果要同時可以呈現 HTML 跟 JSON 的話，得另外設計網址，有點麻煩...讓我們接著看看另一種做法。

## <a name="output-json-with-jbuilder"></a>輸出成 JSON 格式 - 使用 Jbuilder

不知道大家有沒注意到，其實在使用 Scaffold 產生一堆檔案的時候，在 View 的目錄裡有多一些特別的檔案：

![image](/images/chapter21/views.png)

這幾個結尾是 `.json.jbuilder` 的檔案，就跟 `.html.erb` 的概念一樣，`.html.erb` 是指會使用 ERB 樣版引擎來解讀這個檔案，並轉換成 HTML 格式；而 `.json.jbuilder` 則是使用 [Jbuilder](https://github.com/rails/jbuilder) 這個 gem，把結果輸出成 JSON 格式。

讓我們看一下 `app/views/users/index.json.jbuilder` 這個檔案的內容：

```ruby
json.array! @users, partial: 'users/user', as: :user
```

看來是去 render `users/_user` 這個檔案，裡面的內容是：

```
json.extract! user, :id, :name, :email, :created_at, :updated_at
json.url user_url(user, format: :json)
```

意思大概就是會輸出 `user` 物件的 `id`、`name`...`updated_at` 等欄位的資料。

原本網址 `http://localhost:3000/users` 可以看到使用者列表，如果把網址後面加上 `.json` 變成 `http://localhost:3000/users.json` 就能得到跟前面 `render json: @users` 一樣的結果。

不只如此，`/users/2` 會用 HTML 格式輸出 2 號使用者資料，如果是 `/users/2.json` 則是會以 JSON 格式輸出這筆資料。

另外，像是如果我只想印出姓名跟 Email，可以把 `users/_user.json.jbuilder` 的內容修改成這樣：

```
json.extract! user, :name, :email
```

輸出結果就會變成（經過排版結果）：

```json
[
   {
      "name":"孫悟空",
      "email":"eddie@5xruby.tw"
   },
   {
      "name":"貝曲達",
      "email":"hi@5xruby.tw"
   },
   {
      "name":"丁小雨",
      "email":"rain@5xruby.tw"
   }
]
```

使用 `render json: @users` 的寫法只能一口氣把 `@users` 的內容全部印出來，而使用 Jbuilder 則是可以微調要輸出的欄位。另一個好處就是原本 `/users` 的頁面還是可保持 HTML 的頁面呈現，只有 `/users.json` 的時候才會以 JSON 格式輸出。

### 為什麼加 `.json` 就可以了？

其實這算是 Route 做的好事，先看一下 `rails routes` 的結果：

    $ rails routes
       Prefix Verb   URI Pattern               Controller#Action
        users GET    /users(.:format)          users#index
              POST   /users(.:format)          users#create
     new_user GET    /users/new(.:format)      users#new
    edit_user GET    /users/:id/edit(.:format) users#edit
         user GET    /users/:id(.:format)      users#show
              PATCH  /users/:id(.:format)      users#update
              PUT    /users/:id(.:format)      users#update
              DELETE /users/:id(.:format)      users#destroy

有注意到後面那個 `(.:format)` 嗎？就是這個東西。假設你輸入網址 `/users.html` 的時候，後面的附檔名就會被捕捉成 `format`，然後如果在 Controller 的 Action 沒有特別聲明要以什麼方式 render 的話，就會依據這個 `format` 去找對應的檔案，也就是說會找到 app/views/index.`html`.erb。（還記得結果是 `.html.erb` 的意思嗎？）

同理可證，當網址是輸入 `/users.json` 的時候，它會去找 app/views/index.`json`.jbuilder 來輸出結果。 如果後面沒有附檔名，例如 `/users`，則是預設會去找 `html` 的樣版。

## <a name="api-only"></a>API-Only 模式

Rails 方便歸方便，它本身是個有點肥大的框架，這是一直以來 Rails 被詬病的地方。在 Rails 5 之後，Rails 加入了 API only 的模式，讓你在產生專案的時候減少一些不必要的套件及 middleware。讓我們產生一個全新的專案：

    $ rails new my_blog --api

後面加上了 `--api` 參數產生出來的專案，因為僅需要做 API 的輸出，所以不僅用到的 gem 變少了，連用到的 middleware 也比較少一點，在效能的提昇上是有一些幫助的。

更多詳細資料請參閱 http://guides.rubyonrails.org/api_app.html

但即使如此，Rails 畢竟是一個功能完整的網站開發框架，改成 API only 也只是從很胖的胖子變成普通胖的胖子，如果真的有效能考量，而且沒有要用到那麼完整的功能，也許可直接考慮使用更輕量化的工具，例如 Sinatra ([http://www.sinatrarb.com/](http://www.sinatrarb.com/))。

