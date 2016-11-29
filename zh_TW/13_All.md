# 13 如何設計一個Web框架
前面十二章介紹瞭如何通過Go來開發Web應用，介紹了很多基礎知識、開發工具和開發技巧，那麼我們這一章通過這些知識來實現一個簡易的Web框架。通過Go語言來實現一個完整的框架設計，這框架中主要內容有第一小節介紹的Web框架的結構規劃，例如採用MVC模式來進行開發，程序的執行流程設計等內容；第二小節介紹框架的第一個功能：路由，如何讓訪問的URL映射到相應的處理邏輯；第三小節介紹處理邏輯，如何設計一個公共的controller，對象繼承之後處理函數中如何處理response和request；第四小節介紹框架的一些輔助功能，例如日誌處理、配置信息等；第五小節介紹如何基於Web框架實現一個博客，包括博文的發表、修改、刪除、顯示列表等操作。

通過這麼一個完整的項目例子，我期望能夠讓讀者瞭解如何開發Web應用，如何搭建自己的目錄結構，如何實現路由，如何實現MVC模式等各方面的開發內容。在框架盛行的今天，MVC也不再是神話。經常聽到很多程序員討論哪個框架好，哪個框架不好， 其實框架只是工具，沒有好與不好，只有適合與不適合，適合自己的就是最好的，所以教會大家自己動手寫框架，那麼不同的需求都可以用自己的思路去實現。

## 目錄
  ![](images/navi13.png?raw=true)

## links
   * [目錄](<preface.md>)
   * 上一章: [第十二章總結](<12.5.md>)
   * 下一節: [項目規劃](<13.1.md>)

---

# 13.1 項目規劃
做任何事情都需要做好規劃，那麼我們在開發博客系統之前，同樣需要做好項目的規劃，如何設置目錄結構，如何理解整個項目的流程圖，當我們理解了應用的執行過程，那麼接下來的設計編碼就會變得相對容易了
## gopath以及項目設置
假設指定gopath是文件系統的普通目錄名，當然我們可以隨便設置一個目錄名，然後將其路徑存入GOPATH。前面介紹過GOPATH可以是多個目錄：在window系統設置環境變量；在linux/MacOS系統只要輸入終端命令`export gopath=/home/astaxie/gopath`，但是必須保證gopath這個代碼目錄下面有三個目錄pkg、bin、src。新建項目的源碼放在src目錄下面，現在暫定我們的博客目錄叫做beeblog，下面是在window下的環境變量和目錄結構的截圖：

![](images/13.1.gopath.png?raw=true)

圖13.1 環境變量GOPATH設置

![](images/13.1.gopath2.png?raw=true)

圖13.2 工作目錄在$gopath/src下

## 應用程序流程圖
博客系統是基於模型-視圖-控制器這一設計模式的。MVC是一種將應用程序的邏輯層和表現層進行分離的結構方式。在實踐中，由於表現層從Go中分離了出來，所以它允許你的網頁中只包含很少的腳本。

- 模型 (Model) 代表數據結構。通常來說，模型類將包含取出、插入、更新數據庫資料等這些功能。
- 視圖 (View) 是展示給用戶的信息的結構及樣式。一個視圖通常是一個網頁，但是在Go中，一個視圖也可以是一個頁面片段，如頁頭、頁尾。它還可以是一個 RSS 頁面，或其它類型的“頁面”，Go實現的template包已經很好的實現了View層中的部分功能。
- 控制器 (Controller) 是模型、視圖以及其他任何處理HTTP請求所必須的資源之間的中介，並生成網頁。

下圖顯示了項目設計中框架的數據流是如何貫穿整個系統:

![](images/13.1.flow.png?raw=true)

圖13.3 框架的數據流

1. main.go作為應用入口，初始化一些運行博客所需要的基本資源，配置信息，監聽端口。
2. 路由功能檢查HTTP請求，根據URL以及method來確定誰(控制層)來處理請求的轉發資源。
3. 如果緩存文件存在，它將繞過通常的流程執行，被直接發送給瀏覽器。
4. 安全檢測：應用程序控制器調用之前，HTTP請求和任一用戶提交的數據將被過濾。
5. 控制器裝載模型、核心庫、輔助函數，以及任何處理特定請求所需的其它資源，控制器主要負責處理業務邏輯。
6. 輸出視圖層中渲染好的即將發送到Web瀏覽器中的內容。如果開啟緩存，視圖首先被緩存，將用於以後的常規請求。

## 目錄結構
根據上面的應用程序流程設計，博客的目錄結構設計如下：

	|——main.go         入口文件
	|——conf            配置文件和處理模塊
	|——controllers     控制器入口
	|——models          數據庫處理模塊
	|——utils           輔助函數庫
	|——static          靜態文件目錄
    |——views           視圖庫

