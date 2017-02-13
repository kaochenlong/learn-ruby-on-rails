---

title: 購物車 Part 2
comments: true
permalink: /chapters/26-shopping-cart-part-2.html

---

# 購物車 Part 2

- [先寫測試，再寫程式](#tdd)

接續前一個章節，繼續把後續的功能以 TDD 的方式完成。

> 本章節程式碼可於 GitHub 上取得 https://github.com/kaochenlong/shopping_mall

## <a name="tdd">先寫測試，再寫程式

### 測試 Step 4

在開始下一個測試之前，先看一下這個測試：

```ruby
require 'rails_helper'

RSpec.describe Cart, type: :model do
  describe "購物車基本功能" do
    # ...[略]...
    it "每個 Cart Item 都可以計算它自己的金額（小計）" do
    end

    # ...[略]...
  end
end
```

這個看起來是跟 `CartItem` 比較有關，雖然要全部寫在同一個 spec 檔裡面也不是不行，但隨著測試越來越多，建議還是另外開一 spec 來做這件事。一樣使用產生器來產生 spec 檔案：

    $ rails g rspec:model CartItem
    Running via Spring preloader in process 96855
      create  spec/models/cart_item_spec.rb
      invoke  factory_girl
      create    spec/factories/cart_items.rb

然後把 Cart Item 相關的測試移過去：

```ruby
require 'rails_helper'

RSpec.describe CartItem, type: :model do
  it "每個 Cart Item 都可以計算它自己的金額（小計）" do
    p1 = Product.create(title:"七龍珠", price: 80)      # 建立商品 1
    p2 = Product.create(title:"冒險野郎", price: 200)   # 建立商品 2

    cart = Cart.new
    3.times { cart.add_item(p1.id) }  # 加 3 次商品 1
    4.times { cart.add_item(p2.id) }  # 加 4 次商品 2
    2.times { cart.add_item(p1.id) }  # 再加 2 次商品 1

    expect(cart.items.first.price).to be 400   # 第 1 條 cart item 的價錢應該是 400 塊
    expect(cart.items.second.price).to be 800  # 第 2 條 cart item 應該是 800 塊
  end
end
```

在測試裡，期待 `CartItem` 本身可以計算自己這條 item 的價錢的能力。這時候跑測試，自然是一定會失敗的...

### 實作 Step 4

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

  def price
    product.price * quantity
  end
end
```

說明：

因為目前每個 item 本身都可以知道對應到的商品以及數量，所以只要一行：

```ruby
product.price * quantity
```

就可以算出這個 item 的價錢了。

### 測試 Step 5

即然每個 CartItem 都可以自己算錢，接下來要讓整台購物車也能算錢就會不太難做了。測試如下：

```ruby
require 'rails_helper'

RSpec.describe Cart, type: :model do
  describe "購物車基本功能" do
    #...[略]...

    it "可以計算整台購物車的總消費金額" do
      cart = Cart.new
      p1 = Product.create(title:"七龍珠", price: 80)      # 建立商品 1
      p2 = Product.create(title:"冒險野郎", price: 200)   # 建立商品 2

      3.times {
        cart.add_item(p1.id)
        cart.add_item(p2.id)
      }

      expect(cart.total_price).to be 840
    end
  end

  # ...[略]...
end
```

商品 1 跟商品 2 各買了 3 份，整台購物車的 `total_price` 應該是 840 元。

### 實作 Step 5

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

  def total_price
    items.reduce(0) { |sum, item| sum + item.price }
  end
end
```

在 `Cart` 類別加上了 `total_price` 方法，並且用 Ruby 內建的 `reduce` 方法來計算所有 item 的價錢，這樣測試應該就可以通過了。

### 測試 Step 6

基本功能做完了，接下來要做的是比較進階的功能。因為預計會使用 Session 在存購物車的資料，所以會需要把購物車物件轉換成 Hash 格式：

```ruby
require 'rails_helper'

RSpec.describe Cart, type: :model do
  # ...[略]...

  describe "購物車進階功能" do
    it "可以將購物車內容轉換成 Hash，存到 Session 裡" do
      cart = Cart.new
      3.times { cart.add_item(2) }   # 新增商品 id 2
      4.times { cart.add_item(5) }   # 新增商品 id 5

      expect(cart.serialize).to eq session_hash
    end

    it "可以把 Session 的內容（Hash 格式），還原成購物車的內容" do
    end
  end

  private
  def session_hash
    {
      "items" => [
        {"product_id" => 2, "quantity" => 3},
        {"product_id" => 5, "quantity" => 4}
      ]
    }
  end
end
```

在這個測試，我手動把商品 id 2 以及 5 透過 `add_item` 方法丟到購物車裡，然後期待購物車的 `serialize` 方法可以回傳一個格式正確的 Hash...

### 實作 Step 6

為了符合預期的規格，在這裡我定義了一個 `serialize` 的方法，想辦法讓它回傳期望的 Hash 格式：

```ruby
class Cart
  # ...[略]...

  def serialize
    all_items = items.map { |item|
      { "product_id" => item.product_id, "quantity" => item.quantity}
    }

    { "items" => all_items }
  end
end
```

要把物件收集成一個陣列雖然常會使用 `.each` 方法，但使用 `.map` 方法寫起來會更漂亮一點。如果沒打錯字的話，這個測試應該可以順利通過。

### 測試 Step 7

可以由購物車物件轉成 Hash，接下來是要可以反向的把 Hash 轉成購物車：

```ruby
require 'rails_helper'

RSpec.describe Cart, type: :model do
  describe "購物車基本功能" do
    # ...[略]...
  end

  describe "購物車進階功能" do
    it "可以將購物車內容轉換成 Hash，存到 Session 裡" do
      # ...[略]...
    end

    it "可以把 Session 的內容（Hash 格式），還原成購物車的內容" do
      cart = Cart.from_hash(session_hash)

      expect(cart.items.first.product_id).to be 2
      expect(cart.items.first.quantity).to be 3
      expect(cart.items.second.product_id).to be 5
      expect(cart.items.second.quantity).to be 4
    end
  end

  private
  def session_hash
    {
      "items" => [
        {"product_id" => 2, "quantity" => 3},
        {"product_id" => 5, "quantity" => 4}
      ]
    }
  end
end
```

期待購物車有個類別方法 `from_hash`，可以接收一個 Hash 轉回購物車物件，並且期待轉回來的商品跟數量都是正確的...

### 實作 Step 7

這個實作可能會比前面幾個要來得複雜一點：

```ruby
class Cart
  attr_reader :items

  def initialize(items = [])
    @items = items
  end

  def add_item(product_id)
    #...[略]...
  end

  def empty?
    #...[略]...
  end

  def total_price
    #...[略]...
  end

  def serialize
    #...[略]...
  end

  def self.from_hash(hash)
    if hash.nil?
      new []
    else
      new hash["items"].map { |item_hash|
        CartItem.new(item_hash["product_id"], item_hash["quantity"])
      }
    end
  end
end
```

說明：

1. 因為期望 `from_hash` 是類別方法，所以在定義的時候加上了 `self.`
2. 在 `self.from_hash` 方法中，不管傳進來的 Hash 是空的還是有資料，最終都還是呼叫 `new` 方法產生一個 `Cart` 實體，並且把傳入的 Hash 的內容轉換成 `CartItem` 物件。
3. 因此，在 `Cart` 類別的 `initialize` 方法需要稍做調整，讓它可以接收一個參數，並把參數直接指定給 `@items` 實體變數。

這樣測試應該就可以通過了！YES！

## 小結

請記得，TDD（Test-Driven Development）的重點在於「Development」而不在於「Test」，它是一種「測試先行」的「開發方法」。c 使用 TDD 方式進行開發一開始會有不小的阻力，畢竟跟平常的開發習慣不同，但逐漸習慣後會開始嘗到甜頭，並慢慢建立自信。甚到當你熟悉這個流程之後，沒先寫測試就開始實作反而會覺得晚上睡不著覺。

