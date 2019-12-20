# AJAX 方式的頁面導航(page navigation)
過去 JSP 時代使用者每次要送出資料都得 submit form 到一個 URL，因此我們傾向把每個單一功能實作在獨立的頁面，換功能的時候也是整個頁面換掉。但是使用 ZK 後，我們可以只更改部分畫面，使用者送出資料透過 AJAX 被 event listener 處理了。URL 跟處理使用者的 request 已經沒有直接關聯，你就可以把多個功能實作在同一個頁面，例如我的範例就把事項列表、新增、完成都做在同一個頁面上，可以避免頁面切換。但如果真的要切換頁面的時候，也不用整個頁面換掉，只要更新不同的部分就好。


# 套用範本 (Template Injection)
Shadow component 中的 [`<apply>`](http://books.zkoss.org/zk-mvvm-book/8.0/shadow_elements/ui_template_injection.html) 就適合實作頁面導航。它可以讓你插入一個 zul 到當前的頁面中：
`<apply templateURI="anotherPage.zul"/>`

如果搭配 data binding 來動態改變 URI 如：
`<apply templateURI="＠load(vm.page)"/>`

只要改變 `vm.page` 的值，就可以達到切換頁面的效果。


# 頁面切換
接下來，我希望點選一個事項，就可編輯其內容。為了讓單一頁面維持精簡，我將「事項列表」與「編輯事項」分出來做成 2 個獨立的頁面，另有一個主頁面，當使用者點選一個事項時，我就把「事項列表」那一塊切換成「編輯事項」，編輯後再切換回來。因此總共有三個頁面，每個頁面套用各自的 ViewModel，這樣每個 ViewModel 只會包含相關性較高的應用程式邏輯，就可做到每個 ViewModel 內部內聚性高 (high coherence)，但 ViewModel 間耦合性低 (low coupling)。

![]({{site.baseurl}}/assets/navigation.png)

主頁面 myTodoList.zul
```xml
<?link rel="stylesheet" type="text/css" href="/tutorial/style.css"?>
<window viewModel="@id('vm') @init('zkcoach.mvvm.tutorial.controller.MyTodoListViewModel')"
	border="normal" width="40%" vflex="1" contentStyle="overflow:auto" sclass="todo-window">
	<caption iconSclass="z-icon-list-ul" style="font-size:22px;"
		sclass="fn-caption" label="@init(vm.title)" />
	<apply templateURI="@load(vm.currentPage)"/>
</window>
```

主頁面只有一個 window 作為應用程式外框，內容只有一個 `<apply>`，我們要透過 data binding 改變 `vm.currentPage` 來切換頁面。

# ViewModel 間的溝通 - global command binding
拆開成兩個 ViewModel 之後，問題就來了，現在主頁面與 todoList.zul 分屬不同的 ViewModel，兩者要怎麼溝通呢？其中一種方式就是透過 global command binding，它跟 command binding 類似，都是綁定在元件事件屬性上，事件發生時會觸發對應的 global command method，而預設會呼叫同一個 desktop 下所有 ViewModel 內的 global command method，是一種「一對多」的呼叫。

![]({{site.baseurl}}/assets/global-command.png)

我們讓 myTodoList.zul 負責控制頁面導向，因此實作一個 global command `navigate()` 讓其他頁面呼叫。實作的方式跟 command method 沒有什麼差別只是 annotation 換成 `@GlobalCommand`。
```java
public class MyTodoListViewModel {
	private String title = "我的待辦清單";
	private String currentPage = "todoList.zul";

	@GlobalCommand @NotifyChange("currentPage")
	public void navigate(@BindingParam("page") String page){
		currentPage = page;
	}
...
}
```


# 列表到編輯

點選一個事項會發出 onSelect 事件，因此我將 global command 綁定在該事件，並將要切換的目的 （page.zul）頁面作為參數傳入。
```xml
<listbox model="@init(vm.todoListModel)"
onSelect="@global-command('navigate', page='todo.zul')" vflex="1">
```

如此一來，每次選一個項目，畫面就切換成編輯畫面了。


# ViewModel 間的溝通 - scoped attribute
雖然畫面是切過去了，但是要被編輯的待辦事項卻沒有傳過去，所以編輯畫面沒有資料可編。這時我們可運用另一種溝通的方式：將資料存入某個 scope 中。因為兩個 ViewModel 都屬於同一個 desktop，存入 desktop 範圍是一個好選擇。雖然兩個 ViewModel 也同時屬於同一個 session，但謹記分享資料的原則是選擇「最小夠用的範圍」。這可以避免堆積無用的資料垃圾在記憶體中，減少伺服器的負擔，

```java
public class TodoListViewModel {
    ...
    @Command
    public void edit(@ContextParam(ContextType.DESKTOP) Desktop desktop) {
        desktop.setAttribute("todo", selectedTodo); //存入 desktop 讓編輯頁面存取
    }
...
}
```

在編輯頁面的 ViewModel 中，就可以在初始化過程將 desktop attribute 取出來，並存在自己的 member field 中。
```java
public class TodoEditorViewModel{
    ...
    @Init
    public void init(@ScopeParam("todo")Todo todo){
        this.todo = todo;
    }
    ...
}
```
* `@ScopeParam` 會從最小範圍的物件 (execution) 開始搜尋你指定 key 的 attribute，並依次往較大的範圍物件搜尋 (Desktop, Session, Application)，如果找到就會回傳。因為我之前用 todo 這個 key 存入 Desktop，因此現在需指定同一個 key 抓取 attribute。


# 編輯到列表
修改後，可不儲存就「返回」也可以「儲存」，兩個 button 都呼叫 `@global-command` 就可以喚回列表畫面。

todo.zul
```xml
...
<hlayout>
    <button onClick="@global-command('navigate', page='todoList.zul')" label="返回" />
    <button onClick="@command('save') @global-command('navigate', page='todoList.zul')" label="儲存" />
</hlayout>
```
`@command` 跟 `@global-command` 可以同時使用，而 `@command` 會先被執行。