## 框架設計
為了實現博客的快速搭建，打算基於上面的流程設計開發一個最小化的框架，框架包括路由功能、支持REST的控制器、自動化的模板渲染，日誌系統、配置管理等。

## 總結
本小節介紹了博客系統從設置GOPATH到目錄建立這樣的基礎信息，也簡單介紹了框架結構採用的MVC模式，博客系統中數據流的執行流程，最後通過這些流程設計了博客系統的目錄結構，至此，我們基本完成一個框架的搭建，接下來的幾個小節我們將會逐個實現。
## links
   * [目錄](<preface.md>)
   * 上一章: [構建博客系統](<13.0.md>)
   * 下一節: [自定義路由器設計](<13.2.md>)

---

# 13.2 自定義路由器設計

## HTTP路由
HTTP路由組件負責將HTTP請求交到對應的函數處理(或者是一個struct的方法)，如前面小節所描述的結構圖，路由在框架中相當於一個事件處理器，而這個事件包括：

- 用戶請求的路徑(path)(例如:/user/123,/article/123)，當然還有查詢串信息(例如?id=11)
- HTTP的請求方法(method)(GET、POST、PUT、DELETE、PATCH等)

路由器就是根據用戶請求的事件信息轉發到相應的處理函數(控制層)。
## 默認的路由實現
在3.4小節有過介紹Go的http包的詳解，裡面介紹了Go的http包如何設計和實現路由，這裡繼續以一個例子來說明：

	func fooHandler(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
	}

	http.HandleFunc("/foo", fooHandler)

	http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
	})

	log.Fatal(http.ListenAndServe(":8080", nil))
	
上面的例子調用了http默認的DefaultServeMux來添加路由，需要提供兩個參數，第一個參數是希望用戶訪問此資源的URL路徑(保存在r.URL.Path)，第二參數是即將要執行的函數，以提供用戶訪問的資源。路由的思路主要集中在兩點：

- 添加路由信息
- 根據用戶請求轉發到要執行的函數

Go默認的路由添加是通過函數`http.Handle`和`http.HandleFunc`等來添加，底層都是調用了`DefaultServeMux.Handle(pattern string, handler Handler)`,這個函數會把路由信息存儲在一個map信息中`map[string]muxEntry`，這就解決了上面說的第一點。

Go監聽端口，然後接收到tcp連接會扔給Handler來處理，上面的例子默認nil即為`http.DefaultServeMux`，通過`DefaultServeMux.ServeHTTP`函數來進行調度，遍歷之前存儲的map路由信息，和用戶訪問的URL進行匹配，以查詢對應註冊的處理函數，這樣就實現了上面所說的第二點。

	for k, v := range mux.m {
		if !pathMatch(k, path) {
			continue
		}
		if h == nil || len(k) > n {
			n = len(k)
			h = v.h
		}
	}


## beego框架路由實現
目前幾乎所有的Web應用路由實現都是基於http默認的路由器，但是Go自帶的路由器有幾個限制：

- 不支持參數設定，例如/user/:uid 這種泛類型匹配
- 無法很好的支持REST模式，無法限制訪問的方法，例如上面的例子中，用戶訪問/foo，可以用GET、POST、DELETE、HEAD等方式訪問
- 一般網站的路由規則太多了，編寫繁瑣。我前面自己開發了一個API應用，路由規則有三十幾條，這種路由多了之後其實可以進一步簡化，通過struct的方法進行一種簡化

beego框架的路由器基於上面的幾點限制考慮設計了一種REST方式的路由實現，路由設計也是基於上面Go默認設計的兩點來考慮：存儲路由和轉發路由

### 存儲路由
針對前面所說的限制點，我們首先要解決參數支持就需要用到正則，第二和第三點我們通過一種變通的方法來解決，REST的方法對應到struct的方法中去，然後路由到struct而不是函數，這樣在轉發路由的時候就可以根據method來執行不同的方法。

根據上面的思路，我們設計了兩個數據類型controllerInfo(保存路徑和對應的struct，這裡是一個reflect.Type類型)和ControllerRegistor(routers是一個slice用來保存用戶添加的路由信息，以及beego框架的應用信息)

	type controllerInfo struct {
		regex          *regexp.Regexp
		params         map[int]string
		controllerType reflect.Type
	}

	type ControllerRegistor struct {
		routers     []*controllerInfo
		Application *App
	}
	

ControllerRegistor對外的接口函數有

	func (p *ControllerRegistor) Add(pattern string, c ControllerInterface)

