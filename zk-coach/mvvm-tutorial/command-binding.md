# 刪除待辦事項
在每個待辦事項後都有一個刪除按鈕，我們希望按下去可以刪除該項，這時就要用到 command binding 來處理按鈕事件。


## Command Binding 就是事件傾聽器 (Event Listener)
這個語法是用來 **將元件的事件屬性（如 onClick）綁定到 ViewModel 中的方法 (method)**，之後在該事件發生時（如按下按鈕），ZK 就呼叫綁定的方法，該方法通常會實作你的應用程式邏輯，如新增、刪除、搜尋等，其實就跟註冊一個事件傾聽器的概念是一樣的。

首先我們必須在 ViewModel 中定義一個方法，並且加上`@Command`，這樣 ZK 才能辨識出這個 method 是一個 command method，可以被 zul 上的 command binging 呼叫：

```java
public class TodoListViewModel {
    @Command
    public void deleteTodo() {
        System.out.println("deleteTodo");
    }
}
```

之後就可以在 zul 上綁定該個 command，預設的 command 名稱就是 method 名稱：`deleteTodo`。

```xml
    <button onClick="@command('deleteTodo')"
        iconSclass="z-icon-times-circle" style="color:#FFAAAA"/>
```
這樣一來，你按下按鈕，伺服器就會印出 "deleteTodo"。

不過這樣還無法完成我們要的功能，為了要刪除該列顯示的事項，我們將該 Todo 物件透過 command binding 傳進 command method 中如下：

`/tutorial/command/myTodoList.zul`
```xml
<listcell>
    <button onClick="@command('deleteTodo',todo=each)"
        iconSclass="z-icon-times-circle" style="color:#FFAAAA"/>
</listcell>
```
command binding 中可以附加一到多個參數，參數之間用逗號分隔，每一個參數的語法是 key=EL-expression， key 是你自訂的名稱，等號後面是 EL 表達式，因此可以傳入任何可以用 EL 存取的物件，在這裡 `each` 這個保留字代表每一個 Listitem 顯示的 Todo。

在 command method 上我們也要宣告對應的參數，ZK 才能將參數傳入：

```java
@Command
public void deleteTodo(@BindingParam("todo") Todo todo) {
    //刪除後端資料
    todoListService.deleteTodo(todo);
    //刪除畫面上的資料
    todoListModel.remove(todo);
}
```
`@BindingParam("todo")` 是用來告訴 ZK，這個參數是要從 command binding 中傳過來，其中 "todo"對應到我們在 command binding 上參數的 key。而這個 method 中就要實作應用程式邏輯，所以我們刪除資料庫中跟刪除元件 data model 中對應的 Todo。

這樣我們就完成了「刪除」的實作。
