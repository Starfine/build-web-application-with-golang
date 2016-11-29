# 12 部署與維護
到目前為止，我們前面已經介紹瞭如何開發程序、調試程序以及測試程序，正如人們常說的：開發最後的10%需要花費90%的時間，所以這一章我們將強調這最後的10%部分，要真正成為讓人信任並使用的優秀應用，需要考慮到一些細節，以上所說的10%就是指這些小細節。

本章我們將通過四個小節來介紹這些小細節的處理，第一小節介紹如何在生產服務上記錄程序產生的日誌，如何記錄日誌，第二小節介紹發生錯誤時我們的程序如何處理，如何保證儘量少的影響到用戶的訪問，第三小節介紹如何來部署Go的獨立程序，由於目前Go程序還無法像C那樣寫成daemon，那麼我們如何管理這樣的進程程序後臺運行呢？第四小節將介紹應用數據的備份和恢復，儘量保證應用在崩潰的情況能夠保持數據的完整性。
## 目錄
 ![](images/navi12.png?raw=true)

## links
   * [目錄](<preface.md>)
   * 上一章: [第十一章總結](<11.4.md>)
   * 下一節: [應用日誌](<12.1.md>)

---

# 12.1 應用日誌
我們期望開發的Web應用程序能夠把整個程序運行過程中出現的各種事件一一記錄下來，Go語言中提供了一個簡易的log包，我們使用該包可以方便的實現日誌記錄的功能，這些日誌都是基於fmt包的打印再結合panic之類的函數來進行一般的打印、拋出錯誤處理。Go目前標準包只是包含了簡單的功能，如果我們想把我們的應用日誌保存到文件，然後又能夠結合日誌實現很多複雜的功能（編寫過Java或者C++的讀者應該都使用過log4j和log4cpp之類的日誌工具），可以使用第三方開發的一個日誌系統，`https://github.com/cihub/seelog`，它實現了很強大的日誌功能。接下來我們介紹如何通過該日誌系統來實現我們應用的日誌功能。

## seelog介紹
seelog是用Go語言實現的一個日誌系統，它提供了一些簡單的函數來實現複雜的日誌分配、過濾和格式化。主要有如下特性：

- XML的動態配置，可以不用重新編譯程序而動態的加載配置信息
- 支持熱更新，能夠動態改變配置而不需要重啟應用
- 支持多輸出流，能夠同時把日誌輸出到多種流中、例如文件流、網絡流等
- 支持不同的日誌輸出

	- 命令行輸出
	- 文件輸出
	- 緩存輸出
	- 支持log rotate
	- SMTP郵件

上面只列舉了部分特性，seelog是一個特別強大的日誌處理系統，詳細的內容請參看官方wiki。接下來我將簡要介紹一下如何在項目中使用它：

首先安裝seelog

	go get -u github.com/cihub/seelog
	
然後我們來看一個簡單的例子：

	package main

	import log "github.com/cihub/seelog"

	func main() {
	    defer log.Flush()
	    log.Info("Hello from Seelog!")
	}

編譯後運行如果出現了`Hello from seelog`，說明seelog日誌系統已經成功安裝並且可以正常運行了。

## 基於seelog的自定義日誌處理
seelog支持自定義日誌處理，下面是我基於它自定義的日誌處理包的部分內容：

	package logs
	
	import (
		"errors"
		"fmt"
		seelog "github.com/cihub/seelog"
		"io"
	)
	
	var Logger seelog.LoggerInterface
	
	func loadAppConfig() {
		appConfig := `
	<seelog minlevel="warn">
	    <outputs formatid="common">
	        <rollingfile type="size" filename="/data/logs/roll.log" maxsize="100000" maxrolls="5"/>
			<filter levels="critical">
	            <file path="/data/logs/critical.log" formatid="critical"/>
	            <smtp formatid="criticalemail" senderaddress="astaxie@gmail.com" sendername="ShortUrl API" hostname="smtp.gmail.com" hostport="587" username="mailusername" password="mailpassword">
	                <recipient address="xiemengjun@gmail.com"/>
	            </smtp>
	        </filter>
	    </outputs>
	    <formats>
	        <format id="common" format="%Date/%Time [%LEV] %Msg%n" />
		    <format id="critical" format="%File %FullPath %Func %Msg%n" />
		    <format id="criticalemail" format="Critical error on our server!\n    %Time %Date %RelFile %Func %Msg \nSent by Seelog"/>
	    </formats>
	</seelog>
	`
		logger, err := seelog.LoggerFromConfigAsBytes([]byte(appConfig))
		if err != nil {
			fmt.Println(err)
			return
		}
		UseLogger(logger)
	}
	
	func init() {
		DisableLog()
		loadAppConfig()
	}
	
	// DisableLog disables all library log output
	func DisableLog() {
		Logger = seelog.Disabled
	}
	
	// UseLogger uses a specified seelog.LoggerInterface to output library log.
	// Use this func if you are using Seelog logging system in your app.
	func UseLogger(newLogger seelog.LoggerInterface) {
		Logger = newLogger
	}

