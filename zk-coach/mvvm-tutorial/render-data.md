# 顯示單一物件
畫面繪製好後，接下來就是將伺服器的資料顯示到畫面上，MVVM 模式下，我們要利用 data binding 將 ViewModel 的屬性（property）連結到元件的屬性上，運行後，ZK 就會將 ViewModel 中的資料自動設到元件上。

我們的應用程式 ViewModel 如下：
```java
public class MyTodoListViewModel {
	private String title = "我的待辦清單";

	public String getTitle() {
		return title;
	}
}
```

你可以看到 ViewModel 就是一個 POJO，**不需繼承特定父類別**，也**不需實作特定介面**，其中包含我們要顯示在頁面上的資料：視窗標題：`String title`，為求簡單，這裡是固定值，當然它可以是從資料庫或其它來來源查詢得來。

## 套用 ViewModel
為了讓 ZK 可以從 ViewModel 中取得資料，你必須提供符合 Java bean 命名規範的 getter：getTitle()，首字要大寫。然後我要將其套用到頁面上的元件：
```xml
<window viewModel="@id('vm') @init('zkcoach.mvvm.tutorial.controller.MyTodoListViewModel')" ...>
</window>
```
用 viewModel 屬性來套用 MyTodoListViewModel 到 Window 上，其所包含的子元件也就都可以存取到該 ViewModel。其中 `@id` 用來宣告 ViewModel 變數名稱，寫 data binding 時要用這變數來存取 ViewModel 中的屬性(property)。`@init` 用來告訴 ZK 怎麼產生 ViewModel 物件，如果給 FQCN 字串，那 ZK 就依此產生物件，或是給一個變數，ZK 會把解析到的的物件作為 ViewModel

## 將 ViewModel 載入到元件
在 MVVM 中是透過寫在 zul 的 data binding 敘述將元件的屬性綁定到 ViewModel 中的屬性來載入資料的，如：
![]({{site.baseurl}}/assets/caption-title.png)

ZK 提供兩種語法來載入資料：
### `@init` -- 一次性載入
`@init` 若寫在元件屬性上則代表「一次性載入」，ZK 將 ViewModel 中的屬性（title）讀出並設到元件屬性上（如Caption 的 label），只會在第一次頁面載入的時候設一次，之後若 title 改變了， ZK 也不會再將其值設定到 Caption 上，這就是一次性的意思 -- 只載入一次。適合用在那些不會隨著使用者互動而改變的 ViewModel 屬性上。

### `@load` -- 載入並同步
將 ViewModel 中的屬性（title）讀出並設到元件屬性上，並監控該屬性變化且適時同步到元件上。ZK 會在伺服器上建立追蹤節點（tracking node），它會記錄元件屬性與 ViewModel 屬性的綁定關係，並在必要的時候同步兩者的資料。

因此可以看出，使用 `@load` 雖然可以保持同步，但是追蹤節點需耗用額外的記憶體，若是該 ViewModel 屬性不會變化，可以採用 `@init` 減少記憶體消耗。

本例中，title 並不改變，因此我們用 `@init` 即可：
**/tutorial/render/myTodoList.zul**
```xml
<window ...>
	<caption ... label="@init(vm.title)" />
</window>
```

# 顯示集合物件
有些 ViewModel 中的物件是集合物件如 Java `List`, `Map`, `Set`，這些物件綁定到單一元件上的屬性沒有意義，通常需要綁定到對等數量的元件上。接下來我們就用 Listbox 來顯示 ViewModel 中的「待辦事項」集合物件。

![]({{site.baseurl}}/assets/todoList.png)

首先定義資料類別：

```java
/**
 * 待辦事項
 */
public class Todo implements Serializable, Cloneable {
    private static final long serialVersionUID = 1L;

    private Integer id;
    private String subject; //主題
    private String description;
    private Integer priority;
    private Date date;
    private boolean complete;

    //每個屬性的 getter 與 setter 方法
}
```
以上是我們的資料物件「待辦事項」，每個屬性**必須要有對應的 getter、setter 方法**，你才能用 data binding 語法來存取它的值。


## 多層架構
雖然這個應用很小，我們還是以多層架構（multi-tier）來實作，這是 web application 常見的架構，也就是將整個應用程式依照功能分成多層，每一層都有其主要的功能，且由一個或多個類別來實作。一般會分成下列幾層：