詳細的實現如下所示：

	func (p *ControllerRegistor) Add(pattern string, c ControllerInterface) {
		parts := strings.Split(pattern, "/")
	
		j := 0
		params := make(map[int]string)
		for i, part := range parts {
			if strings.HasPrefix(part, ":") {
				expr := "([^/]+)"

				//a user may choose to override the defult expression
				// similar to expressjs: ‘/user/:id([0-9]+)’
 
				if index := strings.Index(part, "("); index != -1 {
					expr = part[index:]
					part = part[:index]
				}
				params[j] = part
				parts[i] = expr
				j++
			}
		}
	
		//recreate the url pattern, with parameters replaced
		//by regular expressions. then compile the regex

		pattern = strings.Join(parts, "/")
		regex, regexErr := regexp.Compile(pattern)
		if regexErr != nil {

			//TODO add error handling here to avoid panic
			panic(regexErr)
			return
		}
	
		//now create the Route
		t := reflect.Indirect(reflect.ValueOf(c)).Type()
		route := &controllerInfo{}
		route.regex = regex
		route.params = params
		route.controllerType = t
	
		p.routers = append(p.routers, route)
	
	}
	
### 靜態路由實現
上面我們實現的動態路由的實現，Go的http包默認支持靜態文件處理FileServer，由於我們實現了自定義的路由器，那麼靜態文件也需要自己設定，beego的靜態文件夾路徑保存在全局變量StaticDir中，StaticDir是一個map類型，實現如下：

	func (app *App) SetStaticPath(url string, path string) *App {
		StaticDir[url] = path
		return app
	}

應用中設置靜態路徑可以使用如下方式實現：

	beego.SetStaticPath("/img","/static/img")
	

### 轉發路由
轉發路由是基於ControllerRegistor裡的路由信息來進行轉發的，詳細的實現如下代碼所示：

	// AutoRoute
	func (p *ControllerRegistor) ServeHTTP(w http.ResponseWriter, r *http.Request) {
		defer func() {
			if err := recover(); err != nil {
				if !RecoverPanic {
					// go back to panic
					panic(err)
				} else {
					Critical("Handler crashed with error", err)
					for i := 1; ; i += 1 {
						_, file, line, ok := runtime.Caller(i)
						if !ok {
							break
						}
						Critical(file, line)
					}
				}
			}
		}()
		var started bool
		for prefix, staticDir := range StaticDir {
			if strings.HasPrefix(r.URL.Path, prefix) {
				file := staticDir + r.URL.Path[len(prefix):]
				http.ServeFile(w, r, file)
				started = true
				return
			}
		}
		requestPath := r.URL.Path
	
		//find a matching Route
		for _, route := range p.routers {
	
			//check if Route pattern matches url
			if !route.regex.MatchString(requestPath) {
				continue
			}
	
			//get submatches (params)
			matches := route.regex.FindStringSubmatch(requestPath)
	
			//double check that the Route matches the URL pattern.
			if len(matches[0]) != len(requestPath) {
				continue
			}
	
			params := make(map[string]string)
			if len(route.params) > 0 {
				//add url parameters to the query param map
				values := r.URL.Query()
				for i, match := range matches[1:] {
					values.Add(route.params[i], match)
					params[route.params[i]] = match
				}
	
				//reassemble query params and add to RawQuery
				r.URL.RawQuery = url.Values(values).Encode() + "&" + r.URL.RawQuery
				//r.URL.RawQuery = url.Values(values).Encode()
			}
			//Invoke the request handler
			vc := reflect.New(route.controllerType)
			init := vc.MethodByName("Init")
			in := make([]reflect.Value, 2)
			ct := &Context{ResponseWriter: w, Request: r, Params: params}
			in[0] = reflect.ValueOf(ct)
			in[1] = reflect.ValueOf(route.controllerType.Name())
			init.Call(in)
			in = make([]reflect.Value, 0)
			method := vc.MethodByName("Prepare")
			method.Call(in)
			if r.Method == "GET" {
				method = vc.MethodByName("Get")
				method.Call(in)
			} else if r.Method == "POST" {
				method = vc.MethodByName("Post")
				method.Call(in)
			} else if r.Method == "HEAD" {
				method = vc.MethodByName("Head")
				method.Call(in)
			} else if r.Method == "DELETE" {
				method = vc.MethodByName("Delete")
				method.Call(in)
			} else if r.Method == "PUT" {
				method = vc.MethodByName("Put")
				method.Call(in)
			} else if r.Method == "PATCH" {
				method = vc.MethodByName("Patch")
				method.Call(in)
			} else if r.Method == "OPTIONS" {
				method = vc.MethodByName("Options")
				method.Call(in)
			}
			if AutoRender {
				method = vc.MethodByName("Render")
				method.Call(in)
			}
			method = vc.MethodByName("Finish")
			method.Call(in)
			started = true
			break
		}
	
		//if no matches to url, throw a not found exception
		if started == false {
			http.NotFound(w, r)
		}
	}

