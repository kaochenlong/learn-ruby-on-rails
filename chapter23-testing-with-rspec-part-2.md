---

title: 寫測試讓你更有信心 Part 2
permalink: /chapters/23-testing-with-rspec-part-2

---

# 寫測試讓你更有信心 Part 2

- [哥寫的不是測試，是規格](#spec)
- [把規格轉成測試](#turn-specs-into-test-code)
- [紅綠燈](#red-green-light)
- [小結](#note)

前面介紹了什麼是測試，接下來就讓我們捲起袖子，動手寫測試吧！

## <a name="spec"></a>哥寫的不是測試，是規格

> 客人：「我想要做一個銀行帳戶系統，很簡單的，只要可以存錢、領錢以及顯示餘額就行了」

雖然客人開的需求有點「簡單」，但這個時候先不要電腦打開就開始寫 code，先來把規格寫出來吧：

- 存錢功能
    - 原本帳戶有 10 元，存入 5 元之後，帳戶餘額變 15 元
    - 原本帳戶有 10 元，存入 -5 元之後，帳戶餘額還是 10 元（不能存入小於等於零的金額）
- 領錢功能
    - 原本帳戶有 10 元，領出 5 元之後，帳戶餘額變 5 元
    - 原本帳戶有 10 元，試圖領出 20 元，帳戶餘額還是 10 元，但無法領出（餘額不足）
    - 原本帳戶有 10 元，領出 -5 元之後，帳戶餘額還是 10 元（不能領出小於或等於零的金額）

以上這些就是「規格」。再次強調一次，我們其實不是在「寫測試」，而是在「寫規格」，然後藉由一步一步滿足這些規格的過程來完成系統功能。

### 安裝 RSpec

在 Ruby/Rails 的世界有好幾套測試用的框架，目前比較受歡迎的主要有 `minitest` 跟 `RSpec`，這邊我們將使用 RSpec 來做介紹。安裝 RSpec 只要一行：

    $ gem install rspec
    Successfully installed rspec-3.5.0
    Parsing documentation for rspec-3.5.0
    Installing ri documentation for rspec-3.5.0
    Done installing documentation for rspec after 0 seconds
    1 gem installed

這樣就行了。

## <a name="turn-specs-into-test-code"></a>把規格轉成測試

接下來，我們要把上面提到的規格轉成程式碼。

因為這不是一個 Rails 專案，所以可以找一個你喜歡的地方，隨便開一個資料夾，並在裡面新增一個名為 `bank_account_spec.rb` 的檔案，然後試著把上面的「規格」轉換成「測試」：

```ruby
# 檔案：bank_account_spec.rb

RSpec.describe BankAccount do
  describe "存錢功能" do
    it "原本帳戶有 10 元，存入 5 元之後，帳戶餘額變 15 元" do
    end

    it "原本帳戶有 10 元，存入 -5 元之後，帳戶餘額還是 10 元（不能存入小於等於零的金額）" do
    end
  end

  describe "領錢功能" do
    it "原本帳戶有 10 元，領出 5 元之後，帳戶餘額變 5 元" do
    end

    it "原本帳戶有 10 元，試圖領出 20 元，帳戶餘額還是 10 元，但無法領出（餘額不足）" do
    end

    it "原本帳戶有 10 元，領出 -5 元之後，帳戶餘額還是 10 元（不能領出小於或等於零的金額）" do
    end
  end
end
```

說明：

1. 檔名不一定要叫這個名字，只是習慣上會在要測試的對象的後面加上 `_spec` 或是 `_test` 以表示是測試用的檔案。
2. 這只是先把要測試的方向列出來而已，還沒開始寫。
3. 可以用中文沒關係，重點是清楚就好。

## <a name="red-green-light"></a>紅綠燈

TDD 的流程，大概是一個「紅綠燈」的概念:

1. 先寫規格（測試），執行它，這時候一定會發生錯誤（紅燈）。
2. 實作功能，想辦法通過第 1 步發生錯誤的測試（綠燈）。
3. 回到第 1 步。

### 測試失敗！（紅燈）

接下來使用 `rspec` 程式來執行一下剛剛這個「規格」：

    $ rspec bank_account_spec.rb
    /private/tmp/bank/bank_spec.rb:1:in `<top (required)>': uninitialized constant BankAccount (NameError)
      from /Users/user/.rvm/gems/ruby-2.4.0/gems/rspec-core-3.5.4/lib/rspec/core/configuration.rb:1435:in `load'
      from /Users/user/.rvm/gems/ruby-2.4.0/gems/rspec-core-3.5.4/lib/rspec/core/configuration.rb:1435:in `block in load_spec_files'
      ...[略]...
      from /Users/user/.rvm/gems/ruby-2.4.0/bin/ruby_executable_hooks:15:in `eval'
      from /Users/user/.rvm/gems/ruby-2.4.0/bin/ruby_executable_hooks:15:in `<main>'

咦？發生錯誤了！如果你是第一次接觸 TDD 開發流程的話，可能會驚訝「哇！好多錯誤訊息啊！這怎麼解決？」

請不要擔心，也請大家要開始習慣，在寫測試的時候，這樣的錯誤是很常見的，甚至應該說這是正常的。仔細看一下錯誤訊息，錯誤訊息是 `uninitialized constant BankAccount`，表示還沒有 `BankAccount` 這個類別。這是當然的，因為我們根本還沒寫啊，這時候執行測試如果會過，如果不是你有養小精靈幫你寫程式，就是你在做夢還沒醒。

### 想辦法解決錯誤（綠燈）

我們來把 `BankAccount` 類別做出來吧，在同一個目錄下新增一個名為 `bank_account.rb` 的檔案，內容如下：

```ruby
class BankAccount
end
```

再回到剛剛的 `bank_account_spec.rb` 的第一行加上：

```ruby
require "./bank_account"
```

`require` 可以引入這個檔案的內容。接著，再執行一次測試：

    $ rspec bank_account_spec.rb
    .....

    Finished in 0.00096 seconds (files took 0.13211 seconds to load)
    5 examples, 0 failures

雖然我們的測試裡面還沒有寫任何的內容，但至少沒有剛剛的錯誤訊息了。

### 繼續寫測試（紅燈）

```ruby
require "./bank_account"

RSpec.describe BankAccount do
  describe "存錢功能" do
    it "原本帳戶有 10 元，存入 5 元之後，帳戶餘額變 15 元" do
      account = BankAccount.new(10)
      account.deposit 5
      expect(account.balance).to be 15
    end

    it "原本帳戶有 10 元，存入 -5 元之後，帳戶餘額還是 10 元（不能存入小於等於零的金額）" do
    end
  end

  describe "領錢功能" do
    # ... [略] ...
  end
end
```

等等！那個 `deposit` 跟 `balance` 功能是哪來的？其實現在這個當下還沒寫，我們只是先假設（或期待）這個類別有這些功能，而且在最後算出來的餘額數字是對的。

所以這時候執行測試，沒意外的話應該會出錯：

    $ rspec bank_account_spec.rb
    F....

    Failures:

      1) BankAccount 存錢功能 原本帳戶有 10 元，存入 5 元之後，帳戶餘額變 15 元
         Failure/Error: account = BankAccount.new(10)

         ArgumentError:
           wrong number of arguments (given 1, expected 0)
         # ./bank_account_spec.rb:6:in `initialize'
         # ./bank_account_spec.rb:6:in `new'
         # ./bank_account_spec.rb:6:in `block (3 levels) in <top (required)>'

    Finished in 0.00122 seconds (files took 0.09661 seconds to load)
    5 examples, 1 failure

    Failed examples:

    rspec ./bank_account_spec.rb:5 # BankAccount 存錢功能 原本帳戶有 10 元，存入 5 元之後，帳戶餘額變 15 元

果然出錯了！有錯才是正確的，因為我們根本還沒寫真正的功能啊，這個測試只是先假設我們有這些功能而已。

### 繼續想辦法解決錯誤（綠燈）

好啦，即然知道這個 `BankAccount` 類別要有 `deposit` 跟 `balance` 功能，那就來寫吧。回到 `bank_account.rb`，先讓我們把上面需要的功能寫出來：

```ruby
class BankAccount
  def initialize(amount)
    @amount = amount
  end

  def balance
    @amount
  end

  def deposit(amount)
    @amount += amount
  end
end
```

執行測試：

    $ rspec bank_account_spec.rb
    .....

    Finished in 0.00143 seconds (files took 0.09033 seconds to load)
    5 examples, 0 failures

過了！搞定一個測試，再讓我們繼續往下看下一個測試。

### 繼續寫測試（紅燈）

接下來是存錢功能的第二個測試：

```ruby
require "./bank_account"

RSpec.describe BankAccount do
  describe "存錢功能" do
    it "原本帳戶有 10 元，存入 5 元之後，帳戶餘額變 15 元" do
      account = BankAccount.new(10)
      account.deposit 5
      expect(account.balance).to be 15
    end

    it "原本帳戶有 10 元，存入 -5 元之後，帳戶餘額還是 10 元（不能存入小於等於零的金額）" do
      account = BankAccount.new(10)
      account.deposit -5
      expect(account.balance).to be 10
    end
  end

  describe "領錢功能" do
    #...[略]...
  end
end
```

在寫測試的時候，測試的程式碼不需要寫得很漂亮，只要寫得清楚就好。在我們加上「不能存入小於等於零的金額」的測試之後，執行 `rspec`：

    $ rspec bank_account_spec.rb
    .F...

    Failures:

      1) BankAccount 存錢功能 原本帳戶有 10 元，存入 -5 元之後，帳戶餘額還是 10 元（不能存入小於等於零的金額）
         Failure/Error: expect(account.balance).to be 10

           expected #<Fixnum:21> => 10
                got #<Fixnum:11> => 5

          ...[略]...

    Finished in 0.02562 seconds (files took 0.09308 seconds to load)
    5 examples, 1 failure

    Failed examples:

    rspec ./bank_account_spec.rb:11 # BankAccount 存錢功能 原本帳戶有 10 元，存入 -5 元之後，帳戶餘額還是 10 元（不能存入小於等於零的金額）

咦？又壞了！先想一下，為什麼測試會壞掉？

### 繼續想辦法解決錯誤（綠燈）

如果回去看我們 `BankAccount` 的 `deposit` 方法的實作就會發現，我們根本沒檢查傳入的金額是不是小於零。修正一下：

```ruby
class BankAccount
  def initialize(amount)
    @amount = amount
  end

  def balance
    @amount
  end

  def deposit(amount)
    @amount += amount if amount > 0
  end
end
```

在 `deposit` 方法裡加上了 `if amount > 0` 的判斷後，再執行一次測試：

    $ rspec bank_account_spec.rb
    .....

    Finished in 0.00233 seconds (files took 0.09252 seconds to load)
    5 examples, 0 failures

這樣就過了，而且之前的那個測試也沒壞。很好，就是維持這個手感，再讓我們繼續往下一個測試前進。

### 再繼續測試

避免拖太長的篇幅，我先一口氣先把剩下的三個測試寫完：

```ruby
require "./bank_account"

RSpec.describe BankAccount do
  describe "存錢功能" do
    # ... [略] ...
  end

  describe "領錢功能" do
    it "原本帳戶有 10 元，領出 5 元之後，帳戶餘額變 5 元" do
      account = BankAccount.new(10)
      amount = account.withdraw 5
      expect(amount).to be 5
      expect(account.balance).to be 5
    end

    it "原本帳戶有 10 元，試圖領出 20 元，帳戶餘額還是 10 元，但無法領出（餘額不足）" do
      account = BankAccount.new(10)
      amount = account.withdraw(20)
      expect(amount).to be 0
      expect(account.balance).to be 10
    end

    it "原本帳戶有 10 元，領出 -5 元之後，帳戶餘額還是 10 元（不能領出小於或等於零的金額）" do
      account = BankAccount.new(10)
      amount = account.withdraw(-5)
      expect(amount).to be 0
      expect(account.balance).to be 10
    end
  end
end
```

這時候執行測試，想當然爾一定是會發生錯誤訊息的：

      $ rspec bank_account_spec.rb
      ..FFF

      Failures:

        1) BankAccount 領錢功能 原本帳戶有 10 元，領出 5 元之後，帳戶餘額變 5 元
           Failure/Error: amount = account.withdraw 5

           NoMethodError:
             undefined method `withdraw' for #<BankAccount:0x007ffe4c02e818 @amount=10>
           # ./bank_account_spec.rb:21:in `block (3 levels) in <top (required)>'

        2) BankAccount 領錢功能 原本帳戶有 10 元，試圖領出 20 元，帳戶餘額還是 10 元，但無法領出（餘額不足）
           Failure/Error: amount = account.withdraw(20)

           NoMethodError:
             undefined method `withdraw' for #<BankAccount:0x007ffe4cd43300 @amount=10>
           # ./bank_account_spec.rb:28:in `block (3 levels) in <top (required)>'

        3) BankAccount 領錢功能 原本帳戶有 10 元，領出 -5 元之後，帳戶餘額還是 10 元（不能領出小於或等於零的金額）
           Failure/Error: amount = account.withdraw(-5)

           NoMethodError:
             undefined method `withdraw' for #<BankAccount:0x007ffe4cd41b18 @amount=10>
           # ./bank_account_spec.rb:35:in `block (3 levels) in <top (required)>'

      Finished in 0.00252 seconds (files took 0.09026 seconds to load)
      5 examples, 3 failures

      Failed examples:

      rspec ./bank_account_spec.rb:19 # BankAccount 領錢功能 原本帳戶有 10 元，領出 5 元之後，帳戶餘額變 5 元
      rspec ./bank_account_spec.rb:26 # BankAccount 領錢功能 原本帳戶有 10 元，試圖領出 20 元，帳戶餘額還是 10 元，但無法領出（餘額不足）
      rspec ./bank_account_spec.rb:33 # BankAccount 領錢功能 原本帳戶有 10 元，領出 -5 元之後，帳戶餘額還是 10 元（不能領出小於或等於零的金額）

