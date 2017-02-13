---

title: 購物車 Part 1
comments: true
permalink: /chapters/25-shopping-cart-part-1.html

---

# 購物車 Part 1

- [功能設計](#requirement)
- [測試環境設定](#rspec-env)
- [先寫測試，再寫程式](#tdd)

終於進入真正的實作階段了，接下來讓我們來實作一個很多電子商務網站都會有的購物車功能吧。

在這個實作範例中，我們將使用：

1. TDD（Test-Driven Development）方式進行開發。
2. 購物車的內容不會建立資料表。

關於第 2 點，有些人會習慣使用建立資料表來存購物車的資料，不過我們這裡選擇使用 Session 來存放資料。

> 本章節程式碼可於 GitHub 上取得 https://github.com/kaochenlong/shopping_mall

## <a name="requirement"></a>功能設計

![image](/images/chapter25/cart.png)

說明：

一台購物車（Cart,①）會有很多的購買項目（CartItem ②），每個購買項目都有一項商品（Product ③）以及數量（Quantity ④）

### 基本功能

1. 可以把商品丟到到購物車裡，然後購物車裡就有東西了。
2. 如果加了相同種類的商品到購物車裡，購買項目（CartItem）並不會增加，但商品的數量會改變。
3. 商品可以放到購物車裡，也可以再拿出來。
4. 每個 Cart Item 都可以計算它自己的金額（小計）。
5. 可以計算整台購物車的總消費金額。
6. 特別活動可能可搭配折扣（例如聖誕節的時候全面打 9 折，或是滿額滿千送百）。

### 進階功能

因為購物車將以 Session 方式儲存，所以：

1. 可以將購物車內容轉換成 Hash，存到 Session 裡
2. 也可以把 Session 的內容（Hash 格式），還原成購物車的內容。

## <a name="rspec-env"></a>測試環境設定

在 `Gemfile` 裡加上需要的 gem：

```ruby
source 'https://rubygems.org'

# ...[略]...

group :development, :test do
  gem 'byebug', platform: :mri
  gem 'rspec-rails'
  gem 'factory_girl_rails'
  gem 'faker'
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
```

存檔後別忘了執行 `bundle install`，確保套件有安裝成功。其中，本體是 `rspec-rails`，其它像 `factory_girl_rails` 以及 `faker` 則是輔助產生測試資料用的。

Gem 安裝完成之後，接著照 `rspec-rails` 的說明頁面，安裝 `rspec` 到 Rails 專案裡：

    $ rails g rspec:install
    Running via Spring preloader in process 85482
      create  .rspec
      create  spec
      create  spec/spec_helper.rb
      create  spec/rails_helper.rb

這個指令會建立一個 `spec` 目錄，並且建立 2 個 Helper 檔案。接著試一下執行 `rspec` 指令：

    $ rspec
    No examples found.

    Finished in 0.00036 seconds (files took 0.15527 seconds to load)
    0 examples, 0 failures

這時候因為都還沒有任何測試，所以這個結果是正常的。

另外，因為我們會用 Rspec 取代原本內建的測試，所以原本專案裡的 `test` 目錄也可移除。


## <a name="tdd"></a>先寫測試，再寫程式

### 先寫測試

我們在第 23 章是自己手動建立 `bank_account_spec.rb` 檔案，但如果有安裝 `rspec-rails` 的話，可以請產生器幫我們做這件事。首先，先產生一個針對 Cart 這個 Model 的測試：

    $ rails g rspec:model Cart
    Running via Spring preloader in process 86133
        create  spec/models/cart_spec.rb
        invoke  factory_girl
        create    spec/factories/carts.rb

接著執行測試看看，應該是會失敗（錯誤訊息是「沒有 Cart 這個常數」）：

    $ rspec
    /private/tmp/shopping_mall/spec/models/cart_spec.rb:3:in `<top (required)>': uninitialized constant Cart (NameError)
      from /Users/user/.rvm/gems/ruby-2.4.0/gems/rspec-core-3.5.4/lib/rspec/core/configuration.rb:1435:in `load'
      ...[略]...
      from /Users/user/.rvm/gems/ruby-2.4.0/bin/ruby_executable_hooks:15:in `eval'
      from /Users/user/.rvm/gems/ruby-2.4.0/bin/ruby_executable_hooks:15:in `<main>'

不意外，因為我們根本還沒寫。

### 建立 Cart Model

首先，大家看到 Model 這個字，就會以為跟資料庫有關，然後就準備用 `rails g model Cart...` 指令來產生這個檔案了。事實上並不需要的，因為我們這個購物車只需要是個一般的純 Ruby 類別就行了，並不需要資料庫功能。所以請自己手動新增一個 `cart.rb` 檔案到 `app/models` 目錄裡：

```ruby
# 檔案：app/models/cart.rb
class Cart
end
```

什麼內容還不用寫，也不需要繼承自 Rails 的類別，只要先定義一個 `Cart` 類別就好。接著執行 `rspec` 指令，跑一下測試：

    $ rspec
    *

    Pending: (Failures listed here are expected and do not affect your suite's status)

      1) Cart add some examples to (or delete) /private/tmp/shopping_mall/spec/models/cart_spec.rb
         # Not yet implemented
         # ./spec/models/cart_spec.rb:4


    Finished in 0.0008 seconds (files took 3.31 seconds to load)
    1 example, 0 failures, 1 pending

這樣剛才找不到 `Cart` 類別的問題就解決了。這個 `pending` 的訊息是因為在 `spec/models/cart_spec.rb` 檔案裡有一行 `pending`，把它刪掉再執行一次就不會有錯誤訊息了。

### 把規格轉成測試

回到 `spec/models/cart_spec.rb` 檔案，把我們要測試的內容補上去：

```ruby
require 'rails_helper'

RSpec.describe Cart, type: :model do
  describe "購物車基本功能" do
    it "可以把商品丟到到購物車裡，然後購物車裡就有東西了" do
    end

    it "如果加了相同種類的商品到購物車裡，購買項目（CartItem）並不會增加，但商品的數量會改變" do
    end

    it "商品可以放到購物車裡，也可以再拿出來" do
    end

    it "每個 Cart Item 都可以計算它自己的金額（小計）" do
    end

    it "可以計算整台購物車的總消費金額" do
    end

    it "特別活動可能可搭配折扣（例如聖誕節的時候全面打 9 折，或是滿額滿千送百）" do
    end
  end


  describe "購物車進階功能" do
    it "可以將購物車內容轉換成 Hash，存到 Session 裡" do
    end

    it "可以把 Session 的內容（Hash 格式），還原成購物車的內容" do
    end
  end
end
```

### 測試 Step 1

先從第一個測試開始寫吧：

```ruby
require 'rails_helper'

RSpec.describe Cart, type: :model do
  describe "購物車基本功能" do
    it "可以把商品丟到到購物車裡，然後購物車裡就有東西了" do
      cart = Cart.new                   # 新增一台購物車
      cart.add_item 1                   # 隨便丟一個東西到購物車裡
      expect(cart.empty?).to be false   # 它應該不是空的
    end

    #...[略]...
end
```

這時候執行測試，應該是會失敗的，因為 `add_item` 跟 `empty?` 方法都還沒有寫。

### 實作 Step 1

回到 `Cart` 類別加上這兩個方法，想辦法通過測試：

```ruby
class Cart
  def initialize
    @items = []
  end

  def add_item(product_id)
    @items << product_id
  end

  def empty?
    @items.empty?
  end
end
```

說明：

1. 在 `initialize` 的時候初始化一個空陣列 `@items`
2. 在 `add_item` 的時候，把傳進來的東西往 `@items` 陣列裡丟
3. `empty?` 方法則是回傳 `@items` 陣列是不是空的

沒打錯字的話，應該可以通過測試。

### 測試 Step 2

繼續加測試：

```ruby
require 'rails_helper'

RSpec.describe Cart, type: :model do
  describe "購物車基本功能" do
    # ...[略]

    it "如果加了相同種類的商品到購物車裡，購買項目（CartItem）並不會增加，但商品的數量會改變" do
      cart = Cart.new                             # 新增一台購物車
      3.times { cart.add_item(1) }                # 加了 3 次的 1
      5.times { cart.add_item(2) }                # 加了 5 次的 2
      2.times { cart.add_item(3) }                # 加了 2 次的 3

      expect(cart.items.length).to be 3           # 總共應該會有 3 個 item
      expect(cart.items.first.quantity).to be 3   # 第 1 個 item 的數量會是 3
      expect(cart.items.second.quantity).to be 5  # 第 2 個 item 的數量會是 5
    end

    # ...[略]...
  end
end
```

### 實作 Step 2

一樣想辦法來通過測試。我們在 step 1 寫的 `add_item` 不僅沒有任何檢查，只是一股腦兒的把收到的 `product_id` 的往 `@items` 陣列裡丟，而且光靠 `product_id` 也不足以記錄傳進來的數量，看起來會需要另外的類別來存放收到的商品以及數量。先修改原來 `Cart` 類別的內容：

```ruby
class Cart
  attr_reader :items

  def initialize
    @items = []
  end

  def add_item(product_id)
    found_item = items.find { |item| item.product_id == product_id }

    if found_item
      found_item.increment
    else
      @items << CartItem.new(product_id)
    end
  end

  def empty?
    items.empty?
  end
end
```

說明：

1. 加了一個 `items` 的 `attr_reader`，讓內、外部的存取更方便一些。
2. `add_item` 方法裡使用 `find` 方法來找看看是不是有 item 的 product_id 跟傳進來的 product_id 是一樣的。這個 `find` 方法不是一般 Model 的 find 方法，它只是一個一般的陣列方法，可以找到是否有符合條件的元素。
3. 如果找到的話，就叫那個 `found_item` 增加數量
4. 找不到的話，就是用 `CartItem` 類別包一個物件，然後丟往 `@items` 陣列。

接下來，跟新增 `Cart` 類別一樣，新增一個 `cart_item.rb` 在 `app/models` 底下，而且也同樣因為不需要用到資料庫相關的功能，所以也不需要繼承自 Rails 的類別：

```ruby
# 檔案：app/models/cart_item.rb

class CartItem
  attr_reader :product_id, :quantity

  def initialize(product_id, quantity = 1)
    @product_id = product_id
    @quantity = quantity
  end

  def increment(n = 1)
    @quantity += n
  end
end
```

說明：

1. 加了 2 個 `attr_reader`，分別是 `product_id` 以及 `quantity`，方便外部取用。
2. 在初始化的時候會接收 `product_id` 以及 `quantity` 兩個參數，但如果沒有傳 `quantity` 則是使用預設值 1
3. `increment` 方法會接收一次要新增的數量，預設值 1

這樣一來測試應該就可以通過了

### 測試 Step 3

繼續測試：

```ruby
require 'rails_helper'

RSpec.describe Cart, type: :model do
  describe "購物車基本功能" do
    # ...[略]...

    it "商品可以放到購物車裡，也可以再拿出來" do
      cart = Cart.new
      p1 = Product.create(title:"七龍珠")              # 建立商品 1
      p2 = Product.create(title:"冒險野郎")            # 建立商品 2

      4.times { cart.add_item(p1.id) }                 # 放了 4 次的商品 1
      2.times { cart.add_item(p2.id) }                 # 放了 2 次的商品 2

      expect(cart.items.first.product_id).to be p1.id  # 第 1 個 item 的商品 id 應該會等於商品 1 的 id
      expect(cart.items.second.product_id).to be p2.id # 第 2 個 item 的商品 id 應該會等於商品 2 的 id
      expect(cart.items.first.product).to be_a Product # 第 1 個 item 拿出來的東西應該是一種商品
    end
  end

  # ...[略]...
end
```

執行測試，沒意外應該是會出錯的...

### 實作 Step 3

因為我們要透過 `Product` Model 來建立資料，然後放到購物車裡，所以這時候就真的需要請產生器來幫我們建一個 `Product` Model 了：

    $ rails g model Product title description:text price:integer
    Running via Spring preloader in process 88733
      invoke  active_record
      create    db/migrate/20170110151200_create_products.rb
      create    app/models/product.rb
      invoke    rspec
      create      spec/models/product_spec.rb
      invoke      factory_girl
      create        spec/factories/products.rb

別忘了執行 `rails db:migrate` 喔！

接下來，回頭來幫 `CartItem` 加上 `product` 方法，讓它可以根據目前這條 item 的 `product_id` 查出產品是什麼：

```ruby
class CartItem
  attr_reader :product_id, :quantity

  def initialize(product_id, quantity = 1)
    @product_id = product_id
    @quantity = quantity
  end

  def increment(n = 1)
    @quantity += n
  end

  def product
    Product.find_by(id: product_id)
  end
end
```

如此一來，透過 `Cart` 類別的 `add_item` 方法，把 A 商品加到購物車裡，拿出來之後還是 A 商品，而不是放蘋果進去，結拿拿出來變香蕉。

### 小結

TDD 的手感：

1. 確認規格，把規格轉成測試
2. 執行測試，一定會失敗！（除非你有小精靈）
3. 想辦法讓測試通過
4. 回到第一步

寫測試的時候，先不用擔心寫得好不好看或優不優雅，先用最直覺的方式把你想測的內容寫出來，然後把實作內容做出來，想辦法通過測試。到這裡，我們大概完成了購物車一半的功能，下個章節將會繼續完成另外一半...

> 本章節程式碼可於 GitHub 上取得 https://github.com/kaochenlong/shopping_mall

