---

title: 開發工具與常用命令列指令
comments: true
permalink: /chapters/03-command-line-tools.html

---

# 開發工具與常用命令列指令

在上一個章節介紹安裝 Ruby 及 Rails，在正式開始寫我們第一個應用程式之前，先介紹一下開發工具以及在開發 Rails 專案過程中常會用到的指令。

- [開發工具](#dev-tools)
- [常用命令列指令](#command-line)
- [不要害怕指令、不要害怕錯誤](#dont-be-scared-of-command-line)

## <a name="dev-tools"></a>開發工具

剛接觸 Ruby 或 Rails 的朋友常會問道：「我是 Ruby/Rails 的新手，請問有像 Apple 的 Xcode、Microsoft 的 Visual Studio 或至少像 Dreamweaver 之類方便或視覺化的整合開發工具（Integrated Development Environment, IDE）嗎？」

簡單的答案：「目前沒有」

那大家都用什麼工具在開發？其實每種程式語言的開發環境都不相同，大部份的 Ruby/Rails 開發者，不太使用這樣的東西。 但其實不是不用，而是幾乎沒有這樣的工具可以用。

如果程式語法不熟，沒有程式碼提醒或是語法自動補完的工具怎麼辦？Ruby/Rails 語法不熟，就多查手冊、多用、多寫幾次就會熟了。身為一名稱職的開發者，也不應該太過依賴這樣的提醒功能。別太依賴工具，蹲好馬步、練好正拳把基礎打穩才是正途，別被開發工具寵壞了。而且 Ruby/Rails 的語法都短短的，語法本身也相當直覺、易懂，沒有程式碼提醒或語法自動補完也不是太大的問題。

Ruby/Rails 的開發者只要手上有任何一款文字編輯器就能進行開發（就跟龍五手上只要有槍....類似的概念吧），雖然沒有好用的開發工具對新手來說是個不小的門檻，但根據幾年在學校或課堂上教授 Ruby/Rails 課程的經驗來看，這都不是真正造成學習者會卡關的地方。

以下介紹幾款曾經使用比較順手的文字編輯器。

### Sublime Text

[Sublime Text](https://www.sublimetext.com/) 是一款商業軟體，雖然需要付費購買，但即使沒有付費也可使用（超佛心！），它的優點除了有程式碼上色之外，好用的外掛也非常多。

### Atom

[Atom](https://atom.io/) 是由 GitHub 出資的開發的編輯器，不僅完全免費，連原始碼都直接開放了，同樣也有程式碼上色功能，外掛也越來越豐富。

### Vim / Emacs

這兩款文字編輯器的年紀已經有三、四十歲了，說不定都比大家還要老。雖然很老，但到現在還是很多開發者會使用，而且各有各的擁護者，一款稱之「編輯器之神」，另一款稱「神之編輯器」（可參閱「[編輯器之戰](https://zh.wikipedia.org/wiki/%E7%BC%96%E8%BE%91%E5%99%A8%E4%B9%8B%E6%88%98)」）。

我自己目前主要使用 Vim，並不是說它特別強大，主要是它跟終端機可以無縫整合（因為 Vim 本身就在終端機裡）。在 Rails 專案開發過程中，有很多機會需要在終端機環境下輸入指令，所以對我來說開發起來比較順手，另外主要的原因是因為已經用習慣了。

### RubyMine

說沒有 IDE 其實是騙人的，還是有商業公司推出一套名為 [RubyMine](https://www.jetbrains.com/ruby/) 的整合開發工具。它的優點可以提醒或自動完成語法，對 Ruby 語法還不熟的新手來說應該有幫助；但缺點是執行速度有比較慢一點點，另一個不太算缺點的缺點就是它的收費比其它的軟體要來得貴一些。

## <a name="command-line"></a>常用命令列指令

在 Rails 的開發過程中，許多指令都是在終端機（Terminal）環境操作。由於大部份的初學者較習慣圖形介面工具，不熟悉指令該怎麼輸入，或是輸入的指令是什麼意思，這點是讓新手覺得容易挫折的地方。以下介紹幾個在終端機環境常會用到的指令。

| 指令          | 說明                     |
| ------------- |:-------------------------|
| cd            | 切換目錄                 |
| pwd           | 取得目前所在的位置       |
| ls            | 列出目前的檔案列表       |
| mkdir         | 建立新的目錄             |
| touch         | 建立檔案                 |
| cp            | 複製檔案                 |
| mv            | 移動檔案                 |
| rm            | 刪除檔案                 |
| sudo          | 暫時取得權限             |

### 目錄切換

在 Rails 專案開發過程中，指令需要下在正確的目錄裡才能正常運作，所以學會目錄的切換是很重要的。

    # 切換到 /tmp 目錄（絕對路徑）
    $ cd /tmp

    # 切換到 my_project 目錄（相對路徑）
    $ cd my_project

    # 往上一層目錄移動
    $ cd ..

    # 切換到使用者的 home 目錄中的 project 裡的 namecards 目錄
    $ cd ~/project/namecards/

    # 顯示目前所在目錄
    $ pwd
    /tmp

### 檔案列表

`ls` 指令可列出在目前目錄所有的檔案及目錄，後面接的 `-al` 參數，`a` 是指連小數點開頭的檔案（例如.gitignore）也會顯示，`l` 則是完整檔案的權限、擁有者以及建立、修改時間：

    $ ls -al
    total 56
    drwxr-xr-x  18 user  wheel   612 Dec 18 02:20 .
    drwxrwxrwt  24 root  wheel   816 Dec 18 02:19 ..
    -rw-r--r--   1 user  wheel   543 Dec 18 02:19 .gitignore
    -rw-r--r--   1 user  wheel  1729 Dec 18 02:19 Gemfile
    -rw-r--r--   1 user  wheel  4331 Dec 18 02:20 Gemfile.lock
    -rw-r--r--   1 user  wheel   374 Dec 18 02:19 README.md
    -rw-r--r--   1 user  wheel   227 Dec 18 02:19 Rakefile
    drwxr-xr-x  10 user  wheel   340 Dec 18 02:19 app
    drwxr-xr-x   8 user  wheel   272 Dec 18 02:20 bin
    drwxr-xr-x  14 user  wheel   476 Dec 18 02:19 config
    -rw-r--r--   1 user  wheel   130 Dec 18 02:19 config.ru
    drwxr-xr-x   4 user  wheel   136 Dec 18 02:41 db
    drwxr-xr-x   4 user  wheel   136 Dec 18 02:19 lib
    drwxr-xr-x   4 user  wheel   136 Dec 18 02:23 log
    drwxr-xr-x   9 user  wheel   306 Dec 18 02:19 public
    drwxr-xr-x   9 user  wheel   306 Dec 18 02:19 test
    drwxr-xr-x   7 user  wheel   238 Dec 18 02:23 tmp
    drwxr-xr-x   3 user  wheel   102 Dec 18 02:19 vendor

### 建立檔案、目錄

    $ touch index.html

如果 index.html 這個檔案本來不存在，`touch` 指令會建立一個名為 index.html 的空白檔案；如果本來就已經存在，則只會改變這個檔案的最後修改時間，並不會變更其內容。

    $ mkdir demo

`mkdir` 指令會在目前所在目錄，建立一個名為 demo 的目錄。

### 檔案操作

    # 複製 index.html 成 about.html
    $ cp index.html about.html

    # 把 index.html 更名成 info.html
    $ mv index.html info.html

    # 刪除 index.html
    $ rm index.html

    # 刪除在這個目錄裡所有的 html 檔
    $ rm *.html

### 取得權限

有些指令需要有系統管理權限（root 權限）才能執行（例如要幫使用者變更密碼）。這時可在指令前面再加上 `sudo` 指令，只要你本身有可以使用 `sudo` 指令的權限，就可暫時的透過這個指令取得 root 權限：

    $ sudo passwd john

## <a name="dont-be-scared-of-command-line"><a>不要害怕指令、不要害怕錯誤

在 Rails 專案開發過程中會在終端機環境輸入許多指令，對新手來說是個不小的障礙。但請不要擔心，在開發過程用到的指令其實都不會太複雜，應該多用幾次就能上手，千萬不要因為指令輸入錯誤而造成挫折。
 另外，在終端機執行指令後，不管成功或失敗，通常都會有訊息顯示在指令之後，這些訊息請多花幾秒鐘仔細的閱讀。很多的新手以為看到訊息就等於是指令執行成功，但事實上可能是錯誤訊息。

看到錯誤訊息不用擔心，因為通常答案就在錯誤訊息中，舉個例子來說：

![image](/images/chapter03/pending_migration.png)

這紅紅的錯誤訊息 `ActiveRecord::PendingMigrationError` 看起來一開始有點嚇人，但仔細看，它底下寫著一行貼心小提示：

    Migrations are pending. To resolve this issue, run: bin/rails db:migrate RAILS_ENV=development

意思就是說你有個 Migration 還沒處理，只要執行一下 `rails db:migrate` 就解決了。

不要害怕輸入指令，不要害怕錯誤訊息，加油！

