# 14 擴展Web框架
第十三章介紹瞭如何開發一個Web框架，通過介紹MVC、路由、日誌處理、配置處理完成了一個基本的框架系統，但是一個好的框架需要一些方便的輔助工具來快速的開發Web，那麼我們這一章將就如何提供一些快速開發Web的工具進行介紹，第一小節介紹如何處理靜態文件，如何利用現有的twitter開源的bootstrap進行快速的開發美觀的站點，第二小節介紹如何利用前面介紹的session來進行用戶登錄處理，第三小節介紹如何方便的輸出表單、這些表單如何進行數據驗證，如何快速的結合model進行數據的增刪改操作，第四小節介紹如何進行一些用戶認證，包括http basic認證、http digest認證，第五小節介紹如何利用前面介紹的i18n支持多語言的應用開發。第六小節介紹瞭如何集成Go的pprof包用於性能調試。

通過本章的擴展，beego框架將具有快速開發Web的特性，最後我們將講解如何利用這些擴展的特性擴展開發第十三章開發的博客系統，通過開發一個完整、美觀的博客系統讓讀者瞭解beego開發帶給你的快速。

## 目錄
![](images/navi14.png?raw=true)

## links
   * [目錄](<preface.md>)
   * 上一章: [第十三章總結](<13.6.md>)
   * 下一節: [靜態文件支持](<14.1.md>)

---

# 14.1 靜態文件支持
我們在前面已經講過如何處理靜態文件，這小節我們詳細的介紹如何在beego裡面設置和使用靜態文件。通過再介紹一個twitter開源的html、css框架bootstrap，無需大量的設計工作就能夠讓你快速地建立一個漂亮的站點。

## beego靜態文件實現和設置
Go的net/http包中提供了靜態文件的服務，`ServeFile`和`FileServer`等函數。beego的靜態文件處理就是基於這一層處理的，具體的實現如下所示：

	//static file server
	for prefix, staticDir := range StaticDir {
		if strings.HasPrefix(r.URL.Path, prefix) {
			file := staticDir + r.URL.Path[len(prefix):]
			http.ServeFile(w, r, file)
			w.started = true
			return
		}
	}
	
StaticDir裡面保存的是相應的url對應到靜態文件所在的目錄，因此在處理URL請求的時候只需要判斷對應的請求地址是否包含靜態處理開頭的url，如果包含的話就採用http.ServeFile提供服務。

舉例如下：

	beego.StaticDir["/asset"] = "/static"

那麼請求url如`http://www.beego.me/asset/bootstrap.css`就會請求`/static/bootstrap.css`來提供反饋給客戶端。	

## bootstrap集成
Bootstrap是Twitter推出的一個開源的用於前端開發的工具包。對於開發者來說，Bootstrap是快速開發Web應用程序的最佳前端工具包。它是一個CSS和HTML的集合，它使用了最新的HTML5標準，給你的Web開發提供了時尚的版式，表單，按鈕，表格，網格系統等等。

- 組件
　　Bootstrap中包含了豐富的Web組件，根據這些組件，可以快速的搭建一個漂亮、功能完備的網站。其中包括以下組件：
　　下拉菜單、按鈕組、按鈕下拉菜單、導航、導航條、麵包屑、分頁、排版、縮略圖、警告對話框、進度條、媒體對象等
- Javascript插件
　　Bootstrap自帶了13個jQuery插件，這些插件為Bootstrap中的組件賦予了“生命”。其中包括：
　　模式對話框、標籤頁、滾動條、彈出框等。
- 定製自己的框架代碼
　　可以對Bootstrap中所有的CSS變量進行修改，依據自己的需求裁剪代碼。

![](images/14.1.bootstrap.png?raw=true)

圖14.1 bootstrap站點

接下來我們利用bootstrap集成到beego框架裡面來，快速的建立一個漂亮的站點。

1. 首先把下載的bootstrap目錄放到我們的項目目錄，取名為static，如下截圖所示

	![](images/14.1.bootstrap2.png?raw=true)
	
	圖14.2 項目中靜態文件目錄結構

2. 因為beego默認設置了StaticDir的值，所以如果你的靜態文件目錄是static的話就無須再增加了：

	StaticDir["/static"] = "static"
	
3. 模板中使用如下的地址就可以了：

		//css文件
		<link href="/static/css/bootstrap.css" rel="stylesheet">
		
		//js文件
		<script src="/static/js/bootstrap-transition.js"></script>
		
		//圖片文件
		<img src="/static/img/logo.png">