上面主要實現了三個函數，

- `DisableLog`
	
	初始化全局變量Logger為seelog的禁用狀態，主要為了防止Logger被多次初始化
- `loadAppConfig`

	根據配置文件初始化seelog的配置信息，這裡我們把配置文件通過字符串讀取設置好了，當然也可以通過讀取XML文件。裡面的配置說明如下：
	
	- seelog 
	
		minlevel參數可選，如果被配置,高於或等於此級別的日誌會被記錄，同理maxlevel。
	- outputs
		
		輸出信息的目的地，這裡分成了兩份數據，一份記錄到log rotate文件裡面。另一份設置了filter，如果這個錯誤級別是critical，那麼將發送報警郵件。
		
	- formats
	
		定義了各種日誌的格式
	
- `UseLogger`

	設置當前的日誌器為相應的日誌處理
	
上面我們定義了一個自定義的日誌處理包，下面就是使用示例：

	package main
	
	import (
		"net/http"
		"project/logs"
		"project/configs"
		"project/routes"
	)
	
	func main() {
		addr, _ := configs.MainConfig.String("server", "addr")
		logs.Logger.Info("Start server at:%v", addr)
		err := http.ListenAndServe(addr, routes.NewMux())
		logs.Logger.Critical("Server err:%v", err)
	}

## 發生錯誤發送郵件
上面的例子解釋瞭如何設置發送郵件，我們通過如下的smtp配置用來發送郵件：

	<smtp formatid="criticalemail" senderaddress="astaxie@gmail.com" sendername="ShortUrl API" hostname="smtp.gmail.com" hostport="587" username="mailusername" password="mailpassword">
		<recipient address="xiemengjun@gmail.com"/>
	</smtp>

郵件的格式通過criticalemail配置，然後通過其他的配置發送郵件服務器的配置，通過recipient配置接收郵件的用戶，如果有多個用戶可以再添加一行。

要測試這個代碼是否正常工作，可以在代碼中增加類似下面的一個假消息。不過記住過後要把它刪除，否則上線之後就會收到很多垃圾郵件。

	logs.Logger.Critical("test Critical message")

現在，只要我們的應用在線上記錄一個Critical的信息，你的郵箱就會收到一個Email，這樣一旦線上的系統出現問題，你就能立馬通過郵件獲知，就能及時的進行處理。			
## 使用應用日誌
對於應用日誌，每個人的應用場景可能會各不相同，有些人利用應用日誌來做數據分析，有些人利用應用日誌來做性能分析，有些人來做用戶行為分析，還有些就是純粹的記錄，以方便應用出現問題的時候輔助查找問題。

舉一個例子，我們需要跟蹤用戶嘗試登陸系統的操作。這裡會把成功與不成功的嘗試都記錄下來。記錄成功的使用"Info"日誌級別，而不成功的使用"warn"級別。如果想查找所有不成功的登陸，我們可以利用linux的grep之類的命令工具，如下：

	# cat /data/logs/roll.log | grep "failed login"
	2012-12-11 11:12:00 WARN : failed login attempt from 11.22.33.44 username password

通過這種方式我們就可以很方便的查找相應的信息，這樣有利於我們針對應用日誌做一些統計和分析。另外我們還需要考慮日誌的大小，對於一個高流量的Web應用來說，日誌的增長是相當可怕的，所以我們在seelog的配置文件裡面設置了logrotate，這樣就能保證日誌文件不會因為不斷變大而導致我們的磁盤空間不夠引起問題。

## 小結
通過上面對seelog系統及如何基於它進行自定義日誌系統的學習，現在我們可以很輕鬆的隨需構建一個合適的功能強大的日誌處理系統了。日誌處理系統為數據分析提供了可靠的數據源，比如通過對日誌的分析，我們可以進一步優化系統，或者應用出現問題時方便查找定位問題，另外seelog也提供了日誌分級功能，通過對minlevel的配置，我們可以很方便的設置測試或發佈版本的輸出消息級別。

## links
   * [目錄](<preface.md>)
   * 上一章: [部署與維護](<12.0.md>)
   * 下一節: [網站錯誤處理](<12.2.md>)
<!-- {% raw %} -->

---

# 12.2 網站錯誤處理
我們的Web應用一旦上線之後，那麼各種錯誤出現的概率都有，Web應用日常運行中可能出現多種錯誤，具體如下所示：

- 數據庫錯誤：指與訪問數據庫服務器或數據相關的錯誤。例如，以下可能出現的一些數據庫錯誤。

	- 連接錯誤：這一類錯誤可能是數據庫服務器網絡斷開、用戶名密碼不正確、或者數據庫不存在。
	- 查詢錯誤：使用的SQL非法導致錯誤，這樣子SQL錯誤如果程序經過嚴格的測試應該可以避免。
	- 數據錯誤：數據庫中的約束衝突，例如一個唯一字段中插入一條重複主鍵的值就會報錯，但是如果你的應用程序在上線之前經過了嚴格的測試也是可以避免這類問題。