畫面層（View）
控制器層（Controller）
業務（服務）層（Service）
儲存層（Persistence）

各層間的依賴關係是由上往下。畫面層負責繪製畫面、顯示資料、提供介面與使用者互動、向後端發出請求，在本範例中由 ZK 元件扮演。控制器層負責處理畫面層的請求，並呼叫業務層來滿足請求，本例中是由 ViewModel 扮演。業務（服務）層就是實作應用程式主要邏輯的地方，例如登入、搜尋、訂房等，本例中就是 `TodoListService`。儲存層就是處理資料操作（CRUD），直接存取實際儲存裝置如資料庫的一層，一般會以 JPA, Hibernate 這類 ORM 框架來實作，為求簡單，本範例省略未實作這一層。

TodoListService 就是實作應用程式邏輯的類別，裡面實作了可以讓我們操作整個待辦事項清單的方法：
```java
public class TodoListService {
    public synchronized List<Todo> getTodoList() {...}
    public synchronized Todo getTodo(Integer id) {...}
    public synchronized Todo saveTodo(Todo todo) {...}
    public synchronized Todo updateTodo(Todo todo) {...}
    public synchronized void deleteTodo(Todo todo) {...}
}    
```
看到這些方法好像只跟新增查改有關，你會覺得這個 class 看起來好像在做儲存層，不過那是因為這個應用的邏輯很簡單才會如此，如果應用程式功能更多一些，就會有一些更複雜的應用邏輯，而不只是新增刪除這種簡單的動作。


## 資料型元件
ZK 中有些元件是設計來顯示大量資料的，如：`Listbox`、`Grid`，這類元件跟另外兩種物件分工合作來顯示大量資料。一個是 model 物件，負責儲存資料物件（Todo）與選擇狀態，可以用集合物件（List, Set, Map）或 ZK ListModel 來實作，這物件完全不依賴元件，也就是同樣的 Model 可以賦予 Listbox 也可以賦予 Grid；另一個是 Renderer（繪製器），負責依據 model 內容以對應的子元件繪製出來，例如 Listbox renderer 就會產生 Listitem。ListModel 中若有資料變動會發出事件通知元件更新，若是元件收到使用者選擇某筆資料，也會通知 ListModel 更新選擇狀態。三者之間的設計符合「單一責任原則」，各自有明確而單一的「責任」。

**TODO 3 者關係圖**

Listbox 和 Grid 都有一個 `model` 屬性可接受 ListModel 物件。在 MVVM 方法下，我們通常不自行實作 renderer，而是定義「模板」(`<template>`)，而元件內建的 renderer 就會依此模板來產生子元件。


## 優化的 ZK ListModel
為了儲存從 TodoListService 查詢到的所有「待辦事項」，我在 ViewModel 中宣告一個集合物件：`ListModelList`。
```java
public class TodoListViewModel{

    //畫面上所需要呈現或儲存的資料(view state)
    private ListModelList<Todo> todoListModel;

}
```

你可能注意到為何不用 Java 內建的 `List` 類別，而是用 ZK 內建的 `ListModelList`，那是因為這個類別有特別針對 ZK 元件優化，只要你變更其內容，如呼叫 `add()`、`remove()`，它會自動通知所綁定的元件告知變動的範圍，並且在瀏覽器上只繪製變動範圍的資料。例如 `ListModelList` 內若有 100 個物件，你若增加了一個物件，它會通知對應的 ZK 元件只畫那新增的一個物件，而不會 100 個都重繪。如果你使用得是 Java 內建 List 類別，那 ZK 元件無從得知變動範圍，因此元件只能每次變動後都把所有資料都重新繪製一次，效能較差。

## ViewModel 初始化方法
ZK 會在載入 zul 的過程自動產生你指定的 ViewModel 物件，然後就呼叫你標註 `@Init` 的方法，你可以用來初始化 ViewModel 內的屬性，當然跟 ZK 無關的變數，也可在建構子內初始化。我們透過服務層物件（`todoListService`）查詢所有儲存的「代辦事項」並初始化清單：`todoListModel`。

```java
public class TodoListViewModel{

    //應用程式邏輯
    private TodoListService todoListService = new TodoListService();

    //畫面上所需要呈現或儲存的資料(view state)
    private ListModelList<Todo> todoListModel;

    @Init // 初始化
    public void init() {
        //從後端抓資料
        List<Todo> todoList = todoListService.getTodoList();
        //建議使用 ListModelList 可以優化繪製效率，避免每次都重新繪製所有資料
        todoListModel = new ListModelList<Todo>(todoList);
    }
}
```


