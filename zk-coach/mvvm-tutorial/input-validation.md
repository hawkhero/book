# 驗證輸入：待辦事項主題
照目前為止的實作，如果還沒輸入待辦事項主題就不小心按到 enter 鍵，就會產生空白主題的待辦事項，我決定要防止這種不合理的情況，這時就可以使用驗證綁定語法： [`@validator`](http://books.zkoss.org/zk-mvvm-book/8.0/data_binding/validator.html)，來將一個驗證器綁定到元件上。而我們需要自行實作驗證器 (Validator)，於是我們就實作一個檢查字串是否為空的驗證器如下：

```java
public class SubjectValidator extends AbstractValidator {

	@Override
	public void validate(ValidationContext ctx) {
		//取得驗證目標值
		String subject = (String)ctx.getProperty().getValue();

		if(Strings.isBlank(subject)){
			Clients.showNotification("必須輸入待辦內容喔！",ctx.getBindContext().getComponent());
			//設定驗證結果為不合法，因此：
			//使用者輸入不會存入 ViewModel 中
			//不會執行相關的 command
			ctx.setInvalid();
		}
	}

}
```
* 雖然可以實作 `Validator`介面，但是繼承 ZK 提供的 `AbstractValidator` 後覆寫 `validate()` 是較簡單的作法
* 上面的範例用 showNotification() 顯示錯誤訊息，也可以顯示錯誤訊息在其他元件上，請參考 [Validator](http://books.zkoss.org/zk-mvvm-book/8.0/data_binding/validator.html)中 validation message holder 的用法。

接下來將 `@validator` 跟 `@save` 合併使用，如此一來，當要儲存輸入到 vm.subject 時，會先呼叫驗證器驗證，如果失敗就不會存值入 ViewModel，驗證成功才會存入，於是我改寫成：
```xml
<textbox value="@save(vm.subject, before='addTodo')@validator('zkcoach.mvvm.tutorial.controller.SubjectValidator')" onOK="@command('addTodo')"
			hflex="1" placeholder="該做啥?" />
```

## 條件式資料綁定 （conditional data binding)
你會注意到 `@save` 中加上了 `before='addTodo'`，這稱為 [conditional binding](http://books.zkoss.org/zk-mvvm-book/8.0/data_binding/property_binding.html#conditional-binding)，用來指定資料存取的時機點。Textbox 預設的 `@save` 儲存時機是 `onChange` 事件發生時，會將 Textbox 的植存回 ViewModel，如果加上 `before='addTodo'` 代表我們指定儲存的時機點為**執行 addTodo 命令之前**。因此當我按下 enter 鍵或點加號按鈕時，就會存值，也就會觸發驗證器。