### 使用入門
基於這樣的路由設計之後就可以解決前面所說的三個限制點，使用的方式如下所示：

基本的使用註冊路由：

	beego.BeeApp.RegisterController("/", &controllers.MainController{})
	
參數註冊：

	beego.BeeApp.RegisterController("/:param", &controllers.UserController{})
	
正則匹配：

	beego.BeeApp.RegisterController("/users/:uid([0-9]+)", &controllers.UserController{})

## links
   * [目錄](<preface.md>)
   * 上一章: [項目規劃](<13.1.md>)
   * 下一節: [controller設計](<13.3.md>)
<!-- {% raw %} -->

---

# 13.3 controller設計

傳統的MVC框架大多數是基於Action設計的後綴式映射，然而，現在Web流行REST風格的架構。儘管使用Filter或者rewrite能夠通過URL重寫實現REST風格的URL，但是為什麼不直接設計一個全新的REST風格的 MVC框架呢？本小節就是基於這種思路來講述如何從頭設計一個基於REST風格的MVC框架中的controller，最大限度地簡化Web應用的開發，甚至編寫一行代碼就可以實現“Hello, world”。

## controller作用
MVC設計模式是目前Web應用開發中最常見的架構模式，通過分離 Model（模型）、View（視圖）和 Controller（控制器），可以更容易實現易於擴展的用戶界面(UI)。Model指後臺返回的數據；View指需要渲染的頁面，通常是模板頁面，渲染後的內容通常是HTML；Controller指Web開發人員編寫的處理不同URL的控制器，如前面小節講述的路由就是URL請求轉發到控制器的過程，controller在整個的MVC框架中起到了一個核心的作用，負責處理業務邏輯，因此控制器是整個框架中必不可少的一部分，Model和View對於有些業務需求是可以不寫的，例如沒有數據處理的邏輯處理，沒有頁面輸出的302調整之類的就不需要Model和View，但是controller這一環節是必不可少的。

## beego的REST設計
前面小節介紹了路由實現了註冊struct的功能，而struct中實現了REST方式，因此我們需要設計一個用於邏輯處理controller的基類，這裡主要設計了兩個類型，一個struct、一個interface

	type Controller struct {
		Ct        *Context
		Tpl       *template.Template
		Data      map[interface{}]interface{}
		ChildName string
		TplNames  string
		Layout    []string
		TplExt    string
	}

	type ControllerInterface interface {
		Init(ct *Context, cn string)    //初始化上下文和子類名稱
		Prepare()                       //開始執行之前的一些處理
		Get()                           //method=GET的處理
		Post()                          //method=POST的處理
		Delete()                        //method=DELETE的處理
		Put()                           //method=PUT的處理
		Head()                          //method=HEAD的處理
		Patch()                         //method=PATCH的處理
		Options()                       //method=OPTIONS的處理
		Finish()                        //執行完成之後的處理		
		Render() error                  //執行完method對應的方法之後渲染頁面
	}

那麼前面介紹的路由add函數的時候是定義了ControllerInterface類型，因此，只要我們實現這個接口就可以，所以我們的基類Controller實現如下的方法：

	func (c *Controller) Init(ct *Context, cn string) {
		c.Data = make(map[interface{}]interface{})
		c.Layout = make([]string, 0)
		c.TplNames = ""
		c.ChildName = cn
		c.Ct = ct
		c.TplExt = "tpl"
	}

	func (c *Controller) Prepare() {

	}

	func (c *Controller) Finish() {

	}

	func (c *Controller) Get() {
		http.Error(c.Ct.ResponseWriter, "Method Not Allowed", 405)
	}

	func (c *Controller) Post() {
		http.Error(c.Ct.ResponseWriter, "Method Not Allowed", 405)
	}

	func (c *Controller) Delete() {
		http.Error(c.Ct.ResponseWriter, "Method Not Allowed", 405)
	}

	func (c *Controller) Put() {
		http.Error(c.Ct.ResponseWriter, "Method Not Allowed", 405)
	}

	func (c *Controller) Head() {
		http.Error(c.Ct.ResponseWriter, "Method Not Allowed", 405)
	}

	func (c *Controller) Patch() {
		http.Error(c.Ct.ResponseWriter, "Method Not Allowed", 405)
	}

	func (c *Controller) Options() {
		http.Error(c.Ct.ResponseWriter, "Method Not Allowed", 405)
	}

	func (c *Controller) Render() error {
		if len(c.Layout) > 0 {
			var filenames []string
			for _, file := range c.Layout {
				filenames = append(filenames, path.Join(ViewsPath, file))
			}
			t, err := template.ParseFiles(filenames...)
			if err != nil {
				Trace("template ParseFiles err:", err)
			}
			err = t.ExecuteTemplate(c.Ct.ResponseWriter, c.TplNames, c.Data)
			if err != nil {
				Trace("template Execute err:", err)
			}
		} else {
			if c.TplNames == "" {
				c.TplNames = c.ChildName + "/" + c.Ct.Request.Method + "." + c.TplExt
			}
			t, err := template.ParseFiles(path.Join(ViewsPath, c.TplNames))
			if err != nil {
				Trace("template ParseFiles err:", err)
			}
			err = t.Execute(c.Ct.ResponseWriter, c.Data)
			if err != nil {
				Trace("template Execute err:", err)
			}
		}
		return nil
	}

	func (c *Controller) Redirect(url string, code int) {
		c.Ct.Redirect(code, url)
	}

