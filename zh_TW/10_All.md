# 10 國際化和本地化
為了適應經濟的全球一體化，作為開發者，我們需要開發出支持多國語言、國際化的Web應用，即同樣的頁面在不同的語言環境下需要顯示不同的效果，也就是說應用程序在運行時能夠根據請求所來自的地域與語言的不同而顯示不同的用戶界面。這樣，當需要在應用程序中添加對新的語言的支持時，無需修改應用程序的代碼，只需要增加語言包即可實現。

國際化與本地化（Internationalization and localization,通常用i18n和L10N表示），國際化是將針對某個地區設計的程序進行重構，以使它能夠在更多地區使用，本地化是指在一個面向國際化的程序中增加對新地區的支持。

目前，Go語言的標準包沒有提供對i18n的支持，但有一些比較簡單的第三方實現，這一章我們將實現一個go-i18n庫，用來支持Go語言的i18n。

所謂的國際化：就是根據特定的locale信息，提取與之相應的字符串或其它一些東西（比如時間和貨幣的格式）等等。這涉及到三個問題：

1、如何確定locale。

2、如何保存與locale相關的字符串或其它信息。

3、如何根據locale提取字符串和其它相應的信息。

在第一小節裡，我們將介紹如何設置正確的locale以便讓訪問站點的用戶能夠獲得與其語言相應的頁面。第二小節將介紹如何處理或存儲字符串、貨幣、時間日期等與locale相關的信息，第三小節將介紹如何實現國際化站點，即如何根據不同locale返回不同合適的內容。通過這三個小節的學習，我們將獲得一個完整的i18n方案。

## 目錄

  ![](images/navi10.png?raw=true)

## links
   * [目錄](<preface.md>)
   * 上一章: [第九章總結](<09.7.md>)
   * 下一節: [設置默認地區](<10.1.md>)

---

# 10.1 設置默認地區
## 什麼是Locale
Locale是一組描述世界上某一特定區域文本格式和語言習慣的設置的集合。locale名通常由三個部分組成：第一部分，是一個強制性的，表示語言的縮寫，例如"en"表示英文或"zh"表示中文。第二部分，跟在一個下劃線之後，是一個可選的國家說明符，用於區分講同一種語言的不同國家，例如"en_US"表示美國英語，而"en_UK"表示英國英語。最後一部分，跟在一個句點之後，是可選的字符集說明符，例如"zh_CN.gb2312"表示中國使用gb2312字符集。

GO語言默認採用"UTF-8"編碼集，所以我們實現i18n時不考慮第三部分，接下來我們都採用locale描述的前面兩部分來作為i18n標準的locale名。


>在Linux和Solaris系統中可以通過`locale -a`命令列舉所有支持的地區名，讀者可以看到這些地區名的命名規範。對於BSD等系統，沒有locale命令，但是地區信息存儲在/usr/share/locale中。

## 設置Locale
有了上面對locale的定義，那麼我們就需要根據用戶的信息(訪問信息、個人信息、訪問域名等)來設置與之相關的locale，我們可以通過如下幾種方式來設置用戶的locale。

### 通過域名設置Locale
設置Locale的辦法之一是在應用運行的時候採用域名分級的方式，例如，我們採用www.asta.com當做我們的英文站(默認站)，而把域名www.asta.cn當做中文站。這樣通過在應用裡面設置域名和相應的locale的對應關係，就可以設置好地區。這樣處理有幾點好處：

- 通過URL就可以很明顯的識別
- 用戶可以通過域名很直觀的知道將訪問那種語言的站點
- 在Go程序中實現非常的簡單方便，通過一個map就可以實現
- 有利於搜索引擎抓取，能夠提高站點的SEO

我們可以通過下面的代碼來實現域名的對應locale：

	if r.Host == "www.asta.com" {
		i18n.SetLocale("en")
	} else if r.Host == "www.asta.cn" {
		i18n.SetLocale("zh-CN")
	} else if r.Host == "www.asta.tw" {
		i18n.SetLocale("zh-TW")
	}

