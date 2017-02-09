---

title: Model、View、Controller 三分天下
comments: true
permalink: /chapters/10-mvc

---

# Model、View、Controller 三分天下

- [為什麼要這麼麻煩？](#why-mvc)
- [圖解 MVC](#mvc-flow)
- [目錄結構](#project-folders)

Rails 的專案是採用 Model、View、Controller（簡稱 MVC）的方式設計的。當年我在開發 PHP 專案的時候都沒有這麼麻煩，就是在一個 .php 檔案就處理完查詢資料庫、展示資料等工作（那時候沒有使用其它框架）。所以在我剛開始學習 Rails 的時候一直有這個疑問，就是「為什麼 Rails 要搞這麼複雜？檔案分得這麼細要一直切換實在很麻煩耶！」

## <a name="why-mvc"></a>為什麼要這麼麻煩？

### 分工容易

拆解成 MVC 結構之後，雖然檔案變多、變分散了，但也因此更容易進行分工，當團隊人數增加，每個人可以在各自負責的部份進行開發，較不易互相衝突、干擾。

### 開發慣例

另一個好處，就是因為整個 Rails 專案都是遵循 MVC 的結構，所以即使是不同程度的開發者寫出來的 Rails 專案，Controller 通常會放在 `app/controllers` 目錄裡，Model 應該也會放在 `app/models` 裡，不會有太大的差別。

## <a name="mvc-flow"></a>圖解 MVC

我們用一張圖來說明 Rails 裡的 MVC 是怎麼運作的：

![image](/images/chapter10/mvc.png)

1. 當有使用者輸入網址，連到你的網站的時候，第一關會遇到的是路徑對照表（Route，檔案 `config/routes.rb`）。
2. 在這個路徑對照表裡，記錄著這個網站對外開放的路徑對照表。Rails 會根據使用者輸入的網址及參數，比對這個路徑對照表的資料，然後告訴你應該去找哪個 Controller 上的哪個 Action；或是在對照表裡查不到相關資料，然後就會告訴你 `HTTP 404` 找不到頁面。
3. 在 Controller 上通常會有好幾個 Action，其實這些 Action 說穿了就是一般的方法而已。透過路徑對照表，找到了對應的 Action，這個 Action 會決定要做什麼事。
4. 舉例個子來說，在這個 Action 可能會需要查閱「目前所有的商品列表」，接著它就會去請 Model 幫忙要資料。
5. 雖然 Model 本身並不是資料庫，但它可以幫你把你跟 Model 說的「人話」轉成資料庫看得懂的資料庫查詢語言（SQL）。
6. 透過資料庫查詢語言，Model 從資料庫那邊取得你想要的資料。
7. Model 把這包資料交回 Controller/Action 手上。
8. 雖然 Controller/Action 拿到資料了，但目前這包東西還沒美化、整理過，還不適合給使用者看，所以 Controller/Action 需要跟 View 借一下畫面，讓資料更適合閱讀。
9. Controller/Action 把資料跟 View 的畫面組合，最後呈現給使用者看。

最後第 8、第 9 步，大概就是「I have a data, I have a template, um!... 秀出查詢結果」之類的概念吧。

## <a name="project-folders"></a>目錄結構

針對 Rails 的 Route + MVC 的組合，介紹一下這些角色在專案裡對應的目錄結構及慣例。

### Route

跟其它 MVC 相比，Route 相對的較為單純，全部的路徑設定都放在 `config/routes.rb` 這個檔案裡：

![image](/images/chapter10/folder-config.png)

`config` 目錄裡面除了 `routes.rb` 之外，基本上跟整個專案設定有關的幾乎都是放在這裡，例如 `database.yml` 就是專門用來設定資料庫連線資訊的地方。關於 Route 的使用會在下一篇做更詳細的介紹。

### Controller

Controller 就是放在專案的 `app/controllers` 目錄裡：

![image](/images/chapter10/folder-controller.png)

通常每個 Controller 會有自己獨立的檔案，而且檔案的名字跟類別的名字是對得起來的。規則很簡單，就是「大寫字元改成底線加小寫」。舉個例子來說，如果類別的名稱如果叫做 `PostsController` 的話，那這個檔案的名字就是會是 `posts_controller.rb`。

如果有興趣，你也可以進到 `rails console` 裡使用字串類別的 `underscore` 以及 `camelcase` 兩個方法來玩看看：

    $ rails console
    Running via Spring preloader in process 38922
    Loading development environment (Rails 5.0.1)
    >> "PostsController".underscore
    => "posts_controller"

    >> "UsersController".underscore
    => "users_controller"

    >> "posts_controller".camelcase
    => "PostsController"

另外，如果你打開每個 Controller 的內容，會發現預設都是繼承自 `ApplicationController` 這個類別。根據前面的規則，這個檔案的檔名自然就是 `application_controller.rb` 了

### Model

Model 相關的檔案都是放在 `app/models` 目錄裡：

![image](/images/chapter10/folder-model.png)

它的類別與檔名規則跟 Controller 是一樣的，例如 `Post` Model，它的檔名是 `post.rb`；如果是 `UserStory` 的話，則是 `user_story.rb`，以此類推。

另外，如果資料庫中有 Model 相對應的資料表（Table）的話，資料表的命名慣例是「小寫 + 複數」。簡單整理如下：

| Model 類別名稱 |  檔案名稱          | 資料表名稱    |
|----------------|--------------------|---------------|
| User           |  user.rb           | users         |
| Post           |  post.rb           | posts         |
| ProductItem    |  product_item.rb   | product_items |

當然資料表的命名慣例是可以修改的，但沒必要的話通常不會特別去改它，儘量維持「慣例優於設定」（CoC, Convention Over Configuration）的原則。

### View

View 主要是負責畫面輸出，通常是一群 HTML 之類的檔案，就放在 `app/views/` 目錄底下，而且預設會隨著 Controller 的名字而集中在某個資料夾：

![image](/images/chapter10/folder-view.png)

像是 `PostsController` 相關的 View，就會放在 `app/views/posts` 目錄裡。如果執行的是 `PostsController` 的 `index` Action，沒特別聲明 render 方法的話，預設會來找 `app/views/posts/index.html.erb` 這個檔案。

而這個 `index.html.erb` 的附檔名本身也是有特別意義的：

1. `erb` 是 Embedded Ruby 的縮寫，表示這個檔案會由 Ruby 標準函式庫中的 [ERB](http://ruby-doc.org/stdlib/libdoc/erb/rdoc/ERB.html) 樣版引擎進行解讀。你可以在這個檔案裡寫一些 Ruby 語法，例如陣列的 `each` 或 `map` 方法以及像是產生超連結的 `link_to` 方法。
2. `html` 表示這個檔案在被 ERB 樣版引擎處理後會被輸出成 HTML。

大概了解 Route 跟 MVC 的運作方式，以及每個角色的所在目錄後，下個章節就可以準備來寫一些簡單的程式碼，熟悉一下 Rails 開發的手感。