### 實作領錢功能，繼續想辦法解決錯誤

`BankAccount` 的存錢功能實作如下：

```ruby
class BankAccount
  def initialize(amount)
    @amount = amount
  end

  def balance
    @amount
  end

  def deposit(amount)
    @amount += amount if amount > 0
  end

  def withdraw(amount)
    if amount > 0 && @amount >= amount
      @amount -= amount
      amount
    else
      0
    end
  end
end
```

執行一下測試：

    $ rspec bank_account_spec.rb
    .....

    Finished in 0.00238 seconds (files took 0.09287 seconds to load)
    5 examples, 0 failures

三個測試全部都過了，之前寫的存錢功能也沒有因此被弄壞。

## <a name="note"></a>小結

雖然這個例子可能有點簡單，但希望藉由這樣一連串的「紅綠燈」練習，可以讓大家更了解寫測試的手感，以及為什麼要寫測試。

因為在正式開工之前，先把規格好好的想清楚，可以讓開發者多想幾分鐘，不僅在類別及方法的命名上可以用比較適合的名字，也因為我們的類別或方法也都是照著「規格」寫出來的，所以相對的不會寫出多餘的類別或方法。

不過，測試全部都通過也不表示程式就完全不會有 Bug，只能說「目前的程式碼實作都有滿足現有的規格」。但藉由越完整的測試，除了可以減少 Bug 的出現，最重要的是可避免「修改完 A 功能結果 B 功能跟著壞掉」的問題。

在 Ruby/Rails 的世界，寫測試是算是業界標準的技能，希望大家可以趕快習慣寫測試的「紅綠燈」手感，藉由測試讓你對你的程式碼實作更有信心！