當然除了整域名設置地區之外，我們還可以通過子域名來設置地區，例如"en.asta.com"表示英文站點，"cn.asta.com"表示中文站點。實現代碼如下所示：

	prefix := strings.Split(r.Host,".")

	if prefix[0] == "en" {
		i18n.SetLocale("en")
	} else if prefix[0] == "cn" {
		i18n.SetLocale("zh-CN")
	} else if prefix[0] == "tw" {
		i18n.SetLocale("zh-TW")
	}

通過域名設置Locale有如上所示的優點，但是我們一般開發Web應用的時候不會採用這種方式，因為首先域名成本比較高，開發一個Locale就需要一個域名，而且往往統一名稱的域名不一定能申請的到，其次我們不願意為每個站點去本地化一個配置，而更多的是採用url後面帶參數的方式，請看下面的介紹。

### 從域名參數設置Locale
目前最常用的設置Locale的方式是在URL裡面帶上參數，例如www.asta.com/hello?locale=zh或者www.asta.com/zh/hello。這樣我們就可以設置地區：`i18n.SetLocale(params["locale"])`。

這種設置方式幾乎擁有前面講的通過域名設置Locale的所有優點，它採用RESTful的方式，以使得我們不需要增加額外的方法來處理。但是這種方式需要在每一個的link裡面增加相應的參數locale，這也許有點複雜而且有時候甚至相當的繁瑣。不過我們可以寫一個通用的函數url，讓所有的link地址都通過這個函數來生成，然後在這個函數裡面增加`locale=params["locale"]`參數來緩解一下。

也許我們希望URL地址看上去更加的RESTful一點，例如：www.asta.com/en/books(英文站點)和www.asta.com/zh/books(中文站點)，這種方式的URL更加有利於SEO，而且對於用戶也比較友好，能夠通過URL直觀的知道訪問的站點。那麼這樣的URL地址可以通過router來獲取locale(參考REST小節裡面介紹的router插件實現)：

	mux.Get("/:locale/books", listbook)

### 從客戶端設置地區
在一些特殊的情況下，我們需要根據客戶端的信息而不是通過URL來設置Locale，這些信息可能來自於客戶端設置的喜好語言(瀏覽器中設置)，用戶的IP地址，用戶在註冊的時候填寫的所在地信息等。這種方式比較適合Web為基礎的應用。

- Accept-Language

客戶端請求的時候在HTTP頭信息裡面有`Accept-Language`，一般的客戶端都會設置該信息，下面是Go語言實現的一個簡單的根據`Accept-Language`實現設置地區的代碼：

	AL := r.Header.Get("Accept-Language")
	if AL == "en" {
		i18n.SetLocale("en")
	} else if AL == "zh-CN" {
		i18n.SetLocale("zh-CN")
	} else if AL == "zh-TW" {
		i18n.SetLocale("zh-TW")
	}

當然在實際應用中，可能需要更加嚴格的判斷來進行設置地區
- IP地址

	另一種根據客戶端來設定地區就是用戶訪問的IP，我們根據相應的IP庫，對應訪問的IP到地區，目前全球比較常用的就是GeoIP Lite Country這個庫。這種設置地區的機制非常簡單，我們只需要根據IP數據庫查詢用戶的IP然後返回國家地區，根據返回的結果設置對應的地區。

- 用戶profile

	當然你也可以讓用戶根據你提供的下拉菜單或者別的什麼方式的設置相應的locale，然後我們將用戶輸入的信息，保存到與它帳號相關的profile中，當用戶再次登陸的時候把這個設置複寫到locale設置中，這樣就可以保證該用戶每次訪問都是基於自己先前設置的locale來獲得頁面。

## 總結
通過上面的介紹可知，設置Locale可以有很多種方式，我們應該根據需求的不同來選擇不同的設置Locale的方法，以讓用戶能以它最熟悉的方式，獲得我們提供的服務，提高應用的用戶友好性。

## links
  * [目錄](<preface.md>)
  * 上一節: [國際化和本地化](<10.0.md>)
  * 下一節: [本地化資源](<10.2.md>)
<!-- {% raw %} -->

---

