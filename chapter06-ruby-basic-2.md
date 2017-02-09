---

title: 數字、字串、陣列、雜湊、符號
comments: true
permalink: /chapters/06-ruby-basic-2

---

# 數字、字串、陣列、雜湊、符號

Rails 不是一種程式語言，它是一種用 Ruby 這個程式語言所開發出來的網頁開發框架（Web Framework）。

接下來幾個章節的目的並不是要詳細的介紹 Ruby 這個程式語言所有的功能，而是希望讓大家對 Ruby 有足夠的基本認識，之後大家在閱讀或撰寫 Rails 專案的時候，會比較知道 Rails 在寫些什麼。

- [數字（Number）](#number)
- [字串（String）](#string)
- [陣列（Array）](#array)
- [雜湊（Hash）](#hash)
- [符號（Symbol）](#symbol)

## <a name="number"></a>數字（Number）

在 Ruby 的世界裡幾乎什麼東西都是物件，所以即使是看起來很單純的數字，事實上它也是一個數字物件，它身上有一些好用的方法，像是在上個章節提到的迴圈寫法：

```ruby
# 印出 5 次的 Ruby
5.times do
  puts "Ruby"
end

# 從 1 印到 10
1.upto(10) { |i|
  puts i
}

# 從 10 印到 1
10.downto(1) { |i|
  puts i
}
```

### 整數除法

在進行除法的時候，如果除數跟被除數都是整數的話，結果也會是整數，除不盡的小數會被無條件捨去：

```ruby
puts 10 / 3   # => 得到 3
```

我想這個結果可能不是你想要的。如果要解決這個問題的話，只要讓除數或被除數其中一個加上小數點就可以解決了：

```ruby
puts 10 / 3.0   # => 3.3333333333333335
puts 10.0 / 3   # => 3.3333333333333335
```

### 其實四則運算跟你想的不太一樣

在 Ruby 裡，很多東西都不是它看起來的樣子，其中一個就是四則運算

```ruby
puts 1 + 2     # => 3
```

這看起來簡單到不行的運算，但其實那個 `+` 號其實並不是普通的加號，它在 Ruby 是一個一般的方法（method），上面這行的原形應該是：

```ruby
puts 1.+(2)    # => 3
```

這個加號，事實上是「數字物件 1」呼叫了 `+` 這個方法，並且把「數字物件 2」當做參數傳進去。也因為它是一個方法，所以就有機會可以重新改寫它原來的功能，讓 1 + 1 不等於 2 都是有可能的，這個在後面物件導向程式設計會有更詳細的說明。

### 浮點數是不太準確的

浮點數就是帶有小數點的數字，讓我們來看一段簡單的運算式：

```ruby
puts 4.51212 == (3.51212 + 1)
```

我們光用肉眼就看得出來這應該是相等的，但執行之後會發現結果是 false，表示這兩個值不相等。這是因為浮點數本身就是沒辦法非常精準，如果真的需要精準的計算，可使用 Ruby 的標準函式庫 [BigDecimal](http://ruby-doc.org/stdlib/libdoc/bigdecimal/rdoc/BigDecimal.html) 做個轉換：

```ruby
require 'bigdecimal'
puts BigDecimal("4.51212") == BigDecimal("3.51212") + BigDecimal("1")    # => true
```

### 問題：要怎麼知道某個數字是不是奇數？

如果數字可以被 2 整除，表示它是偶數，反之則是奇數，所以程式可以這樣寫：

```ruby
num = 20

if num % 2 == 0
  puts "偶數"
else
  puts "奇數"
end
```

這個 `%` 是表示「取餘數」的意思，如果餘數為 0 表示為偶數，不然就是奇數。事實上，在數字物件上已經有有內建的判斷方法：

```ruby
puts 20.odd?    # => false
puts 20.even?   # => true
```

### 問題：要怎麼做「四捨五入到小數點第二位」？

使用數字物件的 `round` 方法，可以對數字進行四捨五入計算：

```ruby
puts 3.333.round      # => 3
puts 3.834.round      # => 4
```

如果想要四捨五入到小數第 2 位，就加個參數給它：

```ruby
puts 3.333.round(2)   # => 3.33
puts 4.518.round(2)   # => 4.52
```

----

## <a name="string"></a>字串（String）

字串大概是所有程式語言最常用的東西了。在 Ruby 的字串有兩款，一種使用單引號（Single Quote），另一種使用雙引號（Double Quote）：

```ruby
name = "鳴人"
name = '鳴人'
```

### 把字串當陣列玩

字串，其實可以看成是許多字元的組合，在 Ruby 的字串可以這樣玩：

```ruby
name = "This is a book"
title = "紅寶石鑑定商"

puts name[0]        # => T
puts title[1]       # => 寶

title[0..1] = "鑽"
puts title          # => 鑽石鑑定商
```

感覺就像是把字串當做陣列在處理。

### 字串安插（String Interpolation）

字串可以使用加號來進行組合、串接，像這樣：

```ruby
name = "eddie"
age = 18

puts "你好，我是 " + name
# => 印出「你好，我是 eddie」

puts "你好，我是 " + name + " 我今年 " + age + " 歲"
# => 字串跟數字無法直接串接，會出現 TypeErro

# 需要先把 age 轉換成字串（使用 to_s 方法）
puts "你好，我是 " + name + " 我今年 " + age.to_s + " 歲"
# => 印出「你好，我是 eddie 我今年 18 歲」
```

以上面這個例子來說，有時候需要先把數字轉換成字串才能串接，寫起來語法也較不好看。Ruby 的字串有提供了字串安插的寫法，寫起來更簡單也更清楚一些：

```ruby
name = "eddie"
age = 18

puts "你好，我是 #{name}，我今年 #{age} 歲"
# => 印出「你好，我是 eddie，我今年 18 歲」
```

要注意的是，單引號組成的字串沒有這個效果，它將會完整呈現 `#{ ... }` 裡的東西。

```ruby
name = "eddie"
age = 18

puts '你好，我是 #{name}，我今年 #{age} 歲'
# => 印出「你好，我是 #{name}，我今年 #{age} 歲」
```

### 引號裡的引號

有時候可能會在引號裡面再放別的引號，例如：

```ruby
puts "Hello, I'm 悟空"
```

這是在雙引號裡放了一個單引號，這樣沒什麼問題，但如果想要在雙引號裡放雙引號，或是單引號裡放單引號，就得做一些額外的工作。為了可以正常呈現字串，需要使用反斜線來跳脫（escape）這個引號，告訴 Ruby 說「這是一個普通的引號，不是用來包字串的那個引號喔」

```ruby
puts "我說\"雙引號需要使用反斜線來處理!\""
# => 印出「我說"雙引號需要使用反斜線來處理!"」
```

在 Ruby 有提供另一種字串的表現方式，分別是 `%Q` 跟 `%q`，各代表雙引號跟單引號，而且不太需要使用跳脫方式來處理多餘的引號：

```ruby
name = "紅寶石"

puts %Q(你好，#{name})                   # 跟雙引號一樣，可以使用字串安插
# => 印出「你好，紅寶石」

puts %Q(你好，紅寶石"'"'"'"'"''"'"'")    # 要放幾個引號都可以
# => 印出「你好，紅寶石"'"'"'"'"''"'"'"」

puts %q(你好，#{name})                   # 跟單引號一樣，不會處理字串安插
# => 印出「你好，#{name}」
```

### 問題：字串的長度怎麼算？

如果都是英文字母，字串的長度容易算的，就是直接使用 `size` 方法就可以算得出來這個字串有幾個字：

```ruby
puts "hello, ruby".size          # => 11
puts "你好，紅寶石".size         # => 6
puts "你好，紅寶石".bytesize     # => 18
```

### 問題：如何計算一段文字中共有幾個字（Word）？

這裡指的字，是指英文的 Word，例如 "hello ruby" 這樣算是 2 個字。如果想要計算一段文字中共有幾個字，可以這樣做：

```ruby
words = "Ut in aliquam mauris. Donec dolor quam, sagittis id efficitur vel, convallis vitae tortor"
puts words.split.count    # => 14
```

字串的 `split` 方法可把字串依照所輸入的參數拆成陣列（若未輸入將以空白字元做為拆解字元），然後再計算陣列的數量即可得知共總有幾個字。

### 問題：字串的大小寫轉換？

使用 `downcase` 方法可讓字母全部變小寫、使用 `upcase` 方法可讓方法全部變大寫，`swapcase` 則是讓大小寫互相轉換：

```ruby
puts "hello, ruby".upcase     # => HELLO, RUBY
puts "HELLO, RUBY".downcase   # => hello, ruby
puts "Hello, Ruby".swapcase   # => hELLO, rUBY
```

如果只是想首字大寫的話，可使用 `capitalize` 方法：

```ruby
puts "eddie".capitalize     # => Eddie
```

### 問題：檢查是否為空字串？

想要檢查是否為空字串，最直覺的方式可能是這樣：

```ruby
if name == ''
  # ...
else
  # ...
end
```

但 Ruby 有內建了一個 `empty?` 方法，可檢查該字串是否為空字串，程式碼看起來更口語：

```ruby
puts "hello".empty?    # => false
puts " ".empty?        # => 裡面有一個空白字元，所以得到 false
puts "".empty?         # => true
```

注意，空白字元（space）不算是空字串喔！

### 問題：想要知道某個字母在字串中共出現幾次？

這個題目雖然可以把字串的所有字母拆開再跑 for 迴圈或 each 方法來計算，但 Ruby 的字串有一個 `count` 方法，可簡單的算出字串中某些字母出現的次數：

```ruby
words = "Lorem Ipsum Dolor Sit Amet, Consectetur Adipiscing Elit."

puts words.count("i")    # => 算出有幾個 i，共有 5 個
puts words.count("A-Z")  # => 算出所有大寫字母，共有 8 個
puts words.count("a-z")  # => 算出所有小寫字母，共有 39 個
```

### 問題：想知道字串是不是特定字元開題或結尾？或是包含指定字串或字元？

使用 `start_with?` 及 `end_with?` 方法可檢查該字串是否某字元（或字串）開頭及結尾：

```ruby
puts "Hello, Ruby".start_with?("H")  # => true
puts "Hello, Ruby".start_with?("h")  # => false
puts "Hello, Ruby".end_with?("y")    # => true
```

使用 `include?` 方法則可檢查是否包含指定的字串或字元：

```ruby
puts "Hello, Ruby".include?("r")     # => false
puts "Hello, Ruby".include?("R")     # => true
puts "Hello, Ruby".include?("Ruby")  # => true
puts "Hello, Ruby".include?("Rudy")  # => false
```

在方法名稱後面的問號 `?` 也是方法名子的一部份，別忘了在使用的時候要一併加上去。

### 問題：想把字串裡的某些字換成其它字？

使用 `sub` 或 `gsub` 方法，可換掉一個或全部符合條件的字串：

```ruby
puts "PHP is good, and I love PHP".sub(/PHP/, "Ruby")
# sub 只會換掉最先遇到的那個字串
# => 印出「Ruby is good, and I love PHP」

puts "PHP is good, and I love PHP".gsub(/PHP/, "Ruby")
# gsub 會換掉全部符合的字串
# => 印出「Ruby is good, and I love Ruby」
```

這樣就可以把 PHP 換成 Ruby 了！

## <a name="array"></a>陣列（Array）

在 Ruby 要建立一個陣列有幾個方式：

1. 使用 `Array.new` 方法
2. 直接使用中括號 `[ ]`

```ruby
p Array.new
# 產生一個空的陣列
# => []

p Array.new(5)
# 產生一個擁有 5 個元素並放滿 nil 的陣列
# => [nil, nil, nil, nil, nil]

p Array.new(5, "Ruby")
# 產生一個擁有 5 個元素並放滿 "Ruby" 字串的陣列
# => ["Ruby", "Ruby", "Ruby", "Ruby", "Ruby"]
```

比起上面這樣的寫法，我們更常使用中括號 `[ ... ]` 來定義、操作陣列，而且元素不限定只能存放同一種類型，想要把數字跟字串混著放也沒問題。

```ruby
friends = ["魯夫", "孫悟空", "黑崎一護", "旋渦嗚人"]
secret_number = [4, 8, 15, 16, 23, 42]
```

除了放一般的元素之外，陣列裡面也可以再放陣列，像是這樣：

```ruby
list = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
```

陣列在寫的時候，如果為了方便檢視也不一定要全部寫在同一行，像這樣寫也是可以的：

```ruby
friends = [["魯夫", "娜美", "羅賓"],
           ["孫悟空", "克林", "天津飯"],
           ["黑崎一護", "朽木露琪亞", "阿散井戀次"],
           ["旋渦嗚人", "我愛羅", "宇智波佐助"]]
```

### 陣列取用

要取用陣列的內容，可使用陣列的索引值（Index）來取用，但要注意，Ruby 的陣列索引是從 0 開始算的，所以 0 表示是第 1 個元素，負的索引值則是表示從後面開始算：

```ruby
friends = ["魯夫", "孫悟空", "黑崎一護", "旋渦嗚人"]
puts friends[0]     # => 魯夫
puts friends[1]     # => 孫悟空
puts friends[-1]    # => 旋渦嗚人
```

### 問題：請把陣列 [1, 2, 3, 4, 5] 變成 [1, 3, 5, 7, 9]

如果是剛從別的程式語言轉過來的話，可能會這樣寫：

```ruby
list = [1, 2, 3, 4, 5]

result = []
list.each do |i|
  result << i * 2 - 1
end

p result    # [1, 3, 5, 7, 9]
```

就是先做一個空的陣列（result），然後在 each 迴圈裡面把元素計算過之後再丟回那個空陣列裡。雖然這樣執行結果是對的，但在 Ruby 可以有更好的寫法：

```ruby
list = [1, 2, 3, 4, 5]
p list.map { |i| i * 2 - 1 }    # [1, 3, 5, 7, 9]
```

簡單的說，就是這個 `map` 方法會「對 list 這個陣列裡的每一個元素做某件事之後再收集成一個新的陣列」，所以也不需要另外再建立一個空的陣列。

這一開始會有點不習慣，但用習慣之後就會覺得很好用了 :)

### 問題：請計算從 1 加到 100 的總和。

你可能會這樣寫：

```ruby
total = 0

(1..100).to_a.each do |i|
  total = total + i
end

puts total    # 得到 5050
```

其中，前面的 `(1..100)` 會做出一個 1 到 100 的範圍，`to_a` 則是把這個範圍轉換成陣列。 `(1..100).to_a` 還可以改用星號 `*` 方式展開，連 `to_a` 都可以省略，寫起來會再更短一點：

```ruby
total = 0

[*1..100].each do |i|
  total = total + i
end

puts total    # 得到 5050
```

這樣寫沒什麼問題，但在 Ruby 還可以有更精簡的寫法：

```ruby
puts [*1..100].reduce(0) { |total, i| total + i }
```

這個 `reduce` 方法會「把裡面每個元素不斷的"互動"（這裡是相加），然後再把最後結果收集起來」。其實 `reduce` 方法只在意傳進去的 `total` 跟 `i` 這兩個之間的「互動」，以這個範例來說，它只在意「相加」這件事，所以如果想要再更精簡的話還可以這樣寫：

```ruby
puts [*1..100].reduce(:+)
```

只是這可能程式碼可讀性上有稍微差一點點了。

參考資料：(https://ruby-doc.org/core/Enumerable.html#method-i-reduce)

### 問題：請印出 1 ~ 100 數字中所有的單數

你可能會這樣寫：

```ruby
result = []

[*1..100].each do |i|
  result << i if i % 2 == 1
end

p result    # [1, 3, 5, ... 95, 97, 99]
```

但 Ruby 有個 `select` 方法，可以挑選出「符合條件」的元素，並收集成陣列：

```ruby
p [*1..100].select { |i| i % 2 == 1 }
```

再配合數字物件檢查單、偶數的方法：

```ruby
p [*1..100].select { |i| i.odd? }
```

再精簡一點

```ruby
p [*1..100].select(&:odd?)
```

### 問題：如果想要從 1 到 100 中隨便取 5 個不重複的亂數該怎麼做？

這有幾種做法，一種是不斷的跑迴圈，一次隨機抽一個顆球，連續抽 5 次，如果抽到重複的再重新抽一次。但其實可換個方式想，如果題目改成「你有一副牌，共有 52 張，請你隨便發 5 張不重複的牌給我」，你會怎麼做？

大部份的人應該都是先洗牌（shuffle），然後再從最上面直接發 5 張出來。用程式寫的話大概會長這樣：

```ruby
puts [*1..52].shuffle.first 5
```

陣列可透過 `shuffle` 洗牌，然後直接用 `first` 方法取前 5 張。其實還可以再更短一點：

```ruby
puts [*1..52].sample 5
```

所以原題目想要取 1 到 100 不重複亂數的話，就是：

```ruby
puts [*1..100].sample 5
```

## <a name="hash"></a>雜湊（Hash）

Hash 有兩種寫法，一種有箭頭式的寫法：

```ruby
old_hash = {:title => "Ruby", :price => 350}
```

在 Ruby 1.9 版之後，有加入了類似 JSON 式的寫法：

```ruby
new_hash = {title: "Ruby", price: 350 }
```

不管是新式或是舊式的，其實本質上都是一樣的。

```ruby
old_hash = {:title => "Ruby", :price => 350}
new_hash = {title: "Ruby", price: 350 }

p old_hash == new_hash        # => true
```

這裡的 `title` 跟 `price` 稱之 hash 的 key，而 `"Ruby"` 及 `350` 則稱之 value。

### 要拿對的鑰匙才能拿到對的資料

假設我有個 hash 長這樣：

```ruby
profile = {name: "5xRuby", age: 18, tel: "02-28825252" }
```

如果想要取得這個 hash 的 `name` 資料，從別的程式語言剛轉過來的人，可能會這樣寫：

```ruby
puts profile["name"]
```

你會發現什麼都沒有（其實是取到 nil），要改成這樣寫才會拿得到：

```ruby
puts profile[:name]    # => 5xRuby
```

因為，在這個 hash 中，它的 key 是 `:name` 而不是字串 `"name"`，所以會拿不到你想要的東西，這點要特別注意。`:name` 是一個符號（Symbol），至於什麼是符號，在下一個章節會說明。

### 問題：要怎麼把 hash 裡的東西一個一個印出來？

假設我的 hash 是這個樣子：

```ruby
profile = {name: "5xRuby", age: 18, tel: "02-28825252" }
```

如果想要取得所有的 key 或是所有的 value，可以直接使用 `keys` 或 `values` 方法取得：

```ruby
p profile.keys     # => [:name, :age, :tel]
p profile.values   # => ["5xRuby", 18, "02-28825252"]
```

如果是想要一個一個東西印出來的話，可以這樣寫：

```ruby
profile.each do |element|
  p element
end

# 執行後得到
# [:name, "5xRuby"]
# [:age, 18]
# [:tel, "02-28825252"]
```

但如果後面的 block 多加一個變數的話：

```ruby
profile.each do |key, value|
  puts key
  puts value
end
```

這樣就可各別取得 key 跟 value 了。

## <a name="symbol"></a>符號（Symbol）

在 Rails 專案裡，你可能看過這樣的程式碼：

```ruby
class User < ActiveRecord::Base
  has_many :products
  validates :name, presence: true
end

class Product < ActiveRecord::Base
  belongs_to :user
end
```

或是這樣：

```ruby
class ProductController < ApplicationControl
  before_action :find_product

  # .. 中略

  private
  def find_product
    @product = Product.find_by(id: params[:id])
  end
end
```

這在 Rails 專案裡是很常見的寫法，但這裡 `:products`、`:user`、`:name` 以及 `:find_product` 是什麼意思呢？

這可能是在大部份的人在學習 Ruby / Rails 的時候覺得困擾的問題。這東西在 Ruby 裡稱之 Symbol，中文翻譯做符號，它的寫法是在前面加上個冒號。常見的 Symbol 的命名規則跟一般的變數差不多，是以用英文字母或數字的組合，例如 `:name` 或 `:title`，非英文也可以，像是 `:姓名`、`:おはよう`也行，甚至要在中間加上空白也沒問題。但如果要使用空白字元的話，需要用引號包起來，例如 `:"hello world"`。不過以使用頻率來說，大多還是以英文字母的組合為主。

### Symbol 是什麼

Symbol 其實是有點玄的東西，有些人認為它就是的變數，或就是只是個名字，但事實上它不就是變數或名字這麼簡單，你可以想像它是一個「帶有名字的物件」：

Symbol 就是一個 `Symbol` [類別](http://ruby-doc.org/core/Symbol.html)的實體，它可用來表示某個狀態，例如這樣：

```ruby
class Order
  attr_reader :status

  def initialize(items, status = :pending)
    @items = items
    @status = status
  end

  def compete
    @status = :complete
  end
end

order = Order.new(["item A", "item B", "item C"])

if order.status == :pending
  puts "order is pending"
end
```

你也許會好奇這裡的 `:pending` 跟 `:complete` 是什麼？其實它就是代表 pending 跟 complete 這兩個狀態，前面提到 Symbol 是一種「帶有名字的物件」，正如其名，Symbol 就是符號，這個符號表示「已完成」或「未完成」。

那.. 上面這個例子，把 Symbol 改用字串可以嗎？當然是可以的。

### Symbol 跟變數有什麼不同？

變數是一個指向某個物件的名字，例如：

```ruby
greeting = "Hello Ruby"
```

上面這行語法，是指 `greeting` 這個名字指向 `"Hello Ruby"` 這個字串物件，但如果沒有 `"Hello Ruby"` 這個字串給它指，這個名字本身是沒辦法單獨存在的。

而 Symbol 是一個「帶有名字的物件」，本身不需要指向任何東西也可以拿來用，例如上面的 `:pending` 跟 `:complete` 的例子。而且你也沒辦法直接拿 Symbol 來當變數，像這樣會出現語法錯誤：

```ruby
:name = "見龍"   # 這得到 SyntaxError 的錯誤訊息
```

### Symbol 跟字串有什麼不同？

上課時候最常被問到的問題之一，就是「Symbol 跟字串有什麼不一樣？」

#### 字串的內容可以變，但 Symbol 不行

簡單的說，Symbol 跟字串有點像，Symbol 也有一些跟字串長得很像的方法，例如 `length`、`upcase`、`downcase` 等等。不過 Symbol 本身是不能被修改的，但字串可以。一樣開 `irb` 來試試：

```ruby
# 像字串一樣的操作
>> :hello.length
=> 5
>> :hello.upcase
=> :HELLO

# 假設有個 hello 字串，可以使用中括號 + 索引來取得其中某個字元
>> "hello"[0]
=> "h"
>> "hello"[3]
=> "l"

# Symbol 也行
>> :hello[0]
=> "h"
>> :hello[3]
=> "l"

# 這回來試著修改某個字母，例如想把 h 換成 k
>> "hello"[0] = "k"
=> "k"

# 但在 Symbol 上卻行不通，會發生錯誤（因為 Symbol 類別並沒有 []= 這個方法）
>> :hello[0] = "k"
NoMethodError: undefined method `[]=' for :hello:Symbol'`
```

所以，其實你也可以把 Symbol 看做是一種「不可變（immutable）的字串」。

#### 字串的效能稍微差一點點點點

在 Ruby 裡，每次要產生一個新的字串的時候，它會都向去要一塊新的記憶體，來看看這個例子：

```ruby
5.times do
  puts "hello".object_id
end

# => 70199659402580
# => 70199659366640
# => 70199659366560
# => 70199659366500
# => 70199659366420
```

`object_id` 方法會取得該物件在 Ruby 世界裡的唯一的數字編號，在不同的電腦或不同的 Ruby 版本所得到的結果可能會不太一樣。

相同的物件會有相同的 object id，不同的物件則會有不同的 object id。從上面這個例子可以發現，即使一樣都是 `"hello"` 字串，Ruby 每次在產生字串的時候都會產生一個新的 object id，表示它們是在記憶體裡其實是不同的 5 個物件。

但 Symbol 就不同了：

```ruby
5.times do
  puts :hello.object_id
end

# => 899228
# => 899228
# => 899228
# => 899228
# => 899228
```

只要是一樣的 Symbol，就會有相同的 object id，表示他們是同一顆東西，當 Ruby 第二次要取用同一個 Symbol 的時候，它會直接從記憶體裡拿，而不用重新產生一份，所以 Symbol 相對的較節省記憶體。

#### Symbol 的比較（Comparison）比字串快

直接寫一段 benchmark，讓它跑個 1 億次看看結果：

```ruby
require 'benchmark'
loop_times = 100000000

str = Benchmark.measure do
  loop_times.times do
    "hello" == "hello"
  end
end.total

sym = Benchmark.measure do
  loop_times.times do
    :hello == :hello
  end
end.total

puts "Benchmark"
puts "String: #{str}"
puts "Symbol: #{sym}"

# => Benchmark
# => String: 12.299999999999999
# => Symbol: 5.750000000000002
```

Symbol 的處理速度明顯比字串快得多，那是因為 Symbol 在做比較的時候，是直接比對這兩顆物件的 object id 是不是相同，而字串比較則是一個字母一個字母逐一比對。

所以在效能上來說，字串在做比較的時間複雜度會隨著字母的數量（N）而增加，但 Symbol 的比較因為只比較是不是同一顆物件，所以它的複雜度是固定的。

#### 字串跟 Symbol 是可以互相轉換的

字串及 Symbol 類別都有提供一些方法可以互相轉換，例如：

```ruby
# 使用 to_sym 可把字串轉成 symbol
>> "name".to_sym
=> :name

# intern 方法比較不常看到其它人用（因為這個方法看起來不怎麼直覺），但其實跟 to_sym 是一樣的效果
>> "name".intern
=> :name

# 另外也可用 %s 來做轉換
>> %s(name)
=> :name

# 用 to_s 方法可以把 symbol 轉成字串
>> :name.to_s
=> "name"

# 還有個不常用但功能跟 to_s 一樣的方法
>> :name.id2name
=> "name"
```

### 使用時機

講這麼多，那到底什麼時候該用 Symbol，什麼時候該用字串？

#### Hash 裡的 Key

```ruby
>> profile = { name: "見龍", age: 18 }
=> {:name=>"見龍", :age=>18}
```

這裡的 `:name` 跟 `:age` 就是 Symbol。因為 Symbol 不可變（immutable）的特性，以及它的查找、比較的速度比字串還快，它很適合用來當 Hash 的 Key。

#### 字串的方法比較多、比較好用

雖然也是可以把 Symbol 當字串用，但畢竟 Symbol 類別內建的方法不像字串類別那麼豐富，所以如果你想要用字串那些好用的功能，就選擇用字串而不要選擇 Symbol。

又，如果你只是想在畫面上把內容印出來，那就選用字串，是因為 Symbol 被輸出的時候，還是會先轉型成字串，所以效能就又差了那麼一點點。

#### 要怎麼知道那些方法的參數是要使用字串還是使用 Symbol？

先看個例子：

```ruby
class Cat
  attr_accessor :name
end

kitty = Cat.new
kitty.name = "Nancy"
puts kitty.name         # => Nancy
```

如果把 `attr_accessor :name` 換成 `attr_accessor "name"` 一樣可以正常運轉。

有的方法的參數是用字串，有的是用 Symbol，有的是兩種都能用，那該怎麼知道該用哪一種？

答案很簡單，就是，看．手．冊！遇到不知道怎麼用的，去查它的 API 手冊最直接了。

* [Ruby API](http://ruby-doc.org/core/)
* [Rails API](http://api.rubyonrails.org/)