- 應用運行時錯誤：這類錯誤範圍很廣，涵蓋了代碼中出現的幾乎所有錯誤。可能的應用錯誤的情況如下：

	- 文件系統和權限：應用讀取不存在的文件，或者讀取沒有權限的文件、或者寫入一個不允許寫入的文件，這些都會導致一個錯誤。應用讀取的文件如果格式不正確也會報錯，例如配置文件應該是ini的配置格式，而設置成了json格式就會報錯。
	- 第三方應用：如果我們的應用程序耦合了其他第三方接口程序，例如應用程序發表文章之後自動調用接發微博的接口，所以這個接口必須正常運行才能完成我們發表一篇文章的功能。

- HTTP錯誤：這些錯誤是根據用戶的請求出現的錯誤，最常見的就是404錯誤。雖然可能會出現很多不同的錯誤，但其中比較常見的錯誤還有401未授權錯誤(需要認證才能訪問的資源)、403禁止錯誤(不允許用戶訪問的資源)和503錯誤(程序內部出錯)。
- 操作系統出錯：這類錯誤都是由於應用程序上的操作系統出現錯誤引起的，主要有操作系統的資源被分配完了，導致死機，還有操作系統的磁盤滿了，導致無法寫入，這樣就會引起很多錯誤。
- 網絡出錯：指兩方面的錯誤，一方面是用戶請求應用程序的時候出現網絡斷開，這樣就導致連接中斷，這種錯誤不會造成應用程序的崩潰，但是會影響用戶訪問的效果；另一方面是應用程序讀取其他網絡上的數據，其他網絡斷開會導致讀取失敗，這種需要對應用程序做有效的測試，能夠避免這類問題出現的情況下程序崩潰。

## 錯誤處理的目標
在實現錯誤處理之前，我們必須明確錯誤處理想要達到的目標是什麼，錯誤處理系統應該完成以下工作：

- 通知訪問用戶出現錯誤了：不論出現的是一個系統錯誤還是用戶錯誤，用戶都應當知道Web應用出了問題，用戶的這次請求無法正確的完成了。例如，對於用戶的錯誤請求，我們顯示一個統一的錯誤頁面(404.html)。出現系統錯誤時，我們通過自定義的錯誤頁面顯示系統暫時不可用之類的錯誤頁面(error.html)。
- 記錄錯誤：系統出現錯誤，一般就是我們調用函數的時候返回err不為nil的情況，可以使用前面小節介紹的日誌系統記錄到日誌文件。如果是一些致命錯誤，則通過郵件通知系統管理員。一般404之類的錯誤不需要發送郵件，只需要記錄到日誌系統。
- 回滾當前的請求操作：如果一個用戶請求過程中出現了一個服務器錯誤，那麼已完成的操作需要回滾。下面來看一個例子：一個系統將用戶遞交的表單保存到數據庫，並將這個數據遞交到一個第三方服務器，但是第三方服務器掛了，這就導致一個錯誤，那麼先前存儲到數據庫的表單數據應該刪除(應告知無效)，而且應該通知用戶系統出現錯誤了。
- 保證現有程序可運行可服務：我們知道沒有人能保證程序一定能夠一直正常的運行著，萬一哪一天程序崩潰了，那麼我們就需要記錄錯誤，然後立刻讓程序重新運行起來，讓程序繼續提供服務，然後再通知系統管理員，通過日誌等找出問題。

## 如何處理錯誤
錯誤處理其實我們已經在十一章第一小節裡面有過介紹如何設計錯誤處理，這裡我們再從一個例子詳細的講解一下，如何來處理不同的錯誤：