# 10.2 本地化資源
前面小節我們介紹瞭如何設置Locale，設置好Locale之後我們需要解決的問題就是如何存儲相應的Locale對應的信息呢？這裡面的信息包括：文本信息、時間和日期、貨幣值、圖片、包含文件以及視圖等資源。那麼接下來我們將對這些信息一一進行介紹，Go語言中我們把這些格式信息存儲在JSON中，然後通過合適的方式展現出來。(接下來以中文和英文兩種語言對比舉例,存儲格式文件en.json和zh-CN.json)
## 本地化文本消息
文本信息是編寫Web應用中最常用到的，也是本地化資源中最多的信息，想要以適合本地語言的方式來顯示文本信息，可行的一種方案是:建立需要的語言相應的map來維護一個key-value的關係，在輸出之前按需從適合的map中去獲取相應的文本，如下是一個簡單的示例：

	package main

	import "fmt"

	var locales map[string]map[string]string

	func main() {
		locales = make(map[string]map[string]string, 2)
		en := make(map[string]string, 10)
		en["pea"] = "pea"
		en["bean"] = "bean"
		locales["en"] = en
		cn := make(map[string]string, 10)
		cn["pea"] = "豌豆"
		cn["bean"] = "毛豆"
		locales["zh-CN"] = cn
		lang := "zh-CN"
		fmt.Println(msg(lang, "pea"))
		fmt.Println(msg(lang, "bean"))
	}

	func msg(locale, key string) string {
		if v, ok := locales[locale]; ok {
			if v2, ok := v[key]; ok {
				return v2
			}
		}
		return ""
	}


上面示例演示了不同locale的文本翻譯，實現了中文和英文對於同一個key顯示不同語言的實現，上面實現了中文的文本消息，如果想切換到英文版本，只需要把lang設置為en即可。

有些時候僅是key-value替換是不能滿足需要的，例如"I am 30 years old",中文表達是"我今年30歲了"，而此處的30是一個變量，該怎麼辦呢？這個時候，我們可以結合`fmt.Printf`函數來實現，請看下面的代碼：

	en["how old"] ="I am %d years old"
	cn["how old"] ="我今年%d歲了"

	fmt.Printf(msg(lang, "how old"), 30)

上面的示例代碼僅用以演示內部的實現方案，而實際數據是存儲在JSON裡面的，所以我們可以通過`json.Unmarshal`來為相應的map填充數據。

## 本地化日期和時間
因為時區的關係，同一時刻，在不同的地區，表示是不一樣的，而且因為Locale的關係，時間格式也不盡相同，例如中文環境下可能顯示：`2012年10月24日 星期三 23時11分13秒 CST`，而在英文環境下可能顯示:`Wed Oct 24 23:11:13 CST 2012`。這裡面我們需要解決兩點:

1. 時區問題
2. 格式問題

$GOROOT/lib/time包中的timeinfo.zip含有locale對應的時區的定義，為了獲得對應於當前locale的時間，我們應首先使用`time.LoadLocation(name string)`獲取相應於地區的locale，比如`Asia/Shanghai`或`America/Chicago`對應的時區信息，然後再利用此信息與調用`time.Now`獲得的Time對象協作來獲得最終的時間。詳細的請看下面的例子(該例子採用上面例子的一些變量):

	en["time_zone"]="America/Chicago"
	cn["time_zone"]="Asia/Shanghai"

	loc,_:=time.LoadLocation(msg(lang,"time_zone"))
	t:=time.Now()
	t = t.In(loc)
	fmt.Println(t.Format(time.RFC3339))

我們可以通過類似處理文本格式的方式來解決時間格式的問題，舉例如下:

	en["date_format"]="%Y-%m-%d %H:%M:%S"
	cn["date_format"]="%Y年%m月%d日 %H時%M分%S秒"

	fmt.Println(date(msg(lang,"date_format"),t))

	func date(fomate string,t time.Time) string{
		year, month, day = t.Date()
		hour, min, sec = t.Clock()
		//解析相應的%Y %m %d %H %M %S然後返回信息
		//%Y 替換成2012
		//%m 替換成10
		//%d 替換成24
	}

## 本地化貨幣值
各個地區的貨幣表示也不一樣，處理方式也與日期差不多，細節請看下面代碼:

	en["money"] ="USD %d"
	cn["money"] ="￥%d元"

	fmt.Println(date(msg(lang,"date_format"),100))

	func money_format(fomate string,money int64) string{
		return fmt.Sprintf(fomate,money)
	}


