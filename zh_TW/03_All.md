# 3 Web基礎

學習基於Web的編程可能正是你讀本書的原因。事實上，如何通過Go來編寫Web應用也是我編寫這本書的初衷。前面已經介紹過，Go目前已經擁有了成熟的HTTP處理包，這使得編寫能做任何事情的動態Web程序易如反掌。在接下來的各章中將要介紹的內容，都是屬於Web編程的範疇。本章則集中討論一些與Web相關的概念和Go如何運行Web程序的話題。

## 目錄
![](images/navi3.png?raw=true)

## links
   * [目錄](<preface.md>)
   * 上一章: [第二章總結](<02.8.md>)
   * 下一節: [Web工作方式](<03.1.md>)

---

# 3.1 Web工作方式

我們平時瀏覽網頁的時候,會打開瀏覽器，輸入網址後按下回車鍵，然後就會顯示出你想要瀏覽的內容。在這個看似簡單的用戶行為背後，到底隱藏了些什麼呢？

對於普通的上網過程，系統其實是這樣做的：瀏覽器本身是一個客戶端，當你輸入URL的時候，首先瀏覽器會去請求DNS服務器，通過DNS獲取相應的域名對應的IP，然後通過IP地址找到IP對應的服務器後，要求建立TCP連接，等瀏覽器發送完HTTP Request（請求）包後，服務器接收到請求包之後才開始處理請求包，服務器調用自身服務，返回HTTP Response（響應）包；客戶端收到來自服務器的響應後開始渲染這個Response包裡的主體（body），等收到全部的內容隨後斷開與該服務器之間的TCP連接。

![](images/3.1.web2.png?raw=true)

圖3.1 用戶訪問一個Web站點的過程

 一個Web服務器也被稱為HTTP服務器，它通過HTTP協議與客戶端通信。這個客戶端通常指的是Web瀏覽器(其實手機端客戶端內部也是瀏覽器實現的)。

Web服務器的工作原理可以簡單地歸納為：

- 客戶機通過TCP/IP協議建立到服務器的TCP連接
- 客戶端向服務器發送HTTP協議請求包，請求服務器裡的資源文檔
- 服務器向客戶機發送HTTP協議應答包，如果請求的資源包含有動態語言的內容，那麼服務器會調用動態語言的解釋引擎負責處理“動態內容”，並將處理得到的數據返回給客戶端
- 客戶機與服務器斷開。由客戶端解釋HTML文檔，在客戶端屏幕上渲染圖形結果

一個簡單的HTTP事務就是這樣實現的，看起來很複雜，原理其實是挺簡單的。需要注意的是客戶機與服務器之間的通信是非持久連接的，也就是當服務器發送了應答後就與客戶機斷開連接，等待下一次請求。

## URL和DNS解析
我們瀏覽網頁都是通過URL訪問的，那麼URL到底是怎麼樣的呢？

URL(Uniform Resource Locator)是“統一資源定位符”的英文縮寫，用於描述一個網絡上的資源, 基本格式如下

	scheme://host[:port#]/path/.../[?query-string][#anchor]
	scheme         指定低層使用的協議(例如：http, https, ftp)
	host           HTTP服務器的IP地址或者域名
	port#          HTTP服務器的默認端口是80，這種情況下端口號可以省略。如果使用了別的端口，必須指明，例如 http://www.cnblogs.com:8080/
	path           訪問資源的路徑
	query-string   發送給http服務器的數據
	anchor         錨

 DNS(Domain Name System)是“域名系統”的英文縮寫，是一種組織成域層次結構的計算機和網絡服務命名系統，它用於TCP/IP網絡，它從事將主機名或域名轉換為實際IP地址的工作。DNS就是這樣的一位“翻譯官”，它的基本工作原理可用下圖來表示。

![](images/3.1.dns_hierachy.png?raw=true)

圖3.2 DNS工作原理

更詳細的DNS解析的過程如下，這個過程有助於我們理解DNS的工作模式

1. 在瀏覽器中輸入www.qq.com域名，操作系統會先檢查自己本地的hosts文件是否有這個網址映射關係，如果有，就先調用這個IP地址映射，完成域名解析。

2. 如果hosts裡沒有這個域名的映射，則查找本地DNS解析器緩存，是否有這個網址映射關係，如果有，直接返回，完成域名解析。

