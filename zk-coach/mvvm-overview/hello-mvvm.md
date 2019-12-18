# 一個簡單的例子

這個簡單的頁面中只顯示一個 hello。
```xml
	<div>
		<label value="hello"/>
	</div>
```
## 建立 ViewModel
但是進一步我們希望資料可以來自 server，而不是固定值，不然就跟靜態 HTML 沒有兩樣。首先新建一個 ViewModel，因為畫面上只有一個狀態，就是問候語，所以我們就宣告一個 String 來儲存這個狀態。

```java
public class HelloVM {

	private String hello = "hello";

	public String getHello() {
		return hello;
	}
}
```
這裡我們寫一個 getter `getHello()` 也是 ZK 的要求，因為 ZK 只能透過 getter 來從 ViewModel 中來取得資料。

## 套用 ViewModel
接下來我們要將控制器 ViewModel 套用在畫面上，讓畫面上的元件在 ViewModel 的控制之下。我們要在 `viewModel` 屬性中以 data binding 語法來指定 ViewModel:

```xml
	<div viewModel="@id('vm')@init('zkcoach.mvvm.HelloVM')">
		<label value="hello"/>
	</div>
```

`@id` 指定 ViewModel 的變數名稱
`@init` 指定 ViewModel 的完整 class 名稱 (FQCN)字串，注意要加單引號

這樣，該 `<div>` 及其下的子元件都在 `HelloVM` 的控制之下。


## 載入資料
在 MVVM 模式下是透過 data binding 來連結頁面與 ViewModel 中的資料。如果要將資料從 ViewModel 中載入到頁面上，要使用 `@load`

```xml
	<div viewModel="@id('vm')@init('zkcoach.mvvm.HelloVM')">
		<label value="@load(vm.hello)"/>
	</div>
```
重點是這一句：
    `value = @load(vm.hello)`
它意義是叫 ZK 將 ViewModel 中的 hello 屬性值存到 Textbox 的 value 中。

`vm.hello` 是一個 EL (Expression Language) 表達式，代表是 ViewModel 中的 hello 屬性。ZK 會透過 Java reflection 機制呼叫該屬性的對應 getter，也就是 `getHello()`，來取得 hello 屬性的值，然後存入 Textbox 的 value 屬性中 （當然 ZK 也是透過呼叫 `Textbox.setValue()` 來存入值）。

如此就實作了載入資料，ZK 自動會在畫面建立好之後將 ViewModel 中的資料載入並顯示在畫面上。

## 加上動作
接下來，我們加上一點互動的功能：按下按鈕之後，頁面上的 hello 會變成 hello world。這個互動就是在觸發一個 **命令（應用程式邏輯）**，而不只是單純資料存取，所以我們要在 ViewModel 中用一個方法, `helloWorld()` 來實作。

```java
public class HelloVM {

	private String hello = "hello";

	public String getHello() {
		return hello;
	}

	@Command @NotifyChange("hello")
	public void helloWorld(){
		hello = "hello world";
	}

}
```
但是 ZK 並不知道哪一個 method 才是可以被頁面呼叫的應用程式邏輯，因此你需要在該方法上加上 @Command。而命令被執行過後，如果有屬性改變了，ZK 也不知道，你必須要通知 ZK，因此要加上 @NotifyChange("hello")，代表這個 ViewModel 中的 hello 被改變了，ZK 就會在執行命令之後，再次呼叫 getHello() 去取得更新的值並顯示在頁面上。

## 繫結命令
實作好命令之後，還必須要跟元件的 event 繫結，才能夠讓使用者透過頁面操作來觸發該命令，我們將其跟 Button 的 onClick 繫結：

```xml
<div viewModel="@id('vm')@init('zkcoach.mvvm.HelloVM')">
    <label value="@load(vm.hello)"/>
    <button label="hello world" onClick="@command('helloWorld')"/>
</div>
```
注意 `@command` 是全小寫，其中的字串是命令的名稱，預設為方法的名字，因為 ViewModel 中的方法為 `helloWorld()`，因此命令名稱就是 'helloWorld'。寫完後，只要你點擊該按鈕，頁面上的文字就會變成 hello world。