## 本地化視圖和資源
我們可能會根據Locale的不同來展示視圖，這些視圖包含不同的圖片、css、js等各種靜態資源。那麼應如何來處理這些信息呢？首先我們應按locale來組織文件信息，請看下面的文件目錄安排：

	views
	|--en  //英文模板
		|--images     //存儲圖片信息
		|--js         //存儲JS文件
		|--css        //存儲css文件
		index.tpl     //用戶首頁
		login.tpl     //登陸首頁
	|--zh-CN //中文模板
		|--images
		|--js
		|--css
		index.tpl
		login.tpl

有了這個目錄結構後我們就可以在渲染的地方這樣來實現代碼：


	s1, _ := template.ParseFiles("views"+lang+"index.tpl")
	VV.Lang=lang
	s1.Execute(os.Stdout, VV)

而對於裡面的index.tpl裡面的資源設置如下：

	// js文件
	<script type="text/javascript" src="views/{{.VV.Lang}}/js/jquery/jquery-1.8.0.min.js"></script>
	// css文件
	<link href="views/{{.VV.Lang}}/css/bootstrap-responsive.min.css" rel="stylesheet">
	// 圖片文件
	<img src="views/{{.VV.Lang}}/images/btn.png">

採用這種方式來本地化視圖以及資源時，我們就可以很容易的進行擴展了。

## 總結
本小節介紹瞭如何使用及存儲本地資源，有時需要通過轉換函數來實現，有時通過lang來設置，但是最終都是通過key-value的方式來存儲Locale對應的數據，在需要時取出相應於Locale的信息後，如果是文本信息就直接輸出，如果是時間日期或者貨幣，則需要先通過`fmt.Printf`或其他格式化函數來處理，而對於不同Locale的視圖和資源則是最簡單的，只要在路徑裡面增加lang就可以實現了。

## links
  * [目錄](<preface.md>)
  * 上一節: [設置默認地區](<10.1.md>)
  * 下一節: [國際化站點](<10.3.md>)
<!-- {% endraw %} -->
<!-- {% raw %} -->

---

# 10.3 國際化站點
前面小節介紹瞭如何處理本地化資源，即Locale一個相應的配置文件，那麼如果處理多個的本地化資源呢？而對於一些我們經常用到的例如：簡單的文本翻譯、時間日期、數字等如果處理呢？本小節將一一解決這些問題。
## 管理多個本地包
在開發一個應用的時候，首先我們要決定是隻支持一種語言，還是多種語言，如果要支持多種語言，我們則需要制定一個組織結構，以方便將來更多語言的添加。在此我們設計如下：Locale有關的文件放置在config/locales下，假設你要支持中文和英文，那麼你需要在這個文件夾下放置en.json和zh.json。大概的內容如下所示：

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

