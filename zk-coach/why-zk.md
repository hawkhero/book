[ZK](https://www.zkoss.org) 是一個基於 Java EE 與 AJAX 的使用者介面框架 (UI framework)，它提供上百個元件讓你打造快速反應的網頁應用程式 (web application)的介面，每個元件透過 AJAX 跟伺服器溝通後，只更新部分畫面，因此可以帶來類似於桌面應用程式的使用體驗。並可用 xml 語法來描述介面，可以快速建構畫面，支援事件驅動(event-driven)的方式來實作應用程式邏輯，簡單易上手。

以下列出幾個特點：
# 不需要寫 JavaScript 就能擁有 AJAX
[AJAX](https://zh.wikipedia.org/wiki/AJAX) 能夠使你的網頁應用程式提供的使用者經驗提升。因為每次使用者觸發畫面更新時，不用重新載入整個畫面，只會局部改變頁面，讓使用者有類似桌上型應用程式的使用體驗。但是要把 AJAX 用得好又易於維護並不容易，ZK 的每個元件都已經內建 AJAX 技術，因此開發者只需要懂 Java API 就能輕鬆得到 AJAX 的好處。

# 元件化介面
ZK 為一個基於 Java EE 技術的使用者介面框架 (UI framework)，提供豐富的 UI 元件來幫助你打造應用程式介面，從簡單的下拉選單到複雜的樹狀圖都有，你可以從 [ZK Demo](https://www.zkoss.org/zkdemo) 看到這些元件實際在瀏覽器中呈現的效果跟他們提供的功能。

**ZK Demo**

![](/assets/zkdemo.png)

你可以輕易地用一種 XML 格式的語法，我們稱 **ZUL** (ZK User Interface Markup Language)來建構介面，或是使用ZK Java API。

# 事件驅動 (event driven)

使用 ZK 時，基本上開發模式基本上是「事件驅動」(event driven)的，使用者跟元件互動時會觸發對應的事件 (event)，這些事件都透過 **AJAX** 傳到伺服器， ZK 會呼叫事件對應的傾聽器(你所實作的)來完成業務邏輯，並將對應的更新資料傳回瀏覽器，元件收到後會自行重新繪製必要更新的部份，並不會整頁更新。

# 基於 Java EE
採用 ZK 可以只使用 Java，不需要撰寫 JavaScript 程式，不管是操作元件、處理事件都只需要撰寫 Java，這讓 ZK 可以很輕易地整合既有的基於 Java EE的企業級應用系統，當然包括整個 Java 的生態系 （如第三方函式庫等）。


# 架構清楚
一般來說，我們會將網頁應用程式打造成多層(multi-tiers)架構（如下圖），每一層都是由功能相近的一群 class 組成，這樣的目的是讓整個系統易拓展、好維護、重用性高。

![](/assets/multi-tiers.png)

ZK 對於自身的定位很明確是「UI 框架」，並不預設你要用何種服務層框架例如 Spring，或儲存相關的框架如 JPA 、Hibernate，當然也不會限制使用何種資料庫。所有跟控制 ZK 元件相關的邏輯主要都集中在控制器層 (Controller)，而該層就是由 Java class 實作，可以是 Composer/ViewModel，因此你可以輕易呼叫任何其他 Java class。因此跟 ZK 相關的程式碼完全不會混入你的其他層的實作。


# 免安裝線上試用
[ZK fiddle](http://zkfiddle.org/) 是類似 [JSFiddle](https://jsfiddle.net/) 的網站，讓你線上撰寫 ZK ，無須安裝、設定環境，主要是讓你試玩元件的效果，真正開發時還是需要在本地端建立環境。

![](/assets/zkfiddle.png)