3. 如果hosts與本地DNS解析器緩存都沒有相應的網址映射關係，首先會找TCP/IP參數中設置的首選DNS服務器，在此我們叫它本地DNS服務器，此服務器收到查詢時，如果要查詢的域名，包含在本地配置區域資源中，則返回解析結果給客戶機，完成域名解析，此解析具有權威性。

4. 如果要查詢的域名，不由本地DNS服務器區域解析，但該服務器已緩存了此網址映射關係，則調用這個IP地址映射，完成域名解析，此解析不具有權威性。

5. 如果本地DNS服務器本地區域文件與緩存解析都失效，則根據本地DNS服務器的設置（是否設置轉發器）進行查詢，如果未用轉發模式，本地DNS就把請求發至 “根DNS服務器”，“根DNS服務器”收到請求後會判斷這個域名(.com)是誰來授權管理，並會返回一個負責該頂級域名服務器的一個IP。本地DNS服務器收到IP信息後，將會聯繫負責.com域的這臺服務器。這臺負責.com域的服務器收到請求後，如果自己無法解析，它就會找一個管理.com域的下一級DNS服務器地址(qq.com)給本地DNS服務器。當本地DNS服務器收到這個地址後，就會找qq.com域服務器，重複上面的動作，進行查詢，直至找到www.qq.com主機。

6. 如果用的是轉發模式，此DNS服務器就會把請求轉發至上一級DNS服務器，由上一級服務器進行解析，上一級服務器如果不能解析，或找根DNS或把轉請求轉至上上級，以此循環。不管是本地DNS服務器用是是轉發，還是根提示，最後都是把結果返回給本地DNS服務器，由此DNS服務器再返回給客戶機。

![](images/3.1.dns_inquery.png?raw=true)

圖3.3 DNS解析的整個流程

> 所謂 `遞歸查詢過程` 就是 “查詢的遞交者” 更替, 而 `迭代查詢過程` 則是 “查詢的遞交者”不變。
>
> 舉個例子來說，你想知道某個一起上法律課的女孩的電話，並且你偷偷拍了她的照片，回到寢室告訴一個很仗義的哥們兒，這個哥們兒二話沒說，拍著胸脯告訴你，甭急，我替你查(此處完成了一次遞歸查詢，即，問詢者的角色更替)。然後他拿著照片問了學院大四學長，學長告訴他，這姑娘是xx系的；然後這哥們兒馬不停蹄又問了xx系的辦公室主任助理同學，助理同學說是xx系yy班的，然後很仗義的哥們兒去xx系yy班的班長那裡取到了該女孩兒電話。(此處完成若干次迭代查詢，即，問詢者角色不變，但反覆更替問詢對象)最後，他把號碼交到了你手裡。完成整個查詢過程。

通過上面的步驟，我們最後獲取的是IP地址，也就是瀏覽器最後發起請求的時候是基於IP來和服務器做信息交互的。

## HTTP協議詳解

HTTP協議是Web工作的核心，所以要了解清楚Web的工作方式就需要詳細的瞭解清楚HTTP是怎麼樣工作的。

HTTP是一種讓Web服務器與瀏覽器(客戶端)通過Internet發送與接收數據的協議,它建立在TCP協議之上，一般採用TCP的80端口。它是一個請求、響應協議--客戶端發出一個請求，服務器響應這個請求。在HTTP中，客戶端總是通過建立一個連接與發送一個HTTP請求來發起一個事務。服務器不能主動去與客戶端聯繫，也不能給客戶端發出一個回調連接。客戶端與服務器端都可以提前中斷一個連接。例如，當瀏覽器下載一個文件時，你可以通過點擊“停止”鍵來中斷文件的下載，關閉與服務器的HTTP連接。

HTTP協議是無狀態的，同一個客戶端的這次請求和上次請求是沒有對應關係，對HTTP服務器來說，它並不知道這兩個請求是否來自同一個客戶端。為了解決這個問題， Web程序引入了Cookie機制來維護連接的可持續狀態。