為了支持國際化，在此我們使用了一個國際化相關的包——[go-i18n](https://github.com/astaxie/go-i18n)，首先我們向go-i18n包註冊config/locales這個目錄,以加載所有的locale文件

	Tr:=i18n.NewLocale()
	Tr.LoadPath("config/locales")

這個包使用起來很簡單，你可以通過下面的方式進行測試：

	fmt.Println(Tr.Translate("submit"))
	//輸出Submit
	Tr.SetLocale("zn")
	fmt.Println(Tr.Translate("submit"))
	//輸出“遞交”

## 自動加載本地包
上面我們介紹瞭如何自動加載自定義語言包，其實go-i18n庫已經預加載了很多默認的格式信息，例如時間格式、貨幣格式，用戶可以在自定義配置時改寫這些默認配置，請看下面的處理過程：


	//加載默認配置文件，這些文件都放在go-i18n/locales下面

	//文件命名zh.json、en-json、en-US.json等，可以不斷的擴展支持更多的語言

	func (il *IL) loadDefaultTranslations(dirPath string) error {
		dir, err := os.Open(dirPath)
		if err != nil {
			return err
		}
		defer dir.Close()

		names, err := dir.Readdirnames(-1)
		if err != nil {
			return err
		}

		for _, name := range names {
			fullPath := path.Join(dirPath, name)

			fi, err := os.Stat(fullPath)
			if err != nil {
				return err
			}

			if fi.IsDir() {
				if err := il.loadTranslations(fullPath); err != nil {
					return err
				}
			} else if locale := il.matchingLocaleFromFileName(name); locale != "" {
				file, err := os.Open(fullPath)
				if err != nil {
					return err
				}
				defer file.Close()

				if err := il.loadTranslation(file, locale); err != nil {
					return err
				}
			}
		}

		return nil
	}

通過上面的方法加載配置信息到默認的文件，這樣我們就可以在我們沒有自定義時間信息的時候執行如下的代碼獲取對應的信息:

	//locale=zh的情況下，執行如下代碼：

	fmt.Println(Tr.Time(time.Now()))
	//輸出：2009年1月08日 星期四 20:37:58 CST

	fmt.Println(Tr.Time(time.Now(),"long"))
	//輸出：2009年1月08日

	fmt.Println(Tr.Money(11.11))
	//輸出:￥11.11

## template mapfunc
上面我們實現了多個語言包的管理和加載，而一些函數的實現是基於邏輯層的，例如："Tr.Translate"、"Tr.Time"、"Tr.Money"等，雖然我們在邏輯層可以利用這些函數把需要的參數進行轉換後在模板層渲染的時候直接輸出，但是如果我們想在模版層直接使用這些函數該怎麼實現呢？不知你是否還記得，在前面介紹模板的時候說過：Go語言的模板支持自定義模板函數，下面是我們實現的方便操作的mapfunc：

1. 文本信息

文本信息調用`Tr.Translate`來實現相應的信息轉換，mapFunc的實現如下：

	func I18nT(args ...interface{}) string {
		ok := false
		var s string
		if len(args) == 1 {
			s, ok = args[0].(string)
		}
		if !ok {
			s = fmt.Sprint(args...)
		}
		return Tr.Translate(s)
	}

註冊函數如下：

	t.Funcs(template.FuncMap{"T": I18nT})

模板中使用如下：

	{{.V.Submit | T}}


2. 時間日期

時間日期調用`Tr.Time`函數來實現相應的時間轉換，mapFunc的實現如下：

	func I18nTimeDate(args ...interface{}) string {
		ok := false
		var s string
		if len(args) == 1 {
			s, ok = args[0].(string)
		}
		if !ok {
			s = fmt.Sprint(args...)
		}
		return Tr.Time(s)
	}

註冊函數如下：

	t.Funcs(template.FuncMap{"TD": I18nTimeDate})

模板中使用如下：

	{{.V.Now | TD}}

3. 貨幣信息

貨幣調用`Tr.Money`函數來實現相應的時間轉換，mapFunc的實現如下：

	func I18nMoney(args ...interface{}) string {
		ok := false
		var s string
		if len(args) == 1 {
			s, ok = args[0].(string)
		}
		if !ok {
			s = fmt.Sprint(args...)
		}
		return Tr.Money(s)
	}

註冊函數如下：

	t.Funcs(template.FuncMap{"M": I18nMoney})

模板中使用如下：

	{{.V.Money | M}}

## 總結
通過這小節我們知道了如何實現一個多語言包的Web應用，通過自定義語言包我們可以方便的實現多語言，而且通過配置文件能夠非常方便的擴充多語言，默認情況下，go-i18n會自定加載一些公共的配置信息，例如時間、貨幣等，我們就可以非常方便的使用，同時為了支持在模板中使用這些函數，也實現了相應的模板函數，這樣就允許我們在開發Web應用的時候直接在模板中通過pipeline的方式來操作多語言包。

## links
  * [目錄](<preface.md>)
  * 上一節: [本地化資源](<10.2.md>)
  * 下一節: [小結](<10.4.md>)
<!-- {% endraw %} -->

---

# 10.4 小結
通過這一章的介紹，讀者應該對如何操作i18n有了深入的瞭解，我也根據這一章介紹的內容實現了一個開源的解決方案go-i18n：https://github.com/astaxie/go-i18n  通過這個開源庫我們可以很方便的實現多語言版本的Web應用，使得我們的應用能夠輕鬆的實現國際化。如果你發現這個開源庫中的錯誤或者那些缺失的地方，請一起參與到這個開源項目中來，讓我們的這個庫爭取成為Go的標準庫。
## links
  * [目錄](<preface.md>)
  * 上一節: [國際化站點](<10.3.md>)
  * 下一節: [錯誤處理，故障排除和測試](<11.0.md>)
