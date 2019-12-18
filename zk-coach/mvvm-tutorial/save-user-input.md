# 變更完成狀態
在待辦事項清單上，原本只使用 `@init` 顯示資料，但現在我們要讓使用者能勾選 Checkbox 來改變完成狀態，這時我們就要用 `＠save` 來**將元件屬性綁定到 ViewModel 屬性上**，這個語法會告訴 ZK 在使用者輸入完之後，將瀏覽器的資料傳回伺服器上，並透過 setter 存回 ViewModel中，因此我們要將語法改成：
```xml
<listcell>
	<checkbox checked="@init(each.complete)@save(each.complete)" />
</listcell>
```

因為既要讀取又要寫入同一個 ViewModel 屬性，因此我們可以改用 `@bind`，它的意義等於 `@load` + `@save`，這樣我們就不用寫兩次：
```xml
<listcell>
	<checkbox checked="@bind(each.complete)" />
</listcell>
```


# 新增待辦事項
同樣道理，我們要做一個輸入事項名稱就能新增 Todo 的功能，也就是要將使用者輸入 Textbox 的內容存回 ViewModel 的 `subject`，也是要利用 `@save`：
```xml
<textbox value="@save(vm.subject)"
			hflex="1" placeholder="該做啥?" />
```
* placeholder 可以在尚未輸入前，讓 Textbox 顯示提示文字，讓使用者知道這一欄要輸入的內容


接下來放一個「加號」按鈕來新增事項。
```xml
<textbox value="@save(vm.subject, before='addTodo')"
    hflex="1" placeholder="該做啥?" />
<button onClick="@command('addTodo')" hflex="min"
    iconSclass="z-icon-plus-circle" style="color:#D4EE9F"/>

```

並在 ViewModel 中實作對應的功能：
```java
public class TodoListViewModel {
    ...
    @Command //@Command 用來宣告此方法為命令 (command)
    @NotifyChange("subject") //@NotifyChange 通知 ZK 更新哪些 property
    public void addTodo() {
        selectedTodo = todoListService.saveTodo(new Todo(subject));
        //更新頁面資料，無需用 @NotifyChange 通知 ZK 我們改變了 todoListModel，它自行會通知元件繪製新增的一筆
        todoListModel.add(selectedTodo);
        //清空輸入，方便輸入下一個事項
        subject = "";
    }
    ...
}
```

## 更新畫面
在 MVVM 模式中，ZK 雖然幫你呼叫 command method，但無法得知你的 method 改變了哪些 ViewModel 中的屬性，因此你必須要主動透過 [`@NotifyChange`](http://books.zkoss.org/zk-mvvm-book/8.0/viewmodel/notification.html) 告知，根據改變的屬性個數不同，語法各自為：

一個屬性：
`@NotifyChange("oneProperty")`

多個屬性：
`@NotifyChange({"property01", "property02"})`

所有 ViewModel 中的屬性：
`@NotifyChange("*")`

以上面 `addTodo()` 來說，我們清空了 `subject`，因此需要告知 ZK 幫我們更新所有綁定 `subject` 的元件的狀態，於是我們就要寫：

`@NotifyChange("subject")`

當然你同時要確保 ViewModel 中有 `getSubject()`。

### 內建 ListModel / TreeModel 免通知
先前提過，ZK 內建的 model 物件如 `ListModelList` 會在其內容變動時自動發內部事件給綁定的元件，因此不需要透過 `@NotifyChange` 來通知，這算是唯一的特例。


## 重用 command binding
為求讓使用者方便，輸入完按下 enter 鍵就可以新增，我們可以也在 onOK 上用 command binding，而且完全不需要修改 ViewModel。（在 Textbox  中按下 enter 就會產生 onOK 事件）
```xml
<textbox value="@save(vm.subject)" onOK="@command('addTodo')"
    hflex="1" placeholder="該做啥?" />
<button onClick="@command('addTodo')" hflex="min"
    iconSclass="z-icon-plus-circle" style="color:#D4EE9F"/>
```
這也是 MVVM 模式的優勢之一，command 就像 ViewModel 提供給 zul 的介面，只要透過 command binding 呼叫，ViewModel 不需要知道元件是哪一個。