>HTTP協議是建立在TCP協議之上的，因此TCP攻擊一樣會影響HTTP的通訊，例如比較常見的一些攻擊：SYN Flood是當前最流行的DoS（拒絕服務攻擊）與DdoS（分佈式拒絕服務攻擊）的方式之一，這是一種利用TCP協議缺陷，發送大量偽造的TCP連接請求，從而使得被攻擊方資源耗盡（CPU滿負荷或內存不足）的攻擊方式。

### HTTP請求包（瀏覽器信息）

我們先來看看Request包的結構, Request包分為3部分，第一部分叫Request line（請求行）, 第二部分叫Request header（請求頭）,第三部分是body（主體）。header和body之間有個空行，請求包的例子所示:

	GET /domains/example/ HTTP/1.1		//請求行: 請求方法 請求URI HTTP協議/協議版本
	Host：www.iana.org				//服務端的主機名
	User-Agent：Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.4 (KHTML, like Gecko) Chrome/22.0.1229.94 Safari/537.4			//瀏覽器信息
	Accept：text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8	//客戶端能接收的mine
	Accept-Encoding：gzip,deflate,sdch		//是否支持流壓縮
	Accept-Charset：UTF-8,*;q=0.5		//客戶端字符編碼集
	//空行,用於分割請求頭和消息體
	//消息體,請求資源參數,例如POST傳遞的參數

HTTP協議定義了很多與服務器交互的請求方法，最基本的有4種，分別是GET,POST,PUT,DELETE。一個URL地址用於描述一個網絡上的資源，而HTTP中的GET, POST, PUT, DELETE就對應著對這個資源的查，改，增，刪4個操作。我們最常見的就是GET和POST了。GET一般用於獲取/查詢資源信息，而POST一般用於更新資源信息。

通過fiddler抓包可以看到如下請求信息:

![](images/3.1.http.png?raw=true)

圖3.4 fiddler抓取的GET信息

![](images/3.1.httpPOST.png?raw=true)

圖3.5 fiddler抓取的POST信息

我們看看GET和POST的區別:

1. 我們可以看到GET請求消息體為空，POST請求帶有消息體。
2. GET提交的數據會放在URL之後，以`?`分割URL和傳輸數據，參數之間以`&`相連，如`EditPosts.aspx?name=test1&id=123456`。POST方法是把提交的數據放在HTTP包的body中。
3. GET提交的數據大小有限制（因為瀏覽器對URL的長度有限制），而POST方法提交的數據沒有限制。
4. GET方式提交數據，會帶來安全問題，比如一個登錄頁面，通過GET方式提交數據時，用戶名和密碼將出現在URL上，如果頁面可以被緩存或者其他人可以訪問這臺機器，就可以從歷史記錄獲得該用戶的賬號和密碼。

### HTTP響應包（服務器信息）
我們再來看看HTTP的response包，他的結構如下：

	HTTP/1.1 200 OK						//狀態行
	Server: nginx/1.0.8					//服務器使用的WEB軟件名及版本
	Date:Date: Tue, 30 Oct 2012 04:14:25 GMT		//發送時間
	Content-Type: text/html				//服務器發送信息的類型
	Transfer-Encoding: chunked			//表示發送HTTP包是分段發的
	Connection: keep-alive				//保持連接狀態
	Content-Length: 90					//主體內容長度
	//空行 用來分割消息頭和主體
	<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"... //消息體

Response包中的第一行叫做狀態行，由HTTP協議版本號， 狀態碼， 狀態消息 三部分組成。

狀態碼用來告訴HTTP客戶端,HTTP服務器是否產生了預期的Response。HTTP/1.1協議中定義了5類狀態碼， 狀態碼由三位數字組成，第一個數字定義了響應的類別

- 1XX  提示信息 		- 表示請求已被成功接收，繼續處理
- 2XX  成功 			- 表示請求已被成功接收，理解，接受
- 3XX  重定向 		- 要完成請求必須進行更進一步的處理
- 4XX  客戶端錯誤 	- 請求有語法錯誤或請求無法實現
- 5XX  服務器端錯誤 	- 服務器未能實現合法的請求

我們看下面這個圖展示了詳細的返回信息，左邊可以看到有很多的資源返回碼，200是常用的，表示正常信息，302表示跳轉。response header裡面展示了詳細的信息。

![](images/3.1.response.png?raw=true)

圖3.6 訪問一次網站的全部請求信息

