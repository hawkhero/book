# 元件初探
初學者可能遇到的問題就是不知道哪些元件可以符合畫面的需要，你可以從以下 3 個地方來查詢元件的功能與用法：
* [ZK Demo](https://www.zkoss.org/zkdemo/grid)

元件線上展示，你可以從這裡看到每個元件實際上在瀏覽器中繪製出來的樣子，試著操作看看來確定哪個元件的功能是不是你要的，或是要怎麼用，找到之後下方就有該頁面的說明，程式碼分成 View, Controller, Model 是為了讓架構清楚。

![](/assets/descriptionCode.png)

點 "Test Code online" 你可以連到 ZK Sandbox 可以讓你線上修改程式碼並即時呈現修改後的結果，如果想即時看一些功能的效果，可以透過這個方法。或是貼到 [zkfiddle](/zkfiddle.org)。
"Dowhload This Demo" 可以下載該展示範例的程式碼，方便你複製到你的專案中使用。
* [Component Reference](http://books.zkoss.org/wiki/ZK_Component_Reference)

這是介紹文件功能的主文件，你可以從這本手冊來查到每個元件上各種功能的用法與技術細節。
* [ZK Javadoc](http://zkoss.org/javadoc/latest/zk/)

有些功能較簡單、易於理解的元件屬性，或是 Component Reference 沒寫到的屬性，可以查 Javadoc。

# 元件用法
ZUL 是一個 XML 格式的 UI 語言，每個 tag 代表一個元件，tag 的名稱會對應到一個 Java class。

![](/assets/window-window.png)

元件主要透過屬性 (attribute) 來控制其行為與功能。每個屬性都對應到其 Java class 上的一個 setter，所以如果 `<button>` 上有`disabled` 屬性，代表 `Button.java` 上必定有 `setDisabled()`。

ZK 元件也可以用 Java API 來產生，跟 Swing 的寫法類似，但是用 zul 寫法的可讀性較高。

# 主畫面
為了營造跟桌上應用程式類似的介面體驗，我們用 Window 當作主畫面外框。
```xml
<?link rel="stylesheet" type="text/css" href="style.css"?>
<window border="normal" width="500px" vflex="1" contentStyle="overflow:auto" sclass="todo-window">

</window>
```
讓我解釋一下幾個看起來比較難懂的屬性。

`<?link?>` 是告訴 ZK 把 `<link>` 放在產生的 HTML 的 `<head>` 中。

他是一種給 ZUL 剖析器的處理指示 (processing instruction)，ZK 中稱為 directive，用來告訴剖析器怎麼處理這頁 zul，[所有的 directive 可參考 ZUML Reference](https://www.zkoss.org/wiki/ZUML%20Reference/ZUML/Processing%20Instructions)。

## 自動計算元件尺寸：vflex/hflex
vflex 指得是 vertical flexible，就是**元件高度的自動調整模式**，其值為 1 或 true 代表「填滿可用空間」，所以父元件給它多大，Window 就會撐滿，因為 Window 就是最上層根元件，因此就會撐滿整個瀏覽器可見高度，如果你調整瀏覽器高度，Window 高度也會自動調整。這是 ZK 透過 javascript 來幫你計算可用空間並透過 CSS 設定在 HTML element 上。

若給 min 代表**依據子元件的總和高度來設定高度**，不過子元件增加時，並父元件**不會自動調整尺寸**。另外也有 hflex，用來設定寬度的自動調整模式，用法一模一樣。運用這兩個模式可以幫助你不用把元件的大小寫死，配合各種螢幕大小，詳細使用技巧請參閱 [ZK Developer's Reference](https://www.zkoss.org/wiki/ZK%20Developer's%20Reference/UI%20Patterns/Hflex%20and%20Vflex)。

## 調整元件 CSS style
style 可以 inline 方式設定 CSS rule，也就是直接設定在 HTML element style 屬性上。比較直接方便，但是如果想要重用 CSS rule 就必須用 sclass。

`contentStyle="overflow:auto`

用來設定 Window 內容區域的 CSS style。因為避免容器類元件內容過多將畫面撐得過大，因此 zk 容器類元件都預設 overflow:hidden，如此超過 Window 大小的內容都會被截掉，而且沒有捲動軸。我這樣設定可以讓 Window 內容過多的時候，自動產生捲動軸。


`sclass="todo-window"`

sclass 代表得就是 CSS class，只是因為 class 是 Java 的保留字，因此名稱上避免用 class（但其實你寫 class 仍然可以通過剖析）。這個屬性是用來增加 CSS class 到元件上，用來客製化元件外觀。所以這裡的 todo-window 就定義在第一行的 style.css 中。我用它來設定 margin，使得 Window 外框跟瀏灠器內緣有一些留白，視覺上不會太擁擠。

## 圖示
iconSclass 讓你輸入 [font awesome 圖示](http://fontawesome.io/)的 CSS class，就可以顯示該圖示，但是前綴字串要改成 **z-icon**。例如 font awesome 圖示名稱叫 fa-list-ul，在 ZK 你就可以用 **z-icon**-list-ul來用這個圖示，細節請參考  [Component Reference](https://www.zkoss.org/wiki/ZK_Component_Reference/Base_Components/LabelImageElement#IconSclass)。並不是所有的元件都支援 iconSclass，只有 [LabelImageElement](http://www.zkoss.org/javadoc/latest/zk/org/zkoss/zul/impl/LabelImageElement.html)的子類別才支援。

## 視窗標題

title 屬性只能給文字，如果要樣式複雜一點的標題，就要用 `<caption>`。

**/tutorial/buildView/myTodoList.zul**
```xml
...
<window border="normal" width="500px" vflex="1" contentStyle="overflow:auto" sclass="todo-window">
    <caption iconSclass="z-icon-list-ul" style="font-size:22px" sclass="fn-caption" label="我的待辦清單" />
</window>
```