上面可以實現把bootstrap集成到beego中來，如下展示的圖就是集成進來之後的展現效果圖：

![](images/14.1.bootstrap3.png?raw=true)

圖14.3 構建的基於bootstrap的站點界面

這些模板和格式bootstrap官方都有提供，這邊就不再重複貼代碼，大家可以上bootstrap官方網站學習如何編寫模板。


## links
   * [目錄](<preface.md>)
   * 上一節: [擴展Web框架](<14.0.md>)
   * 下一節: [Session支持](<14.2.md>)

---

# 14.2 Session支持
第六章的時候我們介紹過如何在Go語言中使用session，也實現了一個sessionManger，beego框架基於sessionManager實現了方便的session處理功能。

## session集成
beego中主要有以下的全局變量來控制session處理：

	//related to session 
	SessionOn            bool   // 是否開啟session模塊，默認不開啟
	SessionProvider      string // session後端提供處理模塊，默認是sessionManager支持的memory
	SessionName          string // 客戶端保存的cookies的名稱
	SessionGCMaxLifetime int64  // cookies有效期

	GlobalSessions *session.Manager //全局session控制器
	
當然上面這些變量需要初始化值，也可以按照下面的代碼來配合配置文件以設置這些值：

	if ar, err := AppConfig.Bool("sessionon"); err != nil {
		SessionOn = false
	} else {
		SessionOn = ar
	}
	if ar := AppConfig.String("sessionprovider"); ar == "" {
		SessionProvider = "memory"
	} else {
		SessionProvider = ar
	}
	if ar := AppConfig.String("sessionname"); ar == "" {
		SessionName = "beegosessionID"
	} else {
		SessionName = ar
	}
	if ar, err := AppConfig.Int("sessiongcmaxlifetime"); err != nil && ar != 0 {
		int64val, _ := strconv.ParseInt(strconv.Itoa(ar), 10, 64)
		SessionGCMaxLifetime = int64val
	} else {
		SessionGCMaxLifetime = 3600
	}	
	
在beego.Run函數中增加如下代碼：

	if SessionOn {
		GlobalSessions, _ = session.NewManager(SessionProvider, SessionName, SessionGCMaxLifetime)
		go GlobalSessions.GC()
	}
	
這樣只要SessionOn設置為true，那麼就會默認開啟session功能，獨立開一個goroutine來處理session。

為了方便我們在自定義Controller中快速使用session，作者在`beego.Controller`中提供瞭如下方法：

	func (c *Controller) StartSession() (sess session.Session) {
		sess = GlobalSessions.SessionStart(c.Ctx.ResponseWriter, c.Ctx.Request)
		return
	}		

## session使用
通過上面的代碼我們可以看到，beego框架簡單地繼承了session功能，那麼在項目中如何使用呢？

首先我們需要在應用的main入口處開啟session：

	beego.SessionOn = true
	

然後我們就可以在控制器的相應方法中如下所示的使用session了：		

	func (this *MainController) Get() {
		var intcount int
		sess := this.StartSession()
		count := sess.Get("count")
		if count == nil {
			intcount = 0
		} else {
			intcount = count.(int)
		}
		intcount = intcount + 1
		sess.Set("count", intcount)
		this.Data["Username"] = "astaxie"
		this.Data["Email"] = "astaxie@gmail.com"
		this.Data["Count"] = intcount
		this.TplNames = "index.tpl"
	}
	
上面的代碼展示瞭如何在控制邏輯中使用session，主要分兩個步驟：

1. 獲取session對象
	
		//獲取對象,類似PHP中的session_start()
		sess := this.StartSession()

2. 使用session進行一般的session值操作
	
		//獲取session值，類似PHP中的$_SESSION["count"]
		sess.Get("count")
		
		//設置session值
		sess.Set("count", intcount)
	
從上面代碼可以看出基於beego框架開發的應用中使用session相當方便，基本上和PHP中調用`session_start()`類似。


## links
   * [目錄](<preface.md>)
   * 上一節: [靜態文件支持](<14.1.md>)
   * 下一節: [表單及驗證支持](<14.3.md>)<!-- {% raw %} -->

---

# 14.3 表單及驗證支持
在Web開發中對於這樣的一個流程可能很眼熟：

- 打開一個網頁顯示出表單。
- 用戶填寫並提交了表單。
- 如果用戶提交了一些無效的信息，或者可能漏掉了一個必填項，表單將會連同用戶的數據和錯誤問題的描述信息返回。
- 用戶再次填寫，繼續上一步過程，直到提交了一個有效的表單。

