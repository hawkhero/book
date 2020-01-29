# 常見問題

## 什麼是 Desktop
ZK 專有名詞。簡單說，一個瀏覽器頁籤 (tab) 就對應到一個 Desktop。當你訪問某個 zul 頁面時，ZK 就會在伺服器建立一個 Desktop 物件，裡面存有這個頁面的 UI 元件樹，每次重載頁面 (reload) 則會新建一個 Desktop。

### 為何需要有 Desktop
Application server 只支援 session scope，瀏覽器上兩個 tab 皆屬於同一個 session，但是不同的 tab 上的兩個 zk javascript widget 物件卻不是互通的，所以伺服器上需要有一個比 session 小的範圍來區別這兩個 tab。

例如我開了兩個 tab 都連到 abc.zul，兩個畫面上各有一個按鈕，伺服器上也有兩個 Button 物件，如果沒有 Desktop，兩個物件都屬於同一個 session，我點了某一個按鈕，瀏覽器將不知道要將事件發到哪一個 Button 物件上。或是我呼叫 Button 的 method，伺服器也不知道應該要叫哪一個 tab 的 javascript widget 更新畫面。


## 什麼是 execution
你可以把他當成 `HttpServletRequest` wrapper 來運用，因為他封裝了從瀏覽器送來的 `HttpServletRequest`。它的 `setAttribute()`, `getAtribute()` 都是直接呼叫 `HttpServletRequest` 上的。


## 什麼是 ZK AU
AU 代表 Asynchronous Update，就是 ZK 所發出的非同步 AJAX 請求。當你跟 ZK 元件互動而產生事件時 (e.g. onClick)，ZK 框架就會透過 AU 將該事件的相關資料傳到伺服器端，並呼叫對應的事件傾聽器，通常是你寫的程式。可以透過 developer tool 觀察得到。

## 如何在 zul 中內嵌 HTML
如果是靜態內容，可透過[宣告 namespace](https://www.zkoss.org/wiki/ZUML_Reference/ZUML/Namespaces/Native) 來插入 HTML tag 到 zul 中。如果是動態產生的 HTML，例如從資料庫產生，則可以使用 [`<html>`](https://www.zkoss.org/wiki/ZK%20Component%20Reference/Essential%20Components/Html) 元件。


## 為何建議用`<apply>` 取代 `<include>`
比起 `<include>` 用 `<apply>` 有以下好處：
* `<apply>` 不會在畫面上產生額外的 DOM 元素

 `<include>` 會產生 `<div>`。
* `<apply>` 本身不會佔用記憶體 (跟 EL 一起使用時)

 `<include>` 本身是元件因此會佔用記憶體。
* `<apply>` 不會產生 [ID space](https://www.zkoss.org/wiki/ZK%20Developer's%20Reference/UI%20Composing/ID%20Space)

 `<include>` 會產生一個 ID space。
* `<apply>` 可用 template 名稱與 zul 路徑來插入一個 zul 片段

 `<include>` 只接受由路徑插入檔案。
* 如果用 `@load` 傳入的參數改變了，`<apply>` 會自動重繪插入的頁面

`<include>` 只有在 `src` 改變時才能重繪插入畫面，所以如果要重繪插入的頁面，需要將 `src` 改成 `null` 再改成原路徑。


## 如何取得 `HttpServletRequest`
`(HttpServletRequest)Executions.getCurrent().getNativeRequest()`


## 如何取得 `HttpSession`
`(HttpSession)Sessions.getCurrent().getNativeSession()`


## ZK 可以跟 MySql, Oracle Database...（等各類資料庫）一起使用嗎
可以。ZK 並不倚賴任何資料庫，所以任何可以透過 Java 連接的資料庫都可以用。


## 如何客制整套元件佈景主題 (Theme)
ZK 8 之後建議以 ZK Theme Template 為基礎來客制元件佈景主題，[這是一個基於 git 的方法](https://www.zkoss.org/wiki/Small_Talks/2016/May/New_Approach_for_Building_Custom_ZK_Theme)，你從 [ZK Theme Template github repository](https://github.com/zkoss/zkThemeTemplate) fork 出自己的 repository，然後基於一個最接近自己預期的 theme 修改 LESS 檔。這樣的好處是如果 ZK 升級或修 bug，你的客制 theme 可以輕易透過 git 來合併變化。


## 怎麼預覽 zul 繪製的結果
要能快速執行你的 Maven 專案，我推薦使用 maven jetty plugin:

```xml
	<build>
		<plugins>
			<plugin>
				<groupId>org.mortbay.jetty</groupId>
				<artifactId>maven-jetty-plugin</artifactId>
				<version>${jetty.version}</version>
			</plugin>
		</plugins>
	</build>
```
優點是啟動快速、無須安裝。


## 常見錯誤訊息

### `java.lang.IllegalStateException: Access denied: component, <Listcell z_27_b53>, belongs to another desktop: [Desktop g272]`
使用 MVC pattern 可能會看過這個錯誤，原因為：
* 你在 composer 中宣告一個 ZK 元件變數為 static，而且還呼叫其 setter
解決方法：將該變數改成非 static
* 你將一個 ZK 元件透過 event queue 傳到另一個 Desktop，而且還呼叫其 setter

**解決方法**

不要傳元件參照，傳資料即可


<span style="font-size: 34px;color:skyblue">
  找不到自己的想知道的基本概念？歡迎 Email 到 hawkhero at gmail
</span>