- 通知用戶出現錯誤：

	通知用戶在訪問頁面的時候我們可以有兩種錯誤：404.html和error.html，下面分別顯示了錯誤頁面的源碼：

		<html lang="en">
		<head>
		    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
		    <title>找不到頁面</title>
		    <meta name="viewport" content="width=device-width, initial-scale=1.0">

		</head>
		<body>
		<div class="container">
		    <div class="row">
		        <div class="span10">
		            <div class="hero-unit">
		                <h1>404!</h1>
		                <p>{{.ErrorInfo}}</p>
		            </div>
		        </div><!--/span-->
		    </div>
		</div>
		</body>
		</html>
	另一個源碼：

		<html lang="en">
		<head>
		    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
		    <title>系統錯誤頁面</title>
		    <meta name="viewport" content="width=device-width, initial-scale=1.0">

		</head>
		<body>
		<div class="container">
		    <div class="row">
		        <div class="span10">
		            <div class="hero-unit">
		                <h1>系統暫時不可用!</h1>
		                <p>{{.ErrorInfo}}</p>
		            </div>
		        </div><!--/span-->
		    </div>
		</div>
		</body>
		</html>

	404的錯誤處理邏輯，如果是系統的錯誤也是類似的操作，同時我們看到在：

		func (p *MyMux) ServeHTTP(w http.ResponseWriter, r *http.Request) {
		    if r.URL.Path == "/" {
		        sayhelloName(w, r)
		        return
		    }
		    NotFound404(w, r)
		    return
		}

		func NotFound404(w http.ResponseWriter, r *http.Request) {
			log.Error("頁面找不到")   //記錄錯誤日誌
			t, _ = t.ParseFiles("tmpl/404.html", nil)  //解析模板文件
	    	ErrorInfo := "文件找不到" //獲取當前用戶信息
	    	t.Execute(w, ErrorInfo)  //執行模板的merger操作
		}

		func SystemError(w http.ResponseWriter, r *http.Request) {
			log.Critical("系統錯誤")   //系統錯誤觸發了Critical，那麼不僅會記錄日誌還會發送郵件
			t, _ = t.ParseFiles("tmpl/error.html", nil)  //解析模板文件
	    	ErrorInfo := "系統暫時不可用" //獲取當前用戶信息
	    	t.Execute(w, ErrorInfo)  //執行模板的merger操作
		}

## 如何處理異常
我們知道在很多其他語言中有try...catch關鍵詞，用來捕獲異常情況，但是其實很多錯誤都是可以預期發生的，而不需要異常處理，應該當做錯誤來處理，這也是為什麼Go語言採用了函數返回錯誤的設計，這些函數不會panic，例如如果一個文件找不到，os.Open返回一個錯誤，它不會panic；如果你向一箇中斷的網絡連接寫數據，net.Conn系列類型的Write函數返回一個錯誤，它們不會panic。這些狀態在這樣的程序裡都是可以預期的。你知道這些操作可能會失敗，因為設計者已經用返回錯誤清楚地表明瞭這一點。這就是上面所講的可以預期發生的錯誤。

但是還有一種情況，有一些操作幾乎不可能失敗，而且在一些特定的情況下也沒有辦法返回錯誤，也無法繼續執行，這樣情況就應該panic。舉個例子：如果一個程序計算x[j]，但是j越界了，這部分代碼就會導致panic，像這樣的一個不可預期嚴重錯誤就會引起panic，在默認情況下它會殺掉進程，它允許一個正在運行這部分代碼的goroutine從發生錯誤的panic中恢復運行，發生panic之後，這部分代碼後面的函數和代碼都不會繼續執行，這是Go特意這樣設計的，因為要區別於錯誤和異常，panic其實就是異常處理。如下代碼，我們期望通過uid來獲取User中的username信息，但是如果uid越界了就會拋出異常，這個時候如果我們沒有recover機制，進程就會被殺死，從而導致程序不可服務。因此為了程序的健壯性，在一些地方需要建立recover機制。

	func GetUser(uid int) (username string) {
		defer func() {
			if x := recover(); x != nil {
				username = ""
			}
		}()

		username = User[uid]
		return
	}

上面介紹了錯誤和異常的區別，那麼我們在開發程序的時候如何來設計呢？規則很簡單：如果你定義的函數有可能失敗，它就應該返回一個錯誤。當我調用其他package的函數時，如果這個函數實現的很好，我不需要擔心它會panic，除非有真正的異常情況發生，即使那樣也不應該是我去處理它。而panic和recover是針對自己開發package裡面實現的邏輯，針對一些特殊情況來設計。

## 小結
本小節總結了當我們的Web應用部署之後如何處理各種錯誤：網絡錯誤、數據庫錯誤、操作系統錯誤等，當錯誤發生時，我們的程序如何來正確處理：顯示友好的出錯界面、回滾操作、記錄日誌、通知管理員等操作，最後介紹瞭如何來正確處理錯誤和異常。一般的程序中錯誤和異常很容易混淆的，但是在Go中錯誤和異常是有明顯的區分，所以告訴我們在程序設計中處理錯誤和異常應該遵循怎麼樣的原則。
## links
   * [目錄](<preface.md>)
   * 上一章: [應用日誌](<12.1.md>)
   * 下一節: [應用部署](<12.3.md>)
<!-- {% endraw %} -->

---

# 12.3 應用部署
程序開發完畢之後，我們現在要部署Web應用程序了，但是我們如何來部署這些應用程序呢？因為Go程序編譯之後是一個可執行文件，編寫過C程序的讀者一定知道採用daemon就可以完美的實現程序後臺持續運行，但是目前Go還無法完美的實現daemon，因此，針對Go的應用程序部署，我們可以利用第三方工具來管理，第三方的工具有很多，例如Supervisord、upstart、daemontools等，這小節我介紹目前自己系統中採用的工具Supervisord。
## daemon
目前Go程序還不能實現daemon，詳細的見這個Go語言的bug：<`http://code.google.com/p/go/issues/detail?id=227`>，大概的意思說很難從現有的使用的線程中fork一個出來，因為沒有一種簡單的方法來確保所有已經使用的線程的狀態一致性問題。