在接收端，腳本必須：

- 檢查用戶遞交的表單數據。
- 驗證數據是否為正確的類型，合適的標準。例如，如果一個用戶名被提交，它必須被驗證是否只包含了允許的字符。它必須有一個最小長度，不能超過最大長度。用戶名不能與已存在的他人用戶名重複，甚至是一個保留字等。
- 過濾數據並清理不安全字符，保證邏輯處理中接收的數據是安全的。
- 如果需要，預格式化數據（數據需要清除空白或者經過HTML編碼等等。）
- 準備好數據，插入數據庫。

儘管上面的過程並不是很複雜，但是通常情況下需要編寫很多代碼，而且為了顯示錯誤信息，在網頁中經常要使用多種不同的控制結構。創建表單驗證雖簡單，實施起來實在枯燥無味。

## 表單和驗證
對於開發者來說，一般開發過程都是相當複雜，而且大多是在重複一樣的工作。假設一個場景項目中忽然需要增加一個表單數據，那麼局部代碼的整個流程都需要修改。我們知道Go裡面struct是常用的一個數據結構，因此beego的form採用了struct來處理表單信息。

首先定義一個開發Web應用時相對應的struct，一個字段對應一個form元素，通過struct的tag來定義相應的元素信息和驗證信息，如下所示：

	type User struct{
		Username 	string 	`form:text,valid:required`
		Nickname 	string 	`form:text,valid:required`
		Age		int 	`form:text,valid:required|numeric`
		Email 		string 	`form:text,valid:required|valid_email`
		Introduce 	string 	`form:textarea`
	}

定義好struct之後接下來在controller中這樣操作

	func (this *AddController) Get() {
		this.Data["form"] = beego.Form(&User{})
		this.Layout = "admin/layout.html"
		this.TplNames = "admin/add.tpl"
	}		

在模板中這樣顯示錶單

	<h1>New Blog Post</h1>
	<form action="" method="post">
	{{.form.render()}}
	</form>

上面我們定義好了整個的第一步，從struct到顯示錶單的過程，接下來就是用戶填寫信息，服務器端接收數據然後驗證，最後插入數據庫。

	func (this *AddController) Post() {
		var user User
		form := this.GetInput(&user)
		if !form.Validates() {
			return
		}
		models.UserInsert(&user)
		this.Ctx.Redirect(302, "/admin/index")
	}		

## 表單類型
以下列表列出來了對應的form元素信息：
<table cellpadding="0" cellspacing="1" border="0" style="width:100%" class="tableborder">
<tbody><tr>
<th>名稱</th>
<th>參數</th>
<th>功能描述</th>
</tr>

<tr>
<td class="td"><strong>text</strong></td>
<td class="td">No</td>
<td class="td">textbox輸入框</td>
</tr>

<tr>
<td class="td"><strong>button</strong></td>
<td class="td">No</td>
<td class="td">按鈕</td>
</tr>

<tr>
<td class="td"><strong>checkbox</strong></td>
<td class="td">No</td>
<td class="td">多選擇框</td>
</tr>

<tr>
<td class="td"><strong>dropdown</strong></td>
<td class="td">No</td>
<td class="td">下拉選擇框</td>
</tr>

<tr>
<td class="td"><strong>file</strong></td>
<td class="td">No</td>
<td class="td">文件上傳</td>
</tr>

<tr>
<td class="td"><strong>hidden</strong></td>
<td class="td">No</td>
<td class="td">隱藏元素</td>
</tr>

<tr>
<td class="td"><strong>password</strong></td>
<td class="td">No</td>
<td class="td">密碼輸入框</td>
</tr>

<tr>
<td class="td"><strong>radio</strong></td>
<td class="td">No</td>
<td class="td">單選框</td>
</tr>

<tr>
<td class="td"><strong>textarea</strong></td>
<td class="td">No</td>
<td class="td">文本輸入框</td>
</tr>

</tbody></table>


## 表單驗證		
以下列表將列出可被使用的原生規則
<table cellpadding="0" cellspacing="1" border="0" style="width:100%" class="tableborder">
<tbody><tr>
<th>規則</th>
<th>參數</th>
<th>描述</th>
<th>舉例</th>
</tr>

<tr>
<td class="td"><strong>required</strong></td>
<td class="td">No</td>
<td class="td">如果元素為空，則返回FALSE</td>
<td class="td">&nbsp;</td>
</tr>