### HTTP協議是無狀態的和Connection: keep-alive的區別
無狀態是指協議對於事務處理沒有記憶能力，服務器不知道客戶端是什麼狀態。從另一方面講，打開一個服務器上的網頁和你之前打開這個服務器上的網頁之間沒有任何聯繫。

HTTP是一個無狀態的面向連接的協議，無狀態不代表HTTP不能保持TCP連接，更不能代表HTTP使用的是UDP協議（面對無連接）。

從HTTP/1.1起，默認都開啟了Keep-Alive保持連接特性，簡單地說，當一個網頁打開完成後，客戶端和服務器之間用於傳輸HTTP數據的TCP連接不會關閉，如果客戶端再次訪問這個服務器上的網頁，會繼續使用這一條已經建立的TCP連接。

Keep-Alive不會永久保持連接，它有一個保持時間，可以在不同服務器軟件（如Apache）中設置這個時間。

## 請求實例

![](images/3.1.web.png?raw=true)

圖3.7 一次請求的request和response

上面這張圖我們可以瞭解到整個的通訊過程，同時細心的讀者是否注意到了一點，一個URL請求但是左邊欄裡面為什麼會有那麼多的資源請求(這些都是靜態文件，go對於靜態文件有專門的處理方式)。

這個就是瀏覽器的一個功能，第一次請求url，服務器端返回的是html頁面，然後瀏覽器開始渲染HTML：當解析到HTML DOM裡面的圖片連接，css腳本和js腳本的鏈接，瀏覽器就會自動發起一個請求靜態資源的HTTP請求，獲取相對應的靜態資源，然後瀏覽器就會渲染出來，最終將所有資源整合、渲染，完整展現在我們面前的屏幕上。

>網頁優化方面有一項措施是減少HTTP請求次數，就是把儘量多的css和js資源合併在一起，目的是儘量減少網頁請求靜態資源的次數，提高網頁加載速度，同時減緩服務器的壓力。

## links
   * [目錄](<preface.md>)
   * 上一節: [Web基礎](<03.0.md>)
   * 下一節: [Go搭建一個Web服務器](<03.2.md>)

---

# 3.2 Go搭建一個Web服務器

前面小節已經介紹了Web是基於http協議的一個服務，Go語言裡面提供了一個完善的net/http包，通過http包可以很方便的就搭建起來一個可以運行的Web服務。同時使用這個包能很簡單地對Web的路由，靜態文件，模版，cookie等數據進行設置和操作。

## http包建立Web服務器

	package main

	import (
		"fmt"
		"net/http"
		"strings"
		"log"
	)

	func sayhelloName(w http.ResponseWriter, r *http.Request) {
		r.ParseForm()  //解析參數，默認是不會解析的
		fmt.Println(r.Form)  //這些信息是輸出到服務器端的打印信息
		fmt.Println("path", r.URL.Path)
		fmt.Println("scheme", r.URL.Scheme)
		fmt.Println(r.Form["url_long"])
		for k, v := range r.Form {
			fmt.Println("key:", k)
			fmt.Println("val:", strings.Join(v, ""))
		}
		fmt.Fprintf(w, "Hello astaxie!") //這個寫入到w的是輸出到客戶端的
	}

	func main() {
		http.HandleFunc("/", sayhelloName) //設置訪問的路由
		err := http.ListenAndServe(":9090", nil) //設置監聽的端口
		if err != nil {
			log.Fatal("ListenAndServe: ", err)
		}
	}

上面這個代碼，我們build之後，然後執行web.exe,這個時候其實已經在9090端口監聽http鏈接請求了。

在瀏覽器輸入`http://localhost:9090`

可以看到瀏覽器頁面輸出了`Hello astaxie!`

可以換一個地址試試：`http://localhost:9090/?url_long=111&url_long=222`

看看瀏覽器輸出的是什麼，服務器輸出的是什麼？

在服務器端輸出的信息如下：

![](images/3.2.goweb.png?raw=true)

圖3.8 用戶訪問Web之後服務器端打印的信息

我們看到上面的代碼，要編寫一個Web服務器很簡單，只要調用http包的兩個函數就可以了。

>如果你以前是PHP程序員，那你也許就會問，我們的nginx、apache服務器不需要嗎？Go就是不需要這些，因為他直接就監聽tcp端口了，做了nginx做的事情，然後sayhelloName這個其實就是我們寫的邏輯函數了，跟php裡面的控制層（controller）函數類似。