上面的controller基類已經實現了接口定義的函數，通過路由根據url執行相應的controller的原則，會依次執行如下：

	Init()      初始化
	Prepare()   執行之前的初始化，每個繼承的子類可以來實現該函數
	method()    根據不同的method執行不同的函數：GET、POST、PUT、HEAD等，子類來實現這些函數，如果沒實現，那麼默認都是403
	Render()    可選，根據全局變量AutoRender來判斷是否執行
	Finish()    執行完之後執行的操作，每個繼承的子類可以來實現該函數

## 應用指南
上面beego框架中完成了controller基類的設計，那麼我們在我們的應用中可以這樣來設計我們的方法：

	package controllers

	import (
		"github.com/astaxie/beego"
	)

	type MainController struct {
		beego.Controller
	}

	func (this *MainController) Get() {
		this.Data["Username"] = "astaxie"
		this.Data["Email"] = "astaxie@gmail.com"
		this.TplNames = "index.tpl"
	}

上面的方式我們實現了子類MainController，實現了Get方法，那麼如果用戶通過其他的方式(POST/HEAD等)來訪問該資源都將返回403，而如果是Get來訪問，因為我們設置了AutoRender=true，那麼在執行完Get方法之後會自動執行Render函數，就會顯示如下界面：

![](images/13.4.beego.png?raw=true)

index.tpl的代碼如下所示，我們可以看到數據的設置和顯示都是相當的簡單方便：

	<!DOCTYPE html>
	<html>
	  <head>
	    <title>beego welcome template</title>
	  </head>
	  <body>
	    <h1>Hello, world!{{.Username}},{{.Email}}</h1>
	  </body>
	</html>


## links
   * [目錄](<preface.md>)
   * 上一章: [自定義路由器設計](<13.2.md>)
   * 下一節: [日誌和配置設計](<13.4.md>)
<!-- {% endraw %} -->

---

# 13.4 日誌和配置設計

## 日誌和配置的重要性
前面已經介紹過日誌在我們程序開發中起著很重要的作用，通過日誌我們可以記錄調試我們的信息，當初介紹過一個日誌系統seelog，根據不同的level輸出不同的日誌，這個對於程序開發和程序部署來說至關重要。我們可以在程序開發中設置level低一點，部署的時候把level設置高，這樣我們開發中的調試信息可以屏蔽掉。

配置模塊對於應用部署牽涉到服務器不同的一些配置信息非常有用，例如一些數據庫配置信息、監聽端口、監聽地址等都是可以通過配置文件來配置，這樣我們的應用程序就具有很強的靈活性，可以通過配置文件的配置部署在不同的機器上，可以連接不同的數據庫之類的。

## beego的日誌設計
beego的日誌設計部署思路來自於seelog，根據不同的level來記錄日誌，但是beego設計的日誌系統比較輕量級，採用了系統的log.Logger接口，默認輸出到os.Stdout,用戶可以實現這個接口然後通過beego.SetLogger設置自定義的輸出，詳細的實現如下所示：

	
	// Log levels to control the logging output.
	const (
		LevelTrace = iota
		LevelDebug
		LevelInfo
		LevelWarning
		LevelError
		LevelCritical
	)
	
	// logLevel controls the global log level used by the logger.
	var level = LevelTrace
	
	// LogLevel returns the global log level and can be used in
	// own implementations of the logger interface.
	func Level() int {
		return level
	}
	
	// SetLogLevel sets the global log level used by the simple
	// logger.
	func SetLevel(l int) {
		level = l
	}
	
