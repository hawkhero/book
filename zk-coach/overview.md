
<a href="https://www.zkoss.org">![]({{site.baseurl}}/assets/zklogo.png)</a>
是一個基於 Java EE 與 AJAX 的使用者介面框架 (UI framework)，它提供上百個元件讓你打造快速反應的網頁應用程式 (web application)的介面，每個元件透過 AJAX 跟伺服器溝通後，只更新部分畫面，因此可以帶來類似於桌面應用程式的使用體驗。並可用 xml 語法來設計介面，既快速也易讀。以支援事件驅動(event-driven)的方式來實作應用程式邏輯，簡單易上手。

更支援以 [MVVM (Model-View-ViewModel) 設計模式](http://books.zkoss.org/zk-mvvm-book/8.0/index.html)開發，讓你的應用程式跟 ZK 元件的耦合性更低，也讓畫面跟畫面控制邏輯解耦合，利用雙向資料繫結語法 (2 ways data binding) 來同步畫面(View) 與應用程式邏輯 (ViewModel) ，更利於打造[響應式設計](https://zh.wikipedia.org/wiki/%E5%93%8D%E5%BA%94%E5%BC%8F%E7%BD%91%E9%A1%B5%E8%AE%BE%E8%AE%A1)的網站。

以下為部分 ZK 元件在預設佈景主題的概觀：

![]({{site.baseurl}}/assets/iceblue.png)

想了解更多可以參觀：
* [ZK 元件展示](https://www.zkoss.org/zkdemo/)
* [24 個佈景主題預覽](https://www.zkoss.org/zk85themedemo/)
* [開始學習 ZK 系列文章](https://www.zkoss.org/documentation)



# 本書目的
本書針對剛開始想要學習 ZK 的人所寫的，主要介紹用 MVVM 開發網頁應用程式的基本觀念。其目的並不是要寫一個中文版的 [ZK Developer's Reference](http://books.zkoss.org/wiki/ZK_Developer%27s_Reference)，是因為在擔任教育訓練講師的過程中，看見中文使用者的需要，包括我自己，看中文還是親切一點；另一個是因為 ZK 文件很豐富而難以找到需要的章節閱讀，因此希望給讀者一個基本引導，讓讀者能了解 ZK 的主要概念，其他未提到的細節部分還是請參照官方文件。不過電子書的好處就是增補內容很容易，如果讀者反應良好，我也會增加更多內容。


# 特色

* 範例專案  
  本書附帶一個範例專案，裡面包含所有書中提到的範例程式。你可以輕易地將整個專案執行在 jetty 上，透過瀏覽器看到執行結果，有助於了解程式碼。

* 練習專案  
  執行 maven 指令可以產生一個練習專案，它以範例專案為基礎，但是刻意移除部分程式碼，好讓讀者練習之用。因此你可以產生範例專案來練習，若真的遇到困難再參考完整程式。

* 圖片解說  
  儘量以各式圖片解說概念。我認同 Head First Design Pattern 一書中提及的理念，學習不是只能靠文字，其他的媒介可以讓大腦得到更多資訊，更增強學習效果。

* 情境 FAQ  
  ZK 目前已經是個頗為成熟的框架，功能很多，因此不熟悉的人常常不知道實務上某個情境要用什麼功能來解決，這裡會列出各個常用的情境並提供連結指向建議使用的 ZK 特性。

# 必要知識

我預設你已經了解 Maven、Java EE、XML，因此與這些相關的基礎知識除非必要我不會詳加說明。

# 專有名詞

我贊同侯捷前輩對於翻譯英文專有名詞的看法，如果中文沒有廣為人知、易懂的譯名的話，我會維持英文原名，畢竟 ZK 本身也創造了一些自訂的術語，因此我會使用英文關鍵字來指稱這些技術，這樣讀者在網路上查詢文件或問答的話，也比較知道用哪個英文關鍵字查詢。

# ZK 版本

本書內容針對 8.5 EE \(Enterprise Edition\)，需要 EE 的功能我會額外標注。

歡迎利用 gitbook 給回饋，或是寄信到 **hawkhero at gmail**

# [點此訂閱本書更新訊息](https://docs.google.com/forms/d/e/1FAIpQLScj0yrVJCeT4239GlsipZpbLha0MRsqa0TzMjNjoCqquk3EOA/viewform?usp=sf_link)