>如果你以前是Python程序員，那麼你一定聽說過tornado，這個代碼和他是不是很像，對，沒錯，Go就是擁有類似Python這樣動態語言的特性，寫Web應用很方便。

>如果你以前是Ruby程序員，會發現和ROR的/script/server啟動有點類似。

我們看到Go通過簡單的幾行代碼就已經運行起來一個Web服務了，而且這個Web服務內部有支持高併發的特性，我將會在接下來的兩個小節裡面詳細的講解一下Go是如何實現Web高併發的。

## links
   * [目錄](<preface.md>)
   * 上一節: [Web工作方式](<03.1.md>)
   * 下一節: [Go如何使得web工作](<03.3.md>)

---

# 3.3 Go如何使得Web工作
前面小節介紹瞭如何通過Go搭建一個Web服務，我們可以看到簡單應用一個net/http包就方便的搭建起來了。那麼Go在底層到底是怎麼做的呢？萬變不離其宗，Go的Web服務工作也離不開我們第一小節介紹的Web工作方式。

## web工作方式的幾個概念

以下均是服務器端的幾個概念

Request：用戶請求的信息，用來解析用戶的請求信息，包括post、get、cookie、url等信息

Response：服務器需要反饋給客戶端的信息

Conn：用戶的每次請求鏈接

Handler：處理請求和生成返回信息的處理邏輯

## 分析http包運行機制

如下圖所示，是Go實現Web服務的工作模式的流程圖

![](images/3.3.http.png?raw=true)

圖3.9 http包執行流程

1. 創建Listen Socket, 監聽指定的端口, 等待客戶端請求到來。

2. Listen Socket接受客戶端的請求, 得到Client Socket, 接下來通過Client Socket與客戶端通信。

3. 處理客戶端的請求, 首先從Client Socket讀取HTTP請求的協議頭, 如果是POST方法, 還可能要讀取客戶端提交的數據, 然後交給相應的handler處理請求, handler處理完畢準備好客戶端需要的數據, 通過Client Socket寫給客戶端。

這整個的過程裡面我們只要瞭解清楚下面三個問題，也就知道Go是如何讓Web運行起來了

- 如何監聽端口？
- 如何接收客戶端請求？
- 如何分配handler？

前面小節的代碼裡面我們可以看到，Go是通過一個函數`ListenAndServe`來處理這些事情的，這個底層其實這樣處理的：初始化一個server對象，然後調用了`net.Listen("tcp", addr)`，也就是底層用TCP協議搭建了一個服務，然後監控我們設置的端口。

下面代碼來自Go的http包的源碼，通過下面的代碼我們可以看到整個的http處理過程：

	func (srv *Server) Serve(l net.Listener) error {
		defer l.Close()
		var tempDelay time.Duration // how long to sleep on accept failure
		for {
			rw, e := l.Accept()
			if e != nil {
				if ne, ok := e.(net.Error); ok && ne.Temporary() {
					if tempDelay == 0 {
						tempDelay = 5 * time.Millisecond
					} else {
						tempDelay *= 2
					}
					if max := 1 * time.Second; tempDelay > max {
						tempDelay = max
					}
					log.Printf("http: Accept error: %v; retrying in %v", e, tempDelay)
					time.Sleep(tempDelay)
					continue
				}
				return e
			}
			tempDelay = 0
			c, err := srv.newConn(rw)
			if err != nil {
				continue
			}
			go c.serve()
		}
	}

監控之後如何接收客戶端的請求呢？上面代碼執行監控端口之後，調用了`srv.Serve(net.Listener)`函數，這個函數就是處理接收客戶端的請求信息。這個函數裡面起了一個`for{}`，首先通過Listener接收請求，其次創建一個Conn，最後單獨開了一個goroutine，把這個請求的數據當做參數扔給這個conn去服務：`go c.serve()`。這個就是高併發體現了，用戶的每一次請求都是在一個新的goroutine去服務，相互不影響。

