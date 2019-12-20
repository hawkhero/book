# 科學化除錯方法
Steve McConnel 所著的[軟體建構之道(Code Complete)](https://g.co/kgs/f4upi7) 中, 第 23 章提到一個科學化的除錯方法，步驟如下：

1. 可以穩定產生錯誤
2. 找出錯誤的根因 <br/>
    a. 收集造成此錯誤的相關資訊<br/>
    b. 分析找到的資料並給出一個對根因的假設<br/>
    c. 證明或反證該假設<br/>
3. 修正該錯誤
4. 測試該修正
5. 尋找其他同樣的錯誤

我就以這套步驟當作基本原則來說明怎麼在使用 ZK 時除錯。

# 1.可以穩定產生錯誤
這階段目標是找到**能產生該錯誤的明確、固定、最少步驟**。

如果你的步驟不明確、不固定、或步驟很多，這代表似乎每個元件都可能造成問題，那問題範圍就會牽涉太廣，難以找到根因。如果能夠除去一個步驟，仍然能夠重現該錯誤，那就代表該步驟與錯誤無關，可以移除。因此經由反覆測試來減少步驟，就能縮小問題範圍，可以讓你接下來比較容易找到根因。


# 2.找出錯誤的根因
## a. 收集造成此錯誤的相關資訊
### 伺服器 log 中的（錯誤）訊息
很多人會忽略伺服器印出的 log 內容，但其實它常常明確的表示出問題的根因，如顯示 exception stack trace，我們就可以根據呼叫的 stack 去找出錯誤發生的程式來推測出錯誤根因。

### 瀏覽器上的錯誤訊息
主流的桌上瀏覽器按 F12 都可以開啟開發者工具，其中 Console 頁會顯示錯誤訊息，這通常也能幫助我們找到根因。
以下是 Chrome 中的開發者工具：

![]({{site.baseurl}}/assets/consoleTab.png)

### 常見錯誤訊息： java.lang.IllegalStateException: Access denied: component, <Listcell z_27_b53>, belongs to another desktop: [Desktop g272]
使用 MVC pattern 可能會看過這個錯誤，原因為：
* 你在 composer 中宣告一個 ZK 元件變數為 static，而且還呼叫其 setter
解決方法：將該變數改成非 static
* 你將一個 ZK 元件透過 event queue 傳到另一個 Desktop，而且還呼叫其 setter
解決方法：不要傳元件參照，要傳資料即可



### 縮小問題範圍
為了找到錯誤的根因，縮小問題的範圍將會非常有幫助，因為問題牽涉的範圍越大，潛在造成問題的因素就越多，必須耗費非常多時間一一檢視這範圍裡的每個因素來找出根因，因此縮小問題範圍可以增進除錯的效率。你可用以下作法縮小問題：

* 移除（不相關的）元件。
通常 zul 上會有很多元件，但其實真正與錯誤有關的元件不多，因此將其他無關的元件從 zul 中移除可以讓你專注在真正的問題上。
* 減少重製步驟
如果原本要 3 個步驟能重現 bug，若是少做某一個步驟仍然能重現該 bug，就代表那一步跟該 bug 無關，可以省略。
* 移除（不相關的）程式碼
如果一個按鈕按下去，相對應的 event listener 有 500 行，那就得在這 500 行中找問題。如果能移除不相關的程式碼，減少成 100 行，就只需要在這 100 行中找出根因，就可以減少除錯時間。


## b. 分析找到的資料並給出一個對根因的假設
在取得資料之後，就要針對這些資料提出一個假設。如果對 ZK 的架構越了解，就越能夠提出越正確的假設。

讓我們概略地看一次架構：

![]({{site.baseurl}}/assets/architecture.png)

1. 使用者點擊按鈕，觸發 DOM 事件
2. ZK widget 發出事件通知 client engine
3. client engine 將事件透過 AJAX 傳遞到伺服器
4. 伺服器收到事件發給 ZK 元件
5. 元件處理完事件，發給系統
6. 系統呼叫該事件的傾聽器 (event listener)
7. 事件的傾聽器執行所實作的應用程式邏輯，可能包含存取 ZK 元件、存取資料庫、或是再發出事件
8. 當所有的事件都被處理完之後，元件被變更的部分都會被收集起來並轉換成對 ZK widget 的命令
9. 將命令透過 HTTP response 傳回前端
10. ZK client engine 收到 response 之後執行其中的命令，這些命令就會更改瀏覽器的畫面

以上步驟可以簡化為：

1. 瀏覽器發出 ZK AJAX 請求 （或稱 asynchrounous update，簡稱 AU)
2. 伺服器事件傾聽器執行
3. 瀏覽器收到 AJAX 回應 （AU response）

每次當你跟元件互動但結果不如預期的時候，就可以依次檢查以上三個步驟有無正確執行:

1. 是否發出預期的 ZK AU
2. 傾聽器是否執行
3. AU 回應內容是否如預期


## c. 證明或反證該假設
當你提出一個對問題根因的假設之後，如果能夠證明假設成立，恭喜你，已經發現問題根源，可以開始思考怎麼解決。如果假設無法證明或可反證它，那就代表先前的假設錯誤，你要回到前一步，重新提出另一個假設。如此反覆到找到根因為止。以下提供幾個方法幫助你檢查 ZK 運作是否正常來驗證你的假設：

### 是否發出預期的 ZK AU
你可以透過 developer tool 來觀察，以 Chrome 為例，按 F12 打開，選 Network 頁，當你操作某個 ZK 元件而發出 event 時，會有一個路徑為 zkau 的請求發出如下：

![]({{site.baseurl}}/assets/auEvent.png)

確認 AU 的確有發出之後，就該檢查是否是你預期的事件發出。觀察 Form Data 可得知事件的細節，其中包含以下欄位：

* `dtid`, Desktop id
* `cmd`, 事件名稱
* `uuid`, 發生該事件的 DOM element id
* `data`, 事件相關資料，每個事件會帶不同的資料

因此對應使用者與元件互動的行為，就應產生對應的事件，例如按了按鈕就會發出 `onClick`、打開 popup 就發出 `onOepn`。



### 傾聽器是否執行

透過 IDE 設定中斷點，是最常用的方法。

所以你必須要能輕易的在你的 IDE 中執行你的專案，我推薦使用 maven jetty plugin:

```xml
	<build>
		<finalName>${project.artifactId}</finalName>
		<plugins>
			<plugin>
				<groupId>org.mortbay.jetty</groupId>
				<artifactId>maven-jetty-plugin</artifactId>
				<version>${jetty.version}</version>
				<configuration>
					...
				</configuration>
			</plugin>
		</plugins>
	</build>
```
優點是啟動快速、無須安裝。

### AU 回應內容是否如預期

<span style="font-size:34px; color:blue">
（未完待續....）
</span>