上面這一段實現了日誌系統的日誌分級，默認的級別是Trace，用戶通過SetLevel可以設置不同的分級。		
	
	// logger references the used application logger.
	var BeeLogger = log.New(os.Stdout, "", log.Ldate|log.Ltime)
	
	// SetLogger sets a new logger.
	func SetLogger(l *log.Logger) {
		BeeLogger = l
	}
	
	// Trace logs a message at trace level.
	func Trace(v ...interface{}) {
		if level <= LevelTrace {
			BeeLogger.Printf("[T] %v\n", v)
		}
	}
	
	// Debug logs a message at debug level.
	func Debug(v ...interface{}) {
		if level <= LevelDebug {
			BeeLogger.Printf("[D] %v\n", v)
		}
	}
	
	// Info logs a message at info level.
	func Info(v ...interface{}) {
		if level <= LevelInfo {
			BeeLogger.Printf("[I] %v\n", v)
		}
	}
	
	// Warning logs a message at warning level.
	func Warn(v ...interface{}) {
		if level <= LevelWarning {
			BeeLogger.Printf("[W] %v\n", v)
		}
	}
	
	// Error logs a message at error level.
	func Error(v ...interface{}) {
		if level <= LevelError {
			BeeLogger.Printf("[E] %v\n", v)
		}
	}
	
	// Critical logs a message at critical level.
	func Critical(v ...interface{}) {
		if level <= LevelCritical {
			BeeLogger.Printf("[C] %v\n", v)
		}
	}

上面這一段代碼默認初始化了一個BeeLogger對象，默認輸出到os.Stdout，用戶可以通過beego.SetLogger來設置實現了logger的接口輸出。這裡面實現了六個函數：

- Trace（一般的記錄信息，舉例如下：）
	- "Entered parse function validation block"
	- "Validation: entered second 'if'"
	- "Dictionary 'Dict' is empty. Using default value"
- Debug（調試信息，舉例如下：）
	- "Web page requested: http://somesite.com Params='...'"
	- "Response generated. Response size: 10000. Sending."
	- "New file received. Type:PNG Size:20000"
- Info（打印信息，舉例如下：）
	- "Web server restarted"
	- "Hourly statistics: Requested pages: 12345 Errors: 123 ..."
	- "Service paused. Waiting for 'resume' call"
- Warn（警告信息，舉例如下：）
	- "Cache corrupted for file='test.file'. Reading from back-end"
	- "Database 192.168.0.7/DB not responding. Using backup 192.168.0.8/DB"
	- "No response from statistics server. Statistics not sent"
- Error（錯誤信息，舉例如下：）
	- "Internal error. Cannot process request #12345 Error:...."
	- "Cannot perform login: credentials DB not responding"
- Critical（致命錯誤，舉例如下：）
	- "Critical panic received: .... Shutting down"
	- "Fatal error: ... App is shutting down to prevent data corruption or loss"

可以看到每個函數裡面都有對level的判斷，所以如果我們在部署的時候設置了level=LevelWarning，那麼Trace、Debug、Info這三個函數都不會有任何的輸出，以此類推。

## beego的配置設計
配置信息的解析，beego實現了一個key=value的配置文件讀取，類似ini配置文件的格式，就是一個文件解析的過程，然後把解析的數據保存到map中，最後在調用的時候通過幾個string、int之類的函數調用返回相應的值，具體的實現請看下面：

首先定義了一些ini配置文件的一些全局性常量	：

	var (
		bComment = []byte{'#'}
		bEmpty   = []byte{}
		bEqual   = []byte{'='}
		bDQuote  = []byte{'"'}
	)

定義了配置文件的格式：	
	
	// A Config represents the configuration.
	type Config struct {
		filename string
		comment  map[int][]string  // id: []{comment, key...}; id 1 is for main comment.
		data     map[string]string // key: value
		offset   map[string]int64  // key: offset; for editing.
		sync.RWMutex
	}
	
定義瞭解析文件的函數，解析文件的過程是打開文件，然後一行一行的讀取，解析註釋、空行和key=value數據：	
	
	// ParseFile creates a new Config and parses the file configuration from the
	// named file.
	func LoadConfig(name string) (*Config, error) {
		file, err := os.Open(name)
		if err != nil {
			return nil, err
		}
	
		cfg := &Config{
			file.Name(),
			make(map[int][]string),
			make(map[string]string),
			make(map[string]int64),
			sync.RWMutex{},
		}
		cfg.Lock()
		defer cfg.Unlock()
		defer file.Close()
	
		var comment bytes.Buffer
		buf := bufio.NewReader(file)
	
		for nComment, off := 0, int64(1); ; {
			line, _, err := buf.ReadLine()
			if err == io.EOF {
				break
			}
			if bytes.Equal(line, bEmpty) {
				continue
			}
	
			off += int64(len(line))
	
			if bytes.HasPrefix(line, bComment) {
				line = bytes.TrimLeft(line, "#")
				line = bytes.TrimLeftFunc(line, unicode.IsSpace)
				comment.Write(line)
				comment.WriteByte('\n')
				continue
			}
			if comment.Len() != 0 {
				cfg.comment[nComment] = []string{comment.String()}
				comment.Reset()
				nComment++
			}
	
			val := bytes.SplitN(line, bEqual, 2)
			if bytes.HasPrefix(val[1], bDQuote) {
				val[1] = bytes.Trim(val[1], `"`)
			}
	
			key := strings.TrimSpace(string(val[0]))
			cfg.comment[nComment-1] = append(cfg.comment[nComment-1], key)
			cfg.data[key] = strings.TrimSpace(string(val[1]))
			cfg.offset[key] = off
		}
		return cfg, nil
	}

