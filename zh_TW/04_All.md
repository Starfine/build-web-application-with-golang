# 4 表單

表單是我們平常編寫Web應用常用的工具，通過表單我們可以方便的讓客戶端和服務器進行數據的交互。對於以前開發過Web的用戶來說表單都非常熟悉，但是對於C/C++程序員來說，這可能是一個有些陌生的東西，那麼什麼是表單呢？

表單是一個包含表單元素的區域。表單元素是允許用戶在表單中（比如：文本域、下拉列表、單選框、複選框等等）輸入信息的元素。表單使用表單標籤（\<form\>）定義。

	<form>
	...
	input 元素
	...
	</form>

Go裡面對於form處理已經有很方便的方法了，在Request裡面的有專門的form處理，可以很方便的整合到Web開發裡面來，4.1小節裡面將講解Go如何處理表單的輸入。由於不能信任任何用戶的輸入，所以我們需要對這些輸入進行有效性驗證，4.2小節將就如何進行一些普通的驗證進行詳細的演示。

HTTP協議是一種無狀態的協議，那麼如何才能辨別是否是同一個用戶呢？同時又如何保證一個表單不出現多次遞交的情況呢？4.3和4.4小節裡面將對cookie(cookie是存儲在客戶端的信息，能夠每次通過header和服務器進行交互的數據)等進行詳細講解。

表單還有一個很大的功能就是能夠上傳文件，那麼Go是如何處理文件上傳的呢？針對大文件上傳我們如何有效的處理呢？4.5小節我們將一起學習Go處理文件上傳的知識。

## 目錄
![](images/navi4.png?raw=true)

## links
   * [目錄](<preface.md>)
   * 上一章: [第三章總結](<03.5.md>)
   * 下一節: [處理表單的輸入](<04.1.md>)

---

# 4.1 處理表單的輸入

先來看一個表單遞交的例子，我們有如下的表單內容，命名成文件login.gtpl(放入當前新建項目的目錄裡面)

	<html>
	<head>
	<title></title>
	</head>
	<body>
	<form action="/login" method="post">
		用戶名:<input type="text" name="username">
		密碼:<input type="password" name="password">
		<input type="submit" value="登陸">
	</form>
	</body>
	</html>

上面遞交表單到服務器的`/login`，當用戶輸入信息點擊登陸之後，會跳轉到服務器的路由`login`裡面，我們首先要判斷這個是什麼方式傳遞過來，POST還是GET呢？

http包裡面有一個很簡單的方式就可以獲取，我們在前面web的例子的基礎上來看看怎麼處理login頁面的form數據


	package main

	import (
		"fmt"
		"html/template"
		"log"
		"net/http"
		"strings"
	)

	func sayhelloName(w http.ResponseWriter, r *http.Request) {
		r.ParseForm()       //解析url傳遞的參數，對於POST則解析響應包的主體（request body）
		//注意:如果沒有調用ParseForm方法，下面無法獲取表單的數據
		fmt.Println(r.Form) //這些信息是輸出到服務器端的打印信息
		fmt.Println("path", r.URL.Path)
		fmt.Println("scheme", r.URL.Scheme)
		fmt.Println(r.Form["url_long"])
		for k, v := range r.Form {
			fmt.Println("key:", k)
			fmt.Println("val:", strings.Join(v, ""))
		}
		fmt.Fprintf(w, "Hello astaxie!") //這個寫入到w的是輸出到客戶端的
	}

	func login(w http.ResponseWriter, r *http.Request) {
		fmt.Println("method:", r.Method) //獲取請求的方法
		if r.Method == "GET" {
			t, _ := template.ParseFiles("login.gtpl")
			log.Println(t.Execute(w, nil))
		} else {
			//請求的是登陸數據，那麼執行登陸的邏輯判斷
			fmt.Println("username:", r.Form["username"])
			fmt.Println("password:", r.Form["password"])
		}
	}

	func main() {
		http.HandleFunc("/", sayhelloName)       //設置訪問的路由
		http.HandleFunc("/login", login)         //設置訪問的路由
		err := http.ListenAndServe(":9090", nil) //設置監聽的端口
		if err != nil {
			log.Fatal("ListenAndServe: ", err)
		}
	}