但是我們可以看到很多網上的一些實現daemon的方法，例如下面兩種方式：

- MarGo的一個實現思路，使用Commond來執行自身的應用，如果真想實現，那麼推薦這種方案

		d := flag.Bool("d", false, "Whether or not to launch in the background(like a daemon)")
		if *d {
			cmd := exec.Command(os.Args[0],
				"-close-fds",
				"-addr", *addr,
				"-call", *call,
			)
			serr, err := cmd.StderrPipe()
			if err != nil {
				log.Fatalln(err)
			}
			err = cmd.Start()
			if err != nil {
				log.Fatalln(err)
			}
			s, err := ioutil.ReadAll(serr)
			s = bytes.TrimSpace(s)
			if bytes.HasPrefix(s, []byte("addr: ")) {
				fmt.Println(string(s))
				cmd.Process.Release()
			} else {
				log.Printf("unexpected response from MarGo: `%s` error: `%v`\n", s, err)
				cmd.Process.Kill()
			}
		}
		
- 另一種是利用syscall的方案，但是這個方案並不完善：

		package main
		 
		import (
			"log"
			"os"
			"syscall"
		)
		 
		func daemon(nochdir, noclose int) int {
			var ret, ret2 uintptr
			var err uintptr
		 
			darwin := syscall.OS == "darwin"
		 
			// already a daemon
			if syscall.Getppid() == 1 {
				return 0
			}
		 
			// fork off the parent process
			ret, ret2, err = syscall.RawSyscall(syscall.SYS_FORK, 0, 0, 0)
			if err != 0 {
				return -1
			}
		 
			// failure
			if ret2 < 0 {
				os.Exit(-1)
			}
		 
			// handle exception for darwin
			if darwin && ret2 == 1 {
				ret = 0
			}
		 
			// if we got a good PID, then we call exit the parent process.
			if ret > 0 {
				os.Exit(0)
			}
		 
			/* Change the file mode mask */
			_ = syscall.Umask(0)
		 
			// create a new SID for the child process
			s_ret, s_errno := syscall.Setsid()
			if s_errno != 0 {
				log.Printf("Error: syscall.Setsid errno: %d", s_errno)
			}
			if s_ret < 0 {
				return -1
			}
		 
			if nochdir == 0 {
				os.Chdir("/")
			}
		 
			if noclose == 0 {
				f, e := os.OpenFile("/dev/null", os.O_RDWR, 0)
				if e == nil {
					fd := f.Fd()
					syscall.Dup2(fd, os.Stdin.Fd())
					syscall.Dup2(fd, os.Stdout.Fd())
					syscall.Dup2(fd, os.Stderr.Fd())
				}
			}
		 
			return 0
		}	
	
上面提出了兩種實現Go的daemon方案，但是我還是不推薦大家這樣去實現，因為官方還沒有正式的宣佈支持daemon，當然第一種方案目前來看是比較可行的，而且目前開源庫skynet也在採用這個方案做daemon。

## Supervisord
上面已經介紹了Go目前是有兩種方案來實現他的daemon，但是官方本身還不支持這一塊，所以還是建議大家採用第三方成熟工具來管理我們的應用程序，這裡我給大家介紹一款目前使用比較廣泛的進程管理軟件：Supervisord。Supervisord是用Python實現的一款非常實用的進程管理工具。supervisord會幫你把管理的應用程序轉成daemon程序，而且可以方便的通過命令開啟、關閉、重啟等操作，而且它管理的進程一旦崩潰會自動重啟，這樣就可以保證程序執行中斷後的情況下有自我修復的功能。

>我前面在應用中踩過一個坑，就是因為所有的應用程序都是由Supervisord父進程生出來的，那麼當你修改了操作系統的文件描述符之後，別忘記重啟Supervisord，光重啟下面的應用程序沒用。當初我就是系統安裝好之後就先裝了Supervisord，然後開始部署程序，修改文件描述符，重啟程序，以為文件描述符已經是100000了，其實Supervisord這個時候還是默認的1024個，導致他管理的進程所有的描述符也是1024.開放之後壓力一上來系統就開始報文件描述符用光了，查了很久才找到這個坑。

### Supervisord安裝
Supervisord可以通過`sudo easy_install supervisor`安裝，當然也可以通過Supervisord官網下載後解壓並轉到源碼所在的文件夾下執行`setup.py install`來安裝。

- 使用easy_install必須安裝setuptools

	打開`http://pypi.python.org/pypi/setuptools#files`，根據你係統的python的版本下載相應的文件，然後執行`sh setuptoolsxxxx.egg`，這樣就可以使用easy_install命令來安裝Supervisord。