## 綁定 Model 物件

接下來就是要將 Model 物件綁定到 `Listbox`。

```xml
<vlayout hflex="1" vflex="1"
	viewModel="@id('vm') @init('zkcoach.mvvm.tutorial.controller.TodoListViewModel')">
	<listbox model="@init(vm.todoListModel)" vflex="1">
	</listbox>
</vlayout>
```
因為 todoListModel 物件參照本身並不會改變，因此我們可以用 `@init` 即可。之後呼叫 `add()`、`remove()` 時 ListModelList 會自動發事件通知 Listbox 去更新變動的內容。

接下來加上 3 個欄位標頭：
```xml
	<listbox model="@init(vm.todoListModel)" vflex="1">
		<listhead>
			<listheader hflex="min" /> <!--完成勾選框 -->
			<listheader />             <!--待辦事項主題 -->
			<listheader hflex="min" /> <!--刪除按鈕 -->
		</listhead>
	</listbox>
```
一般欄位會加上文字，不過本範例中主要是用來限制欄位的寬度。

在綁定集合資料物件後，就要決定怎麼繪製每個 `Todo` 物件，這裡我們採用範本（`<template>`）的做法，也就是定義一個 zul 片段作為範本，然後 ZK 就會依據這個範本來繪製集合物件中的每一個 `Todo`。

**/tutorial/render/myTodoList.zul**
```xml
	<listbox model="@init(vm.todoListModel)" vflex="1">
        ...
		<template name="model">
			<listitem
				sclass="@init(each.complete?'complete-todo':'')">
				<listcell>
					<checkbox checked="@init(each.complete)"/>
				</listcell>
				<listcell>
					<label value="@init(each.subject)" />
				</listcell>
				<listcell>
					<button iconSclass="z-icon-times-circle" width="40px"
						style="font-size:18px;color:#AA5585" />
				</listcell>
			</listitem>
		</template>
	</listbox>
```

而這個範本的名稱一定要是 `model`，這樣 Listbox 就會知道這範本是用來繪製 model 的。

範本中只能放一個 `<listitem>`，因為我們有 3 個欄位，因此其中要放 3 個 `<listcell>`，依序對到每一欄。

將 Todo 資料繪製出來的關鍵字就是 `each`，這個變數代表的就是 model 中的物件，以本例來說就是 Todo。我們可以用 EL 語法來存取物件的屬性，例如：
`each.complete` （呼叫 `todo.isComplete()`）
`each.subject` （呼叫 `todo.getSubject()`）

透過 each 我們就可將物件綁到適當的元件屬性上將其繪製出來，例如 `<label value="@init(each.subject)" />` 就會將「待辦事項主題」以 Label 繪製出來。而我們希望使用者可以勾選待辦事項完成與否，所以將其綁到 `<checkbox>` 的 checked 屬性上，如果 `each.complete` 為 true，則 Checkbox 就會呈現勾選狀態。

## 用 EL 實做繪製邏輯
有時候簡單的頁面變化可以用 EL 來做到，不需要寫程式，例如我想讓勾選待辦事項時，文字有刪去線效果：
![]({{site.baseurl}}/assets/completeTodo.png)

這可以輕易地用 CSS 做到，所以我們先設計好樣式：
```css
.complete-todo .z-label{
	text-decoration: line-through;
}
```
ZK data binding 語法括號中可以填入任何合法的 Expression Language 敘述，語法格式是這樣（以 `@init` 為例）：
`@init(EL-expression)`

因此我們可以利用三元運算子來決定是否套用該 class：
`<listitem sclass="@init(each.complete?'complete-todo':'')">`


如果你的樣式判斷條件很複雜很難用 EL 表達怎麼辦呢？可以用 Java 在 ViewModel 中的 getter 實作：
```java
public boolean isGranted(){
    // 實做複雜的判斷運算
}
```

然後就可用 EL 取得判斷後的結果：
```xml
<button disabled="@init(not vm.granted)">
```


## 慎選 data binding 語法
在模板內，我多半用 `@init` 因為 Todo 內容不會改變，所以沒必要追蹤其變化，因此用 @init 而不用 @load 可以避免 ZK 在伺服器上建立許多不必要的追蹤節點，節省伺服器的記憶體。