通過上面的代碼我們可以看出獲取請求方法是通過`r.Method`來完成的，這是個字符串類型的變量，返回GET, POST, PUT等method信息。

login函數中我們根據`r.Method`來判斷是顯示登錄界面還是處理登錄邏輯。當GET方式請求時顯示登錄界面，其他方式請求時則處理登錄邏輯，如查詢數據庫、驗證登錄信息等。

當我們在瀏覽器裡面打開`http://127.0.0.1:9090/login`的時候，出現如下界面

![](images/4.1.login.png?raw=true)

如果你看到一個空頁面，可能是你寫的 login.gtpl 文件中有錯誤，請根據控制檯中的日誌進行修復。

圖4.1 用戶登錄界面

我們輸入用戶名和密碼之後發現在服務器端是不會打印出來任何輸出的，為什麼呢？默認情況下，Handler裡面是不會自動解析form的，必須顯式的調用`r.ParseForm()`後，你才能對這個表單數據進行操作。我們修改一下代碼，在`fmt.Println("username:", r.Form["username"])`之前加一行`r.ParseForm()`,重新編譯，再次測試輸入遞交，現在是不是在服務器端有輸出你的輸入的用戶名和密碼了。

`r.Form`裡面包含了所有請求的參數，比如URL中query-string、POST的數據、PUT的數據，所有當你在URL的query-string字段和POST衝突時，會保存成一個slice，裡面存儲了多個值，Go官方文檔中說在接下來的版本里面將會把POST、GET這些數據分離開來。

現在我們修改一下login.gtpl裡面form的action值`http://127.0.0.1:9090/login`修改為`http://127.0.0.1:9090/login?username=astaxie`，再次測試，服務器的輸出username是不是一個slice。服務器端的輸出如下：

![](images/4.1.slice.png?raw=true)

圖4.2 服務器端打印接受到的信息

`request.Form`是一個url.Values類型，裡面存儲的是對應的類似`key=value`的信息，下面展示了可以對form數據進行的一些操作:

	v := url.Values{}
	v.Set("name", "Ava")
	v.Add("friend", "Jess")
	v.Add("friend", "Sarah")
	v.Add("friend", "Zoe")
	// v.Encode() == "name=Ava&friend=Jess&friend=Sarah&friend=Zoe"
	fmt.Println(v.Get("name"))
	fmt.Println(v.Get("friend"))
	fmt.Println(v["friend"])

>**Tips**: 
Request本身也提供了FormValue()函數來獲取用戶提交的參數。如r.Form["username"]也可寫成r.FormValue("username")。調用r.FormValue時會自動調用r.ParseForm，所以不必提前調用。r.FormValue只會返回同名參數中的第一個，若參數不存在則返回空字符串。

## links
   * [目錄](<preface.md>)
   * 上一節: [表單](<04.0.md>)
   * 下一節: [驗證表單的輸入](<04.2.md>)

---

# 4.2 驗證表單的輸入

開發Web的一個原則就是，不能信任用戶輸入的任何信息，所以驗證和過濾用戶的輸入信息就變得非常重要，我們經常會在微博、新聞中聽到某某網站被入侵了，存在什麼漏洞，這些大多是因為網站對於用戶輸入的信息沒有做嚴格的驗證引起的，所以為了編寫出安全可靠的Web程序，驗證表單輸入的意義重大。

我們平常編寫Web應用主要有兩方面的數據驗證，一個是在頁面端的js驗證(目前在這方面有很多的插件庫，比如ValidationJS插件)，一個是在服務器端的驗證，我們這小節講解的是如何在服務器端驗證。

## 必填字段
你想要確保從一個表單元素中得到一個值，例如前面小節裡面的用戶名，我們如何處理呢？Go有一個內置函數`len`可以獲取字符串的長度，這樣我們就可以通過len來獲取數據的長度，例如：

	if len(r.Form["username"][0])==0{
		//為空的處理
	}

