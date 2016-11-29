# 7 文本處理
Web開發中對於文本處理是非常重要的一部分，我們往往需要對輸出或者輸入的內容進行處理，這裡的文本包括字符串、數字、Json、XMl等等。Go語言作為一門高性能的語言，對這些文本的處理都有官方的標準庫來支持。而且在你使用中你會發現Go標準庫的一些設計相當的巧妙，而且對於使用者來說也很方便就能處理這些文本。本章我們將通過四個小節的介紹，讓用戶對Go語言處理文本有一個很好的認識。

XML是目前很多標準接口的交互語言，很多時候和一些Java編寫的webserver進行交互都是基於XML標準進行交互，7.1小節將介紹如何處理XML文本，我們使用XML之後發現它太複雜了，現在很多互聯網企業對外的API大多數採用了JSON格式，這種格式描述簡單，但是又能很好的表達意思，7.2小節我們將講述如何來處理這樣的JSON格式數據。正則是一個讓人又愛又恨的工具，它處理文本的能力非常強大，我們在前面表單驗證裡面已經有所領略它的強大，7.3小節將詳細的更深入的講解如何利用好Go的正則。Web開發中一個很重要的部分就是MVC分離，在Go語言的Web開發中V有一個專門的包來支持`template`,7.4小節將詳細的講解如何使用模版來進行輸出內容。7.5小節將詳細介紹如何進行文件和文件夾的操作。7.6小結介紹了字符串的相關操作。

## 目錄
   ![](images/navi7.png?raw=true)

## links
   * [目錄](<preface.md>)
   * 上一章: [第六章總結](<06.5.md>)
   * 下一節: [XML處理](<07.1.md>)

---

# 7.1 XML處理
XML作為一種數據交換和信息傳遞的格式已經十分普及。而隨著Web服務日益廣泛的應用，現在XML在日常的開發工作中也扮演了愈發重要的角色。這一小節， 我們將就Go語言標準包中的XML相關處理的包進行介紹。

這個小節不會涉及XML規範相關的內容（如需瞭解相關知識請參考其他文獻），而是介紹如何用Go語言來編解碼XML文件相關的知識。

假如你是一名運維人員，你為你所管理的所有服務器生成了如下內容的xml的配置文件：

	<?xml version="1.0" encoding="utf-8"?>
	<servers version="1">
		<server>
			<serverName>Shanghai_VPN</serverName>
			<serverIP>127.0.0.1</serverIP>
		</server>
		<server>
			<serverName>Beijing_VPN</serverName>
			<serverIP>127.0.0.2</serverIP>
		</server>
	</servers>

上面的XML文檔描述了兩個服務器的信息，包含了服務器名和服務器的IP信息，接下來的Go例子以此XML描述的信息進行操作。

## 解析XML
如何解析如上這個XML文件呢？ 我們可以通過xml包的`Unmarshal`函數來達到我們的目的

	func Unmarshal(data []byte, v interface{}) error

data接收的是XML數據流，v是需要輸出的結構，定義為interface，也就是可以把XML轉換為任意的格式。我們這裡主要介紹struct的轉換，因為struct和XML都有類似樹結構的特徵。

示例代碼如下：

	package main

	import (
		"encoding/xml"
		"fmt"
		"io/ioutil"
		"os"
	)

	type Recurlyservers struct {
		XMLName     xml.Name `xml:"servers"`
		Version     string   `xml:"version,attr"`
		Svs         []server `xml:"server"`
		Description string   `xml:",innerxml"`
	}

	type server struct {
		XMLName    xml.Name `xml:"server"`
		ServerName string   `xml:"serverName"`
		ServerIP   string   `xml:"serverIP"`
	}

	func main() {
		file, err := os.Open("servers.xml") // For read access.		
		if err != nil {
			fmt.Printf("error: %v", err)
			return
		}
		defer file.Close()
		data, err := ioutil.ReadAll(file)
		if err != nil {
			fmt.Printf("error: %v", err)
			return
		}
		v := Recurlyservers{}
		err = xml.Unmarshal(data, &v)
		if err != nil {
			fmt.Printf("error: %v", err)
			return
		}

		fmt.Println(v)
	}


XML本質上是一種樹形的數據格式，而我們可以定義與之匹配的go 語言的 struct類型，然後通過xml.Unmarshal來將xml中的數據解析成對應的struct對象。如上例子輸出如下數據

	{{ servers} 1 [{{ server} Shanghai_VPN 127.0.0.1} {{ server} Beijing_VPN 127.0.0.2}]
	<server>
		<serverName>Shanghai_VPN</serverName>
		<serverIP>127.0.0.1</serverIP>
	</server>
	<server>
		<serverName>Beijing_VPN</serverName>
		<serverIP>127.0.0.2</serverIP>
	</server>
	}


上面的例子中，將xml文件解析成對應的struct對象是通過`xml.Unmarshal`來完成的，這個過程是如何實現的？可以看到我們的struct定義後面多了一些類似於`xml:"serverName"`這樣的內容,這個是struct的一個特性，它們被稱為 struct tag，它們是用來輔助反射的。我們來看一下`Unmarshal`的定義：

	func Unmarshal(data []byte, v interface{}) error

我們看到函數定義了兩個參數，第一個是XML數據流，第二個是存儲的對應類型，目前支持struct、slice和string，XML包內部採用了反射來進行數據的映射，所以v裡面的字段必須是導出的。`Unmarshal`解析的時候XML元素和字段怎麼對應起來的呢？這是有一個優先級讀取流程的，首先會讀取struct tag，如果沒有，那麼就會對應字段名。必須注意一點的是解析的時候tag、字段名、XML元素都是大小寫敏感的，所以必須一一對應字段。

Go語言的反射機制，可以利用這些tag信息來將來自XML文件中的數據反射成對應的struct對象，關於反射如何利用struct tag的更多內容請參閱reflect中的相關內容。

解析XML到struct的時候遵循如下的規則：

- 如果struct的一個字段是string或者[]byte類型且它的tag含有`",innerxml"`，Unmarshal將會將此字段所對應的元素內所有內嵌的原始xml累加到此字段上，如上面例子Description定義。最後的輸出是

		<server>
			<serverName>Shanghai_VPN</serverName>
			<serverIP>127.0.0.1</serverIP>
		</server>
		<server>
			<serverName>Beijing_VPN</serverName>
			<serverIP>127.0.0.2</serverIP>
		</server>

- 如果struct中有一個叫做XMLName，且類型為xml.Name字段，那麼在解析的時候就會保存這個element的名字到該字段,如上面例子中的servers。
- 如果某個struct字段的tag定義中含有XML結構中element的名稱，那麼解析的時候就會把相應的element值賦值給該字段，如上servername和serverip定義。
- 如果某個struct字段的tag定義了中含有`",attr"`，那麼解析的時候就會將該結構所對應的element的與字段同名的屬性的值賦值給該字段，如上version定義。
- 如果某個struct字段的tag定義 型如`"a>b>c"`,則解析的時候，會將xml結構a下面的b下面的c元素的值賦值給該字段。
- 如果某個struct字段的tag定義了`"-"`,那麼不會為該字段解析匹配任何xml數據。
- 如果struct字段後面的tag定義了`",any"`，如果他的子元素在不滿足其他的規則的時候就會匹配到這個字段。
- 如果某個XML元素包含一條或者多條註釋，那麼這些註釋將被累加到第一個tag含有",comments"的字段上，這個字段的類型可能是[]byte或string,如果沒有這樣的字段存在，那麼註釋將會被拋棄。