<tr>
<td class="td"><strong>matches</strong></td>
<td class="td">Yes</td>
<td class="td">如果表單元素的值與參數中對應的表單字段的值不相等，則返回FALSE</td>
<td class="td">matches[form_item]</td>
</tr>

  <tr>
    <td class="td"><strong>is_unique</strong></td>
    <td class="td">Yes</td>
    <td class="td">如果表單元素的值與指定數據表欄位有重複，則返回False（譯者注：比如is_unique[User.Email]，那麼驗證類會去查找User表中Email欄位有沒有與表單元素一樣的值，如存重複，則返回false，這樣開發者就不必另寫Callback驗證代碼。）</td>
    <td class="td">is_unique[table.field]</td>
  </tr>

<tr>
<td class="td"><strong>min_length</strong></td>
<td class="td">Yes</td>
<td class="td">如果表單元素值的字符長度少於參數中定義的數字，則返回FALSE</td>
<td class="td">min_length[6]</td>
</tr>

<tr>
<td class="td"><strong>max_length</strong></td>
<td class="td">Yes</td>
<td class="td">如果表單元素值的字符長度大於參數中定義的數字，則返回FALSE</td>
<td class="td">max_length[12]</td>
</tr>

<tr>
<td class="td"><strong>exact_length</strong></td>
<td class="td">Yes</td>
<td class="td">如果表單元素值的字符長度與參數中定義的數字不符，則返回FALSE</td>
<td class="td">exact_length[8]</td>
</tr>

  <tr>
    <td class="td"><strong>greater_than</strong></td>
    <td class="td">Yes</td>
    <td class="td">如果表單元素值是非數字類型，或小於參數定義的值，則返回FALSE</td>
    <td class="td">greater_than[8]</td>
  </tr>

  <tr>
    <td class="td"><strong>less_than</strong></td>
    <td class="td">Yes</td>
    <td class="td">如果表單元素值是非數字類型，或大於參數定義的值，則返回FALSE</td>
    <td class="td">less_than[8]</td>
  </tr>

<tr>
<td class="td"><strong>alpha</strong></td>
<td class="td">No</td>
<td class="td">如果表單元素值中包含除字母以外的其他字符，則返回FALSE</td>
<td class="td">&nbsp;</td>
</tr>

<tr>
<td class="td"><strong>alpha_numeric</strong></td>
<td class="td">No</td>
<td class="td">如果表單元素值中包含除字母和數字以外的其他字符，則返回FALSE</td>
<td class="td">&nbsp;</td>
</tr>

<tr>
<td class="td"><strong>alpha_dash</strong></td>
<td class="td">No</td>
<td class="td">如果表單元素值中包含除字母/數字/下劃線/破折號以外的其他字符，則返回FALSE</td>
<td class="td">&nbsp;</td>
</tr>

<tr>
<td class="td"><strong>numeric</strong></td>
<td class="td">No</td>
<td class="td">如果表單元素值中包含除數字以外的字符，則返回 FALSE</td>
<td class="td">&nbsp;</td>
</tr>

<tr>
<td class="td"><strong>integer</strong></td>
<td class="td">No</td>
<td class="td">如果表單元素中包含除整數以外的字符，則返回FALSE</td>
<td class="td">&nbsp;</td>
</tr>

  <tr>
    <td class="td"><strong>decimal</strong></td>
    <td class="td">Yes</td>
    <td class="td">如果表單元素中輸入（非小數）不完整的值，則返回FALSE</td>
    <td class="td">&nbsp;</td>
  </tr>

<tr>
<td class="td"><strong>is_natural</strong></td>
<td class="td">No</td>
<td class="td">如果表單元素值中包含了非自然數的其他數值 （其他數值不包括零），則返回FALSE。自然數形如：0,1,2,3....等等。</td>
<td class="td">&nbsp;</td>
</tr>

<tr>
<td class="td"><strong>is_natural_no_zero</strong></td>
<td class="td">No</td>
<td class="td">如果表單元素值包含了非自然數的其他數值 （其他數值包括零），則返回FALSE。非零的自然數：1,2,3.....等等。</td>
<td class="td">&nbsp;</td>
</tr>

<tr>
<td class="td"><strong>valid_email</strong></td>
<td class="td">No</td>
<td class="td">如果表單元素值包含不合法的email地址，則返回FALSE</td>
<td class="td">&nbsp;</td>
</tr>