### Supervisord配置
Supervisord默認的配置文件路徑為/etc/supervisord.conf，通過文本編輯器修改這個文件，下面是一個示例的配置文件：

	;/etc/supervisord.conf
	[unix_http_server]
	file = /var/run/supervisord.sock
	chmod = 0777
	chown= root:root

	[inet_http_server]
	# Web管理界面設定
	port=9001
	username = admin
	password = yourpassword

	[supervisorctl]
	; 必須和'unix_http_server'裡面的設定匹配
	serverurl = unix:///var/run/supervisord.sock

	[supervisord]
	logfile=/var/log/supervisord/supervisord.log ; (main log file;default $CWD/supervisord.log)
	logfile_maxbytes=50MB       ; (max main logfile bytes b4 rotation;default 50MB)
	logfile_backups=10          ; (num of main logfile rotation backups;default 10)
	loglevel=info               ; (log level;default info; others: debug,warn,trace)
	pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
	nodaemon=true              ; (start in foreground if true;default false)
	minfds=1024                 ; (min. avail startup file descriptors;default 1024)
	minprocs=200                ; (min. avail process descriptors;default 200)
	user=root                 ; (default is current user, required if root)
	childlogdir=/var/log/supervisord/            ; ('AUTO' child log dir, default $TEMP)

	[rpcinterface:supervisor]
	supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

	; 管理的單個進程的配置，可以添加多個program
	[program:blogdemon]
	command=/data/blog/blogdemon
	autostart = true
	startsecs = 5
	user = root
	redirect_stderr = true
	stdout_logfile = /var/log/supervisord/blogdemon.log

### Supervisord管理
Supervisord安裝完成後有兩個可用的命令行supervisor和supervisorctl，命令使用解釋如下：

- supervisord，初始啟動Supervisord，啟動、管理配置中設置的進程。
- supervisorctl stop programxxx，停止某一個進程(programxxx)，programxxx為[program:blogdemon]裡配置的值，這個示例就是blogdemon。
- supervisorctl start programxxx，啟動某個進程
- supervisorctl restart programxxx，重啟某個進程
- supervisorctl stop all，停止全部進程，注：start、restart、stop都不會載入最新的配置文件。
- supervisorctl reload，載入最新的配置文件，並按新的配置啟動、管理所有進程。

## 小結
這小節我們介紹了Go如何實現daemon化，但是由於目前Go的daemon實現的不足，需要依靠第三方工具來實現應用程序的daemon管理的方式，所以在這裡介紹了一個用python寫的進程管理工具Supervisord，通過Supervisord可以很方便的把我們的Go應用程序管理起來。


## links
   * [目錄](<preface.md>)
   * 上一章: [網站錯誤處理](<12.2.md>)
   * 下一節: [備份和恢復](<12.4.md>)

---

# 12.4 備份和恢復
這小節我們要討論應用程序管理的另一個方面：生產服務器上數據的備份和恢復。我們經常會遇到生產服務器的網絡斷了、硬盤壞了、操作系統崩潰、或者數據庫不可用了等各種異常情況，所以維護人員需要對生產服務器上的應用和數據做好異地災備，冷備熱備的準備。在接下來的介紹中，講解了如何備份應用、如何備份/恢復Mysql數據庫和redis數據庫。

## 應用備份
在大多數集群環境下，Web應用程序基本不需要備份，因為這個其實就是一個代碼副本，我們在本地開發環境中，或者版本控制系統中已經保持這些代碼。但是很多時候，一些開發的站點需要用戶來上傳文件，那麼我們需要對這些用戶上傳的文件進行備份。目前其實有一種合適的做法就是把和網站相關的需要存儲的文件存儲到雲儲存，這樣即使系統崩潰，只要我們的文件還在雲存儲上，至少數據不會丟失。

如果我們沒有采用雲儲存的情況下，如何做到網站的備份呢？這裡我們介紹一個文件同步工具rsync：rsync能夠實現網站的備份，不同系統的文件的同步，如果是windows的話，需要windows版本cwrsync。

### rsync安裝
rysnc的官方網站：http://rsync.samba.org/ 可以從上面獲取最新版本的源碼。當然，因為rsync是一款非常有用的軟件，所以很多Linux的發行版本都將它收錄在內了。

軟件包安裝

	# sudo apt-get  install  rsync  注：在debian、ubuntu 等在線安裝方法；
	# yum install rsync    注：Fedora、Redhat、CentOS 等在線安裝方法；
	# rpm -ivh rsync       注：Fedora、Redhat、CentOS 等rpm包安裝方法；

其它Linux發行版，請用相應的軟件包管理方法來安裝。源碼包安裝

	tar xvf  rsync-xxx.tar.gz
	cd rsync-xxx
	./configure --prefix=/usr  ;make ;make install   注：在用源碼包編譯安裝之前，您得安裝gcc等編譯工具才行；

### rsync配置
rsync主要有以下三個配置文件rsyncd.conf(主配置文件)、rsyncd.secrets(密碼文件)、rsyncd.motd(rysnc服務器信息)。

關於這幾個文件的配置大家可以參考官方網站或者其他介紹rsync的網站，下面介紹服務器端和客戶端如何開啟