上面詳細講述瞭如何定義struct的tag。 只要設置對了tag，那麼XML解析就如上面示例般簡單，tag和XML的element是一一對應的關係，如上所示，我們還可以通過slice來表示多個同級元素。

>注意： 為了正確解析，go語言的xml包要求struct定義中的所有字段必須是可導出的（即首字母大寫）

## 輸出XML
假若我們不是要解析如上所示的XML文件，而是生成它，那麼在go語言中又該如何實現呢？ xml包中提供了`Marshal`和`MarshalIndent`兩個函數，來滿足我們的需求。這兩個函數主要的區別是第二個函數會增加前綴和縮進，函數的定義如下所示：

	func Marshal(v interface{}) ([]byte, error)
	func MarshalIndent(v interface{}, prefix, indent string) ([]byte, error)

兩個函數第一個參數是用來生成XML的結構定義類型數據，都是返回生成的XML數據流。

下面我們來看一下如何輸出如上的XML：

	package main

	import (
		"encoding/xml"
		"fmt"
		"os"
	)

	type Servers struct {
		XMLName xml.Name `xml:"servers"`
		Version string   `xml:"version,attr"`
		Svs     []server `xml:"server"`
	}

	type server struct {
		ServerName string `xml:"serverName"`
		ServerIP   string `xml:"serverIP"`
	}

	func main() {
		v := &Servers{Version: "1"}
		v.Svs = append(v.Svs, server{"Shanghai_VPN", "127.0.0.1"})
		v.Svs = append(v.Svs, server{"Beijing_VPN", "127.0.0.2"})
		output, err := xml.MarshalIndent(v, "  ", "    ")
		if err != nil {
			fmt.Printf("error: %v\n", err)
		}
		os.Stdout.Write([]byte(xml.Header))

		os.Stdout.Write(output)
	}
上面的代碼輸出如下信息：

	<?xml version="1.0" encoding="UTF-8"?>
	<servers version="1">
	<server>
		<serverName>Shanghai_VPN</serverName>
		<serverIP>127.0.0.1</serverIP>
	</server>
	<server>
		<serverName>Beijing_VPN</serverName>
		<serverIP>127.0.0.2</serverIP>
	</server>
	</servers>

和我們之前定義的文件的格式一模一樣，之所以會有`os.Stdout.Write([]byte(xml.Header))` 這句代碼的出現，是因為`xml.MarshalIndent`或者`xml.Marshal`輸出的信息都是不帶XML頭的，為了生成正確的xml文件，我們使用了xml包預定義的Header變量。

我們看到`Marshal`函數接收的參數v是interface{}類型的，即它可以接受任意類型的參數，那麼xml包，根據什麼規則來生成相應的XML文件呢？

- 如果v是 array或者slice，那麼輸出每一個元素，類似<type>value</type>
- 如果v是指針，那麼會Marshal指針指向的內容，如果指針為空，什麼都不輸出
- 如果v是interface，那麼就處理interface所包含的數據
- 如果v是其他數據類型，就會輸出這個數據類型所擁有的字段信息

生成的XML文件中的element的名字又是根據什麼決定的呢？元素名按照如下優先級從struct中獲取：

- 如果v是struct，XMLName的tag中定義的名稱
- 類型為xml.Name的名叫XMLName的字段的值
- 通過struct中字段的tag來獲取
- 通過struct的字段名用來獲取
- marshall的類型名稱

我們應如何設置struct 中字段的tag信息以控制最終xml文件的生成呢？

- XMLName不會被輸出
- tag中含有`"-"`的字段不會輸出
- tag中含有`"name,attr"`，會以name作為屬性名，字段值作為值輸出為這個XML元素的屬性，如上version字段所描述
- tag中含有`",attr"`，會以這個struct的字段名作為屬性名輸出為XML元素的屬性，類似上一條，只是這個name默認是字段名了。
- tag中含有`",chardata"`，輸出為xml的 character data而非element。
- tag中含有`",innerxml"`，將會被原樣輸出，而不會進行常規的編碼過程
- tag中含有`",comment"`，將被當作xml註釋來輸出，而不會進行常規的編碼過程，字段值中不能含有"--"字符串
- tag中含有`"omitempty"`,如果該字段的值為空值那麼該字段就不會被輸出到XML，空值包括：false、0、nil指針或nil接口，任何長度為0的array, slice, map或者string
- tag中含有`"a>b>c"`，那麼就會循環輸出三個元素a包含b，b包含c，例如如下代碼就會輸出

		FirstName string   `xml:"name>first"`
		LastName  string   `xml:"name>last"`

		<name>
		<first>Asta</first>
		<last>Xie</last>
		</name>


上面我們介紹瞭如何使用Go語言的xml包來編/解碼XML文件，重要的一點是對XML的所有操作都是通過struct tag來實現的，所以學會對struct tag的運用變得非常重要，在文章中我們簡要的列舉了如何定義tag。更多內容或tag定義請參看相應的官方資料。

## links
   * [目錄](<preface.md>)
   * 上一節: [文本處理](<07.0.md>)
   * 下一節: [Json處理](<07.2.md>)

---

# 7.2 JSON處理
JSON（Javascript Object Notation）是一種輕量級的數據交換語言，以文字為基礎，具有自我描述性且易於讓人閱讀。儘管JSON是Javascript的一個子集，但JSON是獨立於語言的文本格式，並且採用了類似於C語言家族的一些習慣。JSON與XML最大的不同在於XML是一個完整的標記語言，而JSON不是。JSON由於比XML更小、更快，更易解析,以及瀏覽器的內建快速解析支持,使得其更適用於網絡數據傳輸領域。目前我們看到很多的開放平臺，基本上都是採用了JSON作為他們的數據交互的接口。既然JSON在Web開發中如此重要，那麼Go語言對JSON支持的怎麼樣呢？Go語言的標準庫已經非常好的支持了JSON，可以很容易的對JSON數據進行編、解碼的工作。

前一小節的運維的例子用json來表示，結果描述如下：

	{"servers":[{"serverName":"Shanghai_VPN","serverIP":"127.0.0.1"},{"serverName":"Beijing_VPN","serverIP":"127.0.0.2"}]}

本小節餘下的內容將以此JSON數據為基礎，來介紹go語言的json包對JSON數據的編、解碼。
## 解析JSON

### 解析到結構體
假如有了上面的JSON串，那麼我們如何來解析這個JSON串呢？Go的JSON包中有如下函數

	func Unmarshal(data []byte, v interface{}) error

通過這個函數我們就可以實現解析的目的，詳細的解析例子請看如下代碼：

	package main

	import (
		"encoding/json"
		"fmt"
	)

	type Server struct {
		ServerName string
		ServerIP   string
	}

	type Serverslice struct {
		Servers []Server
	}

	func main() {
		var s Serverslice
		str := `{"servers":[{"serverName":"Shanghai_VPN","serverIP":"127.0.0.1"},{"serverName":"Beijing_VPN","serverIP":"127.0.0.2"}]}`
		json.Unmarshal([]byte(str), &s)
		fmt.Println(s)
	}

