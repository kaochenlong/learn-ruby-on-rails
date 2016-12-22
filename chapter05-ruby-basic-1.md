# 變數、常數、流程控制、迴圈

- [變數 (variable) 與常數 (Constant)](#variable-and-constant)
- [流程控制 (Flow Controller)](#flow-control)
- [迴圈及迭代(Loop and Iteration)](#loop-and-iteration)

Rails 不是一種程式語言，它是一種用 Ruby 這個程式語言所開發出來的網頁開發框架 (Web Framework)。

接下來幾個章節的目的並不是要詳細的介紹 Ruby 這個程式語言所有的功能，而是希望讓大家對 Ruby 有足夠的基本認識，之後大家在閱讀或撰寫 Rails 專案的時候，會比較知道 Rails 在寫些什麼。

----

## <a name="variable-and-constant"></a>變數 (variable) 與常數 (Constant)

### 變數種類

在 Ruby 裡，依寫法及範圍的不同，變數大概有以下幾種：

| 種類                          | 範例     | 預設值  | 說明                    |
| ------------------------------|----------|---------|:------------------------|
| 區域變數 (local variable)     | name     | 沒有    | 非大寫字母開頭的名字    |
| 全域變數 (global variable)    | $name    | nil     | 前面加了 `$` 符號       |
| 實體變數 (instance variable)  | @name    | nil     | 前面加了 `@` 符號       |
| 類別變數 (class variable)     | @@name   | 沒有    | 前面加了 2 個 `@` 符號  |

### 虛擬變數 (Pseudo Variable)

除了上述這幾款變數外，Ruby 還有一種稱之虛擬變數的東西，這是由 Ruby 自己定義的，例如 nil、self、true、false，虛擬變數通常有特別的用途或意義，所以內容不能被改變。

```ruby
self = "123"   # => 發生 Can't change the value of self 錯誤
true = "xyz"   # => 發生 Can't assign to true 錯誤
```

### 變數預設值

沒有初始化的全域變數以及實體變數的預設值是 `nil`，但一般的區域變數就沒有預設值這回事：

```ruby
p @name  # => nil
p $name  # => nil
p name   # => 發生 undefined local variable or method 錯誤
```

### 有效範圍

這些變數的有效範圍(scope)都有些差別。以全域變數來說，就如同它的名字一樣，到處都可以用，但沒事不要亂用全域變數，可能會造成自己或其它人的困擾。區域變數的作用範圍相對的比較「區域」一點，例如：

```ruby
def say_hello
  name = "魯夫"
  puts "hi, 我是#{name}"
end

say_hello     # => 印出 "hi, 我是魯夫"
puts name     # => 發生變數找不到的錯誤
```

定義在 `say_hello` 方法裡的 `name` 變數，在離開 `say_hello` 方法就失效了。再看另一個例子：

```ruby
name = "魯夫"

def say_hello
  puts "hi, 我是#{name}"
end

puts name     # => 印出 "魯夫"
say_hello     # => 發生變數找不到的錯誤
```

雖然 `name` 變數在外面有定義，但在 `say_hello` 方法裡卻找不到。

至於實體變數跟類別變數會在後面物件導向程式設計章節會再有更詳細的說明。

### 使用變數

在 Ruby 使用變數，不需要特別宣告或是指定型態，直接抓來用就可以了。在變數命名規則上，常見會使用英文字母、數字或底線的組合。或是想要用非英文字也可以，例如：

```ruby
my_name = "eddie"
age = 18
姓名 = "見龍"
なまえ = "Yukihiro Matz"
```

### 使用常數

常數用起來其實跟變數差不多，也不需要特別宣告它是常數，但在 Ruby 對常數有特別的命名規定，就是「常數必須要是大寫英文字母開頭」，例如：

```ruby
ExchangeRate = 0.28
Name = "kitty"
```

事實上，所有的類別、模組的名字都必需是常數，在後面的物件導向程式設計會有更詳細的說明。要特別注意的是，在 Ruby 的常數的內容是可以修改而且不會發生錯誤：

```ruby
MyName = "eddie"       # => eddie
MyName << " kao"       # => eddie kao
MyName.prepend "Fr"    # => Freddie kao
```

如果是這樣：

```ruby
MyName = "eddie"
MyName = "hello, ruby"
```

把常數的內容整個換掉會出現警告訊息，但就僅是警告而已，不是錯誤訊息，程式仍可正常執行，這點跟其它程式語言有很大的不同。

### 只是個名字，沒有形態

不管是變數或常數，它本身並沒有形態。你可以把它想像成是「一張有寫著名字的標籤，貼在某個東西上面」，被貼的那個東西有形態，但標籤本身沒有，所以這樣寫是不會有任何問題的：

```ruby
name = "見龍"   # 原本是指向一個字串物件
name = 18       # 這樣會把 name 變數改指向一個數字物件
```

### 命名習慣

幫你的變數取個有意義的好名字是一門學問，像這樣寫：

```ruby
x = "ranny"
a = 1
```

雖然程式不會出錯，但用這樣不太具有意義的命名方式，程式碼的可讀性就變差了。有的程式語言會使用像是 `myShoppingCart` 之類的方式命名(又稱之駝峰式命名法)，在 Ruby 的世界，則是習慣使用小寫英文字母及底線來組合變數名稱，像是 `my_shopping_cart`。

這並沒有對錯，即使使用駝峰式的命名法，程式碼一樣可以正常運作，只是大部份的 Ruby 開發者都習慣這樣的風格。入境隨俗囉!

### 變數多重指定

當有多個變數要指定值的時候，例如這樣：

```ruby
x = 1
y = 2
z = 3
```

利用 Ruby 變數多重指定的特性，上面這三行可以改寫成：

```ruby
x, y, z = 1, 2, 3
```

如果等號的右邊是一個陣列也可以自動拆解出來：

```
x, y, z = [1, 2, 3]
```

#### 如果多重指定的時候數量不一樣多?

讓我們來看看當多重指定時，左右兩邊的數量不一樣多的時候會發生什麼事：

```ruby
# 右邊比較多
a, b = 1, 2, 3, 4
# => a = 1, b = 2, 其它的內容被丟掉了

a, *b = 1, 2, 3, 4
# => a = 1, 變數 b 前面的星號會讓 b 接收剩下的數值變成一個陣列 [2, 3, 4]

x, y, z = [1, 2, 3, 4, 5]
# => x = 1, y = 2, z = 3, 剩下的被忽略

x, y, *z = [1, 2, 3, 4, 5]
# => x = 1, y = 2, 同上，z 會接受其它的值變成陣列 [3, 4, 5]

# 左邊比較多
a, b, c = 1, 2
# => a = 1, b = 2, c 因為分不到值而變成 nil

x, y, z = [1, 2]
# => x = 1, y = 2, z 因為分不到值而變成 nil
```

### 變數再指定

```ruby
a = 1
a = a + 1
puts a       # => 印出數字 2
```

如果你是數學系畢業的，看到 `a = a + 1` 這樣的寫法大概會覺得很不可思議。在大部份的程式語言中，一個等號通常是「指定」(assign) 的意思，所以這行的意思是「把變數 a 的值加 1 之後，再指定回來給 a 這個變數」，變數 a 的值就會由原本的 1 變成 2。

而這裡的 `a = a + 1`，常也會把後面的加號往前搬，可簡化成 `a += 1`。

### 註解 (Comment)

程式碼裡的註解，在程式執行的時候會被忽略，註解通常有兩個主要的用途：

1. 說明這段程式碼的用途
2. 讓該行程式碼不執行

Ruby 使用井字符號 `#` 做為單行註解的符號，多行註解則是使用 `==begin` 及 `==end`：

```ruby
# 這行是註解
# 這行也是註解
# 這些都是註解

=begin
在這裡的內容
全部都是
註解
喔
=end
```

雖然用 `#` 或是 `=begin..=end` 都可以進行註解，但在實務上大多會選用 `#` 符號，畢竟比較容易輸入。

### 動手做做看

#### 問題：有兩個變數，x = 1, y = 2，請寫一小段程式來交換 x 跟 y 這兩個變數的值。

要交換兩個變數的值，通常會需要額外的暫存變數：

```ruby
x = 1
y = 2

tmp = x      # 先把 x 的值放在一個暫時的變數 tmp
x = y        # 把變數 y 的值指定給變數 x
y = tmp      # 最後再把暫時變數 tmp 的值指定給 y
```

這樣就可以把兩個變數的內容換過來了。但在 Ruby 還可以有更簡單的寫法：

```ruby
x = 1
y = 2

x, y = y, x
```

很神奇嗎! 利用 Ruby 可以多重指定變數的特性，一行就搞定了，程式碼簡單又容易閱讀。

----

## <a name="flow-control"></a>流程控制 (Flow Controller)

先看看這個有趣的小故事：

> 回家的時候，太太打電話來交待：
> 「老公，回家的路上，買 10 個包子回來，如果看到西瓜，買 2 個」
> 結果先生回到家，只買了 2 顆包子，因為他在路上看到西瓜了...

流程控制看起來是很簡單的事，大概就是「如果..不然就...」，但寫得不好或是語意不清楚，其實也是會出事的。

另外再看看這個有點極端的例子：

```ruby
a, b = 1, 2
if a > 0
  c = (a * b) / a
else
  c = b
end
```

這看起來好像有點複雜，但仔細想一下，你看得出來上面這段範例在寫什麼嗎? (劇透：不管是 if 或 else，結果都是 `c = b` 喔)

### 真真假假

在各式各樣的程式語言中，有的會把數字 0、數字 -1 或是空陣列當做 false。在 Ruby 裡因為所有的東西都是物件，在 Ruby 只有 `nil` 跟 `false` 會被當假的(false)，除此這兩個之外，其它都是真的(true)，包括數字 0、空陣列、空字串都是 true。

### nil 存在嗎?

雖然 `nil` 跟 `false` 在 Ruby 裡會被當成 false 看待，但並不表示他們不存在。在 Ruby 的世界裡，很多東西都是物件，事實上，nil 跟 false 也都是物件，nil 是 NilClass 類別的實體，false 則是 FalseClass 類別的實體：

```ruby
puts nil.class    # => NilClass
puts false.class  # => FalseClass
```

nil 跟 false 他們是真實存在的物件，只是他們在 Ruby 裡「被當做 false」而已。 特別是 nil，你有想過要怎麼用一個存在的東西來表示一個不存在的東西嗎? 其實這有點玄。nil 是一個實實在在存在的物件，Ruby 只是用它來表示「空的」、「不存在」的概念而已，所以在 Ruby 裡你還是可以對 nil 物件呼叫一些方法，例如：

```ruby
puts nil.nil?     # => true，因為它就是 nil 沒錯
puts nil.to_a     # => 空的 Array []
puts nil.to_h     # => 空的 Hash {}
```

### 如果...不然就...(if .. else ..)

如果.. 不然就... 是簡單的二分法，非黑即白、非藍即綠，寫起來大概像這樣：

```ruby
age = 20

if age >= 18
  puts "你是大人了"
else
  puts "快快長大!"
end
```

### 把 if 放到後面去

在英文的文法裡，有時候可以把 if 放到句子的最後面的倒裝句寫法。在 Ruby 一樣也可以這樣寫，例如：

```ruby
if age >= 18
  puts "你是大人了"
end
```

像這種在 if 區塊裡只有一行內容，就可以把 if 丟到後面去，像這樣：

```ruby
puts "你是大人了" if age >= 18
```

### unless

為了增加程式碼的可讀性，除了 if 之外，Ruby 還有提供 `unless` 語法可以用，unless 等於 `if not` 的效果，所以如果本來長得像這樣的：

```ruby
if not is_adult?(20)
  puts "你是大人了"
end
```

用 `unless` 改寫後就可以變成：

```ruby
unless is_adult?(20)
  puts "你是大人了"
end
```

### 再精簡一點的三元運算子

如果是簡單的 if ... else ...，在 Ruby 可使用三元運算子再讓程式更精簡一點。三元運算子是由 `?` 跟 `:` 組成，像這樣的寫法：

```ruby
gender = 1

if gender == 1
  title = "先生"
else
  title = "小姐"
end

puts title   # => 先生
```

用三元運算子的寫法，可改寫成：

```ruby
gender = 1
title = (gender == 1) ? "先生" : "小姐"
puts title   # => 先生
```

雖然這樣在整體的程式碼行數少好幾行，但對某些人來說這樣的程式碼可讀性似乎有降低一些些。請儘量以程式碼可讀性為優先考量，不要為了讓程式碼變少而這樣寫。

### 如果太多的「如果...不然就...(if .. else ..)」

當遇到太多的 if .. elsif ... 的時候，可考慮使用 `case ... when ...` 來改寫，例如原來是這樣：

```ruby
weather = "下雨"

case weather
when "下雨"
  puts "待在家!"
when "出太陽"
  puts "出去玩!"
else
  puts "在家睡覺!"
end
```

`case ... when ...` 還可以搭配使用範圍技(Range)：

```ruby
age = 10

if age > 0 && age <= 3
  puts "Baby"
elsif age > 3 && age <= 10
  puts "Kids"
elsif age > 10 && age <= 17
  puts "Teenager"
else
  puts "Adult"
end
```

可改寫成：

```ruby
age = 10

case age
when 0..3
  puts "Baby"
when 4..10
  puts "Kids"
when 11..17
  puts "Teenager"
else
  puts "Adult"
end
```

看起來就更清楚、簡潔了。

### 新手常犯錯誤：一個等號不是等於

看看下面這段程式碼範例：

```ruby
age = 20

if age = 18
  puts "yes"
else
  puts "no"
end
```

看起來滿單純的邏輯，因為 `age` 不等於 18，所以理論上應該會印出 "no" 字樣，但你執行程式會發現總是會印出 "yes"。

為什麼?

因為在 Ruby 裡(其實在其它程式語言也適用)，一個等號表示是指定(assign)，二個或三個等號才是比較(compare)。在上面這個例子，`age = 18` 是表示把數字 18 指定給 age 這個變數，而在 Ruby 裡，「指定」這件事的回傳值就是指定的內容本身，這個邏輯判斷式會得到 true，所以不管是 18 或 20，都是會得到 "yes" 字樣。

### 想想看...

#### 問題：要怎麼計算是不是潤年?

潤年的計算公式是「可以被 4 整除、不可 100 整除，但又可以被 400 整除」，根據這個公式可以這樣寫：

```ruby
year = 2000

if year % 4 == 0 && year % 100 != 0 || year % 400 == 0
  puts "潤年"
else
  puts "不是潤年"
end
```

----

## <a name="loop-and-iteration"></a>迴圈及迭代(Loop and Iteration)

在 Ruby 的迴圈主要有幾種：

1. while 迴圈
2. times, upto, downto 方法
3. for..in 迴圈
4. 迭代(iteration)

在 Rails 的開發過程中，比較常用到的是第 3 種跟第 4 種，特別是第 4 種。

### while 迴圈

```ruby
counter = 0

while counter < 5
  puts "hi, #{counter}"
  counter += 1
end

# 執行後得到結果：
# hi, 0
# hi, 1
# hi, 2
# hi, 3
# hi, 4
```

這段程式碼的意思是說「只要 `counter` 這個變數的值小於 5，就一直執行下去吧!」，所以要注意在迴圈裡的那行 `counter += 1` ，如果少了這行，這將會是一個無窮迴圈喔。

就像 if 跟 unless 一樣，while 也有另一個跟它剛好相反意思的好兄弟叫做 `until`，所以上面這行也可改寫成：

```ruby
counter = 0

until counter >= 5
  puts "hi, #{counter}"
  counter += 1
end
```

### times, upto, downto 方法

在 Ruby 的世界裡，幾乎所有的東西都是物件，包括數字也是。在數字物件裡有一個 `times` 方法，可以讓我們指定迴圈要跑幾次：

```ruby
5.times do
  puts "hello, ruby"
end

# 執行後得到結果：
# hello, ruby
# hello, ruby
# hello, ruby
# hello, ruby
# hello, ruby
```

語法一開始可能看不習慣，但看起來相當容易猜是什麼意思吧。後面那個 `do..end` 我們在後面講到 Block 的時候會再另外介紹。除了 `times` 之外，還有 `upto` 跟 `downto` 方法，可以正向或反向的執行迴圈：

```ruby
1.upto(5) do |i|
  puts "hi, ruby #{i}"
end

# 執行後得到結果：
# hi, ruby 1
# hi, ruby 2
# hi, ruby 3
# hi, ruby 4
# hi, ruby 5

5.downto(1) do |i|
  puts "hi, ruby #{i}"
end

# 執行後得到結果：
# hi, ruby 5
# hi, ruby 4
# hi, ruby 3
# hi, ruby 2
# hi, ruby 1
```

在 `do..end` 裡面那個 `|i|` 是 Block 裡的區域變數，同樣也會在 Block 單元再做介紹。

### for..in 迴圈

你手上有一個陣列(Array，會在下個單元介紹)，例如長得像這樣：

```ruby
friends = ["eddie", "joanne", "john", "mark"]
```

如果想要把裡面的元素一個一個印出來，可以這樣做：

```ruby
for friend in friends
  puts friend
end

# 執行後得到結果：
# eddie
# joanne
# john
# mark
```

在變數的命名慣例上，通常會讓陣列是複數型態(例如 `friends`)，而每個元素則是使用單數型態命名(例如 `friend`)。儘量不要有這樣的寫法：

```ruby
for x in friends
  puts x
end
```

雖然執行結果是一樣的，但在程式碼的可讀性上就稍微差了一些，光看變數名稱 `x`，不容易猜到這個變數代表的意義。另外，如果只是想單純的印出 1 ~ 5，不用真的先做出一個含有 1 ~ 5 的陣列，可以利用 Range 的寫法：

```ruby
for i in 1..5 do
  puts i
end

# 執行結果
# 1
# 2
# 3
# 4
# 5
```

### 迭代(iteration)

承上，除了用 `for..in` 的方式把陣列裡的東西印出來，也可使用 `each` 方法做這件事：

```ruby
friends = ["eddie", "joanne", "john", "mark"]

friends.each do |friend|
  puts friend
end
```

陣列的 `each` 方法非常常用到，讓我們來看一下之前用 Scaffold 產生的檔案(檔案：app/views/users/index.html.erb)：

```erb
<tbody>
  <% @users.each do |user| %>
    <tr>
      <td><%= user.name %></td>
      <td><%= user.email %></td>
      <td><%= user.tel %></td>
      <td><%= link_to 'Show', user %></td>
      <td><%= link_to 'Edit', edit_user_path(user) %></td>
      <td><%= link_to 'Destroy', user, method: :delete, data: { confirm: 'Are you sure?' } %></td>
    </tr>
  <% end %>
</tbody>
```

這裡就是使用了 `each` 方法，一個一個的把資料轉出來印在畫面上。