`r.Form`對不同類型的表單元素的留空有不同的處理， 對於空文本框、空文本區域以及文件上傳，元素的值為空值,而如果是未選中的複選框和單選按鈕，則根本不會在r.Form中產生相應條目，如果我們用上面例子中的方式去獲取數據時程序就會報錯。所以我們需要通過`r.Form.Get()`來獲取值，因為如果字段不存在，通過該方式獲取的是空值。但是通過`r.Form.Get()`只能獲取單個的值，如果是map的值，必須通過上面的方式來獲取。

## 數字
你想要確保一個表單輸入框中獲取的只能是數字，例如，你想通過表單獲取某個人的具體年齡是50歲還是10歲，而不是像“一把年紀了”或“年輕著呢”這種描述

如果我們是判斷正整數，那麼我們先轉化成int類型，然後進行處理

	getint,err:=strconv.Atoi(r.Form.Get("age"))
	if err!=nil{
		//數字轉化出錯了，那麼可能就不是數字
	}

	//接下來就可以判斷這個數字的大小範圍了
	if getint >100 {
		//太大了
	}

還有一種方式就是正則匹配的方式

	if m, _ := regexp.MatchString("^[0-9]+$", r.Form.Get("age")); !m {
		return false
	}

對於性能要求很高的用戶來說，這是一個老生常談的問題了，他們認為應該儘量避免使用正則表達式，因為使用正則表達式的速度會比較慢。但是在目前機器性能那麼強勁的情況下，對於這種簡單的正則表達式效率和類型轉換函數是沒有什麼差別的。如果你對正則表達式很熟悉，而且你在其它語言中也在使用它，那麼在Go裡面使用正則表達式將是一個便利的方式。