那麼如何具體分配到相應的函數來處理請求呢？conn首先會解析request:`c.readRequest()`,然後獲取相應的handler:`handler := c.server.Handler`，也就是我們剛才在調用函數`ListenAndServe`時候的第二個參數，我們前面例子傳遞的是nil，也就是為空，那麼默認獲取`handler = DefaultServeMux`,那麼這個變量用來做什麼的呢？對，這個變量就是一個路由器，它用來匹配url跳轉到其相應的handle函數，那麼這個我們有設置過嗎?有，我們調用的代碼裡面第一句不是調用了`http.HandleFunc("/", sayhelloName)`嘛。這個作用就是註冊了請求`/`的路由規則，當請求uri為"/"，路由就會轉到函數sayhelloName，DefaultServeMux會調用ServeHTTP方法，這個方法內部其實就是調用sayhelloName本身，最後通過寫入response的信息反饋到客戶端。


詳細的整個流程如下圖所示：

![](images/3.3.illustrator.png?raw=true)

圖3.10 一個http連接處理流程

至此我們的三個問題已經全部得到了解答，你現在對於Go如何讓Web跑起來的是否已經基本瞭解呢？


## links
   * [目錄](<preface.md>)
   * 上一節: [GO搭建一個簡單的web服務](<03.2.md>)
   * 下一節: [Go的http包詳解](<03.4.md>)

---

# 3.4 Go的http包詳解
前面小節介紹了Go怎麼樣實現了Web工作模式的一個流程，這一小節，我們將詳細地解剖一下http包，看它到底是怎樣實現整個過程的。

Go的http有兩個核心功能：Conn、ServeMux

## Conn的goroutine
與我們一般編寫的http服務器不同, Go為了實現高併發和高性能, 使用了goroutines來處理Conn的讀寫事件, 這樣每個請求都能保持獨立，相互不會阻塞，可以高效的響應網絡事件。這是Go高效的保證。

Go在等待客戶端請求裡面是這樣寫的：

	c, err := srv.newConn(rw)
	if err != nil {
		continue
	}
	go c.serve()

這裡我們可以看到客戶端的每次請求都會創建一個Conn，這個Conn裡面保存了該次請求的信息，然後再傳遞到對應的handler，該handler中便可以讀取到相應的header信息，這樣保證了每個請求的獨立性。

## ServeMux的自定義
我們前面小節講述conn.server的時候，其實內部是調用了http包默認的路由器，通過路由器把本次請求的信息傳遞到了後端的處理函數。那麼這個路由器是怎麼實現的呢？

它的結構如下：

	type ServeMux struct {
		mu sync.RWMutex   //鎖，由於請求涉及到併發處理，因此這裡需要一個鎖機制
		m  map[string]muxEntry  // 路由規則，一個string對應一個mux實體，這裡的string就是註冊的路由表達式
		hosts bool // 是否在任意的規則中帶有host信息
	}

下面看一下muxEntry

	type muxEntry struct {
		explicit bool   // 是否精確匹配
		h        Handler // 這個路由表達式對應哪個handler
		pattern  string  //匹配字符串
	}

接著看一下Handler的定義

	type Handler interface {
		ServeHTTP(ResponseWriter, *Request)  // 路由實現器
	}

Handler是一個接口，但是前一小節中的`sayhelloName`函數並沒有實現ServeHTTP這個接口，為什麼能添加呢？原來在http包裡面還定義了一個類型`HandlerFunc`,我們定義的函數`sayhelloName`就是這個HandlerFunc調用之後的結果，這個類型默認就實現了ServeHTTP這個接口，即我們調用了HandlerFunc(f),強制類型轉換f成為HandlerFunc類型，這樣f就擁有了ServeHTTP方法。

	type HandlerFunc func(ResponseWriter, *Request)

	// ServeHTTP calls f(w, r).
	func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
		f(w, r)
	}

路由器裡面存儲好了相應的路由規則之後，那麼具體的請求又是怎麼分發的呢？請看下面的代碼，默認的路由器實現了`ServeHTTP`：

	func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request) {
		if r.RequestURI == "*" {
			w.Header().Set("Connection", "close")
			w.WriteHeader(StatusBadRequest)
			return
		}
		h, _ := mux.Handler(r)
		h.ServeHTTP(w, r)
	}

如上所示路由器接收到請求之後，如果是`*`那麼關閉鏈接，不然調用`mux.Handler(r)`返回對應設置路由的處理Handler，然後執行`h.ServeHTTP(w, r)`

