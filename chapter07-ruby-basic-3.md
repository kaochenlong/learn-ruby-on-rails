---

title: 方法與程式碼區塊（block）
permalink: /chapters/07-ruby-basic-3

---

# 方法與程式碼區塊（block）

Rails 不是一種程式語言，它是一種用 Ruby 這個程式語言所開發出來的網頁開發框架（Web Framework）。

接下來幾個章節的目的並不是要詳細的介紹 Ruby 這個程式語言所有的功能，而是希望讓大家對 Ruby 有足夠的基本認識，之後大家在閱讀或撰寫 Rails 專案的時候，會比較知道 Rails 在寫些什麼。

- [方法（Method）](#method)
- [程式碼區塊（(Block）](#block)

## <a name="method"></a>方法（Method）

### 定義方法

在 Ruby 定義方法，使用的是 `def` 這個關鍵字：

```ruby
def say_hello_to(name)
  puts "hello, #{name}"
end
```

這樣就定義了一個 `say_hello_to` 方法，後面的 `name` 是這個方法的參數（parameter），不限定只能傳一個，如果要傳多個參數可使用逗號分開。方法的命名慣例跟一般的區域變數差不多，是使用小寫加底線的組合。

### 呼叫方法

要執行已經定義的方法，只要直接呼叫方法的名字即可：

```ruby
say_hello_to("帥哥")    # => hello, 帥哥
```

也可視情況省略小括號：

```ruby
say_hello_to "帥哥"     # => hello, 帥哥
```

在 Ruby 執行方法，經常省略小括號，目的是為了讓程式碼看起來更不像程式碼，反而像是一般的文章。

### 參數預設值

在定義方法時，可幫參數加上預設值：

```ruby
def say_something(message = "something")
  "message: #{message}"
end

p say_something "hi"     # => message: hi
p say_something          # => message: something
```

如果有正確傳參數給方法，那就會使用傳進去的參數；如果沒有，則使用預設值。

### 方法的回傳值

有時候你會希望方法在接收參數並在執行完成之後，回傳執行之後的結果，例如我們可以寫一個 BMI(Body Mass Index，身體質量指數)方法，它可以接收身高與體重，並回傳計算結果：

```ruby
# BMI值計算公式: BMI = 體重（單位：公斤）/ 身高平方（單位：公尺）

def bmi_calculator(height, weight)
  return weight / height ** 2
end

puts bmi_calculator(1.70, 80)   # => 27.681
```

上面這段範例中的 `return` 是指這個方法執行完成之後，把最後的計算結果回傳給呼叫它的方法，在這個範例裡也就是 `puts`，然後會被印出來在畫面上。

在 Ruby 方法裡，最後一行的執行結果會自動被回傳，所以上面這個例子的 `return` 也是可以省略的，像這樣：

```ruby
def bmi_calculator(height, weight)
  weight / height ** 2
end
```

### puts 不是 return，也沒有回傳值

對程式新手來說，有可能會寫出這樣的語法：

```ruby
def bmi_calculator(height, weight)
  puts weight / height ** 2
end
```

執行 `bmi_calculator` 方法，的確是會印出內容，但會印出內容是因為在方法裡面直接 `puts` 把內容印出來，並不是因為這個方法回傳所造成的。事實上，`puts` 方法本身是沒有回傳值的喔。

### 問號跟驚嘆號也是方法的一部份

在 Ruby 定義方法時，方法的名字一般除了使用英文、底線及數字的組合外，也可以使用問號 `?` 跟驚嘆號 `!`（其實等號 `=` 也可以），但僅能放在方法名字的最後面，像這樣：

```ruby
def is_adult?(age)
  age >= 18
end
```

在使用的時候跟一般的方法沒什麼差別，但別忘了要把問號加上去：

```ruby
if is_adult?(20)
  puts "你是成年人了!"
end
```

在通常會使用問號，慣例上是表示這個方法會回傳布林值（true 或 false），不管是 Ruby 或 Rails，都很常可以看到這樣的慣例：

```ruby
puts "".empty?                       # => true
puts [1, 2, 3, 4, 5].include?(3)     # => true
puts "Ruby".start_with?("Ru")        # => true
```

而使用驚嘆號，通常是表示使用這個方法可能會有「副作用」或「驚喜」，舉個例子來說，像是陣列有個叫做 `reverse` 的方法，它可以產生一個跟原來陣列的相反排序的新陣列：

```ruby
original_list = [1, 2, 3, 4, 5]
reversed_list = original_list.reverse

p reversed_list   # => [5, 4, 3, 2, 1]
p original_list   # => [1, 2, 3, 4, 5]
```

`reverse` 方法會回傳一個新的陣列回來，不會影響原來的資料。但如果是呼叫有驚嘆號版本的 `reverse!` 就不同了：

```ruby
original_list = [1, 2, 3, 4, 5]
reversed_list = original_list.reverse!

p reversed_list   # => [5, 4, 3, 2, 1]
p original_list   # => [5, 4, 3, 2, 1]
```

`reverse!` 方法除了會回傳一個陣列之外，原來的陣列也會直接跟著一起被影響了。所以如果你這個方法可能會有一些意外驚喜，在慣例上通常會加上一個驚嘆號，提醒一下使用這個方法的人。

### 問題：是變數還是方法？

因為 Ruby 在執行方法的時候可以適時的省略小括號，可以讓你的方法寫起來像是個區域變數一樣。不過想一下這個情況：

```ruby
age = 18

def age
  20
end

puts age  # => 會得到 18 還是 20？
```

這裡有個區域變數 `age` 指向數字 18，也有一個方法叫 `age` 會回傳數字 20，請問你認為是會印出 18 還是 20？

答案是 18，因為 Ruby 在同一個範圍內，如果遇到同名的區域變數及方法，會以區域變數優先。那如果想要得到 20 的話該怎麼辦？其實超簡單的，就是把最後一行的 `puts age` 改成 `puts age()` 就行了。大家寫 Ruby 省略小括號省到已經習慣了，都忘了其實呼叫方法的基本招使用小括號，這反而是在其它程式語言不太會有的困擾 :)

### 問題：參數有幾個？

在 Rails 裡常會看到 `link_to` 這樣寫：

```erb
<%= link_to '刪除', user, method: :delete, data: { confirm: 'sure?' }, class:'btn' %>
```

你看得出來上面這段範例中，`link_to` 方法共有幾個參數嗎？如果你是用逗號的數量數出來是 5 個，那你就需要繼續往下看了 :)

Ruby 很愛省略東西，像是方法的小括號，所以原來上面的 `link_to` 語法原本應該長這樣：

```erb
<%= link_to('刪除', user, method: :delete, data: { confirm: 'sure?' }, class:'btn') %>
```

除了常常省略小括號外，偶爾也會省略大括號。在 Ruby 中如果最後一個參數是 Hash 的話，它的大括號是可以省略的。舉個例子來說：

```ruby
def say_hello_to(name, options = {})
  # do something
end
```

如果要使用這個方法，可以這樣寫：

```ruby
say_hello_to "eddie", {age: 18, favorite: 'ruby'}
```

又，因為最後一個參數是 Hash，所以 Hash 的大括號也可省略：

```ruby
say_hello_to "eddie", age: 18, favorite: 'ruby'
```

如果你了解有什麼東西被省略的話，一開始的那段 link_to 的範例還原之後會變成：

```erb
<%= link_to('刪除', user, {method: :delete, data: { confirm: 'sure?' }, class:'btn'}) %>
```

所以，其實參數個數只有 3 個，最後一個參數是一個 Hash。也因為最後一個是 Hash，Hash 本身是沒有順序的，所以 Hash 裡的 `method` 要放後面或是 `class` 要放前面其實都可已。

Ruby 的語法可以適時的省略小括號、大括號以及 return，程式碼寫起來雖然會更像在寫文章，但對新手來說可能會容易混淆，需要花一點時間了解到底省略了哪些東西。

## <a name="block"></a>程式碼區塊（Block）

Block 在 Ruby 或 Rails 裡大量的被使用，像是在使用迴圈的時候，可能都寫過這樣的程式碼：

```ruby
5.times { puts "Hello, Ruby" }      # 這會印 5 次的 Hello Ruby

friends = ["魯夫", "孫悟空", "黑崎一護", "旋渦嗚人"]
friends.each do |friend|
  puts friend                       # 這會把陣列裡的元素一個一個印出來
end
```

其中，那個大括號 `{ ... }` 以及 `do ... end`，在 Ruby 稱之一個程式碼區塊（Block）

### Block 不是物件

我們常說，在 Ruby 裡，幾乎什麼東西都是物件，但其實還是有少數的例外，例如 Block 就不是物件。 Block 沒有辦法單獨的存在，也沒辦法把它指定給某個變數，像這樣的寫法都會造成語法錯誤（Syntax Error）：

```ruby
{ puts "Hello, Ruby" }            # 這樣會產生語法錯誤
action = { puts "Hello, Ruby" }   # 這樣也會產生語法錯誤
```

### Block 不是參數

Block 通常得像寄生蟲一樣依附或寄生在其它的方法或物件（或是使用某些類別把它物件化），但它不是參數，例如：

```ruby
def say_hello_to(name)
  # do something here
end

say_hello_to("悟空") {
  puts "這裡是 Block"
}

# 或是 do ... end 寫法
say_hello_to("悟空") do
  puts "這裡是 Block"
end
```

Block 不是參數，在上面這段範例中，`name` 才是參數，但 Block 不是。上面這段程式碼執行之後不會有任何錯誤，但 Block 裡要執行的動作也不會執行。

### 如何執行 Block 的內容？

想像一下這段的對話：

> 某 Block：「嘿，say_hello_to 方法，我要掛在你身上囉」

> say_hello_to ：「隨便啊，你要掛就讓你掛，但要不要讓你執行是我決定的！」

如果想要讓附掛的 Block 執行的話，可使用 `yield` 方法，暫時把控制權交棒給 Block，等 Block 執行結束後再把控制權交回來：

```ruby
def say_hello
  puts "開始"
  yield          # 把控制權暫時讓給 Block
  puts "結束"
end

say_hello {
  puts "這裡是 Block"
}
```

執行上面這段範例會得到：

    開始
    這裡是 Block
    結束

### 傳參數給 Block

有時候你會看到像這樣的寫法：

```ruby
5.times do |i|
  puts i
end
```

那個 `|i|` 是什麼呢？這個在兩根看起來像牆壁中間的 `i`，是在這個 Block 裡專屬的區域變數，Block 執行結束後就會失效了：

```ruby
5.times do |i|
  puts i          # 這個變數 i 只有在 Block 裡有效，會依序印出數字 0 到 4
end

puts i            # 離開 Block 之後就失效，出現找不到變數的錯誤（NameError）
```

所以，到底是這個 i 是怎麼來的？事實上，它就只是你在使用 yield 方法把控制權轉讓給 Block 的時候，順便把值帶給 Block 而已：

```ruby
def say_hello
  puts "開始"
  yield 123         # 把控制權暫時讓給 Block，並且傳數字 123 給 Block
  puts "結束"
end

say_hello { |x|     # 這個 x 是來自 yield 方法
  puts "這裡是 Block，我收到了 #{x}"
}
```

下回大家再看到 `|i|` 的寫法，應該就知道它是什麼意思了。

### Block 的回傳值

其實 `yield` 方法除了把控制權暫時的讓給後面的 Block 之外，Block 最後一行的執行結果也會自動變成 Block 的回傳值，所以可把 Block 當做判斷內容：

```ruby
def pick(list)
  result = []
  list.each do |i|
    result << i if yield(i)            # 如果 yield 的回傳值是 true 的話...
  end
  result
end

p pick([*1..10]) { |x| x % 2 == 0 }    # => [2, 4, 6, 8, 10]
p pick([*1..10]) { |x| x < 5 }         # => [1, 2, 3, 4]
```

上面這段範例的 pick 方法，會根據 Block 的條件，挑出符合條件的元素。

### 用 return 回傳 Block 的結果？

Block 的最後一行執行結果自動會變成 Block 的回傳值，這裡並不是省略了 return，而是不能使用 return 回傳結果。所以如果你在上面那個例子，在 Block 裡試圖用 `return` 回傳結果，像這樣：

```ruby
pick([*1..10]) { |x| return x % 2 == 0 }
```

這會產生 LocalJumpError 錯誤。因為，Block 並不是一個方法，所以它不知道你要 Return 到哪裡去而造成錯誤。

### 問題：`5.times { ... }` 很好用，但你能自己土砲一個類似的方法嗎？

```ruby
def my_times(n)
  i = 0
  while n > i
    i += 1
    yield i
  end
end

my_times(5) { |num|
  puts "hello, #{num}xRuby"
}

# 得到結果
# hello, 1xRuby
# hello, 2xRuby
# hello, 3xRuby
# hello, 4xRuby
# hello, 5xRuby
```

做法就是在執行 `while` 迴圈的同時，不斷的把數字透過 `yield` 傳出來，這樣就可以做出一個類似 `5.times { ... }` 的效果了。

### 大括號跟 do ... end 的差別

大部份的情況，Block 的大括號的寫法跟 `do ... end` 寫法是可以互換的，像這樣：

```ruby
# 使用 do .. end 寫法
5.times do
  puts "哈囉，世界"
end

# 使用大括號寫法
5.times {
  puts "哈囉，世界"
}
```

如果 Block 的內容如果有多行，通常會建議使用 `do .. end` 寫法，如果只有一行，則建議使用大括號寫法，可讓語法看起來精簡一些：

```ruby
# 使用大括號一行寫法
5.times { puts "哈囉，世界" }
```

但事實上，這兩種情況是有一些微妙的差別的，並不是所有情況都互相交換。看看這段程式碼範例：

```ruby
p [*1..10].map { |i| i * 2 }
# => 得到 [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]

p [*1..10].map do |i| i * 2 end
# => 得到 <Enumerator: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]:map>
```

會造成不同結果的原因，有點像是數學的「先乘除後加減」的規則，大括號的優先順序較高：

```ruby
p [*1..10].map { |i| i * 2 }

# 還原省略的小括號
p([*1..10].map { |i| i * 2 })
```

但 `do ... end` 的優先順序較低，會有不一樣的解讀：

```ruby
p [*1..10].map do |i| i * 2 end

# 還原省略的小括號
p([*1..10].map) do |i| i * 2 end
```

因為優先順序較低，所以變成先跟 p 結合了，造成後面附掛的 Block 就不會被處理了。

### 把 Block 物件化

前面提到，Block 本身並不是物件，它沒辦法單獨的存在 Ruby 的世界裡，需要依附在方法或物件後面。

但其實也是可以把 Block 物件化，例如使用 `Proc` 類別：

```ruby
greeting = Proc.new { puts "哈囉，世界" }   # 使用 Proc 類別可把 Block 物件化
```

要使用它的時候，只要執行這個物件上的 `call`：

```ruby
greeting.call  # 印出 "哈囉，世界"
```

如果要帶參數也可以：

```ruby
say_hello_to = Proc.new { |name| puts "你好，#{name}"}
say_hello_to.call("尼特羅會長")
```

### Proc 呼叫方式

要執行一個 Proc 物件，可以使用 `call` 方法，但其實還有其它好幾種使用方法，例如：

```ruby
say_hello_to.call("尼特羅會長")    # 使用 call 方法
say_hello_to.("尼特羅會長")        # 使用小括號（注意，有多一個小數點）
say_hello_to["尼特羅會長"]         # 使用中括號
say_hello_to === "尼特羅會長"      # 使用三個等號
say_hello_to.yield "尼特羅會長"    # 使用 yield 方法
```

這幾種方式都可以呼叫 Proc 物件。

如果是第一次接觸 Ruby 的朋友，使用 Block 一開始可能會有點不習慣。不過因為在 Ruby 或 Rails 的專案裡，Block 被用到的機會非常高，儘早熟悉 Block 的使用是很有幫助的喔，加油！