<tr>
<td class="td"><strong>valid_emails</strong></td>
<td class="td">No</td>
<td class="td">如果表單元素值中任何一個值包含不合法的email地址（地址之間用英文逗號分割），則返回FALSE。</td>
<td class="td">&nbsp;</td>
</tr>

<tr>
<td class="td"><strong>valid_ip</strong></td>
<td class="td">No</td>
<td class="td">如果表單元素的值不是一個合法的IP地址，則返回FALSE。</td>
<td class="td">&nbsp;</td>
</tr>

<tr>
<td class="td"><strong>valid_base64</strong></td>
<td class="td">No</td>
<td class="td">如果表單元素的值包含除了base64 編碼字符之外的其他字符，則返回FALSE。</td>
<td class="td">&nbsp;</td>
</tr>

</tbody></table>


## links
   * [目錄](<preface.md>)
   * 上一節: [Session支持](<14.2.md>)
   * 下一節: [用戶認證](<14.4.md>)
<!-- {% endraw %} -->

---

# 14.4 用戶認證
在開發Web應用過程中，用戶認證是開發者經常遇到的問題，用戶登錄、註冊、登出等操作，而一般認證也分為三個方面的認證

- HTTP Basic和 HTTP Digest認證
- 第三方集成認證：QQ、微博、豆瓣、OPENID、google、github、facebook和twitter等
- 自定義的用戶登錄、註冊、登出，一般都是基於session、cookie認證

beego目前沒有針對這三種方式進行任何形式的集成，但是可以充分的利用第三方開源庫來實現上面的三種方式的用戶認證，不過後續beego會對前面兩種認證逐步集成。

## HTTP Basic和 HTTP Digest認證
這兩個認證是一些應用採用的比較簡單的認證，目前已經有開源的第三方庫支持這兩個認證：
	
	github.com/abbot/go-http-auth 

下面代碼演示瞭如何把這個庫引入beego中從而實現認證：

	package controllers
	
	import (
		"github.com/abbot/go-http-auth"
		"github.com/astaxie/beego"
	)
	
	func Secret(user, realm string) string {
		if user == "john" {
			// password is "hello"
			return "$1$dlPL2MqE$oQmn16q49SqdmhenQuNgs1"
		}
		return ""
	}
	
	type MainController struct {
		beego.Controller
	}
	
	func (this *MainController) Prepare() {
		a := auth.NewBasicAuthenticator("example.com", Secret)
		if username := a.CheckAuth(this.Ctx.Request); username == "" {
			a.RequireAuth(this.Ctx.ResponseWriter, this.Ctx.Request)
		}
	}
	
	func (this *MainController) Get() {
		this.Data["Username"] = "astaxie"
		this.Data["Email"] = "astaxie@gmail.com"
		this.TplNames = "index.tpl"
	}

上面代碼利用了beego的prepare函數，在執行正常邏輯之前調用了認證函數，這樣就非常簡單的實現了http auth，digest的認證也是同樣的原理。

## oauth和oauth2的認證
oauth和oauth2是目前比較流行的兩種認證方式，還好第三方有一個庫實現了這個認證，但是是國外實現的，並沒有QQ、微博之類的國內應用認證集成：

	github.com/bradrydzewski/go.auth

下面代碼演示瞭如何把該庫引入beego中從而實現oauth的認證，這裡以github為例演示：

1. 添加兩條路由

		beego.RegisterController("/auth/login", &controllers.GithubController{})
		beego.RegisterController("/mainpage", &controllers.PageController{})

2. 然後我們處理GithubController登陸的頁面：

		package controllers
		
		import (
			"github.com/astaxie/beego"
			"github.com/bradrydzewski/go.auth"
		)
		
		const (
			githubClientKey = "a0864ea791ce7e7bd0df"
			githubSecretKey = "a0ec09a647a688a64a28f6190b5a0d2705df56ca"
		)
		
		type GithubController struct {
			beego.Controller
		}
		
		func (this *GithubController) Get() {
			// set the auth parameters
			auth.Config.CookieSecret = []byte("7H9xiimk2QdTdYI7rDddfJeV")
			auth.Config.LoginSuccessRedirect = "/mainpage"
			auth.Config.CookieSecure = false
		
			githubHandler := auth.Github(githubClientKey, githubSecretKey)
		
			githubHandler.ServeHTTP(this.Ctx.ResponseWriter, this.Ctx.Request)
		}