- 服務端開啟：

		#/usr/bin/rsync --daemon  --config=/etc/rsyncd.conf

	--daemon參數方式，是讓rsync以服務器模式運行。把rsync加入開機啟動

		echo 'rsync --daemon' >> /etc/rc.d/rc.local
		
	設置rsync密碼

		echo '你的用戶名:你的密碼' > /etc/rsyncd.secrets
		chmod 600 /etc/rsyncd.secrets


- 客戶端同步：

	客戶端可以通過如下命令同步服務器上的文件：
	
		rsync -avzP  --delete  --password-file=rsyncd.secrets   用戶名@192.168.145.5::www /var/rsync/backup
	
	這條命令，簡要的說明一下幾個要點：
	
	1. -avzP是啥，讀者可以使用--help查看
	2. --delete 是為了比如A上刪除了一個文件，同步的時候，B會自動刪除相對應的文件
	3. --password-file 客戶端中/etc/rsyncd.secrets設置的密碼，要和服務端的 /etc/rsyncd.secrets 中的密碼一樣，這樣cron運行的時候，就不需要密碼了
	4. 這條命令中的"用戶名"為服務端的 /etc/rsyncd.secrets中的用戶名
	5. 這條命令中的 192.168.145.5 為服務端的IP地址
	6. ::www，注意是2個 : 號，www為服務端的配置文件 /etc/rsyncd.conf 中的[www]，意思是根據服務端上的/etc/rsyncd.conf來同步其中的[www]段內容，一個 : 號的時候，用於不根據配置文件，直接同步指定目錄。
	
	為了讓同步實時性，可以設置crontab，保持rsync每分鐘同步，當然用戶也可以根據文件的重要程度設置不同的同步頻率。
	

## MySQL備份
應用數據庫目前還是MySQL為主流，目前MySQL的備份有兩種方式：熱備份和冷備份，熱備份目前主要是採用master/slave方式（master/slave方式的同步目前主要用於數據庫讀寫分離，也可以用於熱備份數據），關於如何配置這方面的資料，大家可以找到很多。冷備份的話就是數據有一定的延遲，但是可以保證該時間段之前的數據完整，例如有些時候可能我們的誤操作引起了數據的丟失，那麼master/slave模式是無法找回丟失數據的，但是通過冷備份可以部分恢復數據。

冷備份一般使用shell腳本來實現定時備份數據庫，然後通過上面介紹rsync同步非本地機房的一臺服務器。

下面這個是定時備份mysql的備份腳本，我們使用了mysqldump程序，這個命令可以把數據庫導出到一個文件中。

	#!/bin/bash

    # 以下配置信息請自己修改
    mysql_user="USER" #MySQL備份用戶
    mysql_password="PASSWORD" #MySQL備份用戶的密碼
    mysql_host="localhost"
    mysql_port="3306"
    mysql_charset="utf8" #MySQL編碼
    backup_db_arr=("db1" "db2") #要備份的數據庫名稱，多個用空格分開隔開 如("db1" "db2" "db3")
    backup_location=/var/www/mysql  #備份數據存放位置，末尾請不要帶"/",此項可以保持默認，程序會自動創建文件夾
    expire_backup_delete="ON" #是否開啟過期備份刪除 ON為開啟 OFF為關閉
    expire_days=3 #過期時間天數 默認為三天，此項只有在expire_backup_delete開啟時有效

    # 本行開始以下不需要修改
    backup_time=`date +%Y%m%d%H%M`  #定義備份詳細時間
    backup_Ymd=`date +%Y-%m-%d` #定義備份目錄中的年月日時間
    backup_3ago=`date -d '3 days ago' +%Y-%m-%d` #3天之前的日期
    backup_dir=$backup_location/$backup_Ymd  #備份文件夾全路徑
    welcome_msg="Welcome to use MySQL backup tools!" #歡迎語

    # 判斷MYSQL是否啟動,mysql沒有啟動則備份退出
    mysql_ps=`ps -ef |grep mysql |wc -l`
    mysql_listen=`netstat -an |grep LISTEN |grep $mysql_port|wc -l`
    if [ [$mysql_ps == 0] -o [$mysql_listen == 0] ]; then
            echo "ERROR:MySQL is not running! backup stop!"
            exit
    else
            echo $welcome_msg
    fi

    # 連接到mysql數據庫，無法連接則備份退出
    mysql -h$mysql_host -P$mysql_port -u$mysql_user -p$mysql_password <<end
    use mysql;
    select host,user from user where user='root' and host='localhost';
    exit
    end

    flag=`echo $?`
    if [ $flag != "0" ]; then
            echo "ERROR:Can't connect mysql server! backup stop!"
            exit
    else
            echo "MySQL connect ok! Please wait......"
            # 判斷有沒有定義備份的數據庫，如果定義則開始備份，否則退出備份
            if [ "$backup_db_arr" != "" ];then
                    #dbnames=$(cut -d ',' -f1-5 $backup_database)
                    #echo "arr is (${backup_db_arr[@]})"
                    for dbname in ${backup_db_arr[@]}
                    do
                            echo "database $dbname backup start..."
                            `mkdir -p $backup_dir`
                            `mysqldump -h$mysql_host -P$mysql_port -u$mysql_user -p$mysql_password $dbname --default-character-set=$mysql_charset | gzip > $backup_dir/$dbname-$backup_time.sql.gz`
                            flag=`echo $?`
                            if [ $flag == "0" ];then
                                    echo "database $dbname success backup to $backup_dir/$dbname-$backup_time.sql.gz"
                            else
                                    echo "database $dbname backup fail!"
                            fi
                            
                    done
            else
                    echo "ERROR:No database to backup! backup stop"
                    exit
            fi
            # 如果開啟了刪除過期備份，則進行刪除操作
            if [ "$expire_backup_delete" == "ON" -a  "$backup_location" != "" ];then
                     #`find $backup_location/ -type d -o -type f -ctime +$expire_days -exec rm -rf {} \;`
                     `find $backup_location/ -type d -mtime +$expire_days | xargs rm -rf`
                     echo "Expired backup data delete complete!"
            fi
            echo "All database backup success! Thank you!"
            exit
    fi
    