下面實現了一些讀取配置文件的函數，返回的值確定為bool、int、float64或string：
	
	// Bool returns the boolean value for a given key.
	func (c *Config) Bool(key string) (bool, error) {
		return strconv.ParseBool(c.data[key])
	}
	
	// Int returns the integer value for a given key.
	func (c *Config) Int(key string) (int, error) {
		return strconv.Atoi(c.data[key])
	}
	
	// Float returns the float value for a given key.
	func (c *Config) Float(key string) (float64, error) {
		return strconv.ParseFloat(c.data[key], 64)
	}
	
	// String returns the string value for a given key.
	func (c *Config) String(key string) string {
		return c.data[key]
	}

## 應用指南
下面這個函數是我一個應用中的例子，用來獲取遠程url地址的json數據，實現如下：

	func GetJson() {
		resp, err := http.Get(beego.AppConfig.String("url"))
		if err != nil {
			beego.Critical("http get info error")
			return
		}
		defer resp.Body.Close()
		body, err := ioutil.ReadAll(resp.Body)
		err = json.Unmarshal(body, &AllInfo)
		if err != nil {
			beego.Critical("error:", err)
		}
	}

函數中調用了框架的日誌函數`beego.Critical`函數用來報錯，調用了`beego.AppConfig.String("url")`用來獲取配置文件中的信息，配置文件的信息如下(app.conf)：

	appname = hs
	url ="http://www.api.com/api.html"
	

## links
   * [目錄](<preface.md>)
   * 上一章: [controller設計](<13.3.md>)
   * 下一節: [實現博客的增刪改](<13.5.md>)<!-- {% raw %} -->

---

# 13.5 實現博客的增刪改

前面介紹了beego框架實現的整體構思以及部分實現的偽代碼，這小節介紹通過beego建立一個博客系統，包括博客瀏覽、添加、修改、刪除等操作。
## 博客目錄
博客目錄如下所示：

	.
	├── controllers
	│   ├── delete.go
	│   ├── edit.go
	│   ├── index.go
	│   ├── new.go
	│   └── view.go
	├── main.go
	├── models
	│   └── model.go
	└── views
	    ├── edit.tpl
	    ├── index.tpl
	    ├── layout.tpl
	    ├── new.tpl
	    └── view.tpl

## 博客路由
博客主要的路由規則如下所示：

	//顯示博客首頁
	beego.Router("/", &controllers.IndexController{})
	//查看博客詳細信息
	beego.Router("/view/:id([0-9]+)", &controllers.ViewController{})
	//新建博客博文
	beego.Router("/new", &controllers.NewController{})
	//刪除博文
	beego.Router("/delete/:id([0-9]+)", &controllers.DeleteController{})
	//編輯博文
	beego.Router("/edit/:id([0-9]+)", &controllers.EditController{})


## 數據庫結構
數據庫設計最簡單的博客信息

	CREATE TABLE entries (
	    id INT AUTO_INCREMENT,
	    title TEXT,
	    content TEXT,
	    created DATETIME,
	    primary key (id)
	);

## 控制器
IndexController:

	type IndexController struct {
		beego.Controller
	}

	func (this *IndexController) Get() {
		this.Data["blogs"] = models.GetAll()
		this.Layout = "layout.tpl"
		this.TplNames = "index.tpl"
	}

ViewController:

	type ViewController struct {
		beego.Controller
	}

	func (this *ViewController) Get() {
		id, _ := strconv.Atoi(this.Ctx.Input.Params[":id"])
		this.Data["Post"] = models.GetBlog(id)
		this.Layout = "layout.tpl"
		this.TplNames = "view.tpl"
	}

NewController

	type NewController struct {
		beego.Controller
	}

	func (this *NewController) Get() {
		this.Layout = "layout.tpl"
		this.TplNames = "new.tpl"
	}

	func (this *NewController) Post() {
		inputs := this.Input()
		var blog models.Blog
		blog.Title = inputs.Get("title")
		blog.Content = inputs.Get("content")
		blog.Created = time.Now()
		models.SaveBlog(blog)
		this.Ctx.Redirect(302, "/")
	}		