3. 處理登陸成功之後的頁面

		package controllers
		
		import (
			"github.com/astaxie/beego"
			"github.com/bradrydzewski/go.auth"
			"net/http"
			"net/url"
		)
		
		type PageController struct {
			beego.Controller
		}
		
		func (this *PageController) Get() {
			// set the auth parameters
			auth.Config.CookieSecret = []byte("7H9xiimk2QdTdYI7rDddfJeV")
			auth.Config.LoginSuccessRedirect = "/mainpage"
			auth.Config.CookieSecure = false
		
			user, err := auth.GetUserCookie(this.Ctx.Request)
		
			//if no active user session then authorize user
			if err != nil || user.Id() == "" {
				http.Redirect(this.Ctx.ResponseWriter, this.Ctx.Request, auth.Config.LoginRedirect, http.StatusSeeOther)
				return
			}
		
			//else, add the user to the URL and continue
			this.Ctx.Request.URL.User = url.User(user.Id())
			this.Data["pic"] = user.Picture()
			this.Data["id"] = user.Id()
			this.Data["name"] = user.Name()
			this.TplNames = "home.tpl"
		}

整個的流程如下，首先打開瀏覽器輸入地址：

![](images/14.4.github.png?raw=true)

圖14.4 顯示帶有登錄按鈕的首頁

然後點擊鏈接出現如下界面：

![](images/14.4.github2.png?raw=true)

圖14.5 點擊登錄按鈕後顯示github的授權頁

然後點擊Authorize app就出現如下界面：

![](images/14.4.github3.png?raw=true)

圖14.6 授權登錄之後顯示的獲取到的github信息頁
																												
## 自定義認證
自定義的認證一般都是和session結合驗證的，如下代碼來源於一個基於beego的開源博客：


	//登陸處理
	func (this *LoginController) Post() {
		this.TplNames = "login.tpl"
		this.Ctx.Request.ParseForm()
		username := this.Ctx.Request.Form.Get("username")
		password := this.Ctx.Request.Form.Get("password")
		md5Password := md5.New()
		io.WriteString(md5Password, password)
		buffer := bytes.NewBuffer(nil)
		fmt.Fprintf(buffer, "%x", md5Password.Sum(nil))
		newPass := buffer.String()
	
		now := time.Now().Format("2006-01-02 15:04:05")
	
		userInfo := models.GetUserInfo(username)
		if userInfo.Password == newPass {
			var users models.User
			users.Last_logintime = now
			models.UpdateUserInfo(users)
	
			//登錄成功設置session
			sess := globalSessions.SessionStart(this.Ctx.ResponseWriter, this.Ctx.Request)
			sess.Set("uid", userInfo.Id)
			sess.Set("uname", userInfo.Username)
	
			this.Ctx.Redirect(302, "/")
		}	
	}
	
	//註冊處理
	func (this *RegController) Post() {
		this.TplNames = "reg.tpl"
		this.Ctx.Request.ParseForm()
		username := this.Ctx.Request.Form.Get("username")
		password := this.Ctx.Request.Form.Get("password")
		usererr := checkUsername(username)
		fmt.Println(usererr)
		if usererr == false {
			this.Data["UsernameErr"] = "Username error, Please to again"
			return
		}
	
		passerr := checkPassword(password)
		if passerr == false {
			this.Data["PasswordErr"] = "Password error, Please to again"
			return
		}
	
		md5Password := md5.New()
		io.WriteString(md5Password, password)
		buffer := bytes.NewBuffer(nil)
		fmt.Fprintf(buffer, "%x", md5Password.Sum(nil))
		newPass := buffer.String()
	
		now := time.Now().Format("2006-01-02 15:04:05")
	
		userInfo := models.GetUserInfo(username)
	
		if userInfo.Username == "" {
			var users models.User
			users.Username = username
			users.Password = newPass
			users.Created = now
			users.Last_logintime = now
			models.AddUser(users)
	
			//登錄成功設置session
			sess := globalSessions.SessionStart(this.Ctx.ResponseWriter, this.Ctx.Request)
			sess.Set("uid", userInfo.Id)
			sess.Set("uname", userInfo.Username)
			this.Ctx.Redirect(302, "/")
		} else {
			this.Data["UsernameErr"] = "User already exists"
		}
	
	}
	
	func checkPassword(password string) (b bool) {
		if ok, _ := regexp.MatchString("^[a-zA-Z0-9]{4,16}$", password); !ok {
			return false
		}
		return true
	}
	
	func checkUsername(username string) (b bool) {
		if ok, _ := regexp.MatchString("^[a-zA-Z0-9]{4,16}$", username); !ok {
			return false
		}
		return true
	}
	
