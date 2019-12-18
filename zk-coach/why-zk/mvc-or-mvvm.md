ZK 支援兩種設計模式：

* MVC \(Model View Controller\)
* MVVM \(Model View ViewModel\)

兩者各有其長處，可以根據你的開發習慣與偏好選擇，以下簡單介紹兩者的特性：

# MVC

[MVC](https://zh.wikipedia.org/wiki/MVC) 其實是個通用的軟體模式詞彙，泛指一般把系統分成模型（Model）, 視圖（View）, 控制器（Controller）三部分的架構。ZK 借用這個詞來指稱「**直接透過元件 API 來控制元件**」的設計模式，這個設計模式是ZK 從最初就支援的。 在 ZK 的架構下， View 指得是元件所組成的畫面，可以說是 .zul 檔， Model 代表你的應用程式邏輯，Controller 介於 View 與 Model 之間，責任是控制 ZK 元件、接受並處理元件發出的事件、將後端資料送到畫面上、呼叫 Model 層。

![](/assets/mvc.png)

這種設計模式下，你必需實作一個繼承一個 ZK 內建控制器類別\(`SelectorComposer`\) 的 Controller，然後在 Controller 中實現你的應用程式邏輯。實現的方法就是透過實作 event listener method 傾聽元件上發出的事件（如 `button` 會發出 `onClick` 事件\)，再透過元件提供的 API 來控制畫面，例如我要將文字內容改變，就呼叫 `Label.setValue()`。

本模式的好處是：

* **簡單易懂**。
  基本上就是類似 Swing 的概念：以元件為基礎（Component-based）, 事件驅動（event-driven）。要做什麼就呼叫對應的元件 API，寫出來的程式碼很容易了解，學習門檻較低。

缺點就是：

* **不易自動化測試 Controller**。
  為了要控制元件，Controller 中必須宣告許多 ZK 元件型態的變數，這樣才可以透過變數來控制畫面，但這些元件的物件是由 ZK framework 執行期塞入到 Controller 的變數中的。因此若沒有啟動應用伺服器的的環境跟瀏灠器，你很難測試你的 Controller。
* **Controller 易受畫面變化影響而修改**。
  同樣是因為 Controller 包含 ZK 元件的關係，因為元件一定跟 zul 上所寫的元件有一對一對應關係，因此如果畫面修改，更換了元件，通常 Controller 也一定得修改。


# MVVM

這個模式是 6.0 之後支援，ZK 也強力推廣的模式。這個模式的 Model 跟 View 所代表的角色跟 MVC 相同， **VM** 代表的是 **ViewModel**，其實是另一種形式的控制器角色，只是這個模式下，你不需呼叫元件的 API 來控制畫面，而是「**透過資料繫結 \(data binding\) 來控制元件**」。

![](/assets/mvvm.png)

要實作一個 ViewModel class，你不需要繼承任何父類別，也不需要實作任何介面，只需新建一個 Java class \(POJO\) 即可。ViewModel 扮演控制器的方式是被動的，它只儲存以下兩者：

**畫面的狀態 (state)**：存在 class 的成員變數當中，例如畫面上要輸入使用者名稱，因此你 ViewModel 就要宣告 `String name` 來存

**命令 (command)**：透過方法來實作的業務邏輯，例如你的 ViewModel 可以定義 `public void search(String keyword)` 來實作你畫面上的搜尋邏輯

實作好的 ViewModel 要怎麼跟 zul 上的元件產生連結呢？就是透過**資料繫結 (data binding) 語法**。透過 data binding ，可以將 ViewModel 中的變數跟元件的 attribute 綁在一起，ZK 就會自動將兩邊同步，例如將字串變數 `name` 與 輸入元件  `Textbox` 的 `value` 繫結，而 `value` 代表使用者輸入的值，因此當使用者在瀏灠器頁面中輸入文字時， ZK 就會幫你把該輸入存到伺服器上的變數 `name` 中，反之亦然，你若更改了 ViewModel 中 `name` 的值，ZK 也會幫你顯示到頁面上。這個 View 跟 ViewModel 之間同步的過程你完全不需要寫程式，只要寫好適當的 data binding 語法即可。

呼叫實作業務邏輯的 method 做法也相同，也是利用 data binding 將某個元件的事件（例如 Button 的 onClick）綁定到某個 method 上，而當該事件發生時，ZK 就會自動幫你呼叫該 method。

從以上敘述你可以發現 ViewModel 中完全不需要宣告 ZK 元件型態的變數，也就是 ViewModel 可以跟畫面/元件解耦合 (decoupling)。

這有幾個好處：
* **ViewModel 不易受畫面變動影響**。
因為 ViewModel 中不含任何變數指向 ZK 元件，因此例如將畫面上的 `Grid` 換成 `Listbox`，ViewModel 不需要跟著修改，只要修改 data binding 即可。
* **重用性較高**。
不同頁面只要顯示的資料相同，就可以套用同一個 ViewModel。
* **ViewModel 較易於測試**。
因為 ViewModel 不用繼承 ZK Composer 因此不需要啟動伺服器與 ZK framework 來初始化，你可以自行透過如 JUnit 框架，直接 `new MyViewModel()` ，並呼叫其 method 來測試它。
* **較有利於響應式設計（Responsive Design）**。
響應式設計（Responsive Design）多半需要針對同一個頁面設計兩個（或以上）佈局，一個給桌面瀏覽器，一個給行動裝置。雖然佈局不同，但是顯示的內容大同小異，而且在資料面來說幾乎是相同，也就是可以在不同的頁面上重用同一個 ViewModel，降低實作成本。
* **可同時開發畫面與 ViewModel**。
只要頁面跟 ViewModel 間的溝通介面定義完畢，也就是屬性 (properties) 與命令 (command)，那麼兩邊就可以同時開發，並不會影響彼此。

缺點是：
* **前期需花時間學習 MVVM 的行為**。直接呼叫元件 API 比較容易理解，MVVM 因為透過 ZK 來同步頁面與 ViewModel，因此需要先了解 ZK 幫你做了什麼。

# 避免混用

不過我強烈建議**不要在同一個元件範疇中同時套用2種方法**，例如你已經在某個元件上套用了 Controller，就不要在同一個元件或其下的子元件再套用 ViewModel（反之亦然） 如：

```xml
<div apply="MyController">
    <div viewModel="MyViewModel">
    ...
    </div>
</div>
```

理由是 ZK MVVM 的設計本身並未預期 ZK 在操作元件的過程有其他程式同時操作，因此你自行用 API 操作元件可能會與 ZK 本身的操作產生衝突而出現不預期的結果。