>Go實現的正則是[RE2](http://code.google.com/p/re2/wiki/Syntax)，所有的字符都是UTF-8編碼的。

## 中文
有時候我們想通過表單元素獲取一個用戶的中文名字，但是又為了保證獲取的是正確的中文，我們需要進行驗證，而不是用戶隨便的一些輸入。對於中文我們目前有兩種方式來驗證，可以使用 `unicode` 包提供的 `func Is(rangeTab *RangeTable, r rune) bool` 來驗證，也可以使用正則方式來驗證，這裡使用最簡單的正則方式，如下代碼所示

	if m, _ := regexp.MatchString("^\\p{Han}+$", r.Form.Get("realname")); !m {
		return false
	}

## 英文
我們期望通過表單元素獲取一個英文值，例如我們想知道一個用戶的英文名，應該是astaxie，而不是asta謝。

我們可以很簡單的通過正則驗證數據：

	if m, _ := regexp.MatchString("^[a-zA-Z]+$", r.Form.Get("engname")); !m {
		return false
	}


## 電子郵件地址
你想知道用戶輸入的一個Email地址是否正確，通過如下這個方式可以驗證：

	if m, _ := regexp.MatchString(`^([\w\.\_]{2,10})@(\w{1,}).([a-z]{2,4})$`, r.Form.Get("email")); !m {
		fmt.Println("no")
	}else{
		fmt.Println("yes")
	}


## 手機號碼
你想要判斷用戶輸入的手機號碼是否正確，通過正則也可以驗證：

	if m, _ := regexp.MatchString(`^(1[3|4|5|8][0-9]\d{4,8})$`, r.Form.Get("mobile")); !m {
		return false
	}

## 下拉菜單
如果我們想要判斷表單裡面`<select>`元素生成的下拉菜單中是否有被選中的項目。有些時候黑客可能會偽造這個下拉菜單不存在的值發送給你，那麼如何判斷這個值是否是我們預設的值呢？

我們的select可能是這樣的一些元素

	<select name="fruit">
	<option value="apple">apple</option>
	<option value="pear">pear</option>
	<option value="banane">banane</option>
	</select>

那麼我們可以這樣來驗證

	slice:=[]string{"apple","pear","banane"}
	
	v := r.Form.Get("fruit")
	for item in slice {
		if item == v {
			return true
		}
	}
	
	return false

## 單選按鈕
如果我們想要判斷radio按鈕是否有一個被選中了，我們頁面的輸出可能就是一個男、女性別的選擇，但是也可能一個15歲大的無聊小孩，一手拿著http協議的書，另一隻手通過telnet客戶端向你的程序在發送請求呢，你設定的性別男值是1，女是2，他給你發送一個3，你的程序會出現異常嗎？因此我們也需要像下拉菜單的判斷方式類似，判斷我們獲取的值是我們預設的值，而不是額外的值。

	<input type="radio" name="gender" value="1">男
	<input type="radio" name="gender" value="2">女

那我們也可以類似下拉菜單的做法一樣

	slice:=[]int{1,2}

	for _, v := range slice {
		if v == r.Form.Get("gender") {
			return true
		}
	}
	return false

## 複選框
有一項選擇興趣的複選框，你想確定用戶選中的和你提供給用戶選擇的是同一個類型的數據。

	<input type="checkbox" name="interest" value="football">足球
	<input type="checkbox" name="interest" value="basketball">籃球
	<input type="checkbox" name="interest" value="tennis">網球

對於複選框我們的驗證和單選有點不一樣，因為接收到的數據是一個slice

	slice:=[]string{"football","basketball","tennis"}
	a:=Slice_diff(r.Form["interest"],slice)
	if a == nil{
		return true
	}

	return false

上面這個函數`Slice_diff`包含在我開源的一個庫裡面(操作slice和map的庫)，[https://github.com/astaxie/beeku](https://github.com/astaxie/beeku)

## 日期和時間
你想確定用戶填寫的日期或時間是否有效。例如
，用戶在日程表中安排8月份的第45天開會，或者提供未來的某個時間作為生日。

Go裡面提供了一個time的處理包，我們可以把用戶的輸入年月日轉化成相應的時間，然後進行邏輯判斷

	t := time.Date(2009, time.November, 10, 23, 0, 0, 0, time.UTC)
	fmt.Printf("Go launched at %s\n", t.Local())

獲取time之後我們就可以進行很多時間函數的操作。具體的判斷就根據自己的需求調整。

## 身份證號碼
如果我們想驗證表單輸入的是否是身份證，通過正則也可以方便的驗證，但是身份證有15位和18位，我們兩個都需要驗證

	//驗證15位身份證，15位的是全部數字
	if m, _ := regexp.MatchString(`^(\d{15})$`, r.Form.Get("usercard")); !m {
		return false
	}

	//驗證18位身份證，18位前17位為數字，最後一位是校驗位，可能為數字或字符X。
	if m, _ := regexp.MatchString(`^(\d{17})([0-9]|X)$`, r.Form.Get("usercard")); !m {
		return false
	}

上面列出了我們一些常用的服務器端的表單元素驗證，希望通過這個引導入門，能夠讓你對Go的數據驗證有所瞭解，特別是Go裡面的正則處理。

## links
   * [目錄](<preface.md>)
   * 上一節: [處理表單的輸入](<04.1.md>)
   * 下一節: [預防跨站腳本](<04.3.md>)

---

# 4.3 預防跨站腳本

現在的網站包含大量的動態內容以提高用戶體驗，比過去要複雜得多。所謂動態內容，就是根據用戶環境和需要，Web應用程序能夠輸出相應的內容。動態站點會受到一種名為“跨站腳本攻擊”（Cross Site Scripting, 安全專家們通常將其縮寫成 XSS）的威脅，而靜態站點則完全不受其影響。

攻擊者通常會在有漏洞的程序中插入JavaScript、VBScript、 ActiveX或Flash以欺騙用戶。一旦得手，他們可以盜取用戶帳戶信息，修改用戶設置，盜取/汙染cookie和植入惡意廣告等。

對XSS最佳的防護應該結合以下兩種方法：一是驗證所有輸入數據，有效檢測攻擊(這個我們前面小節已經有過介紹);另一個是對所有輸出數據進行適當的處理，以防止任何已成功注入的腳本在瀏覽器端運行。

那麼Go裡面是怎麼做這個有效防護的呢？Go的html/template裡面帶有下面幾個函數可以幫你轉義

- func HTMLEscape(w io.Writer, b []byte)  //把b進行轉義之後寫到w
- func HTMLEscapeString(s string) string  //轉義s之後返回結果字符串
- func HTMLEscaper(args ...interface{}) string //支持多個參數一起轉義，返回結果字符串


我們看4.1小節的例子

	fmt.Println("username:", template.HTMLEscapeString(r.Form.Get("username"))) //輸出到服務器端
	fmt.Println("password:", template.HTMLEscapeString(r.Form.Get("password")))
	template.HTMLEscape(w, []byte(r.Form.Get("username"))) //輸出到客戶端

如果我們輸入的username是`<script>alert()</script>`,那麼我們可以在瀏覽器上面看到輸出如下所示：

![](images/4.3.escape.png?raw=true)

圖4.3 Javascript過濾之後的輸出

Go的html/template包默認幫你過濾了html標籤，但是有時候你只想要輸出這個`<script>alert()</script>`看起來正常的信息，該怎麼處理？請使用text/template。請看下面的例子：

	import "text/template"
	...
	t, err := template.New("foo").Parse(`{{define "T"}}Hello, {{.}}!{{end}}`)
	err = t.ExecuteTemplate(out, "T", "<script>alert('you have been pwned')</script>")

輸出

	Hello, <script>alert('you have been pwned')</script>!

或者使用template.HTML類型

	import "html/template"
	...
	t, err := template.New("foo").Parse(`{{define "T"}}Hello, {{.}}!{{end}}`)
	err = t.ExecuteTemplate(out, "T", template.HTML("<script>alert('you have been pwned')</script>"))

輸出

	Hello, <script>alert('you have been pwned')</script>!

轉換成`template.HTML`後，變量的內容也不會被轉義

轉義的例子：

	import "html/template"
	...
	t, err := template.New("foo").Parse(`{{define "T"}}Hello, {{.}}!{{end}}`)
	err = t.ExecuteTemplate(out, "T", "<script>alert('you have been pwned')</script>")

轉義之後的輸出：

	Hello, &lt;script&gt;alert(&#39;you have been pwned&#39;)&lt;/script&gt;!



## links
   * [目錄](<preface.md>)
   * 上一節: [驗證的輸入](<04.2.md>)
   * 下一節: [防止多次遞交表單](<04.4.md>)

---

# 4.4 防止多次遞交表單

不知道你是否曾經看到過一個論壇或者博客，在一個帖子或者文章後面出現多條重複的記錄，這些大多數是因為用戶重複遞交了留言的表單引起的。由於種種原因，用戶經常會重複遞交表單。通常這只是鼠標的誤操作，如雙擊了遞交按鈕，也可能是為了編輯或者再次核對填寫過的信息，點擊了瀏覽器的後退按鈕，然後又再次點擊了遞交按鈕而不是瀏覽器的前進按鈕。當然，也可能是故意的——比如，在某項在線調查或者博彩活動中重複投票。那我們如何有效的防止用戶多次遞交相同的表單呢？

解決方案是在表單中添加一個帶有唯一值的隱藏字段。在驗證表單時，先檢查帶有該惟一值的表單是否已經遞交過了。如果是，拒絕再次遞交；如果不是，則處理表單進行邏輯處理。另外，如果是採用了Ajax模式遞交表單的話，當表單遞交後，通過javascript來禁用表單的遞交按鈕。

我繼續拿4.2小節的例子優化：

	<input type="checkbox" name="interest" value="football">足球
	<input type="checkbox" name="interest" value="basketball">籃球
	<input type="checkbox" name="interest" value="tennis">網球	
	用戶名:<input type="text" name="username">
	密碼:<input type="password" name="password">
	<input type="hidden" name="token" value="{{.}}">
	<input type="submit" value="登陸">

我們在模版裡面增加了一個隱藏字段`token`，這個值我們通過MD5(時間戳)來獲取惟一值，然後我們把這個值存儲到服務器端(session來控制，我們將在第六章講解如何保存)，以方便表單提交時比對判定。

	func login(w http.ResponseWriter, r *http.Request) {
		fmt.Println("method:", r.Method) //獲取請求的方法
		if r.Method == "GET" {
			crutime := time.Now().Unix()
			h := md5.New()
			io.WriteString(h, strconv.FormatInt(crutime, 10))
			token := fmt.Sprintf("%x", h.Sum(nil))

			t, _ := template.ParseFiles("login.gtpl")
			t.Execute(w, token)
		} else {
			//請求的是登陸數據，那麼執行登陸的邏輯判斷
			r.ParseForm()
			token := r.Form.Get("token")
			if token != "" {
				//驗證token的合法性
			} else {
				//不存在token報錯
			}
			fmt.Println("username length:", len(r.Form["username"][0]))
			fmt.Println("username:", template.HTMLEscapeString(r.Form.Get("username"))) //輸出到服務器端
			fmt.Println("password:", template.HTMLEscapeString(r.Form.Get("password")))
			template.HTMLEscape(w, []byte(r.Form.Get("username"))) //輸出到客戶端
		}
	}

上面的代碼輸出到頁面的源碼如下：

![](images/4.4.token.png?raw=true)

圖4.4 增加token之後在客戶端輸出的源碼信息

我們看到token已經有輸出值，你可以不斷的刷新，可以看到這個值在不斷的變化。這樣就保證了每次顯示form表單的時候都是唯一的，用戶遞交的表單保持了唯一性。

我們的解決方案可以防止非惡意的攻擊，並能使惡意用戶暫時不知所措，然後，它卻不能排除所有的欺騙性的動機，對此類情況還需要更復雜的工作。

## links
   * [目錄](<preface.md>)
   * 上一節: [預防跨站腳本](<04.3.md>)
   * 下一節: [處理文件上傳](<04.5.md>)

---

# 4.5 處理文件上傳
你想處理一個由用戶上傳的文件，比如你正在建設一個類似Instagram的網站，你需要存儲用戶拍攝的照片。這種需求該如何實現呢？

要使表單能夠上傳文件，首先第一步就是要添加form的`enctype`屬性，`enctype`屬性有如下三種情況:

	application/x-www-form-urlencoded   表示在發送前編碼所有字符（默認）
	multipart/form-data	  不對字符編碼。在使用包含文件上傳控件的表單時，必須使用該值。
	text/plain	  空格轉換為 "+" 加號，但不對特殊字符編碼。

所以，創建新的表單html文件, 命名為upload.gtpl, html代碼應該類似於:

	<html>
	<head>
		<title>上傳文件</title>
	</head>
	<body>
	<form enctype="multipart/form-data" action="/upload" method="post">
	  <input type="file" name="uploadfile" />
	  <input type="hidden" name="token" value="{{.}}"/>
	  <input type="submit" value="upload" />
	</form>
	</body>
	</html>

在服務器端，我們增加一個handlerFunc:

	http.HandleFunc("/upload", upload)

	// 處理/upload 邏輯
	func upload(w http.ResponseWriter, r *http.Request) {
		fmt.Println("method:", r.Method) //獲取請求的方法
		if r.Method == "GET" {
			crutime := time.Now().Unix()
			h := md5.New()
			io.WriteString(h, strconv.FormatInt(crutime, 10))
			token := fmt.Sprintf("%x", h.Sum(nil))

			t, _ := template.ParseFiles("upload.gtpl")
			t.Execute(w, token)
		} else {
			r.ParseMultipartForm(32 << 20)
			file, handler, err := r.FormFile("uploadfile")
			if err != nil {
				fmt.Println(err)
				return
			}
			defer file.Close()
			fmt.Fprintf(w, "%v", handler.Header)
			f, err := os.OpenFile("./test/"+handler.Filename, os.O_WRONLY|os.O_CREATE, 0666)  // 此處假設當前目錄下已存在test目錄
			if err != nil {
				fmt.Println(err)
				return
			}
			defer f.Close()
			io.Copy(f, file)
		}
	}

通過上面的代碼可以看到，處理文件上傳我們需要調用`r.ParseMultipartForm`，裡面的參數表示`maxMemory`，調用`ParseMultipartForm`之後，上傳的文件存儲在`maxMemory`大小的內存裡面，如果文件大小超過了`maxMemory`，那麼剩下的部分將存儲在系統的臨時文件中。我們可以通過`r.FormFile`獲取上面的文件句柄，然後實例中使用了`io.Copy`來存儲文件。

>獲取其他非文件字段信息的時候就不需要調用`r.ParseForm`，因為在需要的時候Go自動會去調用。而且`ParseMultipartForm`調用一次之後，後面再次調用不會再有效果。

通過上面的實例我們可以看到我們上傳文件主要三步處理：

1. 表單中增加enctype="multipart/form-data"
2. 服務端調用`r.ParseMultipartForm`,把上傳的文件存儲在內存和臨時文件中
3. 使用`r.FormFile`獲取文件句柄，然後對文件進行存儲等處理。

文件handler是multipart.FileHeader,裡面存儲瞭如下結構信息

	type FileHeader struct {
		Filename string
		Header   textproto.MIMEHeader
		// contains filtered or unexported fields
	}

我們通過上面的實例代碼打印出來上傳文件的信息如下

![](images/4.5.upload2.png?raw=true)

圖4.5 打印文件上傳後服務器端接受的信息

## 客戶端上傳文件

我們上面的例子演示瞭如何通過表單上傳文件，然後在服務器端處理文件，其實Go支持模擬客戶端表單功能支持文件上傳，詳細用法請看如下示例：

	package main

	import (
		"bytes"
		"fmt"
		"io"
		"io/ioutil"
		"mime/multipart"
		"net/http"
		"os"
	)

	func postFile(filename string, targetUrl string) error {
		bodyBuf := &bytes.Buffer{}
		bodyWriter := multipart.NewWriter(bodyBuf)

		//關鍵的一步操作
		fileWriter, err := bodyWriter.CreateFormFile("uploadfile", filename)
		if err != nil {
			fmt.Println("error writing to buffer")
			return err
		}

		//打開文件句柄操作
		fh, err := os.Open(filename)
		if err != nil {
			fmt.Println("error opening file")
			return err
		}
		defer fh.Close()
		
		//iocopy
		_, err = io.Copy(fileWriter, fh)
		if err != nil {
			return err
		}

		contentType := bodyWriter.FormDataContentType()
		bodyWriter.Close()

		resp, err := http.Post(targetUrl, contentType, bodyBuf)
		if err != nil {
			return err
		}
		defer resp.Body.Close()
		resp_body, err := ioutil.ReadAll(resp.Body)
		if err != nil {
			return err
		}
		fmt.Println(resp.Status)
		fmt.Println(string(resp_body))
		return nil
	}

	// sample usage
	func main() {
		target_url := "http://localhost:9090/upload"
		filename := "./astaxie.pdf"
		postFile(filename, target_url)
	}


上面的例子詳細展示了客戶端如何向服務器上傳一個文件的例子，客戶端通過multipart.Write把文件的文本流寫入一個緩存中，然後調用http的Post方法把緩存傳到服務器。

>如果你還有其他普通字段例如username之類的需要同時寫入，那麼可以調用multipart的WriteField方法寫很多其他類似的字段。

## links
   * [目錄](<preface.md>)
   * 上一節: [防止多次遞交表單](<04.4.md>)
   * 下一節: [小結](<04.6.md>)

---

# 4.6 小結
這一章裡面我們學習了Go如何處理表單信息，我們通過用戶登陸、上傳文件的例子展示了Go處理form表單信息及上傳文件的手段。但是在處理表單過程中我們需要驗證用戶輸入的信息，考慮到網站安全的重要性，數據過濾就顯得相當重要了，因此後面的章節中專門寫了一個小節來講解了不同方面的數據過濾，順帶講一下Go對字符串的正則處理。

通過這一章能夠讓你瞭解客戶端和服務器端是如何進行數據上的交互，客戶端將數據傳遞給服務器系統，服務器接受數據又把處理結果反饋給客戶端。

## links
   * [目錄](<preface.md>)
   * 上一節: [處理文件上傳](<04.5.md>)
   * 下一章: [訪問數據庫](<05.0.md>)