也就是調用對應路由的handler的ServerHTTP接口，那麼mux.Handler(r)怎麼處理的呢？

	func (mux *ServeMux) Handler(r *Request) (h Handler, pattern string) {
		if r.Method != "CONNECT" {
			if p := cleanPath(r.URL.Path); p != r.URL.Path {
				_, pattern = mux.handler(r.Host, p)
				return RedirectHandler(p, StatusMovedPermanently), pattern
			}
		}	
		return mux.handler(r.Host, r.URL.Path)
	}
	
	func (mux *ServeMux) handler(host, path string) (h Handler, pattern string) {
		mux.mu.RLock()
		defer mux.mu.RUnlock()
	
		// Host-specific pattern takes precedence over generic ones
		if mux.hosts {
			h, pattern = mux.match(host + path)
		}
		if h == nil {
			h, pattern = mux.match(path)
		}
		if h == nil {
			h, pattern = NotFoundHandler(), ""
		}
		return
	}

原來他是根據用戶請求的URL和路由器裡面存儲的map去匹配的，當匹配到之後返回存儲的handler，調用這個handler的ServeHTTP接口就可以執行到相應的函數了。

通過上面這個介紹，我們瞭解了整個路由過程，Go其實支持外部實現的路由器 `ListenAndServe`的第二個參數就是用以配置外部路由器的，它是一個Handler接口，即外部路由器只要實現了Handler接口就可以,我們可以在自己實現的路由器的ServeHTTP裡面實現自定義路由功能。

如下代碼所示，我們自己實現了一個簡易的路由器

	package main

	import (
		"fmt"
		"net/http"
	)

	type MyMux struct {
	}

	func (p *MyMux) ServeHTTP(w http.ResponseWriter, r *http.Request) {
		if r.URL.Path == "/" {
			sayhelloName(w, r)
			return
		}
		http.NotFound(w, r)
		return
	}

	func sayhelloName(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Hello myroute!")
	}

	func main() {
		mux := &MyMux{}
		http.ListenAndServe(":9090", mux)
	}

## Go代碼的執行流程

通過對http包的分析之後，現在讓我們來梳理一下整個的代碼執行過程。

- 首先調用Http.HandleFunc

	按順序做了幾件事：

	1 調用了DefaultServeMux的HandleFunc

	2 調用了DefaultServeMux的Handle

	3 往DefaultServeMux的map[string]muxEntry中增加對應的handler和路由規則

- 其次調用http.ListenAndServe(":9090", nil)

	按順序做了幾件事情：

	1 實例化Server

	2 調用Server的ListenAndServe()

	3 調用net.Listen("tcp", addr)監聽端口

	4 啟動一個for循環，在循環體中Accept請求

	5 對每個請求實例化一個Conn，並且開啟一個goroutine為這個請求進行服務go c.serve()

	6 讀取每個請求的內容w, err := c.readRequest()

	7 判斷handler是否為空，如果沒有設置handler（這個例子就沒有設置handler），handler就設置為DefaultServeMux

	8 調用handler的ServeHttp

	9 在這個例子中，下面就進入到DefaultServeMux.ServeHttp

	10 根據request選擇handler，並且進入到這個handler的ServeHTTP

		mux.handler(r).ServeHTTP(w, r)

	11 選擇handler：

	A 判斷是否有路由能滿足這個request（循環遍歷ServerMux的muxEntry）

	B 如果有路由滿足，調用這個路由handler的ServeHttp

	C 如果沒有路由滿足，調用NotFoundHandler的ServeHttp

## links
   * [目錄](<preface.md>)
   * 上一節: [Go如何使得web工作](<03.3.md>)
   * 下一節: [小結](<03.5.md>)

---

# 3.5 小結
這一章我們介紹了HTTP協議, DNS解析的過程, 如何用go實現一個簡陋的web server。並深入到net/http包的源碼中為大家揭開實現此server的祕密。

希望通過這一章的學習，你能夠對Go開發Web有了初步的瞭解，我們也看到相應的代碼了，Go開發Web應用是很方便的，同時又是相當的靈活。

## links
   * [目錄](<preface.md>)
   * 上一節: [Go的http包詳解](<03.4.md>)
   * 下一章: [表單](<04.0.md>)
