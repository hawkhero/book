本書所使用的範例完整原始碼，你都可以在 [Github](https://github.com/hawkhero/zk-coach) 找到。該專案使用 [Maven](https://zh.wikipedia.org/zh-tw/Apache_Maven) 管理。

# 執行專案
切換到專案的目錄下 (\zk-coath)，也就是有 `pom.xml` 的目錄下

## 未安裝 Maven
執行以下命令：

### Linux/Mac
`./mvnw jetty:run`

### Windows
`mvnw jetty:run`

因專案含有 [Maven wrapper](https://github.com/takari/maven-wrapper) ，因此不安裝 Maven 也能執行

## 已安裝 [Maven 3.0+](https://maven.apache.org/download.cgi)
執行以下命令：
`mvn jetty:run`

就可以將整個專案跑在 jetty 伺服器上，然就可以透過瀏灠器訪問以下網址來看所有頁面：
[`http://localhost:8080/zk-coach`](http://localhost:8080/zk-coach)


# 產出練習專案
執行以下命令：
`mvn package`

就會產生一個 /target/zk-coach-exercise.zip ，解壓縮後匯入到 eclipse 中，你會發現裡面的程式碼跟 zul 都是不完整的，目的是讓你做練習，你可以根據書中的內容將完成範例程式。每個標示 TODO 的位置就需要自行實作的提示。建議自行練習之後，才能真正學到 ZK。