EditController

	type EditController struct {
		beego.Controller
	}

	func (this *EditController) Get() {
		id, _ := strconv.Atoi(this.Ctx.Input.Params[":id"])
		this.Data["Post"] = models.GetBlog(id)
		this.Layout = "layout.tpl"
		this.TplNames = "edit.tpl"
	}

	func (this *EditController) Post() {
		inputs := this.Input()
		var blog models.Blog
		blog.Id, _ = strconv.Atoi(inputs.Get("id"))
		blog.Title = inputs.Get("title")
		blog.Content = inputs.Get("content")
		blog.Created = time.Now()
		models.SaveBlog(blog)
		this.Ctx.Redirect(302, "/")
	}

DeleteController

	type DeleteController struct {
		beego.Controller
	}

	func (this *DeleteController) Get() {
		id, _ := strconv.Atoi(this.Ctx.Input.Params[":id"])
		blog := models.GetBlog(id)
		this.Data["Post"] = blog
		models.DelBlog(blog)
		this.Ctx.Redirect(302, "/")
	}

## model層

	package models

	import (
		"database/sql"
		"github.com/astaxie/beedb"
		_ "github.com/ziutek/mymysql/godrv"
		"time"
	)

	type Blog struct {
		Id      int `PK`
		Title   string
		Content string
		Created time.Time
	}

	func GetLink() beedb.Model {
		db, err := sql.Open("mymysql", "blog/astaxie/123456")
		if err != nil {
			panic(err)
		}
		orm := beedb.New(db)
		return orm
	}

	func GetAll() (blogs []Blog) {
		db := GetLink()
		db.FindAll(&blogs)
		return
	}

	func GetBlog(id int) (blog Blog) {
		db := GetLink()
		db.Where("id=?", id).Find(&blog)
		return
	}

	func SaveBlog(blog Blog) (bg Blog) {
		db := GetLink()
		db.Save(&blog)
		return bg
	}

	func DelBlog(blog Blog) {
		db := GetLink()
		db.Delete(&blog)
		return
	}


## view層

layout.tpl

	<html>
	<head>
	    <title>My Blog</title>
	    <style>
	        #menu {
	            width: 200px;
	            float: right;
	        }
	    </style>
	</head>
	<body>

	<ul id="menu">
	    <li><a href="/">Home</a></li>
	    <li><a href="/new">New Post</a></li>
	</ul>

	{{.LayoutContent}}

	</body>
	</html>

index.tpl

	<h1>Blog posts</h1>

	<ul>
	{{range .blogs}}
	    <li>
	        <a href="/view/{{.Id}}">{{.Title}}</a>
	        from {{.Created}}
	        <a href="/edit/{{.Id}}">Edit</a>
	        <a href="/delete/{{.Id}}">Delete</a>
	    </li>
	{{end}}
	</ul>

view.tpl

	<h1>{{.Post.Title}}</h1>
	{{.Post.Created}}<br/>

	{{.Post.Content}}				

new.tpl

	<h1>New Blog Post</h1>
	<form action="" method="post">
	標題:<input type="text" name="title"><br>
	內容：<textarea name="content" colspan="3" rowspan="10"></textarea>
	<input type="submit">
	</form>

edit.tpl

	<h1>Edit {{.Post.Title}}</h1>

	<h1>New Blog Post</h1>
	<form action="" method="post">
	標題:<input type="text" name="title" value="{{.Post.Title}}"><br>
	內容：<textarea name="content" colspan="3" rowspan="10">{{.Post.Content}}</textarea>
	<input type="hidden" name="id" value="{{.Post.Id}}">
	<input type="submit">
	</form>

## links
   * [目錄](<preface.md>)
   * 上一章: [日誌和配置設計](<13.4.md>)
   * 下一節: [小結](<13.6.md>)
<!-- {% endraw %} -->

---

# 13.6 小結
這一章我們主要介紹瞭如何實現一個基礎的Go語言框架，框架包含有路由設計，由於Go內置的http包中路由的一些不足點，我們設計了動態路由規則，然後介紹了MVC模式中的Controller設計，controller實現了REST的實現，這個主要思路來源於tornado框架，然後設計實現了模板的layout以及自動化渲染等技術，主要採用了Go內置的模板引擎，最後我們介紹了一些輔助的日誌、配置等信息的設計，通過這些設計我們實現了一個基礎的框架beego，目前該框架已經開源在github，最後我們通過beego實現了一個博客系統，通過實例代碼詳細的展現瞭如何快速的開發一個站點。

## links
   * [目錄](<preface.md>)
   * 上一章: [實現博客的增刪改](<13.5.md>)
   * 下一節: [擴展Web框架](<14.0.md>)