有了用戶登陸和註冊之後，其他模塊的地方可以增加如下這樣的用戶是否登陸的判斷：

	func (this *AddBlogController) Prepare() {
		sess := globalSessions.SessionStart(this.Ctx.ResponseWriter, this.Ctx.Request)
		sess_uid := sess.Get("userid")
		sess_username := sess.Get("username")
		if sess_uid == nil {
			this.Ctx.Redirect(302, "/admin/login")
			return
		}
		this.Data["Username"] = sess_username
	}

## links
   * [目錄](<preface.md>)
   * 上一節: [表單及驗證支持](<14.3.md>)
   * 下一節: [多語言支持](<14.5.md>)<!-- {% raw %} -->

---

# 14.5 多語言支持
我們在第十章介紹過國際化和本地化，開發了一個go-i18n庫，這小節我們將把該庫集成到beego框架裡面來，使得我們的框架支持國際化和本地化。

## i18n集成
beego中設置全局變量如下：

	Translation	i18n.IL  
	Lang 		string  //設置語言包，zh、en
	LangPath	string  //設置語言包所在位置

初始化多語言函數:

	func InitLang(){
		beego.Translation:=i18n.NewLocale()
		beego.Translation.LoadPath(beego.LangPath)
		beego.Translation.SetLocale(beego.Lang)
	}

為了方便在模板中直接調用多語言包，我們設計了三個函數來處理響應的多語言：

	beegoTplFuncMap["Trans"] = i18n.I18nT
	beegoTplFuncMap["TransDate"] = i18n.I18nTimeDate
	beegoTplFuncMap["TransMoney"] = i18n.I18nMoney

	func I18nT(args ...interface{}) string {
	    ok := false
	    var s string
	    if len(args) == 1 {
	        s, ok = args[0].(string)
	    }
	    if !ok {
	        s = fmt.Sprint(args...)
	    }
	    return beego.Translation.Translate(s)
	}

	func I18nTimeDate(args ...interface{}) string {
	    ok := false
	    var s string
	    if len(args) == 1 {
	        s, ok = args[0].(string)
	    }
	    if !ok {
	        s = fmt.Sprint(args...)
	    }
	    return beego.Translation.Time(s)
	}

	func I18nMoney(args ...interface{}) string {
	    ok := false
	    var s string
	    if len(args) == 1 {
	        s, ok = args[0].(string)
	    }
	    if !ok {
	        s = fmt.Sprint(args...)
	    }
	    return beego.Translation.Money(s)
	}

## 多語言開發使用
1. 設置語言以及語言包所在位置，然後初始化i18n對象：

		beego.Lang = "zh"
		beego.LangPath = "views/lang"
		beego.InitLang()

2. 設計多語言包

	上面講了如何初始化多語言包，現在設計多語言包，多語言包是json文件，如第十章介紹的一樣，我們需要把設計的文件放在LangPath下面，例如zh.json或者en.json

		# zh.json

		{
		"zh": {
		    "submit": "提交",
		    "create": "創建"
		    }
		}

		#en.json

		{
		"en": {
		    "submit": "Submit",
		    "create": "Create"
		    }
		}

3. 使用語言包

	我們可以在controller中調用翻譯獲取響應的翻譯語言，如下所示：

		func (this *MainController) Get() {
			this.Data["create"] = beego.Translation.Translate("create")
			this.TplNames = "index.tpl"
		}

	我們也可以在模板中直接調用響應的翻譯函數：

		//直接文本翻譯
		{{.create | Trans}}

		//時間翻譯
		{{.time | TransDate}}

		//貨幣翻譯
		{{.money | TransMoney}}

## links
   * [目錄](<preface.md>)
   * 上一節: [用戶認證](<14.4.md>)
   * 下一節: [pprof支持](<14.6.md>)
<!-- {% endraw %} -->

---

# 14.6 pprof支持
Go語言有一個非常棒的設計就是標準庫裡面帶有代碼的性能監控工具，在兩個地方有包：

	net/http/pprof
	
	runtime/pprof

其實net/http/pprof中只是使用runtime/pprof包來進行封裝了一下，並在http端口上暴露出來

## beego支持pprof
目前beego框架新增了pprof，該特性默認是不開啟的，如果你需要測試性能，查看相應的執行goroutine之類的信息，其實Go的默認包"net/http/pprof"已經具有該功能，如果按照Go默認的方式執行Web，默認就可以使用，但是由於beego重新封裝了ServHTTP函數，默認的包是無法開啟該功能的，所以需要對beego的內部改造支持pprof。