修改shell腳本的屬性：
    
	chmod 600 /root/mysql_backup.sh
	chmod +x /root/mysql_backup.sh

設置好屬性之後，把命令加入crontab，我們設置了每天00:00定時自動備份，然後把備份的腳本目錄/var/www/mysql設置為rsync同步目錄。

	00 00 * * * /root/mysql_backup.sh

## MySQL恢復
前面介紹MySQL備份分為熱備份和冷備份，熱備份主要的目的是為了能夠實時的恢復，例如應用服務器出現了硬盤故障，那麼我們可以通過修改配置文件把數據庫的讀取和寫入改成slave，這樣就可以儘量少時間的中斷服務。

但是有時候我們需要通過冷備份的SQL來進行數據恢復，既然有了數據庫的備份，就可以通過命令導入：

	mysql -u username -p databse < backup.sql
	
可以看到，導出和導入數據庫數據都是相當簡單，不過如果還需要管理權限，或者其他的一些字符集的設置的話，可能會稍微複雜一些，但是這些都是可以通過一些命令來完成的。

## redis備份
redis是目前我們使用最多的NoSQL，它的備份也分為兩種：熱備份和冷備份，redis也支持master/slave模式，所以我們的熱備份可以通過這種方式實現，相應的配置大家可以參考官方的文檔配置，相當的簡單。我們這裡介紹冷備份的方式：redis其實會定時的把內存裡面的緩存數據保存到數據庫文件裡面，我們備份只要備份相應的文件就可以，就是利用前面介紹的rsync備份到非本地機房就可以實現。

## redis恢復
redis的恢復分為熱備份恢復和冷備份恢復，熱備份恢復的目的和方法同MySQL的恢復一樣，只要修改應用的相應的數據庫連接即可。

但是有時候我們需要根據冷備份來恢復數據，redis的冷備份恢復其實就是隻要把保存的數據庫文件copy到redis的工作目錄，然後啟動redis就可以了，redis在啟動的時候會自動加載數據庫文件到內存中，啟動的速度根據數據庫的文件大小來決定。

## 小結
本小節介紹了我們的應用部分的備份和恢復，即如何做好災備，包括文件的備份、數據庫的備份。同時也介紹了使用rsync同步不同系統的文件，MySQL數據庫和redis數據庫的備份和恢復，希望通過本小節的介紹，能夠給作為開發的你對於線上產品的災備方案提供一個參考方案。 
 
## links
   * [目錄](<preface.md>)
   * 上一章: [應用部署](<12.3.md>)
   * 下一節: [小結](<12.5.md>)

---

# 12.5 小結
本章討論瞭如何部署和維護我們開發的Web應用相關的一些話題。這些內容非常重要，要創建一個能夠基於最小維護平滑運行的應用，必須考慮這些問題。

具體而言，本章討論的內容包括：

- 創建一個強健的日誌系統，可以在出現問題時記錄錯誤並且通知系統管理員
- 處理運行時可能出現的錯誤，包括記錄日誌，並如何友好的顯示給用戶系統出現了問題
- 處理404錯誤，告訴用戶請求的頁面找不到
- 將應用部署到一個生產環境中(包括如何部署更新)
- 如何讓部署的應用程序具有高可用
- 備份和恢復文件以及數據庫

讀完本章內容後，對於從頭開始開發一個Web應用需要考慮那些問題，你應該已經有了全面的瞭解。本章內容將有助於你在實際環境中管理前面各章介紹開發的代碼。

## links
   * [目錄](<preface.md>)
   * 上一章: [備份和恢復](<12.4.md>)
   * 下一節: [如何設計一個Web框架](<13.0.md>)