在上面的示例代碼中，我們首先定義了與json數據對應的結構體，數組對應slice，字段名對應JSON裡面的KEY，在解析的時候，如何將json數據與struct字段相匹配呢？例如JSON的key是`Foo`，那麼怎麼找對應的字段呢？

- 首先查找tag含有`Foo`的可導出的struct字段(首字母大寫)
- 其次查找字段名是`Foo`的導出字段
- 最後查找類似`FOO`或者`FoO`這樣的除了首字母之外其他大小寫不敏感的導出字段

聰明的你一定注意到了這一點：能夠被賦值的字段必須是可導出字段(即首字母大寫）。同時JSON解析的時候只會解析能找得到的字段，找不到的字段會被忽略，這樣的一個好處是：當你接收到一個很大的JSON數據結構而你卻只想獲取其中的部分數據的時候，你只需將你想要的數據對應的字段名大寫，即可輕鬆解決這個問題。

### 解析到interface
上面那種解析方式是在我們知曉被解析的JSON數據的結構的前提下采取的方案，如果我們不知道被解析的數據的格式，又應該如何來解析呢？

我們知道interface{}可以用來存儲任意數據類型的對象，這種數據結構正好用於存儲解析的未知結構的json數據的結果。JSON包中採用map[string]interface{}和[]interface{}結構來存儲任意的JSON對象和數組。Go類型和JSON類型的對應關係如下：

- bool 代表 JSON booleans,
- float64 代表 JSON numbers,
- string 代表 JSON strings,
- nil 代表 JSON null.

現在我們假設有如下的JSON數據

	b := []byte(`{"Name":"Wednesday","Age":6,"Parents":["Gomez","Morticia"]}`)

如果在我們不知道他的結構的情況下，我們把他解析到interface{}裡面

	var f interface{}
	err := json.Unmarshal(b, &f)

這個時候f裡面存儲了一個map類型，他們的key是string，值存儲在空的interface{}裡

	f = map[string]interface{}{
		"Name": "Wednesday",
		"Age":  6,
		"Parents": []interface{}{
			"Gomez",
			"Morticia",
		},
	}

那麼如何來訪問這些數據呢？通過斷言的方式：

	m := f.(map[string]interface{})

通過斷言之後，你就可以通過如下方式來訪問裡面的數據了

	for k, v := range m {
		switch vv := v.(type) {
		case string:
			fmt.Println(k, "is string", vv)
		case int:
			fmt.Println(k, "is int", vv)
		case float64:
			fmt.Println(k,"is float64",vv)
		case []interface{}:
			fmt.Println(k, "is an array:")
			for i, u := range vv {
				fmt.Println(i, u)
			}
		default:
			fmt.Println(k, "is of a type I don't know how to handle")
		}
	}
通過上面的示例可以看到，通過interface{}與type assert的配合，我們就可以解析未知結構的JSON數了。

上面這個是官方提供的解決方案，其實很多時候我們通過類型斷言，操作起來不是很方便，目前bitly公司開源了一個叫做`simplejson`的包,在處理未知結構體的JSON時相當方便，詳細例子如下所示：

	js, err := NewJson([]byte(`{
		"test": {
			"array": [1, "2", 3],
			"int": 10,
			"float": 5.150,
			"bignum": 9223372036854775807,
			"string": "simplejson",
			"bool": true
		}
	}`))

	arr, _ := js.Get("test").Get("array").Array()
	i, _ := js.Get("test").Get("int").Int()
	ms := js.Get("test").Get("string").MustString()

可以看到，使用這個庫操作JSON比起官方包來說，簡單的多,詳細的請參考如下地址：https://github.com/bitly/go-simplejson

## 生成JSON
我們開發很多應用的時候，最後都是要輸出JSON數據串，那麼如何來處理呢？JSON包裡面通過`Marshal`函數來處理，函數定義如下：

	func Marshal(v interface{}) ([]byte, error)

假設我們還是需要生成上面的服務器列表信息，那麼如何來處理呢？請看下面的例子：

	package main

	import (
		"encoding/json"
		"fmt"
	)

	type Server struct {
		ServerName string
		ServerIP   string
	}

	type Serverslice struct {
		Servers []Server
	}

	func main() {
		var s Serverslice
		s.Servers = append(s.Servers, Server{ServerName: "Shanghai_VPN", ServerIP: "127.0.0.1"})
		s.Servers = append(s.Servers, Server{ServerName: "Beijing_VPN", ServerIP: "127.0.0.2"})
		b, err := json.Marshal(s)
		if err != nil {
			fmt.Println("json err:", err)
		}
		fmt.Println(string(b))
	}

輸出如下內容：

	{"Servers":[{"ServerName":"Shanghai_VPN","ServerIP":"127.0.0.1"},{"ServerName":"Beijing_VPN","ServerIP":"127.0.0.2"}]}

我們看到上面的輸出字段名的首字母都是大寫的，如果你想用小寫的首字母怎麼辦呢？把結構體的字段名改成首字母小寫的？JSON輸出的時候必須注意，只有導出的字段才會被輸出，如果修改字段名，那麼就會發現什麼都不會輸出，所以必須通過struct tag定義來實現：

	type Server struct {
		ServerName string `json:"serverName"`
		ServerIP   string `json:"serverIP"`
	}

	type Serverslice struct {
		Servers []Server `json:"servers"`
	}

通過修改上面的結構體定義，輸出的JSON串就和我們最開始定義的JSON串保持一致了。

針對JSON的輸出，我們在定義struct tag的時候需要注意的幾點是:

- 字段的tag是`"-"`，那麼這個字段不會輸出到JSON
- tag中帶有自定義名稱，那麼這個自定義名稱會出現在JSON的字段名中，例如上面例子中serverName
- tag中如果帶有`"omitempty"`選項，那麼如果該字段值為空，就不會輸出到JSON串中
- 如果字段類型是bool, string, int, int64等，而tag中帶有`",string"`選項，那麼這個字段在輸出到JSON的時候會把該字段對應的值轉換成JSON字符串


舉例來說：

	type Server struct {
		// ID 不會導出到JSON中
		ID int `json:"-"`

		// ServerName2 的值會進行二次JSON編碼
		ServerName  string `json:"serverName"`
		ServerName2 string `json:"serverName2,string"`

		// 如果 ServerIP 為空，則不輸出到JSON串中
		ServerIP   string `json:"serverIP,omitempty"`
	}

	s := Server {
		ID:         3,
		ServerName:  `Go "1.0" `,
		ServerName2: `Go "1.0" `,
		ServerIP:   ``,
	}
	b, _ := json.Marshal(s)
	os.Stdout.Write(b)

會輸出以下內容：

	{"serverName":"Go \"1.0\" ","serverName2":"\"Go \\\"1.0\\\" \""}


Marshal函數只有在轉換成功的時候才會返回數據，在轉換的過程中我們需要注意幾點：


- JSON對象只支持string作為key，所以要編碼一個map，那麼必須是map[string]T這種類型(T是Go語言中任意的類型)
- Channel, complex和function是不能被編碼成JSON的
- 嵌套的數據是不能編碼的，不然會讓JSON編碼進入死循環
- 指針在編碼的時候會輸出指針指向的內容，而空指針會輸出null


本小節，我們介紹瞭如何使用Go語言的json標準包來編解碼JSON數據，同時也簡要介紹瞭如何使用第三方包`go-simplejson`來在一些情況下簡化操作，學會並熟練運用它們將對我們接下來的Web開發相當重要。

## links
   * [目錄](<preface.md>)
   * 上一節: [XML處理](<07.1.md>)
   * 下一節: [正則處理](<07.3.md>)

---

# 7.3 正則處理
正則表達式是一種進行模式匹配和文本操縱的複雜而又強大的工具。雖然正則表達式比純粹的文本匹配效率低，但是它卻更靈活。按照它的語法規則，隨需構造出的匹配模式就能夠從原始文本中篩選出幾乎任何你想要得到的字符組合。如果你在Web開發中需要從一些文本數據源中獲取數據,那麼你只需要按照它的語法規則，隨需構造出正確的模式字符串就能夠從原數據源提取出有意義的文本信息。

Go語言通過`regexp`標準包為正則表達式提供了官方支持，如果你已經使用過其他編程語言提供的正則相關功能，那麼你應該對Go語言版本的不會太陌生，但是它們之間也有一些小的差異，因為Go實現的是RE2標準，除了\C，詳細的語法描述參考：`http://code.google.com/p/re2/wiki/Syntax`

其實字符串處理我們可以使用`strings`包來進行搜索(Contains、Index)、替換(Replace)和解析(Split、Join)等操作，但是這些都是簡單的字符串操作，他們的搜索都是大小寫敏感，而且固定的字符串，如果我們需要匹配可變的那種就沒辦法實現了，當然如果`strings`包能解決你的問題，那麼就儘量使用它來解決。因為他們足夠簡單、而且性能和可讀性都會比正則好。

如果你還記得，在前面表單驗證的小節裡，我們已經接觸過正則處理，在那裡我們利用了它來驗證輸入的信息是否滿足某些預設的條件。在使用中需要注意的一點就是：所有的字符都是UTF-8編碼的。接下來讓我們更加深入的來學習Go語言的`regexp`包相關知識吧。

## 通過正則判斷是否匹配
`regexp`包中含有三個函數用來判斷是否匹配，如果匹配返回true，否則返回false

	func Match(pattern string, b []byte) (matched bool, error error)
	func MatchReader(pattern string, r io.RuneReader) (matched bool, error error)
	func MatchString(pattern string, s string) (matched bool, error error)

上面的三個函數實現了同一個功能，就是判斷`pattern`是否和輸入源匹配，匹配的話就返回true，如果解析正則出錯則返回error。三個函數的輸入源分別是byte slice、RuneReader和string。

如果要驗證一個輸入是不是IP地址，那麼如何來判斷呢？請看如下實現

	func IsIP(ip string) (b bool) {
		if m, _ := regexp.MatchString("^[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}$", ip); !m {
			return false
		}
		return true
	}

可以看到，`regexp`的pattern和我們平常使用的正則一模一樣。再來看一個例子：當用戶輸入一個字符串，我們想知道是不是一次合法的輸入：

	func main() {
		if len(os.Args) == 1 {
			fmt.Println("Usage: regexp [string]")
			os.Exit(1)
		} else if m, _ := regexp.MatchString("^[0-9]+$", os.Args[1]); m {
			fmt.Println("數字")
		} else {
			fmt.Println("不是數字")
		}
	}

在上面的兩個小例子中，我們採用了Match(Reader|String)來判斷一些字符串是否符合我們的描述需求，它們使用起來非常方便。

## 通過正則獲取內容
Match模式只能用來對字符串的判斷，而無法截取字符串的一部分、過濾字符串、或者提取出符合條件的一批字符串。如果想要滿足這些需求，那就需要使用正則表達式的複雜模式。

我們經常需要一些爬蟲程序，下面就以爬蟲為例來說明如何使用正則來過濾或截取抓取到的數據：

	package main

	import (
		"fmt"
		"io/ioutil"
		"net/http"
		"regexp"
		"strings"
	)

	func main() {
		resp, err := http.Get("http://www.baidu.com")
		if err != nil {
			fmt.Println("http get error.")
		}
		defer resp.Body.Close()
		body, err := ioutil.ReadAll(resp.Body)
		if err != nil {
			fmt.Println("http read error")
			return
		}

		src := string(body)

		//將HTML標籤全轉換成小寫
		re, _ := regexp.Compile("\\<[\\S\\s]+?\\>")
		src = re.ReplaceAllStringFunc(src, strings.ToLower)

		//去除STYLE
		re, _ = regexp.Compile("\\<style[\\S\\s]+?\\</style\\>")
		src = re.ReplaceAllString(src, "")

		//去除SCRIPT
		re, _ = regexp.Compile("\\<script[\\S\\s]+?\\</script\\>")
		src = re.ReplaceAllString(src, "")

		//去除所有尖括號內的HTML代碼，並換成換行符
		re, _ = regexp.Compile("\\<[\\S\\s]+?\\>")
		src = re.ReplaceAllString(src, "\n")

		//去除連續的換行符
		re, _ = regexp.Compile("\\s{2,}")
		src = re.ReplaceAllString(src, "\n")

		fmt.Println(strings.TrimSpace(src))
	}

從這個示例可以看出，使用複雜的正則首先是Compile，它會解析正則表達式是否合法，如果正確，那麼就會返回一個Regexp，然後就可以利用返回的Regexp在任意的字符串上面執行需要的操作。

解析正則表達式的有如下幾個方法：

	func Compile(expr string) (*Regexp, error)
	func CompilePOSIX(expr string) (*Regexp, error)
	func MustCompile(str string) *Regexp
	func MustCompilePOSIX(str string) *Regexp

CompilePOSIX和Compile的不同點在於POSIX必須使用POSIX語法，它使用最左最長方式搜索，而Compile是採用的則只採用最左方式搜索(例如[a-z]{2,4}這樣一個正則表達式，應用於"aa09aaa88aaaa"這個文本串時，CompilePOSIX返回了aaaa，而Compile的返回的是aa)。前綴有Must的函數表示，在解析正則語法的時候，如果匹配模式串不滿足正確的語法則直接panic，而不加Must的則只是返回錯誤。

在瞭解瞭如何新建一個Regexp之後，我們再來看一下這個struct提供了哪些方法來輔助我們操作字符串，首先我們來看下面這些用來搜索的函數：

	func (re *Regexp) Find(b []byte) []byte
	func (re *Regexp) FindAll(b []byte, n int) [][]byte
	func (re *Regexp) FindAllIndex(b []byte, n int) [][]int
	func (re *Regexp) FindAllString(s string, n int) []string
	func (re *Regexp) FindAllStringIndex(s string, n int) [][]int
	func (re *Regexp) FindAllStringSubmatch(s string, n int) [][]string
	func (re *Regexp) FindAllStringSubmatchIndex(s string, n int) [][]int
	func (re *Regexp) FindAllSubmatch(b []byte, n int) [][][]byte
	func (re *Regexp) FindAllSubmatchIndex(b []byte, n int) [][]int
	func (re *Regexp) FindIndex(b []byte) (loc []int)
	func (re *Regexp) FindReaderIndex(r io.RuneReader) (loc []int)
	func (re *Regexp) FindReaderSubmatchIndex(r io.RuneReader) []int
	func (re *Regexp) FindString(s string) string
	func (re *Regexp) FindStringIndex(s string) (loc []int)
	func (re *Regexp) FindStringSubmatch(s string) []string
	func (re *Regexp) FindStringSubmatchIndex(s string) []int
	func (re *Regexp) FindSubmatch(b []byte) [][]byte
	func (re *Regexp) FindSubmatchIndex(b []byte) []int

上面這18個函數我們根據輸入源(byte slice、string和io.RuneReader)不同還可以繼續簡化成如下幾個，其他的只是輸入源不一樣，其他功能基本是一樣的：

	func (re *Regexp) Find(b []byte) []byte
	func (re *Regexp) FindAll(b []byte, n int) [][]byte
	func (re *Regexp) FindAllIndex(b []byte, n int) [][]int
	func (re *Regexp) FindAllSubmatch(b []byte, n int) [][][]byte
	func (re *Regexp) FindAllSubmatchIndex(b []byte, n int) [][]int
	func (re *Regexp) FindIndex(b []byte) (loc []int)
	func (re *Regexp) FindSubmatch(b []byte) [][]byte
	func (re *Regexp) FindSubmatchIndex(b []byte) []int

對於這些函數的使用我們來看下面這個例子

	package main

	import (
		"fmt"
		"regexp"
	)

	func main() {
		a := "I am learning Go language"

		re, _ := regexp.Compile("[a-z]{2,4}")

		//查找符合正則的第一個
		one := re.Find([]byte(a))
		fmt.Println("Find:", string(one))

		//查找符合正則的所有slice,n小於0表示返回全部符合的字符串，不然就是返回指定的長度
		all := re.FindAll([]byte(a), -1)
		fmt.Println("FindAll", all)

		//查找符合條件的index位置,開始位置和結束位置
		index := re.FindIndex([]byte(a))
		fmt.Println("FindIndex", index)

		//查找符合條件的所有的index位置，n同上
		allindex := re.FindAllIndex([]byte(a), -1)
		fmt.Println("FindAllIndex", allindex)

		re2, _ := regexp.Compile("am(.*)lang(.*)")

		//查找Submatch,返回數組，第一個元素是匹配的全部元素，第二個元素是第一個()裡面的，第三個是第二個()裡面的
		//下面的輸出第一個元素是"am learning Go language"
		//第二個元素是" learning Go "，注意包含空格的輸出
		//第三個元素是"uage"
		submatch := re2.FindSubmatch([]byte(a))
		fmt.Println("FindSubmatch", submatch)
		for _, v := range submatch {
			fmt.Println(string(v))
		}

		//定義和上面的FindIndex一樣
		submatchindex := re2.FindSubmatchIndex([]byte(a))
		fmt.Println(submatchindex)

		//FindAllSubmatch,查找所有符合條件的子匹配
		submatchall := re2.FindAllSubmatch([]byte(a), -1)
		fmt.Println(submatchall)

		//FindAllSubmatchIndex,查找所有字匹配的index
		submatchallindex := re2.FindAllSubmatchIndex([]byte(a), -1)
		fmt.Println(submatchallindex)
	}

前面介紹過匹配函數，Regexp也定義了三個函數，它們和同名的外部函數功能一模一樣，其實外部函數就是調用了這Regexp的三個函數來實現的：

	func (re *Regexp) Match(b []byte) bool
	func (re *Regexp) MatchReader(r io.RuneReader) bool
	func (re *Regexp) MatchString(s string) bool

接下里讓我們來了解替換函數是怎麼操作的？

	func (re *Regexp) ReplaceAll(src, repl []byte) []byte
	func (re *Regexp) ReplaceAllFunc(src []byte, repl func([]byte) []byte) []byte
	func (re *Regexp) ReplaceAllLiteral(src, repl []byte) []byte
	func (re *Regexp) ReplaceAllLiteralString(src, repl string) string
	func (re *Regexp) ReplaceAllString(src, repl string) string
	func (re *Regexp) ReplaceAllStringFunc(src string, repl func(string) string) string

這些替換函數我們在上面的抓網頁的例子有詳細應用示例，

接下來我們看一下Expand的解釋：

	func (re *Regexp) Expand(dst []byte, template []byte, src []byte, match []int) []byte
	func (re *Regexp) ExpandString(dst []byte, template string, src string, match []int) []byte

那麼這個Expand到底用來幹嘛的呢？請看下面的例子：

	func main() {
		src := []byte(`
			call hello alice
			hello bob
			call hello eve
		`)
		pat := regexp.MustCompile(`(?m)(call)\s+(?P<cmd>\w+)\s+(?P<arg>.+)\s*$`)
		res := []byte{}
		for _, s := range pat.FindAllSubmatchIndex(src, -1) {
			res = pat.Expand(res, []byte("$cmd('$arg')\n"), src, s)
		}
		fmt.Println(string(res))
	}

至此我們已經全部介紹完Go語言的`regexp`包，通過對它的主要函數介紹及演示，相信大家應該能夠通過Go語言的正則包進行一些基本的正則的操作了。


## links
   * [目錄](<preface.md>)
   * 上一節: [Json處理](<07.2.md>)
   * 下一節: [模板處理](<07.4.md>)

---

# 7.4 模板處理
## 什麼是模板
你一定聽說過一種叫做MVC的設計模式，Model處理數據，View展現結果，Controller控制用戶的請求，至於View層的處理，在很多動態語言裡面都是通過在靜態HTML中插入動態語言生成的數據，例如JSP中通過插入`<%=....=%>`，PHP中通過插入`<?php.....?>`來實現的。

通過下面這個圖可以說明模板的機制

![](images/7.4.template.png?raw=true)

圖7.1 模板機制圖

Web應用反饋給客戶端的信息中的大部分內容是靜態的，不變的，而另外少部分是根據用戶的請求來動態生成的，例如要顯示用戶的訪問記錄列表。用戶之間只有記錄數據是不同的，而列表的樣式則是固定的，此時採用模板可以複用很多靜態代碼。

## Go模板使用
在Go語言中，我們使用`template`包來進行模板處理，使用類似`Parse`、`ParseFile`、`Execute`等方法從文件或者字符串加載模板，然後執行類似上面圖片展示的模板的merge操作。請看下面的例子：

	func handler(w http.ResponseWriter, r *http.Request) {
		t := template.New("some template") //創建一個模板
		t, _ = t.ParseFiles("tmpl/welcome.html", nil)  //解析模板文件
		user := GetUser() //獲取當前用戶信息
		t.Execute(w, user)  //執行模板的merger操作
	}

通過上面的例子我們可以看到Go語言的模板操作非常的簡單方便，和其他語言的模板處理類似，都是先獲取數據，然後渲染數據。

為了演示和測試代碼的方便，我們在接下來的例子中採用如下格式的代碼

- 使用Parse代替ParseFiles，因為Parse可以直接測試一個字符串，而不需要額外的文件
- 不使用handler來寫演示代碼，而是每個測試一個main，方便測試
- 使用`os.Stdout`代替`http.ResponseWriter`，因為`os.Stdout`實現了`io.Writer`接口

## 模板中如何插入數據？
上面我們演示瞭如何解析並渲染模板，接下來讓我們來更加詳細的瞭解如何把數據渲染出來。一個模板都是應用在一個Go的對象之上，Go對象的字段如何插入到模板中呢？

### 字段操作
Go語言的模板通過`{{}}`來包含需要在渲染時被替換的字段，`{{.}}`表示當前的對象，這和Java或者C++中的this類似，如果要訪問當前對象的字段通過`{{.FieldName}}`,但是需要注意一點：這個字段必須是導出的(字段首字母必須是大寫的),否則在渲染的時候就會報錯，請看下面的這個例子：

	package main

	import (
		"html/template"
		"os"
	)

	type Person struct {
		UserName string
	}

	func main() {
		t := template.New("fieldname example")
		t, _ = t.Parse("hello {{.UserName}}!")
		p := Person{UserName: "Astaxie"}
		t.Execute(os.Stdout, p)
	}

上面的代碼我們可以正確的輸出`hello Astaxie`，但是如果我們稍微修改一下代碼，在模板中含有了未導出的字段，那麼就會報錯

	type Person struct {
		UserName string
		email	string  //未導出的字段，首字母是小寫的
	}

	t, _ = t.Parse("hello {{.UserName}}! {{.email}}")

上面的代碼就會報錯，因為我們調用了一個未導出的字段，但是如果我們調用了一個不存在的字段是不會報錯的，而是輸出為空。

如果模板中輸出`{{.}}`，這個一般應用於字符串對象，默認會調用fmt包輸出字符串的內容。

### 輸出嵌套字段內容
上面我們例子展示瞭如何針對一個對象的字段輸出，那麼如果字段裡面還有對象，如何來循環的輸出這些內容呢？我們可以使用`{{with …}}…{{end}}`和`{{range …}}{{end}}`來進行數據的輸出。

- {{range}} 這個和Go語法裡面的range類似，循環操作數據
- {{with}}操作是指當前對象的值，類似上下文的概念

詳細的使用請看下面的例子：

	package main

	import (
		"html/template"
		"os"
	)

	type Friend struct {
		Fname string
	}

	type Person struct {
		UserName string
		Emails   []string
		Friends  []*Friend
	}

	func main() {
		f1 := Friend{Fname: "minux.ma"}
		f2 := Friend{Fname: "xushiwei"}
		t := template.New("fieldname example")
		t, _ = t.Parse(`hello {{.UserName}}!
				{{range .Emails}}
					an email {{.}}
				{{end}}
				{{with .Friends}}
				{{range .}}
					my friend name is {{.Fname}}
				{{end}}
				{{end}}
				`)
		p := Person{UserName: "Astaxie",
			Emails:  []string{"astaxie@beego.me", "astaxie@gmail.com"},
			Friends: []*Friend{&f1, &f2}}
		t.Execute(os.Stdout, p)
	}

### 條件處理
在Go模板裡面如果需要進行條件判斷，那麼我們可以使用和Go語言的`if-else`語法類似的方式來處理，如果pipeline為空，那麼if就認為是false，下面的例子展示瞭如何使用`if-else`語法：

	package main

	import (
		"os"
		"text/template"
	)

	func main() {
		tEmpty := template.New("template test")
		tEmpty = template.Must(tEmpty.Parse("空 pipeline if demo: {{if ``}} 不會輸出. {{end}}\n"))
		tEmpty.Execute(os.Stdout, nil)

		tWithValue := template.New("template test")
		tWithValue = template.Must(tWithValue.Parse("不為空的 pipeline if demo: {{if `anything`}} 我有內容，我會輸出. {{end}}\n"))
		tWithValue.Execute(os.Stdout, nil)

		tIfElse := template.New("template test")
		tIfElse = template.Must(tIfElse.Parse("if-else demo: {{if `anything`}} if部分 {{else}} else部分.{{end}}\n"))
		tIfElse.Execute(os.Stdout, nil)
	}

通過上面的演示代碼我們知道`if-else`語法相當的簡單，在使用過程中很容易集成到我們的模板代碼中。

> 注意：if裡面無法使用條件判斷，例如.Mail=="astaxie@gmail.com"，這樣的判斷是不正確的，if裡面只能是bool值

### pipelines
Unix用戶已經很熟悉什麼是`pipe`了，`ls | grep "beego"`類似這樣的語法你是不是經常使用，過濾當前目錄下面的文件，顯示含有"beego"的數據，表達的意思就是前面的輸出可以當做後面的輸入，最後顯示我們想要的數據，而Go語言模板最強大的一點就是支持pipe數據，在Go語言裡面任何`{{}}`裡面的都是pipelines數據，例如我們上面輸出的email裡面如果還有一些可能引起XSS注入的，那麼我們如何來進行轉化呢？

	{{. | html}}

在email輸出的地方我們可以採用如上方式可以把輸出全部轉化html的實體，上面的這種方式和我們平常寫Unix的方式是不是一模一樣，操作起來相當的簡便，調用其他的函數也是類似的方式。

### 模板變量
有時候，我們在模板使用過程中需要定義一些局部變量，我們可以在一些操作中申明局部變量，例如`with``range``if`過程中申明局部變量，這個變量的作用域是`{{end}}`之前，Go語言通過申明的局部變量格式如下所示：

	$variable := pipeline

詳細的例子看下面的：

	{{with $x := "output" | printf "%q"}}{{$x}}{{end}}
	{{with $x := "output"}}{{printf "%q" $x}}{{end}}
	{{with $x := "output"}}{{$x | printf "%q"}}{{end}}
### 模板函數
模板在輸出對象的字段值時，採用了`fmt`包把對象轉化成了字符串。但是有時候我們的需求可能不是這樣的，例如有時候我們為了防止垃圾郵件發送者通過採集網頁的方式來發送給我們的郵箱信息，我們希望把`@`替換成`at`例如：`astaxie at beego.me`，如果要實現這樣的功能，我們就需要自定義函數來做這個功能。

每一個模板函數都有一個唯一值的名字，然後與一個Go函數關聯，通過如下的方式來關聯

	type FuncMap map[string]interface{}

例如，如果我們想要的email函數的模板函數名是`emailDeal`，它關聯的Go函數名稱是`EmailDealWith`,那麼我們可以通過下面的方式來註冊這個函數

	t = t.Funcs(template.FuncMap{"emailDeal": EmailDealWith})

`EmailDealWith`這個函數的參數和返回值定義如下：

	func EmailDealWith(args …interface{}) string

我們來看下面的實現例子：

	package main

	import (
		"fmt"
		"html/template"
		"os"
		"strings"
	)

	type Friend struct {
		Fname string
	}

	type Person struct {
		UserName string
		Emails   []string
		Friends  []*Friend
	}

	func EmailDealWith(args ...interface{}) string {
		ok := false
		var s string
		if len(args) == 1 {
			s, ok = args[0].(string)
		}
		if !ok {
			s = fmt.Sprint(args...)
		}
		// find the @ symbol
		substrs := strings.Split(s, "@")
		if len(substrs) != 2 {
			return s
		}
		// replace the @ by " at "
		return (substrs[0] + " at " + substrs[1])
	}

	func main() {
		f1 := Friend{Fname: "minux.ma"}
		f2 := Friend{Fname: "xushiwei"}
		t := template.New("fieldname example")
		t = t.Funcs(template.FuncMap{"emailDeal": EmailDealWith})
		t, _ = t.Parse(`hello {{.UserName}}!
					{{range .Emails}}
						an emails {{.|emailDeal}}
					{{end}}
					{{with .Friends}}
					{{range .}}
						my friend name is {{.Fname}}
					{{end}}
					{{end}}
					`)
		p := Person{UserName: "Astaxie",
			Emails:  []string{"astaxie@beego.me", "astaxie@gmail.com"},
			Friends: []*Friend{&f1, &f2}}
		t.Execute(os.Stdout, p)
	}


上面演示瞭如何自定義函數，其實，在模板包內部已經有內置的實現函數，下面代碼截取自模板包裡面

	var builtins = FuncMap{
		"and":      and,
		"call":     call,
		"html":     HTMLEscaper,
		"index":    index,
		"js":       JSEscaper,
		"len":      length,
		"not":      not,
		"or":       or,
		"print":    fmt.Sprint,
		"printf":   fmt.Sprintf,
		"println":  fmt.Sprintln,
		"urlquery": URLQueryEscaper,
	}


## Must操作
模板包裡面有一個函數`Must`，它的作用是檢測模板是否正確，例如大括號是否匹配，註釋是否正確的關閉，變量是否正確的書寫。接下來我們演示一個例子，用Must來判斷模板是否正確：

	package main

	import (
		"fmt"
		"text/template"
	)

	func main() {
		tOk := template.New("first")
		template.Must(tOk.Parse(" some static text /* and a comment */"))
		fmt.Println("The first one parsed OK.")

		template.Must(template.New("second").Parse("some static text {{ .Name }}"))
		fmt.Println("The second one parsed OK.")

		fmt.Println("The next one ought to fail.")
		tErr := template.New("check parse error with Must")
		template.Must(tErr.Parse(" some static text {{ .Name }"))
	}

將輸出如下內容

	The first one parsed OK.
	The second one parsed OK.
	The next one ought to fail.
	panic: template: check parse error with Must:1: unexpected "}" in command

## 嵌套模板
我們平常開發Web應用的時候，經常會遇到一些模板有些部分是固定不變的，然後可以抽取出來作為一個獨立的部分，例如一個博客的頭部和尾部是不變的，而唯一改變的是中間的內容部分。所以我們可以定義成`header`、`content`、`footer`三個部分。Go語言中通過如下的語法來申明

	{{define "子模板名稱"}}內容{{end}}

通過如下方式來調用：

	{{template "子模板名稱"}}

接下來我們演示如何使用嵌套模板，我們定義三個文件，`header.tmpl`、`content.tmpl`、`footer.tmpl`文件，裡面的內容如下

	//header.tmpl
	{{define "header"}}
	<html>
	<head>
		<title>演示信息</title>
	</head>
	<body>
	{{end}}

	//content.tmpl
	{{define "content"}}
	{{template "header"}}
	<h1>演示嵌套</h1>
	<ul>
		<li>嵌套使用define定義子模板</li>
		<li>調用使用template</li>
	</ul>
	{{template "footer"}}
	{{end}}

	//footer.tmpl
	{{define "footer"}}
	</body>
	</html>
	{{end}}

演示代碼如下：

	package main

	import (
		"fmt"
		"os"
		"text/template"
	)

	func main() {
		s1, _ := template.ParseFiles("header.tmpl", "content.tmpl", "footer.tmpl")
		s1.ExecuteTemplate(os.Stdout, "header", nil)
		fmt.Println()
		s1.ExecuteTemplate(os.Stdout, "content", nil)
		fmt.Println()
		s1.ExecuteTemplate(os.Stdout, "footer", nil)
		fmt.Println()
		s1.Execute(os.Stdout, nil)
	}

通過上面的例子我們可以看到通過`template.ParseFiles`把所有的嵌套模板全部解析到模板裡面，其實每一個定義的{{define}}都是一個獨立的模板，他們相互獨立，是並行存在的關係，內部其實存儲的是類似map的一種關係(key是模板的名稱，value是模板的內容)，然後我們通過`ExecuteTemplate`來執行相應的子模板內容，我們可以看到header、footer都是相對獨立的，都能輸出內容，content 中因為嵌套了header和footer的內容，就會同時輸出三個的內容。但是當我們執行`s1.Execute`，沒有任何的輸出，因為在默認的情況下沒有默認的子模板，所以不會輸出任何的東西。

>同一個集合類的模板是互相知曉的，如果同一模板被多個集合使用，則它需要在多個集合中分別解析

## 總結
通過上面對模板的詳細介紹，我們瞭解瞭如何把動態數據與模板融合：如何輸出循環數據、如何自定義函數、如何嵌套模板等等。通過模板技術的應用，我們可以完成MVC模式中V的處理，接下來的章節我們將介紹如何來處理M和C。

## links
   * [目錄](<preface.md>)
   * 上一節: [正則處理](<07.3.md>)
   * 下一節: [文件操作](<07.5.md>)

---

# 7.5 文件操作
在任何計算機設備中，文件是都是必須的對象，而在Web編程中,文件的操作一直是Web程序員經常遇到的問題,文件操作在Web應用中是必須的,非常有用的,我們經常遇到生成文件目錄,文件(夾)編輯等操作,現在我把Go中的這些操作做一詳細總結並實例示範如何使用。
## 目錄操作
文件操作的大多數函數都是在os包裡面，下面列舉了幾個目錄操作的：

- func Mkdir(name string, perm FileMode) error

	創建名稱為name的目錄，權限設置是perm，例如0777
	
- func MkdirAll(path string, perm FileMode) error

	根據path創建多級子目錄，例如astaxie/test1/test2。
	
- func Remove(name string) error

	刪除名稱為name的目錄，當目錄下有文件或者其他目錄是會出錯

- func RemoveAll(path string) error

	根據path刪除多級子目錄，如果path是單個名稱，那麼該目錄下的子目錄全部刪除。


下面是演示代碼：

	package main

	import (
		"fmt"
		"os"
	)
	
	func main() {
		os.Mkdir("astaxie", 0777)
		os.MkdirAll("astaxie/test1/test2", 0777)
		err := os.Remove("astaxie")
		if err != nil {
			fmt.Println(err)
		}
		os.RemoveAll("astaxie")
	}


## 文件操作

### 建立與打開文件
新建文件可以通過如下兩個方法

- func Create(name string) (file *File, err Error)

	根據提供的文件名創建新的文件，返回一個文件對象，默認權限是0666的文件，返回的文件對象是可讀寫的。

- func NewFile(fd uintptr, name string) *File
	
	根據文件描述符創建相應的文件，返回一個文件對象


通過如下兩個方法來打開文件：

- func Open(name string) (file *File, err Error)

	該方法打開一個名稱為name的文件，但是是隻讀方式，內部實現其實調用了OpenFile。

- func OpenFile(name string, flag int, perm uint32) (file *File, err Error)	

	打開名稱為name的文件，flag是打開的方式，只讀、讀寫等，perm是權限		

### 寫文件
寫文件函數：

- func (file *File) Write(b []byte) (n int, err Error)

	寫入byte類型的信息到文件

- func (file *File) WriteAt(b []byte, off int64) (n int, err Error)

	在指定位置開始寫入byte類型的信息

- func (file *File) WriteString(s string) (ret int, err Error)

	寫入string信息到文件
	
寫文件的示例代碼

	package main

	import (
		"fmt"
		"os"
	)
	
	func main() {
		userFile := "astaxie.txt"
		fout, err := os.Create(userFile)		
		if err != nil {
			fmt.Println(userFile, err)
			return
		}
		defer fout.Close()
		for i := 0; i < 10; i++ {
			fout.WriteString("Just a test!\r\n")
			fout.Write([]byte("Just a test!\r\n"))
		}
	}

### 讀文件
讀文件函數：

- func (file *File) Read(b []byte) (n int, err Error)

	讀取數據到b中

- func (file *File) ReadAt(b []byte, off int64) (n int, err Error)

	從off開始讀取數據到b中

讀文件的示例代碼:

	package main

	import (
		"fmt"
		"os"
	)
	
	func main() {
		userFile := "asatxie.txt"
		fl, err := os.Open(userFile)		
		if err != nil {
			fmt.Println(userFile, err)
			return
		}
		defer fl.Close()
		buf := make([]byte, 1024)
		for {
			n, _ := fl.Read(buf)
			if 0 == n {
				break
			}
			os.Stdout.Write(buf[:n])
		}
	}

### 刪除文件
Go語言裡面刪除文件和刪除文件夾是同一個函數

- func Remove(name string) Error

	調用該函數就可以刪除文件名為name的文件

## links
   * [目錄](<preface.md>)
   * 上一節: [模板處理](<07.4.md>)
   * 下一節: [字符串處理](<07.6.md>)

---

# 7.6 字符串處理
字符串在我們平常的Web開發中經常用到，包括用戶的輸入，數據庫讀取的數據等，我們經常需要對字符串進行分割、連接、轉換等操作，本小節將通過Go標準庫中的strings和strconv兩個包中的函數來講解如何進行有效快速的操作。
## 字符串操作
下面這些函數來自於strings包，這裡介紹一些我平常經常用到的函數，更詳細的請參考官方的文檔。

- func Contains(s, substr string) bool

	字符串s中是否包含substr，返回bool值
	
		fmt.Println(strings.Contains("seafood", "foo"))
		fmt.Println(strings.Contains("seafood", "bar"))
		fmt.Println(strings.Contains("seafood", ""))
		fmt.Println(strings.Contains("", ""))
		//Output:
		//true
		//false
		//true
		//true

- func Join(a []string, sep string) string

	字符串鏈接，把slice a通過sep鏈接起來
	
		s := []string{"foo", "bar", "baz"}
		fmt.Println(strings.Join(s, ", "))
		//Output:foo, bar, baz		
			
- func Index(s, sep string) int 

	在字符串s中查找sep所在的位置，返回位置值，找不到返回-1
	
		fmt.Println(strings.Index("chicken", "ken"))
		fmt.Println(strings.Index("chicken", "dmr"))
		//Output:4
		//-1

- func Repeat(s string, count int) string

	重複s字符串count次，最後返回重複的字符串
	
		fmt.Println("ba" + strings.Repeat("na", 2))
		//Output:banana

- func Replace(s, old, new string, n int) string

	在s字符串中，把old字符串替換為new字符串，n表示替換的次數，小於0表示全部替換
	
		fmt.Println(strings.Replace("oink oink oink", "k", "ky", 2))
		fmt.Println(strings.Replace("oink oink oink", "oink", "moo", -1))
		//Output:oinky oinky oink
		//moo moo moo

- func Split(s, sep string) []string

	把s字符串按照sep分割，返回slice
	
		fmt.Printf("%q\n", strings.Split("a,b,c", ","))
		fmt.Printf("%q\n", strings.Split("a man a plan a canal panama", "a "))
		fmt.Printf("%q\n", strings.Split(" xyz ", ""))
		fmt.Printf("%q\n", strings.Split("", "Bernardo O'Higgins"))
		//Output:["a" "b" "c"]
		//["" "man " "plan " "canal panama"]
		//[" " "x" "y" "z" " "]
		//[""]

- func Trim(s string, cutset string) string

	在s字符串的頭部和尾部去除cutset指定的字符串
	
		fmt.Printf("[%q]", strings.Trim(" !!! Achtung !!! ", "! "))
		//Output:["Achtung"]

- func Fields(s string) []string

	去除s字符串的空格符，並且按照空格分割返回slice
	
		fmt.Printf("Fields are: %q", strings.Fields("  foo bar  baz   "))
		//Output:Fields are: ["foo" "bar" "baz"]


## 字符串轉換
字符串轉化的函數在strconv中，如下也只是列出一些常用的：

- Append 系列函數將整數等轉換為字符串後，添加到現有的字節數組中。

		package main
		
		import (
			"fmt"
			"strconv"
		)
		
		func main() {
			str := make([]byte, 0, 100)
			str = strconv.AppendInt(str, 4567, 10)
			str = strconv.AppendBool(str, false)
			str = strconv.AppendQuote(str, "abcdefg")
			str = strconv.AppendQuoteRune(str, '單')
			fmt.Println(string(str))
		}

- Format 系列函數把其他類型的轉換為字符串

		package main
	
		import (
			"fmt"
			"strconv"
		)
		
		func main() {
			a := strconv.FormatBool(false)
			b := strconv.FormatFloat(123.23, 'g', 12, 64)
			c := strconv.FormatInt(1234, 10)
			d := strconv.FormatUint(12345, 10)
			e := strconv.Itoa(1023)
			fmt.Println(a, b, c, d, e)
		}

- Parse 系列函數把字符串轉換為其他類型
		
		package main

		import (
			"fmt"
			"strconv"
		)
		func checkError(e error){
			if e != nil{
				fmt.Println(e)
			}
		}
		func main() {
			a, err := strconv.ParseBool("false")
			checkError(err)
			b, err := strconv.ParseFloat("123.23", 64)
			checkError(err)
			c, err := strconv.ParseInt("1234", 10, 64)
			checkError(err)
			d, err := strconv.ParseUint("12345", 10, 64)
			checkError(err)
			e, err := strconv.Atoi("1023")
			checkError(err)
			fmt.Println(a, b, c, d, e)
		}

	

## links
   * [目錄](<preface.md>)
   * 上一節: [文件操作](<07.5.md>)
   * 下一節: [小結](<07.7.md>)

---

# 7.7 小結
這一章給大家介紹了一些文本處理的工具，包括XML、JSON、正則和模板技術，XML和JSON是數據交互的工具，通過XML和JSON你可以表達各種含義，通過正則你可以處理文本(搜索、替換、截取)，通過模板技術你可以展現這些數據給用戶。這些都是你開發Web應用過程中需要用到的技術，通過這個小節的介紹你能夠了解如何處理文本、展現文本。

## links
   * [目錄](<preface.md>)
   * 上一節: [字符串處理](<07.6.md>)
   * 下一節: [Web服務](<08.0.md>)
