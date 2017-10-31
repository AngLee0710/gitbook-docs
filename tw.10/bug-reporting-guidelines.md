# 5. 問題回報指南[^1]

> 本篇談的是如何回報問題到 PostgreSQL 官方組織，而本正體中文手冊並非由官方提供，所以如果你希望指出的問題是本手冊的相關問題，請透過[討論區](https://www.gitbook.com/book/pgsql-tw/documents/discussions)，或[台灣 PostgreSQL 使用者社群](https://pgsql-tw.github.io/)所提供的聯絡資訊回報。

如果你在 PostgreSQL 中發現了問題，我們會很希望可以得到通知。你的問題回報可以讓 PostgreSQL 變得更值得信任，因為百密仍有一疏，PostgreSQL 無法保證在任何平台或任何情況下，都一定是完美無缺的。

下面的建議提供你在回報問題時能夠更有效率。你不一定要完全遵照下面的方式，但如果你試著遵循的話，對大家都有幫助。

我們無法保證可以立即修正所有的錯誤。但如果那個問題是明顯的、關鍵的、或是有重大影響的，那就會有人進行瞭解。也可能會回覆你更新你的資料庫版本，如果是因為版本問題的話。我們也可能會判定該錯誤不會被修正，在我們進行重大修改之前；又也許它不容易簡單處理，而且有其他更重要的需求排程已經在進行中。如果你需要立即性的支援，請接洽當地的商業服務。

### 5.1. 確認錯誤

在報告錯誤之前，請再三閱讀文件，以確認你真的在進行你正在嘗試的事情。 如果從文件中不清楚是否可以做某事，請回報；這是屬於文件的一個錯誤。如果確實證明一個程式與文件所描述地不同，那就是一個錯誤。這可能包括但不限於以下情況：

* 程式以致命信號（fatal signal）或程式中某個問題造成作業系統錯誤訊息而終止。（反例可能是“磁盤已滿”的訊息，因為您必須自己修復。）

* 程式對於任何輸入都產生錯誤輸出結果。

* 程式拒絕接受有效的輸入 （如文件中所定義的）。

* 程式接受了無效的輸入，卻沒有警告或錯誤消息。但請記住，您對於無效輸入的認知可能來自於我們對傳統做法的延伸或相容性。

* PostgreSQL 在支援的平台上，按照指示進行編譯，構建或安裝卻失敗了。

這裡的「程式」是指任何可執行文件，不僅僅是後端執行的程序。

緩慢或資源匱乏不一定是一個錯誤。閱讀文件或在某個郵件列表中提問，可以幫助你調整應用程式。不符合 SQL 標準不代表是錯誤，除非明確聲明相容某個特定的功能。

在繼續之前，請檢查 TODO 列表和常見問題解答，看看您的錯誤是否已知。 如果您無法瞭解 TODO 列表中的資訊，請報告你的問題。 我們至少可以做的是使 TODO 列表更清楚。

### 5.2. 回報內容

關於錯誤報告最重要的是陳述所有事實並且只有事實。不要揣測你的想法是錯的，什麼「似乎」，或程式的哪個部分有故障。如果你不熟悉實作的方法，你可能會猜測錯誤，而無法幫助我們。即使你有知識性的解釋，也應該是對事實的補充而不是替代它們。如果我們要修復這個錯誤，我們還是首先要看到它發生。報告裸露的事實是相對簡單的（你可以從屏幕上複製貼上它們），但是經常重要的細節被忽略，因為有人認為這並不重要，或者認為報告被理解是應當的。

每個錯誤報告中都應包含以下內容：

* 程式執行步驟的確切順序是重現問題所必需的。這應該是獨立的；如果輸出應該依賴於資料表中的資料，那麼沒有包含前面的 CREATE TABLE 和 INSERT 語句的 SELECT 語句是不夠的。我們沒有時間來對你的資料庫進行逆向工程，如果我們需要建立自己的資料庫的話，我們很可能會錯過這個問題。

  SQL相關問題的測試最佳格式是可以透過 psql 執行並重現問題。 （請確認您的 ~/ .psqlrc 啟動設定中沒有任何內容。）建立這個檔案的一個簡單方式是使用 pg\_dump 來轉出設定該情境的資料表宣告及資料，然後加入產生問題的查詢語句。我們希望您盡量減少您的問題規模，但這並不是絕對必要的。如果錯誤是可重現的，我們會以任何一種方式找到它。

  如果您的應用程式使用某些其他客戶端界面（如PHP），請嘗試突顯那些問題查詢語句。我們應該不會設置一個 Web 伺服器來重現您的問題。無論如何，請記得提供確切的輸入檔案；不要猜測「大文件」或「中型數據庫」等問題的可能性。因為這些訊息不夠精確，沒有參考價值。

* 你所得到的輸出。請不要說「不能用」或「壞掉了」。如果出現錯誤訊息，請列出來，即使您並不瞭解它。如果程式終止是因為作業系統錯誤，請說明哪個系統錯誤。如果沒有發生任何事情，也如實說明。即使您的測試案例的結果是當機或其他明顯的情況，也不一定會在我們的平台上發生。如果可以的話，最簡單的方法是從終端視窗中複製輸出內容。

  > ### 注意
  >
  > 如果您要報告錯誤訊息，請取得該訊息最詳細的形式。在 psql，事先輸入`\set VERBOSITY verbose`。 如果從伺服器記錄取得訊息，請在執行時將參數 log\_error\_verbosity 設定為 verbose，以便記錄所有詳細訊息。
  >
  > 如果是嚴重錯誤的情況下，客戶端報告的錯誤訊息可能不包含所有可用的信息，還請查看資料庫伺服器的系統記錄輸出。如果你沒有保留伺服器的系統記錄，那麼這是開始這樣做的好時機。

* 你所期待的輸出情況對於情境說明非常重要。如果你只是寫「這個命令給我這樣的輸出」或「這結果不是我的預期」，我們可以自己運行它，檢視該輸出結果，並認為它看起來正常，正是我們的預期。我們不應該把時間花在解讀你的命令背後確切的語意。特別是不要僅僅說「這不是 SQL 或 Oracle 所做的那樣」。從 SQL 挖掘所謂正確的行為並不是一件有趣的事情，我們更不會知道所有其他關連式資料庫的做法。（如果你的問題是當機，很明顯地你可以省略此項。）

* 任何命令列選項和其他啟動選項，包括您從預設值修改的任何相關的環境變數或設定檔案。再次提醒，請提供確切的訊息。如果您正在使用預先封裝好的套件，在開機時啟動資料庫伺服器，您應該嘗試瞭解它如何進行。

* 所有你所做的與安裝說明不同之處。

* PostgreSQL 的版本。你可以執行`SELECT version();` 來找到你正在連線的資料庫系統版本。大多數的程式工具也會支援`--version`選項；至少`postgres --version`和`psql --version`都可以使用。如果功能或選項不存在，那麼您的版本已經太舊而無法進行升級。如果您運行預先編譯的套件（如 RPM），請說明，包括該套件可能具有的子版本。如果您正在使用 Git 某個快照，請說明快照版本，包括提交碼（commit hash）。

  如果您的版本比10還舊，我們幾乎肯定會告訴您進行升級。每個新版本都有很多錯誤修復和改進，所以很可能在PostgreSQL的舊版本中遇到的錯誤已經被修復了。我們只能對使用較早版本的PostgreSQL伺服器提供有限的支援；如果你的需求多於我們所能提供的，請考慮取得商業支援合約。

* 執行平台資訊。這包括作業系統核心名稱和版本，C語言函式庫，中央處理器、記憶體資訊等等。在大多數情況下，報告系統供應商和版本是足夠的，但不要假設每個人都知道「Debian」究竟是什麼，或是每個人都運行在 i386 上。如果您有安裝問題，則還需要你的機器上有關系統工具組的訊息（編譯器、make 工具等等）。

不要擔心你的錯誤報告會因此而變得冗長。這是一個事實過程的呈現。第一次報告所有事情，對我們比較好的做法是，儘量把事實從你身上擠出來。 另一方面，如果你輸入的檔案很大，持平來說，首先要問是否有人有興趣去研究它。這裡有一篇[文章](http://www.chiark.greenend.org.uk/~sgtatham/bugs.html)，概述了有關報告錯誤的更多建議。

不要花所有的時間來指出輸入中的哪些變化會使問題消失。這可能對解決問題沒有幫助。如果事實證明該錯誤不能立即解決，您仍然有時間找到並分享您的解決方案。此外，再一次，不要浪費你的時間猜測為什麼這個問題會存在。我們會很快找到。

在撰寫錯誤報告時，請避免混淆術語。這個軟體整體來說被稱為「PostgreSQL」，有時候會簡稱為「Postgres」。如果你特別在談論後端的程序，請明確指出，而不要只是說「PostgreSQL 當掉了」。其中一個後端程序的當掉與主要的「postgres」程序當掉有很大的不同；當你的意思是某個後端程序終止時，請不要說「伺服器當掉了」，反之亦然。此外，例如交互式前端的「psql」用戶端程序與後端完全分離的。請嘗試具體說明問題是在用戶端還是伺服器端。

### 5.3. Where to Report Bugs

In general, send bug reports to the bug report mailing list at`<`[`pgsql-bugs@postgresql.org`](mailto:pgsql-bugs@postgresql.org)`>`. You are requested to use a descriptive subject for your email message, perhaps parts of the error message.

Another method is to fill in the bug report web-form available at the project's[web site](http://www.postgresql.org/). Entering a bug report this way causes it to be mailed to the`<`[`pgsql-bugs@postgresql.org`](mailto:pgsql-bugs@postgresql.org)`>`mailing list.

If your bug report has security implications and you'd prefer that it not become immediately visible in public archives, don't send it to`pgsql-bugs`. Security issues can be reported privately to`<`[`security@postgresql.org`](mailto:security@postgresql.org)`>`.

Do not send bug reports to any of the user mailing lists, such as`<`[`pgsql-sql@postgresql.org`](mailto:pgsql-sql@postgresql.org)`>`or`<`[`pgsql-general@postgresql.org`](mailto:pgsql-general@postgresql.org)`>`. These mailing lists are for answering user questions, and their subscribers normally do not wish to receive bug reports. More importantly, they are unlikely to fix them.

Also, please do\_not\_send reports to the developers' mailing list`<`[`pgsql-hackers@postgresql.org`](mailto:pgsql-hackers@postgresql.org)`>`. This list is for discussing the development ofPostgreSQL, and it would be nice if we could keep the bug reports separate. We might choose to take up a discussion about your bug report on`pgsql-hackers`, if the problem needs more review.

If you have a problem with the documentation, the best place to report it is the documentation mailing list`<`[`pgsql-docs@postgresql.org`](mailto:pgsql-docs@postgresql.org)`>`. Please be specific about what part of the documentation you are unhappy with.

If your bug is a portability problem on a non-supported platform, send mail to`<`[`pgsql-hackers@postgresql.org`](mailto:pgsql-hackers@postgresql.org)`>`, so we \(and you\) can work on portingPostgreSQLto your platform.

### Note

Due to the unfortunate amount of spam going around, all of the above email addresses are closed mailing lists. That is, you need to be subscribed to a list to be allowed to post on it. \(You need not be subscribed to use the bug-report web form, however.\) If you would like to send mail but do not want to receive list traffic, you can subscribe and set your subscription option to`nomail`. For more information send mail to`<`[`majordomo@postgresql.org`](mailto:majordomo@postgresql.org)`>`with the single word`help`in the body of the message.

---

[^1]: [PostgreSQL: Documentation: 10: 5. Bug Reporting Guidelines](https://www.postgresql.org/docs/10/static/bug-reporting.html)

