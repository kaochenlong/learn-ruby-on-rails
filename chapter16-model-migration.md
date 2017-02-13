---

title: Model Migration
comments: true
permalink: /chapters/16-model-migration.html

---

# Model Migration

- [什麼是 Migration](#what-is-migration)
- [新增 Migration](#add-migration)
- [修改 Migration](#update-migration)
- [種子資料](#seed-data)

資料遷移（Migration）是很多剛接觸 Rails 的新手容易卡關的地方，對 Migration 常見的誤解有：

1. Migration 就是資料庫。
2. 只要在 Migration 修改欄位後，網頁上自動就會呈現修改後的效果。
3. 如果 Migration 有寫錯，只要修改之後再重新執行 `rails db:migrate` 指令就行了。

## <a name="what-is-migration"></a>什麼是 Migration

Migration 是用來描述「資料庫的架構長什麼樣子」的檔案，它會隨著專案開發的過程中逐漸增加。想像一下這個對話內容：

> 同事 A：「嘿，我剛剛建立了一個 User 資料表喔」
>
> 同事 B：「好，那我待會要建一個 Product 資料表用來放產品資訊的」
>
> 同事 C：「咦？等等，這個 User 資料表少一個地址欄位啦，我要加上去喔」
>
> 同事 A：「Product 資料表的 name 欄位不太好記，我要把它改名成 title 喔」
>
> 同事 C：「User 資料表的這個 flag 欄位好像都沒用到，我要把這個欄位刪除掉喔」

這段對話就是所謂的 Migration，上面這樣其實就是發生了 5 次的 Migration，每個 Migration 都是一個描述檔案。透過 Git 共同開發，每位同事應該都能拿到一樣的 Migration 描述檔，只要一個執令就可以同步資料庫的結構，比較不會有「同事 A 直接在 Server 上修改某個欄位的名字，但同事 B 不知情而造成程式無法順利執行」的情況發生。

使用 Migration 的另一個好處，是因為 Migration 檔案應該都會進 Git 版本控制，所以整個資料庫的設計過程全部都可以一目了然。而且假設原本使用 MySQL 資料庫，突然被公司長官要求要換成 PostgreSQL，只要沒有用到太特別或某些資料庫專屬的特異功能，通常只要一行指令就可以再重建資料庫。

## <a name="add-migration"></a>新增 Migration

要新增 Migration 有好幾種管道，例如透過 `rails generate` 指令產生 Model 或 Scaffold 都會順便產生一個 Migration 檔。讓我們先用 generate 產生一個 Model 吧：

    $ rails g model Article title content:text is_online:boolean
    Running via Spring preloader in process 1480
          invoke  active_record
          create    db/migrate/20161231224701_create_articles.rb
          create    app/models/article.rb
          invoke    test_unit
          create      test/models/article_test.rb
          create      test/fixtures/articles.yml

除了 Model 本體外，這個指令也產生了一個名為 `20161231224701_create_articles.rb` 的 Migration 檔案，其中檔名前面的 `20161231224701` 是這個指令執行時候的時間戳記。讓我們看一下這個 Migration 檔案的內容：

```ruby
class CreateArticles < ActiveRecord::Migration[5.0]
  def change
    create_table :articles do |t|
      t.string :title
      t.text :content
      t.boolean :is_online

      t.timestamps
    end
  end
end
```

Migration 檔的內容本質上就是一個 Ruby 程式，從語法大概能猜得出來要建立一個 `articles` 表格，並且有 title、content 以及 is_online 這幾個欄位。

除了這幾個欄位外，在最後一行還有一個 `t.timestamps` 的語法，這個會幫你在這個表格分別建立出 `created_at` 以及 `updated_at` 兩個時間戳記欄位，分別會在資料新增及更新的時候把當下的時間寫進去。如果覺得這個資料表不需要這樣的時間欄位的話，亦可直接把這行刪除。

有了 Migration，記得要執行 `rails db:migrate` 指令，這樣就會把這些描述轉換成真實的資料表：

    $ rails db:migrate
    == 20161231224701 CreateArticles: migrating ===================================
    -- create_table(:articles)
       -> 0.0050s
    == 20161231224701 CreateArticles: migrated (0.0051s) ==========================

### 如果你忘了執行 rails db:migrate 指令...

在 Rails 專案中如果有 Migration 檔案還沒有「處理」過，在你開瀏覽器檢視頁面的時候會看到「ActiveRecord::PendingMigrationError」的錯誤訊息：

![image](/images/chapter16/pending-migration-error.png)

不用太擔心，這時候只要執行一下 `rails db:migrate` 指令就可以解決問題了 :)

### 想要在其它環境下執行 Migrate

預設的 `rails db:migrate` 是會在 development 模式執行，如果你想在 production 或 test 環境執行的話，只要改一下環境變數就行，像這樣：

    $ RAILS_ENV=production rails db:migrate

這樣就會以 production 模式來執行 Migration 了。

## <a name="update-migration"></a>修改 Migration

剛執行完一個 Migration，才發現欄位名字打錯了，想要修改該怎麼做？直覺做法是「修改剛剛那個 Migration 檔案，存檔後再執行一次 `rails db:migrate` 吧」

但這方法行不通的，因為 `rails db:migrate` 這個指令只會針對還沒執行過的 Migration 檔案有效果，已經做過的 Migration ，再做一次是不會有反應的，所以即使修改同一個 Migration 檔再重新執行是沒用的。

那怎辦？其實做法有好幾款，其中一款，就是執行 Rollback 指令，把執行過的 Migration 倒回去：

    $ rails db:rollback
    == 20161231224701 CreateArticles: reverting ===================================
    -- drop_table(:articles)
       -> 0.0024s
    == 20161231224701 CreateArticles: reverted (0.0079s) ==========================

這樣就可以「倒轉」一個 Migration。如果一次想要倒轉 3 個 Migration，可以加上 `STEP=3` 參數：

    $ rails db:rollback STEP=3

雖然我們在 Migration 裡只有寫 `create_table` 語法，但上面這個指令會自動幫我們執行 `drop_table` 來刪除新增的資料表。

在執行 Rollback 的時候，如果正向 Migration 是建立資料表，那逆轉 Migration 就是刪除資料表；同理，如果正向是新增欄位，逆轉就會是刪除欄位。

> 注意：Rollback 是有風險的！
>
> 因為 Rollback 通常會造成刪除資料表或是刪除欄位的效果，所以如果原本該資料表或該欄位已經有資料的話，請儘量不要使用 Rollback 方式來修正 Migration，建議直接再新增一個 Migration 來進行修正。

### Rails 怎麼知道哪些 Migration 有做過？

其實在資料庫裡有一個名為 `schema_migrations` 的資料表，裡面有記錄哪些 Migration 已經做過的。除了可以直接進這個資料表看之外，也可使用這個指令查看：

    $ rails db:migrate:status
    database: /private/tmp/my_candidates/db/development.sqlite3

     Status   Migration ID    Migration Name
    --------------------------------------------------
       up     20161229084544  Create candidates
      down    20161231224701  Create articles
      down    20170101064253  Create comments

其中狀態是 `up` 的表示這個 Migration 已執行過，`down` 則是尚未執行。

### 手工產生 Migration

當想要修正 Migration 的時候，前面提到 Rollback 後修改再重做一次 Migration 的做法其實是不太推薦的，因為這樣做除了可能會刪除原有的資料之外，如果這個專案還有跟其它人協同開發，你也得要求其它同事 Rollback 重做一次，這實在會造成別人的困擾。

除非這個案子剛開始，或是只有你自己一個人在做，否則要進行資料庫結構修改的事，建議另外新增一個 Migration 來修正。例如我想要幫 `articles` 資料表新增一個名為 `photo` 的字串欄位：

    $ rails g migration add_photo_to_articles
    Running via Spring preloader in process 7437
      invoke  active_record
      create    db/migrate/20170101081107_add_photo_to_articles.rb

這邊的 `add_photo_to_articles` 並不一定要這樣寫，你要使用 `abc` 或 `xyz` 都沒問題，但建議使用一眼就看得出意圖的寫法跟單字，日後在維護的時候比較容易依檔名就知道到底這次 Migration 做了什麼事。讓我們看看剛剛產生的那個的 Migration 檔：

```ruby
class AddPhotoToArticles < ActiveRecord::Migration[5.0]
  def change
  end
end
```

其實產生器也只幫你產生了一個空殼而已，沒有真正的實作，所以接下來就要在裡面寫上我想要加的欄位：

```ruby
class AddPhotoToArticles < ActiveRecord::Migration[5.0]
  def change
    add_column :articles, :photo, :string
  end
end
```

`add_column` 這個方法，第一個參數是「資料表名稱」（注意：不是 Model 名稱喔），第二個參數是「要新增的欄位名稱」，第三個參數是這個欄位的「資料型態」。完成並存檔之後，則可繼續執行 Migration：

    $ rails db:migrate
    == 20170101081107 AddPhotoToArticles: migrating ===============================
    -- add_column(:articles, :photo, :string)
       -> 0.0004s
    == 20170101081107 AddPhotoToArticles: migrated (0.0004s) ======================

這樣一來，其它同事透過 Git 收到這個 Migration 檔的時候，同樣只要執行 `rails db:migrate` 指令，就可以跟你有一樣的資料表結構了。

### 魔術 Migration 產生器

Migration 的產生其實還滿神奇的，當你的檔名符合某些字樣的時候，例如 `add ... to ...` 或是 `remove ... from ...`，後面再加一些欄位，可以自動幫你產生一個寫好的 Migration 檔案，例如這樣：

    $ rails g migration add_candidate_id_to_articles candidate_id:integer:index
    Running via Spring preloader in process 7765
      invoke  active_record
      create    db/migrate/20170101081538_add_candidate_id_to_articles.rb

我要 `add` 一個欄位 `to` articles 這個表格，同時幫這個欄位加上索引。看一下它幫我們產生的 Migration 檔：

```ruby
class AddCandidateIdToArticles < ActiveRecord::Migration[5.0]
  def change
    add_column :articles, :candidate_id, :integer
    add_index :articles, :candidate_id
  end
end
```

突然就很魔術的寫好了！

雖然說這樣挺方便，但我沒辦法記得太多這樣的魔術寫法，我個人比較偏好開一個 Migration 再慢慢自己寫，反正也不會慢到哪裡去。如果你對這樣的寫法有興趣，請查閱 Rails Guide 的 [Migration 章節](http://guides.rubyonrails.org/active_record_migrations.html)

### schema.rb 是什麼東西？

在執行 `rails db:migrate` 指令之後，在專案的 `db` 目錄裡有個名為 `schema.rb` 的檔案，內容可能長得像這樣：

```ruby
ActiveRecord::Schema.define(version: 20170101081538) do
  # ...[略]...
  create_table "candidates", force: :cascade do |t|
    t.string   "name"
    t.string   "party"
    t.integer  "age"
    t.text     "politics"
    t.integer  "votes",      default: 0
    t.datetime "created_at",             null: false
    t.datetime "updated_at",             null: false
  end
  # ...[略]...
end
```

這個檔案是你在執行 `rails db:migrate` 指令的時候順便一起產生的，你不需要也沒必要手動修改這個檔案。從這個檔案可以看得出來每個資料表的名字與欄位名稱、型態。這個檔案通常會在版本控制系統裡，如果有些比較老舊的專長，中間有些 Migration 檔因為不明原因壞掉了而無法順利執行 `rails db:migrate`，這時候也可透過 `rails db:schema:load` 把資料表建回來。

另外，因為這個檔案的內容是由 `rails db:migrate` 指令產生，所以偶爾會遇到新手「Migration 寫好但還沒存檔就執行」的狀況，這時候從這個 `schema.rb` 檔案就可以看得出來。

### 「Migration 寫好但還沒存檔就執行」會怎樣？

其實不會怎樣，就只是執行了一個空的 Migration 而已。

但因為執行過的 Migration 檔不會再重複執行，所以有些對 Migration 還不熟的新手，以為已經正確的執行了 Migration 檔，但事實上根本就是執行了一個空的 Migration，這時候即使再存檔也沒效果了。

所以常會發生「奇怪，怎麼明明 Migration 檔案裡就有寫這些欄位，但為什麼 schema.rb 檔案裡卻沒有」的情況。

## <a name="seed-data"></a>種子資料

前面對 Migration 的介紹，好像都是在建立或修改資料庫的結構，事實上如果你想要的話，也是可以在 Migration 的過程順便寫資料進去的，像是這樣：

```ruby
class CreateArticles < ActiveRecord::Migration[5.0]
  def change
    create_table :articles do |t|
      t.string :title
      t.text :content
      t.boolean :is_online

      t.timestamps
    end
  end

  Article.create(title: "五倍紅寶石 part 1", content: "斷開鎖鍊吧!")
  Article.create(title: "五倍紅寶石 part 2", content: "斷開魂結吧!")
end
```

這樣的技巧常用在建立資料表的時候順便建立初始資料，例如預設的系統管理帳號。以上面這段範例來說，當執行 `rails db:migrate` 的同時也會順便一併新增兩筆資料到 `articles` 資料表。

雖然這樣可以寫入預設資料沒錯，但 Migration 的特性之一，就是已經處理過的 Migration 不會再執行（除非 Rollback 回去），所以如果要重新再重建這些預設資料會有點麻煩。在 Rails 裡有個更適合做這件事的地方，就是在 `db/seeds.rb` 這個檔案，請直接編輯這個檔案的內容：

```ruby
Article.create(title: "五倍紅寶石 part 1", content: "斷開鎖鍊吧!")
Article.create(title: "五倍紅寶石 part 2", content: "斷開魂結吧!")
```

存檔後，執行 `rails db:seed` 指令，就可以把資料寫進資料庫裡了，不管是目的性或是實用性來說，把預設資料放在這裡都是比較好的做法。

另外，`rails db:setup` 指令其實除了建立資料庫之外，也隱含了執行 `rails db:seed` 的指令，所以如果是全新的資料庫，執行 `rails db:setup` 可一口氣把資料表建完，順便把預設資料寫入。