- 首先在beego.Run函數中根據變量是否自動加載性能包

		if PprofOn {
			BeeApp.RegisterController(`/debug/pprof`, &ProfController{})
			BeeApp.RegisterController(`/debug/pprof/:pp([\w]+)`, &ProfController{})
		}
	
- 設計ProfConterller

		package beego

		import (
			"net/http/pprof"
		)
		
		type ProfController struct {
			Controller
		}
		
		func (this *ProfController) Get() {
			switch this.Ctx.Params[":pp"] {
			default:
				pprof.Index(this.Ctx.ResponseWriter, this.Ctx.Request)
			case "":
				pprof.Index(this.Ctx.ResponseWriter, this.Ctx.Request)
			case "cmdline":
				pprof.Cmdline(this.Ctx.ResponseWriter, this.Ctx.Request)
			case "profile":
				pprof.Profile(this.Ctx.ResponseWriter, this.Ctx.Request)
			case "symbol":
				pprof.Symbol(this.Ctx.ResponseWriter, this.Ctx.Request)
			}
			this.Ctx.ResponseWriter.WriteHeader(200)
		}
	

## 使用入門

通過上面的設計，你可以通過如下代碼開啟pprof：

	beego.PprofOn = true

然後你就可以在瀏覽器中打開如下URL就看到如下界面：
![](images/14.6.pprof.png?raw=true)

圖14.7 系統當前goroutine、heap、thread信息

點擊goroutine我們可以看到很多詳細的信息：

![](images/14.6.pprof2.png?raw=true)

圖14.8 顯示當前goroutine的詳細信息

我們還可以通過命令行獲取更多詳細的信息

	go tool pprof http://localhost:8080/debug/pprof/profile
	
這時候程序就會進入30秒的profile收集時間，在這段時間內拼命刷新瀏覽器上的頁面，儘量讓cpu佔用性能產生數據。

	(pprof) top10

	Total: 3 samples

       1 33.3% 33.3% 1 33.3% MHeap_AllocLocked

       1 33.3% 66.7% 1 33.3% os/exec.(*Cmd).closeDescriptors

       1 33.3% 100.0% 1 33.3% runtime.sigprocmask

       0 0.0% 100.0% 1 33.3% MCentral_Grow

       0 0.0% 100.0% 2 66.7% main.Compile

       0 0.0% 100.0% 2 66.7% main.compile

       0 0.0% 100.0% 2 66.7% main.run

       0 0.0% 100.0% 1 33.3% makeslice1

       0 0.0% 100.0% 2 66.7% net/http.(*ServeMux).ServeHTTP

       0 0.0% 100.0% 2 66.7% net/http.(*conn).serve	

	(pprof)web
	
![](images/14.6.pprof3.png?raw=true)

圖14.9 展示的執行流程信息

## links
   * [目錄](<preface.md>)
   * 上一節: [多語言支持](<14.5.md>)
   * 下一節: [小結](<14.7.md>)

---

# 14.7 小結
這一章主要闡述瞭如何基於beego框架進行擴展，這包括靜態文件的支持，靜態文件主要講述瞭如何利用beego進行快速的網站開發，利用bootstrap搭建漂亮的站點；第二小結講解了如何在beego中集成sessionManager，方便用戶在利用beego的時候快速的使用session；第三小結介紹了表單和驗證，基於Go語言的struct的定義使得我們在開發Web的過程中從重複的工作中解放出來，而且加入了驗證之後可以儘量做到數據安全，第四小結介紹了用戶認證，用戶認證主要有三方面的需求，http basic和http digest認證，第三方認證，自定義認證，通過代碼演示瞭如何利用現有的第三方包集成到beego應用中來實現這些認證；第五小節介紹了多語言的支持，beego中集成了go-i18n這個多語言包，用戶可以很方便的利用該庫開發多語言的Web應用；第六小節介紹瞭如何集成Go的pprof包，pprof包是用於性能調試的工具，通過對beego的改造之後集成了pprof包，使得用戶可以利用pprof測試基於beego開發的應用，通過這六個小節的介紹我們擴展出來了一個比較強壯的beego框架，這個框架足以應付目前大多數的Web應用，用戶可以繼續發揮自己的想象力去擴展，我這裡只是簡單的介紹了我能想的到的幾個比較重要的擴展。

## links
   * [目錄](<preface.md>)
   * 上一節: [pprof支持](<14.6.md>)