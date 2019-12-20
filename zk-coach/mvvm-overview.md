# 什麼是 MVVM
MVVM[^1] 是一個設計模式 Model-View-ViewModel 的的縮寫，最早源自於 Microsoft WPF[^2]。 這是 MVC 模式的一種變形。這個模式同樣將物件區分為3個角色: View, Model, ViewModel. View 代表畫面呈現，Model 代表應用程式的資料或邏輯，這兩個角色跟原本 MVC 中所代表的一樣。

角色關係圖如下：(箭頭代表呼叫關係)
![]({{site.baseurl}}/assets/view-viewmodel-model.png)

而 ViewModel 其實跟 Controller 扮演的角色相同，只是 ViewModel 並不透過呼叫 UI 元件的 API 來控制畫面，而是被動由 ZK 內建機制來跟畫面 View 溝通。因此 ViewModel 本身不需要繼承任何父類別或實作任何介面，也不需要宣告 ZK 元件類別的變數在其中，只要包含頁面上**所要儲存狀態或顯示的資料**及**業務邏輯**。

View 是由 ZK 元件所組成的畫面，畫面上的資料來自於 ViewModel，而使用者操作元件造成的狀態改變(例如：在 Textbox 中輸入文字)也會傳回到 ViewModel。

![]({{site.baseurl}}/assets/view-data-binding-viewmodel.png)


[^1]: [WPF Apps With The Model-View-ViewModel Design Pattern](http://msdn.microsoft.com/en-us/magazine/dd419663.aspx)

[^2]: [Introduction to Model/View/ViewModel pattern for building WPF apps](http://blogs.msdn.com/b/johngossman/archive/2005/10/08/478683.aspx)
