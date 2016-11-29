# 2 Go語言基礎

Go是一門類似C的編譯型語言，但是它的編譯速度非常快。這門語言的關鍵字總共也就二十五個，比英文字母還少一個，這對於我們的學習來說就簡單了很多。先讓我們看一眼這些關鍵字都長什麼樣：

	break    default      func    interface    select
	case     defer        go      map          struct
	chan     else         goto    package      switch
	const    fallthrough  if      range        type
	continue for          import  return       var

在接下來的這一章中，我將帶領你去學習這門語言的基礎。通過每一小節的介紹，你將發現，Go的世界是那麼地簡潔，設計是如此地美妙，編寫Go將會是一件愉快的事情。等回過頭來，你就會發現這二十五個關鍵字是多麼地親切。

## 目錄
![](images/navi2.png?raw=true)

## links
   * [目錄](<preface.md>)
   * 上一章: [第一章總結](<01.5.md>)
   * 下一節: [你好，Go](<02.1.md>)

---

# 2.1 你好，Go

在開始編寫應用之前，我們先從最基本的程序開始。就像你造房子之前不知道什麼是地基一樣，編寫程序也不知道如何開始。因此，在本節中，我們要學習用最基本的語法讓Go程序運行起來。

## 程序

這就像一個傳統，在學習大部分語言之前，你先學會如何編寫一個可以輸出`hello world`的程序。

準備好了嗎？Let's Go!

	package main

	import "fmt"

	func main() {
		fmt.Printf("Hello, world or 你好，世界 or καλημ ́ρα κóσμ or こんにちはせかい\n")
	}

輸出如下：

	Hello, world or 你好，世界 or καλημ ́ρα κóσμ or こんにちはせかい

## 詳解
首先我們要了解一個概念，Go程序是通過`package`來組織的

`package <pkgName>`（在我們的例子中是`package main`）這一行告訴我們當前文件屬於哪個包，而包名`main`則告訴我們它是一個可獨立運行的包，它在編譯後會產生可執行文件。除了`main`包之外，其它的包最後都會生成`*.a`文件（也就是包文件）並放置在`$GOPATH/pkg/$GOOS_$GOARCH`中（以Mac為例就是`$GOPATH/pkg/darwin_amd64`）。

>每一個可獨立運行的Go程序，必定包含一個`package main`，在這個`main`包中必定包含一個入口函數`main`，而這個函數既沒有參數，也沒有返回值。

為了打印`Hello, world...`，我們調用了一個函數`Printf`，這個函數來自於`fmt`包，所以我們在第三行中導入了系統級別的`fmt`包：`import "fmt"`。

包的概念和Python中的package類似，它們都有一些特別的好處：模塊化（能夠把你的程序分成多個模塊)和可重用性（每個模塊都能被其它應用程序反覆使用）。我們在這裡只是先了解一下包的概念，後面我們將會編寫自己的包。

在第五行中，我們通過關鍵字`func`定義了一個`main`函數，函數體被放在`{}`（大括號）中，就像我們平時寫C、C++或Java時一樣。

大家可以看到`main`函數是沒有任何的參數的，我們接下來就學習如何編寫帶參數的、返回0個或多個值的函數。

第六行，我們調用了`fmt`包裡面定義的函數`Printf`。大家可以看到，這個函數是通過`<pkgName>.<funcName>`的方式調用的，這一點和Python十分相似。

>前面提到過，包名和包所在的文件夾名可以是不同的，此處的`<pkgName>`即為通過`package <pkgName>`聲明的包名，而非文件夾名。

最後大家可以看到我們輸出的內容裡面包含了很多非ASCII碼字符。實際上，Go是天生支持UTF-8的，任何字符都可以直接輸出，你甚至可以用UTF-8中的任何字符作為標識符。


## 結論

Go使用`package`（和Python的模塊類似）來組織代碼。`main.main()`函數(這個函數位於主包）是每一個獨立的可運行程序的入口點。Go使用UTF-8字符串和標識符(因為UTF-8的發明者也就是Go的發明者之一)，所以它天生支持多語言。

## links
   * [目錄](<preface.md>)
   * 上一節: [Go語言基礎](<02.0.md>)
   * 下一節: [Go基礎](<02.2.md>)

---

# 2.2 Go基礎

這小節我們將要介紹如何定義變量、常量、Go內置類型以及Go程序設計中的一些技巧。

## 定義變量

Go語言裡面定義變量有多種方式。

使用`var`關鍵字是Go最基本的定義變量方式，與C語言不同的是Go把變量類型放在變量名後面：

	//定義一個名稱為“variableName”，類型為"type"的變量
	var variableName type

定義多個變量

	//定義三個類型都是“type”的變量
	var vname1, vname2, vname3 type

定義變量並初始化值

	//初始化“variableName”的變量為“value”值，類型是“type”
	var variableName type = value

同時初始化多個變量

	/*
		定義三個類型都是"type"的變量,並且分別初始化為相應的值
		vname1為v1，vname2為v2，vname3為v3
	*/
	var vname1, vname2, vname3 type= v1, v2, v3

你是不是覺得上面這樣的定義有點繁瑣？沒關係，因為Go語言的設計者也發現了，有一種寫法可以讓它變得簡單一點。我們可以直接忽略類型聲明，那麼上面的代碼變成這樣了：

	/*
		定義三個變量，它們分別初始化為相應的值
		vname1為v1，vname2為v2，vname3為v3
		然後Go會根據其相應值的類型來幫你初始化它們
	*/
	var vname1, vname2, vname3 = v1, v2, v3

你覺得上面的還是有些繁瑣？好吧，我也覺得。讓我們繼續簡化：

	/*
		定義三個變量，它們分別初始化為相應的值
		vname1為v1，vname2為v2，vname3為v3
		編譯器會根據初始化的值自動推導出相應的類型
	*/
	vname1, vname2, vname3 := v1, v2, v3

現在是不是看上去非常簡潔了？`:=`這個符號直接取代了`var`和`type`,這種形式叫做簡短聲明。不過它有一個限制，那就是它只能用在函數內部；在函數外部使用則會無法編譯通過，所以一般用`var`方式來定義全局變量。

`_`（下劃線）是個特殊的變量名，任何賦予它的值都會被丟棄。在這個例子中，我們將值`35`賦予`b`，並同時丟棄`34`：

	_, b := 34, 35

Go對於已聲明但未使用的變量會在編譯階段報錯，比如下面的代碼就會產生一個錯誤：聲明瞭`i`但未使用。

	package main

	func main() {
		var i int
	}

## 常量

所謂常量，也就是在程序編譯階段就確定下來的值，而程序在運行時無法改變該值。在Go程序中，常量可定義為數值、布爾值或字符串等類型。

它的語法如下：

	const constantName = value
	//如果需要，也可以明確指定常量的類型：
	const Pi float32 = 3.1415926

下面是一些常量聲明的例子：

	const Pi = 3.1415926
	const i = 10000
	const MaxThread = 10
	const prefix = "astaxie_"

Go 常量和一般程序語言不同的是，可以指定相當多的小數位數(例如200位)，
若指定給float32自動縮短為32bit，指定給float64自動縮短為64bit，詳情參考[鏈接](http://golang.org/ref/spec#Constants)

## 內置基礎類型

### Boolean

在Go中，布爾值的類型為`bool`，值是`true`或`false`，默認為`false`。

	//示例代碼
	var isActive bool  // 全局變量聲明
	var enabled, disabled = true, false  // 忽略類型的聲明
	func test() {
		var available bool  // 一般聲明
		valid := false      // 簡短聲明
		available = true    // 賦值操作
	}


### 數值類型

整數類型有無符號和帶符號兩種。Go同時支持`int`和`uint`，這兩種類型的長度相同，但具體長度取決於不同編譯器的實現。Go裡面也有直接定義好位數的類型：`rune`, `int8`, `int16`, `int32`, `int64`和`byte`, `uint8`, `uint16`, `uint32`, `uint64`。其中`rune`是`int32`的別稱，`byte`是`uint8`的別稱。

>需要注意的一點是，這些類型的變量之間不允許互相賦值或操作，不然會在編譯時引起編譯器報錯。
>
>如下的代碼會產生錯誤：invalid operation: a + b (mismatched types int8 and int32)
>
>>	var a int8

>>	var b int32

>>	c:=a + b
>
>另外，儘管int的長度是32 bit, 但int 與 int32並不可以互用。

浮點數的類型有`float32`和`float64`兩種（沒有`float`類型），默認是`float64`。

這就是全部嗎？No！Go還支持複數。它的默認類型是`complex128`（64位實數+64位虛數）。如果需要小一些的，也有`complex64`(32位實數+32位虛數)。複數的形式為`RE + IMi`，其中`RE`是實數部分，`IM`是虛數部分，而最後的`i`是虛數單位。下面是一個使用複數的例子：

	var c complex64 = 5+5i
	//output: (5+5i)
	fmt.Printf("Value is: %v", c)


### 字符串

我們在上一節中講過，Go中的字符串都是採用`UTF-8`字符集編碼。字符串是用一對雙引號（`""`）或反引號（`` ` `` `` ` ``）括起來定義，它的類型是`string`。

	//示例代碼
	var frenchHello string  // 聲明變量為字符串的一般方法
	var emptyString string = ""  // 聲明瞭一個字符串變量，初始化為空字符串
	func test() {
		no, yes, maybe := "no", "yes", "maybe"  // 簡短聲明，同時聲明多個變量
		japaneseHello := "Konichiwa"  // 同上
		frenchHello = "Bonjour"  // 常規賦值
	}

在Go中字符串是不可變的，例如下面的代碼編譯時會報錯：cannot assign to s[0]

	var s string = "hello"
	s[0] = 'c'


但如果真的想要修改怎麼辦呢？下面的代碼可以實現：

	s := "hello"
	c := []byte(s)  // 將字符串 s 轉換為 []byte 類型
	c[0] = 'c'
	s2 := string(c)  // 再轉換回 string 類型
	fmt.Printf("%s\n", s2)


Go中可以使用`+`操作符來連接兩個字符串：

	s := "hello,"
	m := " world"
	a := s + m
	fmt.Printf("%s\n", a)

修改字符串也可寫為：

	s := "hello"
	s = "c" + s[1:] // 字符串雖不能更改，但可進行切片操作
	fmt.Printf("%s\n", s)

如果要聲明一個多行的字符串怎麼辦？可以通過`` ` ``來聲明：

	m := `hello
		world`

`` ` `` 括起的字符串為Raw字符串，即字符串在代碼中的形式就是打印時的形式，它沒有字符轉義，換行也將原樣輸出。例如本例中會輸出：

    hello
		world

### 錯誤類型
Go內置有一個`error`類型，專門用來處理錯誤信息，Go的`package`裡面還專門有一個包`errors`來處理錯誤：

	err := errors.New("emit macho dwarf: elf header corrupted")
	if err != nil {
		fmt.Print(err)
	}

### Go數據底層的存儲

下面這張圖來源於[Russ Cox Blog](http://research.swtch.com/)中一篇介紹[Go數據結構](http://research.swtch.com/godata)的文章，大家可以看到這些基礎類型底層都是分配了一塊內存，然後存儲了相應的值。

![](images/2.2.basic.png?raw=true)

圖2.1 Go數據格式的存儲

## 一些技巧

### 分組聲明

在Go語言中，同時聲明多個常量、變量，或者導入多個包時，可採用分組的方式進行聲明。

例如下面的代碼：

	import "fmt"
	import "os"

	const i = 100
	const pi = 3.1415
	const prefix = "Go_"

	var i int
	var pi float32
	var prefix string

可以分組寫成如下形式：

	import(
		"fmt"
		"os"
	)

	const(
		i = 100
		pi = 3.1415
		prefix = "Go_"
	)

	var(
		i int
		pi float32
		prefix string
	)

### iota枚舉

Go裡面有一個關鍵字`iota`，這個關鍵字用來聲明`enum`的時候採用，它默認開始值是0，const中每增加一行加1：

	const(
		x = iota  // x == 0
		y = iota  // y == 1
		z = iota  // z == 2
		w  // 常量聲明省略值時，默認和之前一個值的字面相同。這裡隱式地說w = iota，因此w == 3。其實上面y和z可同樣不用"= iota"
	)

	const v = iota // 每遇到一個const關鍵字，iota就會重置，此時v == 0

	const (
	  e, f, g = iota, iota, iota //e=0,f=0,g=0 iota在同一行值相同
	)

	const （
		a = iota    a=0
		b = "B"
		c = iota    //c=2
		d,e,f = iota,iota,iota //d=3,e=3,f=3
		g //g = 4
	）

>除非被顯式設置為其它值或`iota`，每個`const`分組的第一個常量被默認設置為它的0值，第二及後續的常量被默認設置為它前面那個常量的值，如果前面那個常量的值是`iota`，則它也被設置為`iota`。

### Go程序設計的一些規則
Go之所以會那麼簡潔，是因為它有一些默認的行為：
- 大寫字母開頭的變量是可導出的，也就是其它包可以讀取的，是公有變量；小寫字母開頭的就是不可導出的，是私有變量。
- 大寫字母開頭的函數也是一樣，相當於`class`中的帶`public`關鍵詞的公有函數；小寫字母開頭的就是有`private`關鍵詞的私有函數。

## array、slice、map

### array
`array`就是數組，它的定義方式如下：

	var arr [n]type

在`[n]type`中，`n`表示數組的長度，`type`表示存儲元素的類型。對數組的操作和其它語言類似，都是通過`[]`來進行讀取或賦值：

	var arr [10]int  // 聲明瞭一個int類型的數組
	arr[0] = 42      // 數組下標是從0開始的
	arr[1] = 13      // 賦值操作
	fmt.Printf("The first element is %d\n", arr[0])  // 獲取數據，返回42
	fmt.Printf("The last element is %d\n", arr[9]) //返回未賦值的最後一個元素，默認返回0

由於長度也是數組類型的一部分，因此`[3]int`與`[4]int`是不同的類型，數組也就不能改變長度。數組之間的賦值是值的賦值，即當把一個數組作為參數傳入函數的時候，傳入的其實是該數組的副本，而不是它的指針。如果要使用指針，那麼就需要用到後面介紹的`slice`類型了。

數組可以使用另一種`:=`來聲明

	a := [3]int{1, 2, 3} // 聲明瞭一個長度為3的int數組

	b := [10]int{1, 2, 3} // 聲明瞭一個長度為10的int數組，其中前三個元素初始化為1、2、3，其它默認為0

	c := [...]int{4, 5, 6} // 可以省略長度而採用`...`的方式，Go會自動根據元素個數來計算長度

也許你會說，我想數組裡面的值還是數組，能實現嗎？當然咯，Go支持嵌套數組，即多維數組。比如下面的代碼就聲明瞭一個二維數組：

	// 聲明瞭一個二維數組，該數組以兩個數組作為元素，其中每個數組中又有4個int類型的元素
	doubleArray := [2][4]int{[4]int{1, 2, 3, 4}, [4]int{5, 6, 7, 8}}

	// 上面的聲明可以簡化，直接忽略內部的類型
	easyArray := [2][4]int{{1, 2, 3, 4}, {5, 6, 7, 8}}

數組的分配如下所示：

![](images/2.2.array.png?raw=true)

圖2.2 多維數組的映射關係


### slice

在很多應用場景中，數組並不能滿足我們的需求。在初始定義數組時，我們並不知道需要多大的數組，因此我們就需要“動態數組”。在Go裡面這種數據結構叫`slice`

`slice`並不是真正意義上的動態數組，而是一個引用類型。`slice`總是指向一個底層`array`，`slice`的聲明也可以像`array`一樣，只是不需要長度。

	// 和聲明array一樣，只是少了長度
	var fslice []int

接下來我們可以聲明一個`slice`，並初始化數據，如下所示：

	slice := []byte {'a', 'b', 'c', 'd'}

`slice`可以從一個數組或一個已經存在的`slice`中再次聲明。`slice`通過`array[i:j]`來獲取，其中`i`是數組的開始位置，`j`是結束位置，但不包含`array[j]`，它的長度是`j-i`。

	// 聲明一個含有10個元素元素類型為byte的數組
	var ar = [10]byte {'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j'}

	// 聲明兩個含有byte的slice
	var a, b []byte

	// a指向數組的第3個元素開始，併到第五個元素結束，
	a = ar[2:5]
	//現在a含有的元素: ar[2]、ar[3]和ar[4]

	// b是數組ar的另一個slice
	b = ar[3:5]
	// b的元素是：ar[3]和ar[4]

>注意`slice`和數組在聲明時的區別：聲明數組時，方括號內寫明瞭數組的長度或使用`...`自動計算長度，而聲明`slice`時，方括號內沒有任何字符。

它們的數據結構如下所示

![](images/2.2.slice.png?raw=true)

圖2.3 slice和array的對應關係圖

slice有一些簡便的操作

 - `slice`的默認開始位置是0，`ar[:n]`等價於`ar[0:n]`
 - `slice`的第二個序列默認是數組的長度，`ar[n:]`等價於`ar[n:len(ar)]`
 - 如果從一個數組裡面直接獲取`slice`，可以這樣`ar[:]`，因為默認第一個序列是0，第二個是數組的長度，即等價於`ar[0:len(ar)]`

下面這個例子展示了更多關於`slice`的操作：

	// 聲明一個數組
	var array = [10]byte{'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j'}
	// 聲明兩個slice
	var aSlice, bSlice []byte

	// 演示一些簡便操作
	aSlice = array[:3] // 等價於aSlice = array[0:3] aSlice包含元素: a,b,c
	aSlice = array[5:] // 等價於aSlice = array[5:10] aSlice包含元素: f,g,h,i,j
	aSlice = array[:]  // 等價於aSlice = array[0:10] 這樣aSlice包含了全部的元素

	// 從slice中獲取slice
	aSlice = array[3:7]  // aSlice包含元素: d,e,f,g，len=4，cap=7
	bSlice = aSlice[1:3] // bSlice 包含aSlice[1], aSlice[2] 也就是含有: e,f
	bSlice = aSlice[:3]  // bSlice 包含 aSlice[0], aSlice[1], aSlice[2] 也就是含有: d,e,f
	bSlice = aSlice[0:5] // 對slice的slice可以在cap範圍內擴展，此時bSlice包含：d,e,f,g,h
	bSlice = aSlice[:]   // bSlice包含所有aSlice的元素: d,e,f,g

`slice`是引用類型，所以當引用改變其中元素的值時，其它的所有引用都會改變該值，例如上面的`aSlice`和`bSlice`，如果修改了`aSlice`中元素的值，那麼`bSlice`相對應的值也會改變。

從概念上面來說`slice`像一個結構體，這個結構體包含了三個元素：
- 一個指針，指向數組中`slice`指定的開始位置
- 長度，即`slice`的長度
- 最大長度，也就是`slice`開始位置到數組的最後位置的長度

		Array_a := [10]byte{'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j'}
		Slice_a := Array_a[2:5]

上面代碼的真正存儲結構如下圖所示

![](images/2.2.slice2.png?raw=true)

圖2.4 slice對應數組的信息

對於`slice`有幾個有用的內置函數：

- `len`    獲取`slice`的長度
- `cap`    獲取`slice`的最大容量
- `append` 向`slice`裡面追加一個或者多個元素，然後返回一個和`slice`一樣類型的`slice`
- `copy`   函數`copy`從源`slice`的`src`中複製元素到目標`dst`，並且返回複製的元素的個數

注：`append`函數會改變`slice`所引用的數組的內容，從而影響到引用同一數組的其它`slice`。
但當`slice`中沒有剩餘空間（即`(cap-len) == 0`）時，此時將動態分配新的數組空間。返回的`slice`數組指針將指向這個空間，而原數組的內容將保持不變；其它引用此數組的`slice`則不受影響。

從Go1.2開始slice支持了三個參數的slice，之前我們一直採用這種方式在slice或者array基礎上來獲取一個slice

	var array [10]int
	slice := array[2:4]

這個例子裡面slice的容量是8，新版本里面可以指定這個容量

	slice = array[2:4:7]

上面這個的容量就是`7-2`，即5。這樣這個產生的新的slice就沒辦法訪問最後的三個元素。

如果slice是這樣的形式`array[:i:j]`，即第一個參數為空，默認值就是0。

### map

`map`也就是Python中字典的概念，它的格式為`map[keyType]valueType`

我們看下面的代碼，`map`的讀取和設置也類似`slice`一樣，通過`key`來操作，只是`slice`的`index`只能是｀int｀類型，而`map`多了很多類型，可以是`int`，可以是`string`及所有完全定義了`==`與`!=`操作的類型。

	// 聲明一個key是字符串，值為int的字典,這種方式的聲明需要在使用之前使用make初始化
	var numbers map[string]int
	// 另一種map的聲明方式
	numbers := make(map[string]int)
	numbers["one"] = 1  //賦值
	numbers["ten"] = 10 //賦值
	numbers["three"] = 3

	fmt.Println("第三個數字是: ", numbers["three"]) // 讀取數據
	// 打印出來如:第三個數字是: 3

這個`map`就像我們平常看到的表格一樣，左邊列是`key`，右邊列是值

使用map過程中需要注意的幾點：
- `map`是無序的，每次打印出來的`map`都會不一樣，它不能通過`index`獲取，而必須通過`key`獲取
- `map`的長度是不固定的，也就是和`slice`一樣，也是一種引用類型
- 內置的`len`函數同樣適用於`map`，返回`map`擁有的`key`的數量
- `map`的值可以很方便的修改，通過`numbers["one"]=11`可以很容易的把key為`one`的字典值改為`11`
- `map`和其他基本型別不同，它不是thread-safe，在多個go-routine存取時，必須使用mutex lock機制

`map`的初始化可以通過`key:val`的方式初始化值，同時`map`內置有判斷是否存在`key`的方式

通過`delete`刪除`map`的元素：

	// 初始化一個字典
	rating := map[string]float32{"C":5, "Go":4.5, "Python":4.5, "C++":2 }
	// map有兩個返回值，第二個返回值，如果不存在key，那麼ok為false，如果存在ok為true
	csharpRating, ok := rating["C#"]
	if ok {
		fmt.Println("C# is in the map and its rating is ", csharpRating)
	} else {
		fmt.Println("We have no rating associated with C# in the map")
	}

	delete(rating, "C")  // 刪除key為C的元素


上面說過了，`map`也是一種引用類型，如果兩個`map`同時指向一個底層，那麼一個改變，另一個也相應的改變：

	m := make(map[string]string)
	m["Hello"] = "Bonjour"
	m1 := m
	m1["Hello"] = "Salut"  // 現在m["hello"]的值已經是Salut了


### make、new操作

`make`用於內建類型（`map`、`slice` 和`channel`）的內存分配。`new`用於各種類型的內存分配。

內建函數`new`本質上說跟其它語言中的同名函數功能一樣：`new(T)`分配了零值填充的`T`類型的內存空間，並且返回其地址，即一個`*T`類型的值。用Go的術語說，它返回了一個指針，指向新分配的類型`T`的零值。有一點非常重要：

>`new`返回指針。

內建函數`make(T, args)`與`new(T)`有著不同的功能，make只能創建`slice`、`map`和`channel`，並且返回一個有初始值(非零)的`T`類型，而不是`*T`。本質來講，導致這三個類型有所不同的原因是指向數據結構的引用在使用前必須被初始化。例如，一個`slice`，是一個包含指向數據（內部`array`）的指針、長度和容量的三項描述符；在這些項目被初始化之前，`slice`為`nil`。對於`slice`、`map`和`channel`來說，`make`初始化了內部的數據結構，填充適當的值。

>`make`返回初始化後的（非零）值。

下面這個圖詳細的解釋了`new`和`make`之間的區別。

![](images/2.2.makenew.png?raw=true)

圖2.5 make和new對應底層的內存分配


## 零值
關於“零值”，所指並非是空值，而是一種“變量未填充前”的默認值，通常為0。
此處羅列 部分類型 的 “零值”

	int     0
	int8    0
	int32   0
	int64   0
	uint    0x0
	rune    0 //rune的實際類型是 int32
	byte    0x0 // byte的實際類型是 uint8
	float32 0 //長度為 4 byte
	float64 0 //長度為 8 byte
	bool    false
	string  ""

## links
   * [目錄](<preface.md>)
   * 上一章: [你好,Go](<02.1.md>)
   * 下一節: [流程和函數](<02.3.md>)

---

# 2.3 流程和函數
這小節我們要介紹Go裡面的流程控制以及函數操作。
## 流程控制
流程控制在編程語言中是最偉大的發明了，因為有了它，你可以通過很簡單的流程描述來表達很複雜的邏輯。Go中流程控制分三大類：條件判斷，循環控制和無條件跳轉。
### if
`if`也許是各種編程語言中最常見的了，它的語法概括起來就是:如果滿足條件就做某事，否則做另一件事。

Go裡面`if`條件判斷語句中不需要括號，如下代碼所示

	if x > 10 {
		fmt.Println("x is greater than 10")
	} else {
		fmt.Println("x is less than 10")
	}

Go的`if`還有一個強大的地方就是條件判斷語句裡面允許聲明一個變量，這個變量的作用域只能在該條件邏輯塊內，其他地方就不起作用了，如下所示

	// 計算獲取值x,然後根據x返回的大小，判斷是否大於10。
	if x := computedValue(); x > 10 {
		fmt.Println("x is greater than 10")
	} else {
		fmt.Println("x is less than 10")
	}

	//這個地方如果這樣調用就編譯出錯了，因為x是條件裡面的變量
	fmt.Println(x)

多個條件的時候如下所示：

	if integer == 3 {
		fmt.Println("The integer is equal to 3")
	} else if integer < 3 {
		fmt.Println("The integer is less than 3")
	} else {
		fmt.Println("The integer is greater than 3")
	}

### goto

Go有`goto`語句——請明智地使用它。用`goto`跳轉到必須在當前函數內定義的標籤。例如假設這樣一個循環：

	func myFunc() {
		i := 0
	Here:   //這行的第一個詞，以冒號結束作為標籤
		println(i)
		i++
		goto Here   //跳轉到Here去
	}

>標籤名是大小寫敏感的。

### for
Go裡面最強大的一個控制邏輯就是`for`，它即可以用來循環讀取數據，又可以當作`while`來控制邏輯，還能迭代操作。它的語法如下：

	for expression1; expression2; expression3 {
		//...
	}

`expression1`、`expression2`和`expression3`都是表達式，其中`expression1`和`expression3`是變量聲明或者函數調用返回值之類的，`expression2`是用來條件判斷，`expression1`在循環開始之前調用，`expression3`在每輪循環結束之時調用。

一個例子比上面講那麼多更有用，那麼我們看看下面的例子吧：

	package main
	import "fmt"

	func main(){
		sum := 0;
		for index:=0; index < 10 ; index++ {
			sum += index
		}
		fmt.Println("sum is equal to ", sum)
	}
	// 輸出：sum is equal to 45

有些時候需要進行多個賦值操作，由於Go裡面沒有`,`操作符，那麼可以使用平行賦值`i, j = i+1, j-1`


有些時候如果我們忽略`expression1`和`expression3`：

	sum := 1
	for ; sum < 1000;  {
		sum += sum
	}

其中`;`也可以省略，那麼就變成如下的代碼了，是不是似曾相識？對，這就是`while`的功能。

	sum := 1
	for sum < 1000 {
		sum += sum
	}

在循環裡面有兩個關鍵操作`break`和`continue`	,`break`操作是跳出當前循環，`continue`是跳過本次循環。當嵌套過深的時候，`break`可以配合標籤使用，即跳轉至標籤所指定的位置，詳細參考如下例子：

	for index := 10; index>0; index-- {
		if index == 5{
			break // 或者continue
		}
		fmt.Println(index)
	}
	// break打印出來10、9、8、7、6
	// continue打印出來10、9、8、7、6、4、3、2、1

`break`和`continue`還可以跟著標號，用來跳到多重循環中的外層循環

`for`配合`range`可以用於讀取`slice`和`map`的數據：

	for k,v:=range map {
		fmt.Println("map's key:",k)
		fmt.Println("map's val:",v)
	}

由於 Go 支持 “多值返回”, 而對於“聲明而未被調用”的變量, 編譯器會報錯, 在這種情況下, 可以使用`_`來丟棄不需要的返回值
例如

	for _, v := range map{
		fmt.Println("map's val:", v)
	}


### switch
有些時候你需要寫很多的`if-else`來實現一些邏輯處理，這個時候代碼看上去就很醜很冗長，而且也不易於以後的維護，這個時候`switch`就能很好的解決這個問題。它的語法如下

	switch sExpr {
	case expr1:
		some instructions
	case expr2:
		some other instructions
	case expr3:
		some other instructions
	default:
		other code
	}

`sExpr`和`expr1`、`expr2`、`expr3`的類型必須一致。Go的`switch`非常靈活，表達式不必是常量或整數，執行的過程從上至下，直到找到匹配項；而如果`switch`沒有表達式，它會匹配`true`。

	i := 10
	switch i {
	case 1:
		fmt.Println("i is equal to 1")
	case 2, 3, 4:
		fmt.Println("i is equal to 2, 3 or 4")
	case 10:
		fmt.Println("i is equal to 10")
	default:
		fmt.Println("All I know is that i is an integer")
	}

在第5行中，我們把很多值聚合在了一個`case`裡面，同時，Go裡面`switch`默認相當於每個`case`最後帶有`break`，匹配成功後不會自動向下執行其他case，而是跳出整個`switch`, 但是可以使用`fallthrough`強制執行後面的case代碼。

	integer := 6
	switch integer {
	case 4:
		fmt.Println("The integer was <= 4")
		fallthrough
	case 5:
		fmt.Println("The integer was <= 5")
		fallthrough
	case 6:
		fmt.Println("The integer was <= 6")
		fallthrough
	case 7:
		fmt.Println("The integer was <= 7")
		fallthrough
	case 8:
		fmt.Println("The integer was <= 8")
		fallthrough
	default:
		fmt.Println("default case")
	}

上面的程序將輸出

	The integer was <= 6
	The integer was <= 7
	The integer was <= 8
	default case


## 函數
函數是Go裡面的核心設計，它通過關鍵字`func`來聲明，它的格式如下：

	func funcName(input1 type1, input2 type2) (output1 type1, output2 type2) {
		//這裡是處理邏輯代碼
		//返回多個值
		return value1, value2
	}

上面的代碼我們看出

- 關鍵字`func`用來聲明一個函數`funcName`
- 函數可以有一個或者多個參數，每個參數後面帶有類型，通過`,`分隔
- 函數可以返回多個值
- 上面返回值聲明瞭兩個變量`output1`和`output2`，如果你不想聲明也可以，直接就兩個類型
- 如果只有一個返回值且不聲明返回值變量，那麼你可以省略 包括返回值 的括號
- 如果沒有返回值，那麼就直接省略最後的返回信息
- 如果有返回值， 那麼必須在函數的外層添加return語句

下面我們來看一個實際應用函數的例子（用來計算Max值）

	package main
	import "fmt"

	// 返回a、b中最大值.
	func max(a, b int) int {
		if a > b {
			return a
		}
		return b
	}

	func main() {
		x := 3
		y := 4
		z := 5

		max_xy := max(x, y) //調用函數max(x, y)
		max_xz := max(x, z) //調用函數max(x, z)

		fmt.Printf("max(%d, %d) = %d\n", x, y, max_xy)
		fmt.Printf("max(%d, %d) = %d\n", x, z, max_xz)
		fmt.Printf("max(%d, %d) = %d\n", y, z, max(y,z)) // 也可在這直接調用它
	}

上面這個裡面我們可以看到`max`函數有兩個參數，它們的類型都是`int`，那麼第一個變量的類型可以省略（即 a,b int,而非 a int, b int)，默認為離它最近的類型，同理多於2個同類型的變量或者返回值。同時我們注意到它的返回值就是一個類型，這個就是省略寫法。

### 多個返回值
Go語言比C更先進的特性，其中一點就是函數能夠返回多個值。

我們直接上代碼看例子

	package main
	import "fmt"

	//返回 A+B 和 A*B
	func SumAndProduct(A, B int) (int, int) {
		return A+B, A*B
	}

	func main() {
		x := 3
		y := 4

		xPLUSy, xTIMESy := SumAndProduct(x, y)

		fmt.Printf("%d + %d = %d\n", x, y, xPLUSy)
		fmt.Printf("%d * %d = %d\n", x, y, xTIMESy)
	}

上面的例子我們可以看到直接返回了兩個參數，當然我們也可以命名返回參數的變量，這個例子裡面只是用了兩個類型，我們也可以改成如下這樣的定義，然後返回的時候不用帶上變量名，因為直接在函數裡面初始化了。但如果你的函數是導出的(首字母大寫)，官方建議：最好命名返回值，因為不命名返回值，雖然使得代碼更加簡潔了，但是會造成生成的文檔可讀性差。

	func SumAndProduct(A, B int) (add int, Multiplied int) {
		add = A+B
		Multiplied = A*B
		return
	}

### 變參
Go函數支持變參。接受變參的函數是有著不定數量的參數的。為了做到這點，首先需要定義函數使其接受變參：

	func myfunc(arg ...int) {}
`arg ...int`告訴Go這個函數接受不定數量的參數。注意，這些參數的類型全部是`int`。在函數體中，變量`arg`是一個`int`的`slice`：

	for _, n := range arg {
		fmt.Printf("And the number is: %d\n", n)
	}

### 傳值與傳指針
當我們傳一個參數值到被調用函數裡面時，實際上是傳了這個值的一份copy，當在被調用函數中修改參數值的時候，調用函數中相應實參不會發生任何變化，因為數值變化只作用在copy上。

為了驗證我們上面的說法，我們來看一個例子

	package main
	import "fmt"

	//簡單的一個函數，實現了參數+1的操作
	func add1(a int) int {
		a = a+1 // 我們改變了a的值
		return a //返回一個新值
	}

	func main() {
		x := 3

		fmt.Println("x = ", x)  // 應該輸出 "x = 3"

		x1 := add1(x)  //調用add1(x)

		fmt.Println("x+1 = ", x1) // 應該輸出"x+1 = 4"
		fmt.Println("x = ", x)    // 應該輸出"x = 3"
	}

看到了嗎？雖然我們調用了`add1`函數，並且在`add1`中執行`a = a+1`操作，但是上面例子中`x`變量的值沒有發生變化

理由很簡單：因為當我們調用`add1`的時候，`add1`接收的參數其實是`x`的copy，而不是`x`本身。

那你也許會問了，如果真的需要傳這個`x`本身,該怎麼辦呢？

這就牽扯到了所謂的指針。我們知道，變量在內存中是存放於一定地址上的，修改變量實際是修改變量地址處的內存。只有`add1`函數知道`x`變量所在的地址，才能修改`x`變量的值。所以我們需要將`x`所在地址`&x`傳入函數，並將函數的參數的類型由`int`改為`*int`，即改為指針類型，才能在函數中修改`x`變量的值。此時參數仍然是按copy傳遞的，只是copy的是一個指針。請看下面的例子

	package main
	import "fmt"

	//簡單的一個函數，實現了參數+1的操作
	func add1(a *int) int { // 請注意，
		*a = *a+1 // 修改了a的值
		return *a // 返回新值
	}

	func main() {
		x := 3

		fmt.Println("x = ", x)  // 應該輸出 "x = 3"

		x1 := add1(&x)  // 調用 add1(&x) 傳x的地址

		fmt.Println("x+1 = ", x1) // 應該輸出 "x+1 = 4"
		fmt.Println("x = ", x)    // 應該輸出 "x = 4"
	}

這樣，我們就達到了修改`x`的目的。那麼到底傳指針有什麼好處呢？

- 傳指針使得多個函數能操作同一個對象。
- 傳指針比較輕量級 (8bytes),只是傳內存地址，我們可以用指針傳遞體積大的結構體。如果用參數值傳遞的話, 在每次copy上面就會花費相對較多的系統開銷（內存和時間）。所以當你要傳遞大的結構體的時候，用指針是一個明智的選擇。
- Go語言中`channel`，`slice`，`map`這三種類型的實現機制類似指針，所以可以直接傳遞，而不用取地址後傳遞指針。（注：若函數需改變`slice`的長度，則仍需要取地址傳遞指針）

### defer
Go語言中有種不錯的設計，即延遲（defer）語句，你可以在函數中添加多個defer語句。當函數執行到最後時，這些defer語句會按照逆序執行，最後該函數返回。特別是當你在進行一些打開資源的操作時，遇到錯誤需要提前返回，在返回前你需要關閉相應的資源，不然很容易造成資源洩露等問題。如下代碼所示，我們一般寫打開一個資源是這樣操作的：

	func ReadWrite() bool {
		file.Open("file")
	// 做一些工作
		if failureX {
			file.Close()
			return false
		}

		if failureY {
			file.Close()
			return false
		}

		file.Close()
		return true
	}

我們看到上面有很多重複的代碼，Go的`defer`有效解決了這個問題。使用它後，不但代碼量減少了很多，而且程序變得更優雅。在`defer`後指定的函數會在函數退出前調用。

	func ReadWrite() bool {
		file.Open("file")
		defer file.Close()
		if failureX {
			return false
		}
		if failureY {
			return false
		}
		return true
	}

如果有很多調用`defer`，那麼`defer`是採用後進先出模式，所以如下代碼會輸出`4 3 2 1 0`

	for i := 0; i < 5; i++ {
		defer fmt.Printf("%d ", i)
	}

### 函數作為值、類型

在Go中函數也是一種變量，我們可以通過`type`來定義它，它的類型就是所有擁有相同的參數，相同的返回值的一種類型

	type typeName func(input1 inputType1 , input2 inputType2 [, ...]) (result1 resultType1 [, ...])

函數作為類型到底有什麼好處呢？那就是可以把這個類型的函數當做值來傳遞，請看下面的例子

	package main
	import "fmt"

	type testInt func(int) bool // 聲明瞭一個函數類型

	func isOdd(integer int) bool {
		if integer%2 == 0 {
			return false
		}
		return true
	}

	func isEven(integer int) bool {
		if integer%2 == 0 {
			return true
		}
		return false
	}

	// 聲明的函數類型在這個地方當做了一個參數

	func filter(slice []int, f testInt) []int {
		var result []int
		for _, value := range slice {
			if f(value) {
				result = append(result, value)
			}
		}
		return result
	}

	func main(){
		slice := []int {1, 2, 3, 4, 5, 7}
		fmt.Println("slice = ", slice)
		odd := filter(slice, isOdd)    // 函數當做值來傳遞了
		fmt.Println("Odd elements of slice are: ", odd)
		even := filter(slice, isEven)  // 函數當做值來傳遞了
		fmt.Println("Even elements of slice are: ", even)
	}

函數當做值和類型在我們寫一些通用接口的時候非常有用，通過上面例子我們看到`testInt`這個類型是一個函數類型，然後兩個`filter`函數的參數和返回值與`testInt`類型是一樣的，但是我們可以實現很多種的邏輯，這樣使得我們的程序變得非常的靈活。

### Panic和Recover

Go沒有像Java那樣的異常機制，它不能拋出異常，而是使用了`panic`和`recover`機制。一定要記住，你應當把它作為最後的手段來使用，也就是說，你的代碼中應當沒有，或者很少有`panic`的東西。這是個強大的工具，請明智地使用它。那麼，我們應該如何使用它呢？

Panic
>是一個內建函數，可以中斷原有的控制流程，進入一個令人恐慌的流程中。當函數`F`調用`panic`，函數F的執行被中斷，但是`F`中的延遲函數會正常執行，然後F返回到調用它的地方。在調用的地方，`F`的行為就像調用了`panic`。這一過程繼續向上，直到發生`panic`的`goroutine`中所有調用的函數返回，此時程序退出。恐慌可以直接調用`panic`產生。也可以由運行時錯誤產生，例如訪問越界的數組。

Recover
>是一個內建的函數，可以讓進入令人恐慌的流程中的`goroutine`恢復過來。`recover`僅在延遲函數中有效。在正常的執行過程中，調用`recover`會返回`nil`，並且沒有其它任何效果。如果當前的`goroutine`陷入恐慌，調用`recover`可以捕獲到`panic`的輸入值，並且恢復正常的執行。

下面這個函數演示瞭如何在過程中使用`panic`

	var user = os.Getenv("USER")

	func init() {
		if user == "" {
			panic("no value for $USER")
		}
	}

下面這個函數檢查作為其參數的函數在執行時是否會產生`panic`：

	func throwsPanic(f func()) (b bool) {
		defer func() {
			if x := recover(); x != nil {
				b = true
			}
		}()
		f() //執行函數f，如果f中出現了panic，那麼就可以恢復回來
		return
	}

### `main`函數和`init`函數

Go裡面有兩個保留的函數：`init`函數（能夠應用於所有的`package`）和`main`函數（只能應用於`package main`）。這兩個函數在定義時不能有任何的參數和返回值。雖然一個`package`裡面可以寫任意多個`init`函數，但這無論是對於可讀性還是以後的可維護性來說，我們都強烈建議用戶在一個`package`中每個文件只寫一個`init`函數。

Go程序會自動調用`init()`和`main()`，所以你不需要在任何地方調用這兩個函數。每個`package`中的`init`函數都是可選的，但`package main`就必須包含一個`main`函數。

程序的初始化和執行都起始於`main`包。如果`main`包還導入了其它的包，那麼就會在編譯時將它們依次導入。有時一個包會被多個包同時導入，那麼它只會被導入一次（例如很多包可能都會用到`fmt`包，但它只會被導入一次，因為沒有必要導入多次）。當一個包被導入時，如果該包還導入了其它的包，那麼會先將其它包導入進來，然後再對這些包中的包級常量和變量進行初始化，接著執行`init`函數（如果有的話），依次類推。等所有被導入的包都加載完畢了，就會開始對`main`包中的包級常量和變量進行初始化，然後執行`main`包中的`init`函數（如果存在的話），最後執行`main`函數。下圖詳細地解釋了整個執行過程：

![](images/2.3.init.png?raw=true)

圖2.6 main函數引入包初始化流程圖

### import
我們在寫Go代碼的時候經常用到import這個命令用來導入包文件，而我們經常看到的方式參考如下：

	import(
	    "fmt"
	)

然後我們代碼裡面可以通過如下的方式調用

	fmt.Println("hello world")

上面這個fmt是Go語言的標準庫，其實是去`GOROOT`環境變量指定目錄下去加載該模塊，當然Go的import還支持如下兩種方式來加載自己寫的模塊：

1. 相對路徑

	import “./model” //當前文件同一目錄的model目錄，但是不建議這種方式來import

2. 絕對路徑

	import “shorturl/model” //加載gopath/src/shorturl/model模塊
	
	
上面展示了一些import常用的幾種方式，但是還有一些特殊的import，讓很多新手很費解，下面我們來一一講解一下到底是怎麼一回事
	
	
1. 點操作
	
	我們有時候會看到如下的方式導入包
	
		import(
		    . "fmt"
		)
	
	這個點操作的含義就是這個包導入之後在你調用這個包的函數時，你可以省略前綴的包名，也就是前面你調用的fmt.Println("hello world")可以省略的寫成Println("hello world")

2. 別名操作

	別名操作顧名思義我們可以把包命名成另一個我們用起來容易記憶的名字
	
		import(
		    f "fmt"
		)
		
	別名操作的話調用包函數時前綴變成了我們的前綴，即f.Println("hello world")

3. _操作

	這個操作經常是讓很多人費解的一個操作符，請看下面這個import
	
		import (
		    "database/sql"
		    _ "github.com/ziutek/mymysql/godrv"
		)
		
	_操作其實是引入該包，而不直接使用包裡面的函數，而是調用了該包裡面的init函數。


## links
   * [目錄](<preface.md>)
   * 上一章: [Go基礎](<02.2.md>)
   * 下一節: [struct類型](<02.4.md>)

---

# 2.4 struct類型
## struct
Go語言中，也和C或者其他語言一樣，我們可以聲明新的類型，作為其它類型的屬性或字段的容器。例如，我們可以創建一個自定義類型`person`代表一個人的實體。這個實體擁有屬性：姓名和年齡。這樣的類型我們稱之`struct`。如下代碼所示:

	type person struct {
		name string
		age int
	}
看到了嗎？聲明一個struct如此簡單，上面的類型包含有兩個字段
- 一個string類型的字段name，用來保存用戶名稱這個屬性
- 一個int類型的字段age,用來保存用戶年齡這個屬性

如何使用struct呢？請看下面的代碼

	type person struct {
		name string
		age int
	}

	var P person  // P現在就是person類型的變量了

	P.name = "Astaxie"  // 賦值"Astaxie"給P的name屬性.
	P.age = 25  // 賦值"25"給變量P的age屬性
	fmt.Printf("The person's name is %s", P.name)  // 訪問P的name屬性.
除了上面這種P的聲明使用之外，還有另外幾種聲明使用方式：

- 1.按照順序提供初始化值

	P := person{"Tom", 25}

- 2.通過`field:value`的方式初始化，這樣可以任意順序

	P := person{age:24, name:"Tom"}

- 3.當然也可以通過`new`函數分配一個指針，此處P的類型為*person
	
	P := new(person)

下面我們看一個完整的使用struct的例子

	package main
	import "fmt"

	// 聲明一個新的類型
	type person struct {
		name string
		age int
	}

	// 比較兩個人的年齡，返回年齡大的那個人，並且返回年齡差
	// struct也是傳值的
	func Older(p1, p2 person) (person, int) {
		if p1.age>p2.age {  // 比較p1和p2這兩個人的年齡
			return p1, p1.age-p2.age
		}
		return p2, p2.age-p1.age
	}

	func main() {
		var tom person

		// 賦值初始化
		tom.name, tom.age = "Tom", 18

		// 兩個字段都寫清楚的初始化
		bob := person{age:25, name:"Bob"}

		// 按照struct定義順序初始化值
		paul := person{"Paul", 43}

		tb_Older, tb_diff := Older(tom, bob)
		tp_Older, tp_diff := Older(tom, paul)
		bp_Older, bp_diff := Older(bob, paul)

		fmt.Printf("Of %s and %s, %s is older by %d years\n",
			tom.name, bob.name, tb_Older.name, tb_diff)

		fmt.Printf("Of %s and %s, %s is older by %d years\n",
			tom.name, paul.name, tp_Older.name, tp_diff)

		fmt.Printf("Of %s and %s, %s is older by %d years\n",
			bob.name, paul.name, bp_Older.name, bp_diff)
	}

### struct的匿名字段
我們上面介紹瞭如何定義一個struct，定義的時候是字段名與其類型一一對應，實際上Go支持只提供類型，而不寫字段名的方式，也就是匿名字段，也稱為嵌入字段。

當匿名字段是一個struct的時候，那麼這個struct所擁有的全部字段都被隱式地引入了當前定義的這個struct。

讓我們來看一個例子，讓上面說的這些更具體化

	package main
	import "fmt"

	type Human struct {
		name string
		age int
		weight int
	}

	type Student struct {
		Human  // 匿名字段，那麼默認Student就包含了Human的所有字段
		speciality string
	}

	func main() {
		// 我們初始化一個學生
		mark := Student{Human{"Mark", 25, 120}, "Computer Science"}

		// 我們訪問相應的字段
		fmt.Println("His name is ", mark.name)
		fmt.Println("His age is ", mark.age)
		fmt.Println("His weight is ", mark.weight)
		fmt.Println("His speciality is ", mark.speciality)
		// 修改對應的備註信息
		mark.speciality = "AI"
		fmt.Println("Mark changed his speciality")
		fmt.Println("His speciality is ", mark.speciality)
		// 修改他的年齡信息
		fmt.Println("Mark become old")
		mark.age = 46
		fmt.Println("His age is", mark.age)
		// 修改他的體重信息
		fmt.Println("Mark is not an athlet anymore")
		mark.weight += 60
		fmt.Println("His weight is", mark.weight)
	}

圖例如下:

![](images/2.4.student_struct.png?raw=true)

圖2.7 struct組合，Student組合了Human struct和string基本類型

我們看到Student訪問屬性age和name的時候，就像訪問自己所有用的字段一樣，對，匿名字段就是這樣，能夠實現字段的繼承。是不是很酷啊？還有比這個更酷的呢，那就是student還能訪問Human這個字段作為字段名。請看下面的代碼，是不是更酷了。

	mark.Human = Human{"Marcus", 55, 220}
	mark.Human.age -= 1

通過匿名訪問和修改字段相當的有用，但是不僅僅是struct字段哦，所有的內置類型和自定義類型都是可以作為匿名字段的。請看下面的例子

	package main
	import "fmt"

	type Skills []string

	type Human struct {
		name string
		age int
		weight int
	}

	type Student struct {
		Human  // 匿名字段，struct
		Skills // 匿名字段，自定義的類型string slice
		int    // 內置類型作為匿名字段
		speciality string
	}

	func main() {
		// 初始化學生Jane
		jane := Student{Human:Human{"Jane", 35, 100}, speciality:"Biology"}
		// 現在我們來訪問相應的字段
		fmt.Println("Her name is ", jane.name)
		fmt.Println("Her age is ", jane.age)
		fmt.Println("Her weight is ", jane.weight)
		fmt.Println("Her speciality is ", jane.speciality)
		// 我們來修改他的skill技能字段
		jane.Skills = []string{"anatomy"}
		fmt.Println("Her skills are ", jane.Skills)
		fmt.Println("She acquired two new ones ")
		jane.Skills = append(jane.Skills, "physics", "golang")
		fmt.Println("Her skills now are ", jane.Skills)
		// 修改匿名內置類型字段
		jane.int = 3
		fmt.Println("Her preferred number is", jane.int)
	}

從上面例子我們看出來struct不僅僅能夠將struct作為匿名字段、自定義類型、內置類型都可以作為匿名字段，而且可以在相應的字段上面進行函數操作（如例子中的append）。

這裡有一個問題：如果human裡面有一個字段叫做phone，而student也有一個字段叫做phone，那麼該怎麼辦呢？

Go裡面很簡單的解決了這個問題，最外層的優先訪問，也就是當你通過`student.phone`訪問的時候，是訪問student裡面的字段，而不是human裡面的字段。

這樣就允許我們去重載通過匿名字段繼承的一些字段，當然如果我們想訪問重載後對應匿名類型裡面的字段，可以通過匿名字段名來訪問。請看下面的例子

	package main
	import "fmt"

	type Human struct {
		name string
		age int
		phone string  // Human類型擁有的字段
	}

	type Employee struct {
		Human  // 匿名字段Human
		speciality string
		phone string  // 僱員的phone字段
	}

	func main() {
		Bob := Employee{Human{"Bob", 34, "777-444-XXXX"}, "Designer", "333-222"}
		fmt.Println("Bob's work phone is:", Bob.phone)
		// 如果我們要訪問Human的phone字段
		fmt.Println("Bob's personal phone is:", Bob.Human.phone)
	}


## links
   * [目錄](<preface.md>)
   * 上一章: [流程和函數](<02.3.md>)
   * 下一節: [面向對象](<02.5.md>)

---

# 2.5 面向對象
前面兩章我們介紹了函數和struct，那你是否想過函數當作struct的字段一樣來處理呢？今天我們就講解一下函數的另一種形態，帶有接收者的函數，我們稱為`method`

## method
現在假設有這麼一個場景，你定義了一個struct叫做長方形，你現在想要計算他的面積，那麼按照我們一般的思路應該會用下面的方式來實現

	package main
	import "fmt"

	type Rectangle struct {
		width, height float64
	}

	func area(r Rectangle) float64 {
		return r.width*r.height
	}

	func main() {
		r1 := Rectangle{12, 2}
		r2 := Rectangle{9, 4}
		fmt.Println("Area of r1 is: ", area(r1))
		fmt.Println("Area of r2 is: ", area(r2))
	}

這段代碼可以計算出來長方形的面積，但是area()不是作為Rectangle的方法實現的（類似面向對象裡面的方法），而是將Rectangle的對象（如r1,r2）作為參數傳入函數計算面積的。

這樣實現當然沒有問題咯，但是當需要增加圓形、正方形、五邊形甚至其它多邊形的時候，你想計算他們的面積的時候怎麼辦啊？那就只能增加新的函數咯，但是函數名你就必須要跟著換了，變成`area_rectangle, area_circle, area_triangle...`

像下圖所表示的那樣， 橢圓代表函數, 而這些函數並不從屬於struct(或者以面向對象的術語來說，並不屬於class)，他們是單獨存在於struct外圍，而非在概念上屬於某個struct的。

![](images/2.5.rect_func_without_receiver.png?raw=true)

圖2.8 方法和struct的關係圖

很顯然，這樣的實現並不優雅，並且從概念上來說"面積"是"形狀"的一個屬性，它是屬於這個特定的形狀的，就像長方形的長和寬一樣。

基於上面的原因所以就有了`method`的概念，`method`是附屬在一個給定的類型上的，他的語法和函數的聲明語法幾乎一樣，只是在`func`後面增加了一個receiver(也就是method所依從的主體)。

用上面提到的形狀的例子來說，method `area()` 是依賴於某個形狀(比如說Rectangle)來發生作用的。Rectangle.area()的發出者是Rectangle， area()是屬於Rectangle的方法，而非一個外圍函數。

更具體地說，Rectangle存在字段length 和 width, 同時存在方法area(), 這些字段和方法都屬於Rectangle。

用Rob Pike的話來說就是：

>"A method is a function with an implicit first argument, called a receiver."

method的語法如下：

	func (r ReceiverType) funcName(parameters) (results)

下面我們用最開始的例子用method來實現：

	package main
	import (
		"fmt"
		"math"
	)

	type Rectangle struct {
		width, height float64
	}

	type Circle struct {
		radius float64
	}

	func (r Rectangle) area() float64 {
		return r.width*r.height
	}

	func (c Circle) area() float64 {
		return c.radius * c.radius * math.Pi
	}


	func main() {
		r1 := Rectangle{12, 2}
		r2 := Rectangle{9, 4}
		c1 := Circle{10}
		c2 := Circle{25}

		fmt.Println("Area of r1 is: ", r1.area())
		fmt.Println("Area of r2 is: ", r2.area())
		fmt.Println("Area of c1 is: ", c1.area())
		fmt.Println("Area of c2 is: ", c2.area())
	}



在使用method的時候重要注意幾點

- 雖然method的名字一模一樣，但是如果接收者不一樣，那麼method就不一樣
- method裡面可以訪問接收者的字段
- 調用method通過`.`訪問，就像struct裡面訪問字段一樣

圖示如下:

![](images/2.5.shapes_func_with_receiver_cp.png?raw=true)

圖2.9 不同struct的method不同

在上例，method area() 分別屬於Rectangle和Circle， 於是他們的 Receiver 就變成了Rectangle 和 Circle, 或者說，這個area()方法 是由 Rectangle/Circle 發出的。

>值得說明的一點是，圖示中method用虛線標出，意思是此處方法的Receiver是以值傳遞，而非引用傳遞，是的，Receiver還可以是指針, 兩者的差別在於, 指針作為Receiver會對實例對象的內容發生操作,而普通類型作為Receiver僅僅是以副本作為操作對象,並不對原實例對象發生操作。後文對此會有詳細論述。

那是不是method只能作用在struct上面呢？當然不是咯，他可以定義在任何你自定義的類型、內置類型、struct等各種類型上面。這裡你是不是有點迷糊了，什麼叫自定義類型，自定義類型不就是struct嘛，不是這樣的哦，struct只是自定義類型裡面一種比較特殊的類型而已，還有其他自定義類型申明，可以通過如下這樣的申明來實現。

	type typeName typeLiteral

請看下面這個申明自定義類型的代碼

	type ages int

	type money float32

	type months map[string]int

	m := months {
		"January":31,
		"February":28,
		...
		"December":31,
	}

看到了嗎？簡單的很吧，這樣你就可以在自己的代碼裡面定義有意義的類型了，實際上只是一個定義了一個別名,有點類似於c中的typedef，例如上面ages替代了int

好了，讓我們回到`method`

你可以在任何的自定義類型中定義任意多的`method`，接下來讓我們看一個複雜一點的例子

	package main
	import "fmt"

	const(
		WHITE = iota
		BLACK
		BLUE
		RED
		YELLOW
	)

	type Color byte

	type Box struct {
		width, height, depth float64
		color Color
	}

	type BoxList []Box //a slice of boxes

	func (b Box) Volume() float64 {
		return b.width * b.height * b.depth
	}

	func (b *Box) SetColor(c Color) {
		b.color = c
	}

	func (bl BoxList) BiggestColor() Color {
		v := 0.00
		k := Color(WHITE)
		for _, b := range bl {
			if bv := b.Volume(); bv > v {
				v = bv
				k = b.color
			}
		}
		return k
	}

	func (bl BoxList) PaintItBlack() {
		for i, _ := range bl {
			bl[i].SetColor(BLACK)
		}
	}

	func (c Color) String() string {
		strings := []string {"WHITE", "BLACK", "BLUE", "RED", "YELLOW"}
		return strings[c]
	}

	func main() {
		boxes := BoxList {
			Box{4, 4, 4, RED},
			Box{10, 10, 1, YELLOW},
			Box{1, 1, 20, BLACK},
			Box{10, 10, 1, BLUE},
			Box{10, 30, 1, WHITE},
			Box{20, 20, 20, YELLOW},
		}

		fmt.Printf("We have %d boxes in our set\n", len(boxes))
		fmt.Println("The volume of the first one is", boxes[0].Volume(), "cm³")
		fmt.Println("The color of the last one is",boxes[len(boxes)-1].color.String())
		fmt.Println("The biggest one is", boxes.BiggestColor().String())

		fmt.Println("Let's paint them all black")
		boxes.PaintItBlack()
		fmt.Println("The color of the second one is", boxes[1].color.String())

		fmt.Println("Obviously, now, the biggest one is", boxes.BiggestColor().String())
	}

上面的代碼通過const定義了一些常量，然後定義了一些自定義類型

- Color作為byte的別名
- 定義了一個struct:Box，含有三個長寬高字段和一個顏色屬性
- 定義了一個slice:BoxList，含有Box

然後以上面的自定義類型為接收者定義了一些method

- Volume()定義了接收者為Box，返回Box的容量
- SetColor(c Color)，把Box的顏色改為c
- BiggestColor()定在在BoxList上面，返回list裡面容量最大的顏色
- PaintItBlack()把BoxList裡面所有Box的顏色全部變成黑色
- String()定義在Color上面，返回Color的具體顏色(字符串格式)

上面的代碼通過文字描述出來之後是不是很簡單？我們一般解決問題都是通過問題的描述，去寫相應的代碼實現。

### 指針作為receiver
現在讓我們回過頭來看看SetColor這個method，它的receiver是一個指向Box的指針，是的，你可以使用*Box。想想為啥要使用指針而不是Box本身呢？

我們定義SetColor的真正目的是想改變這個Box的顏色，如果不傳Box的指針，那麼SetColor接受的其實是Box的一個copy，也就是說method內對於顏色值的修改，其實只作用於Box的copy，而不是真正的Box。所以我們需要傳入指針。

這裡可以把receiver當作method的第一個參數來看，然後結合前面函數講解的傳值和傳引用就不難理解

這裡你也許會問了那SetColor函數裡面應該這樣定義`*b.Color=c`,而不是`b.Color=c`,因為我們需要讀取到指針相應的值。

你是對的，其實Go裡面這兩種方式都是正確的，當你用指針去訪問相應的字段時(雖然指針沒有任何的字段)，Go知道你要通過指針去獲取這個值，看到了吧，Go的設計是不是越來越吸引你了。

也許細心的讀者會問這樣的問題，PaintItBlack裡面調用SetColor的時候是不是應該寫成`(&bl[i]).SetColor(BLACK)`，因為SetColor的receiver是*Box，而不是Box。

你又說對的，這兩種方式都可以，因為Go知道receiver是指針，他自動幫你轉了。

也就是說：
>如果一個method的receiver是*T,你可以在一個T類型的實例變量V上面調用這個method，而不需要&V去調用這個method

類似的
>如果一個method的receiver是T，你可以在一個*T類型的變量P上面調用這個method，而不需要 *P去調用這個method

所以，你不用擔心你是調用的指針的method還是不是指針的method，Go知道你要做的一切，這對於有多年C/C++編程經驗的同學來說，真是解決了一個很大的痛苦。

### method繼承
前面一章我們學習了字段的繼承，那麼你也會發現Go的一個神奇之處，method也是可以繼承的。如果匿名字段實現了一個method，那麼包含這個匿名字段的struct也能調用該method。讓我們來看下面這個例子

	package main
	import "fmt"

	type Human struct {
		name string
		age int
		phone string
	}

	type Student struct {
		Human //匿名字段
		school string
	}

	type Employee struct {
		Human //匿名字段
		company string
	}

	//在human上面定義了一個method
	func (h *Human) SayHi() {
		fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
	}

	func main() {
		mark := Student{Human{"Mark", 25, "222-222-YYYY"}, "MIT"}
		sam := Employee{Human{"Sam", 45, "111-888-XXXX"}, "Golang Inc"}

		mark.SayHi()
		sam.SayHi()
	}

### method重寫
上面的例子中，如果Employee想要實現自己的SayHi,怎麼辦？簡單，和匿名字段衝突一樣的道理，我們可以在Employee上面定義一個method，重寫了匿名字段的方法。請看下面的例子

	package main
	import "fmt"

	type Human struct {
		name string
		age int
		phone string
	}

	type Student struct {
		Human //匿名字段
		school string
	}

	type Employee struct {
		Human //匿名字段
		company string
	}

	//Human定義method
	func (h *Human) SayHi() {
		fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
	}

	//Employee的method重寫Human的method
	func (e *Employee) SayHi() {
		fmt.Printf("Hi, I am %s, I work at %s. Call me on %s\n", e.name,
			e.company, e.phone) //Yes you can split into 2 lines here.
	}

	func main() {
		mark := Student{Human{"Mark", 25, "222-222-YYYY"}, "MIT"}
		sam := Employee{Human{"Sam", 45, "111-888-XXXX"}, "Golang Inc"}

		mark.SayHi()
		sam.SayHi()
	}

上面的代碼設計的是如此的美妙，讓人不自覺的為Go的設計驚歎！

通過這些內容，我們可以設計出基本的面向對象的程序了，但是Go裡面的面向對象是如此的簡單，沒有任何的私有、公有關鍵字，通過大小寫來實現(大寫開頭的為公有，小寫開頭的為私有)，方法也同樣適用這個原則。
## links
   * [目錄](<preface.md>)
   * 上一章: [struct類型](<02.4.md>)
   * 下一節: [interface](<02.6.md>)

---

# 2.6 interface

## interface
Go語言裡面設計最精妙的應該算interface，它讓面向對象，內容組織實現非常的方便，當你看完這一章，你就會被interface的巧妙設計所折服。
### 什麼是interface
簡單的說，interface是一組method簽名的組合，我們通過interface來定義對象的一組行為。

我們前面一章最後一個例子中Student和Employee都能SayHi，雖然他們的內部實現不一樣，但是那不重要，重要的是他們都能`say hi`

讓我們來繼續做更多的擴展，Student和Employee實現另一個方法`Sing`，然後Student實現方法BorrowMoney而Employee實現SpendSalary。

這樣Student實現了三個方法：SayHi、Sing、BorrowMoney；而Employee實現了SayHi、Sing、SpendSalary。

上面這些方法的組合稱為interface(被對象Student和Employee實現)。例如Student和Employee都實現了interface：SayHi和Sing，也就是這兩個對象是該interface類型。而Employee沒有實現這個interface：SayHi、Sing和BorrowMoney，因為Employee沒有實現BorrowMoney這個方法。
### interface類型
interface類型定義了一組方法，如果某個對象實現了某個接口的所有方法，則此對象就實現了此接口。詳細的語法參考下面這個例子

	type Human struct {
		name string
		age int
		phone string
	}

	type Student struct {
		Human //匿名字段Human
		school string
		loan float32
	}

	type Employee struct {
		Human //匿名字段Human
		company string
		money float32
	}

	//Human對象實現Sayhi方法
	func (h *Human) SayHi() {
		fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
	}

	// Human對象實現Sing方法
	func (h *Human) Sing(lyrics string) {
		fmt.Println("La la, la la la, la la la la la...", lyrics)
	}

	//Human對象實現Guzzle方法
	func (h *Human) Guzzle(beerStein string) {
		fmt.Println("Guzzle Guzzle Guzzle...", beerStein)
	}

	// Employee重載Human的Sayhi方法
	func (e *Employee) SayHi() {
		fmt.Printf("Hi, I am %s, I work at %s. Call me on %s\n", e.name,
			e.company, e.phone) //此句可以分成多行
	}

	//Student實現BorrowMoney方法
	func (s *Student) BorrowMoney(amount float32) {
		s.loan += amount // (again and again and...)
	}

	//Employee實現SpendSalary方法
	func (e *Employee) SpendSalary(amount float32) {
		e.money -= amount // More vodka please!!! Get me through the day!
	}

	// 定義interface
	type Men interface {
		SayHi()
		Sing(lyrics string)
		Guzzle(beerStein string)
	}

	type YoungChap interface {
		SayHi()
		Sing(song string)
		BorrowMoney(amount float32)
	}

	type ElderlyGent interface {
		SayHi()
		Sing(song string)
		SpendSalary(amount float32)
	}

通過上面的代碼我們可以知道，interface可以被任意的對象實現。我們看到上面的Men interface被Human、Student和Employee實現。同理，一個對象可以實現任意多個interface，例如上面的Student實現了Men和YoungChap兩個interface。

最後，任意的類型都實現了空interface(我們這樣定義：interface{})，也就是包含0個method的interface。

### interface值
那麼interface裡面到底能存什麼值呢？如果我們定義了一個interface的變量，那麼這個變量裡面可以存實現這個interface的任意類型的對象。例如上面例子中，我們定義了一個Men interface類型的變量m，那麼m裡面可以存Human、Student或者Employee值。

因為m能夠持有這三種類型的對象，所以我們可以定義一個包含Men類型元素的slice，這個slice可以被賦予實現了Men接口的任意結構的對象，這個和我們傳統意義上面的slice有所不同。

讓我們來看一下下面這個例子:

	package main
	import "fmt"

	type Human struct {
		name string
		age int
		phone string
	}

	type Student struct {
		Human //匿名字段
		school string
		loan float32
	}

	type Employee struct {
		Human //匿名字段
		company string
		money float32
	}

	//Human實現SayHi方法
	func (h Human) SayHi() {
		fmt.Printf("Hi, I am %s you can call me on %s\n", h.name, h.phone)
	}

	//Human實現Sing方法
	func (h Human) Sing(lyrics string) {
		fmt.Println("La la la la...", lyrics)
	}

	//Employee重載Human的SayHi方法
	func (e Employee) SayHi() {
		fmt.Printf("Hi, I am %s, I work at %s. Call me on %s\n", e.name,
			e.company, e.phone)
		}

	// Interface Men被Human,Student和Employee實現
	// 因為這三個類型都實現了這兩個方法
	type Men interface {
		SayHi()
		Sing(lyrics string)
	}

	func main() {
		mike := Student{Human{"Mike", 25, "222-222-XXX"}, "MIT", 0.00}
		paul := Student{Human{"Paul", 26, "111-222-XXX"}, "Harvard", 100}
		sam := Employee{Human{"Sam", 36, "444-222-XXX"}, "Golang Inc.", 1000}
		tom := Employee{Human{"Tom", 37, "222-444-XXX"}, "Things Ltd.", 5000}

		//定義Men類型的變量i
		var i Men

		//i能存儲Student
		i = mike
		fmt.Println("This is Mike, a Student:")
		i.SayHi()
		i.Sing("November rain")

		//i也能存儲Employee
		i = tom
		fmt.Println("This is tom, an Employee:")
		i.SayHi()
		i.Sing("Born to be wild")

		//定義了slice Men
		fmt.Println("Let's use a slice of Men and see what happens")
		x := make([]Men, 3)
		//這三個都是不同類型的元素，但是他們實現了interface同一個接口
		x[0], x[1], x[2] = paul, sam, mike

		for _, value := range x{
			value.SayHi()
		}
	}

通過上面的代碼，你會發現interface就是一組抽象方法的集合，它必須由其他非interface類型實現，而不能自我實現， Go通過interface實現了duck-typing:即"當看到一隻鳥走起來像鴨子、游泳起來像鴨子、叫起來也像鴨子，那麼這隻鳥就可以被稱為鴨子"。

### 空interface
空interface(interface{})不包含任何的method，正因為如此，所有的類型都實現了空interface。空interface對於描述起不到任何的作用(因為它不包含任何的method），但是空interface在我們需要存儲任意類型的數值的時候相當有用，因為它可以存儲任意類型的數值。它有點類似於C語言的void*類型。

	// 定義a為空接口
	var a interface{}
	var i int = 5
	s := "Hello world"
	// a可以存儲任意類型的數值
	a = i
	a = s

一個函數把interface{}作為參數，那麼他可以接受任意類型的值作為參數，如果一個函數返回interface{},那麼也就可以返回任意類型的值。是不是很有用啊！
### interface函數參數
interface的變量可以持有任意實現該interface類型的對象，這給我們編寫函數(包括method)提供了一些額外的思考，我們是不是可以通過定義interface參數，讓函數接受各種類型的參數。

舉個例子：fmt.Println是我們常用的一個函數，但是你是否注意到它可以接受任意類型的數據。打開fmt的源碼文件，你會看到這樣一個定義:

	type Stringer interface {
		 String() string
	}
也就是說，任何實現了String方法的類型都能作為參數被fmt.Println調用,讓我們來試一試

	package main
	import (
		"fmt"
		"strconv"
	)

	type Human struct {
		name string
		age int
		phone string
	}

	// 通過這個方法 Human 實現了 fmt.Stringer
	func (h Human) String() string {
		return "❰"+h.name+" - "+strconv.Itoa(h.age)+" years -  ✆ " +h.phone+"❱"
	}

	func main() {
		Bob := Human{"Bob", 39, "000-7777-XXX"}
		fmt.Println("This Human is : ", Bob)
	}
現在我們再回顧一下前面的Box示例，你會發現Color結構也定義了一個method：String。其實這也是實現了fmt.Stringer這個interface，即如果需要某個類型能被fmt包以特殊的格式輸出，你就必須實現Stringer這個接口。如果沒有實現這個接口，fmt將以默認的方式輸出。

	//實現同樣的功能
	fmt.Println("The biggest one is", boxes.BiggestsColor().String())
	fmt.Println("The biggest one is", boxes.BiggestsColor())

注：實現了error接口的對象（即實現了Error() string的對象），使用fmt輸出時，會調用Error()方法，因此不必再定義String()方法了。
### interface變量存儲的類型

我們知道interface的變量裡面可以存儲任意類型的數值(該類型實現了interface)。那麼我們怎麼反向知道這個變量裡面實際保存了的是哪個類型的對象呢？目前常用的有兩種方法：

- Comma-ok斷言

	Go語言裡面有一個語法，可以直接判斷是否是該類型的變量： value, ok = element.(T)，這裡value就是變量的值，ok是一個bool類型，element是interface變量，T是斷言的類型。

	如果element裡面確實存儲了T類型的數值，那麼ok返回true，否則返回false。

	讓我們通過一個例子來更加深入的理解。

		package main

		import (
			"fmt"
			"strconv"
		)

		type Element interface{}
		type List [] Element

		type Person struct {
			name string
			age int
		}

		//定義了String方法，實現了fmt.Stringer
		func (p Person) String() string {
			return "(name: " + p.name + " - age: "+strconv.Itoa(p.age)+ " years)"
		}

		func main() {
			list := make(List, 3)
			list[0] = 1 // an int
			list[1] = "Hello" // a string
			list[2] = Person{"Dennis", 70}

			for index, element := range list {
				if value, ok := element.(int); ok {
					fmt.Printf("list[%d] is an int and its value is %d\n", index, value)
				} else if value, ok := element.(string); ok {
					fmt.Printf("list[%d] is a string and its value is %s\n", index, value)
				} else if value, ok := element.(Person); ok {
					fmt.Printf("list[%d] is a Person and its value is %s\n", index, value)
				} else {
					fmt.Printf("list[%d] is of a different type\n", index)
				}
			}
		}

	是不是很簡單啊，同時你是否注意到了多個if裡面，還記得我前面介紹流程時講過，if裡面允許初始化變量。

	也許你注意到了，我們斷言的類型越多，那麼if else也就越多，所以才引出了下面要介紹的switch。
- switch測試

	最好的講解就是代碼例子，現在讓我們重寫上面的這個實現

		package main

		import (
			"fmt"
			"strconv"
		)

		type Element interface{}
		type List [] Element

		type Person struct {
			name string
			age int
		}

		//打印
		func (p Person) String() string {
			return "(name: " + p.name + " - age: "+strconv.Itoa(p.age)+ " years)"
		}

		func main() {
			list := make(List, 3)
			list[0] = 1 //an int
			list[1] = "Hello" //a string
			list[2] = Person{"Dennis", 70}

			for index, element := range list{
				switch value := element.(type) {
					case int:
						fmt.Printf("list[%d] is an int and its value is %d\n", index, value)
					case string:
						fmt.Printf("list[%d] is a string and its value is %s\n", index, value)
					case Person:
						fmt.Printf("list[%d] is a Person and its value is %s\n", index, value)
					default:
						fmt.Println("list[%d] is of a different type", index)
				}
			}
		}

	這裡有一點需要強調的是：`element.(type)`語法不能在switch外的任何邏輯裡面使用，如果你要在switch外面判斷一個類型就使用`comma-ok`。

### 嵌入interface
Go裡面真正吸引人的是它內置的邏輯語法，就像我們在學習Struct時學習的匿名字段，多麼的優雅啊，那麼相同的邏輯引入到interface裡面，那不是更加完美了。如果一個interface1作為interface2的一個嵌入字段，那麼interface2隱式的包含了interface1裡面的method。

我們可以看到源碼包container/heap裡面有這樣的一個定義

	type Interface interface {
		sort.Interface //嵌入字段sort.Interface
		Push(x interface{}) //a Push method to push elements into the heap
		Pop() interface{} //a Pop elements that pops elements from the heap
	}

我們看到sort.Interface其實就是嵌入字段，把sort.Interface的所有method給隱式的包含進來了。也就是下面三個方法：

	type Interface interface {
		// Len is the number of elements in the collection.
		Len() int
		// Less returns whether the element with index i should sort
		// before the element with index j.
		Less(i, j int) bool
		// Swap swaps the elements with indexes i and j.
		Swap(i, j int)
	}

另一個例子就是io包下面的 io.ReadWriter ，它包含了io包下面的Reader和Writer兩個interface：

	// io.ReadWriter
	type ReadWriter interface {
		Reader
		Writer
	}

### 反射
Go語言實現了反射，所謂反射就是能檢查程序在運行時的狀態。我們一般用到的包是reflect包。如何運用reflect包，官方的這篇文章詳細的講解了reflect包的實現原理，[laws of reflection](http://golang.org/doc/articles/laws_of_reflection.html)

使用reflect一般分成三步，下面簡要的講解一下：要去反射是一個類型的值(這些值都實現了空interface)，首先需要把它轉化成reflect對象(reflect.Type或者reflect.Value，根據不同的情況調用不同的函數)。這兩種獲取方式如下：

	t := reflect.TypeOf(i)    //得到類型的元數據,通過t我們能獲取類型定義裡面的所有元素
	v := reflect.ValueOf(i)   //得到實際的值，通過v我們獲取存儲在裡面的值，還可以去改變值

轉化為reflect對象之後我們就可以進行一些操作了，也就是將reflect對象轉化成相應的值，例如

	tag := t.Elem().Field(0).Tag  //獲取定義在struct裡面的標籤
	name := v.Elem().Field(0).String()  //獲取存儲在第一個字段裡面的值

獲取反射值能返回相應的類型和數值

	var x float64 = 3.4
	v := reflect.ValueOf(x)
	fmt.Println("type:", v.Type())
	fmt.Println("kind is float64:", v.Kind() == reflect.Float64)
	fmt.Println("value:", v.Float())

最後，反射的話，那麼反射的字段必須是可修改的，我們前面學習過傳值和傳引用，這個裡面也是一樣的道理。反射的字段必須是可讀寫的意思是，如果下面這樣寫，那麼會發生錯誤

	var x float64 = 3.4
	v := reflect.ValueOf(x)
	v.SetFloat(7.1)

如果要修改相應的值，必須這樣寫

	var x float64 = 3.4
	p := reflect.ValueOf(&x)
	v := p.Elem()
	v.SetFloat(7.1)

上面只是對反射的簡單介紹，更深入的理解還需要自己在編程中不斷的實踐。

## links
   * [目錄](<preface.md>)
   * 上一章: [面向對象](<02.5.md>)
   * 下一節: [併發](<02.7.md>)

---

# 2.7 併發

有人把Go比作21世紀的C語言，第一是因為Go語言設計簡單，第二，21世紀最重要的就是並行程序設計，而Go從語言層面就支持了並行。

## goroutine

goroutine是Go並行設計的核心。goroutine說到底其實就是線程，但是它比線程更小，十幾個goroutine可能體現在底層就是五六個線程，Go語言內部幫你實現了這些goroutine之間的內存共享。執行goroutine只需極少的棧內存(大概是4~5KB)，當然會根據相應的數據伸縮。也正因為如此，可同時運行成千上萬個併發任務。goroutine比thread更易用、更高效、更輕便。

goroutine是通過Go的runtime管理的一個線程管理器。goroutine通過`go`關鍵字實現了，其實就是一個普通的函數。

	go hello(a, b, c)

通過關鍵字go就啟動了一個goroutine。我們來看一個例子

	package main

	import (
		"fmt"
		"runtime"
	)

	func say(s string) {
		for i := 0; i < 5; i++ {
			runtime.Gosched()
			fmt.Println(s)
		}
	}

	func main() {
		go say("world") //開一個新的Goroutines執行
		say("hello") //當前Goroutines執行
	}

	// 以上程序執行後將輸出：
	// hello
	// world
	// hello
	// world
	// hello
	// world
	// hello
	// world
	// hello

我們可以看到go關鍵字很方便的就實現了併發編程。
上面的多個goroutine運行在同一個進程裡面，共享內存數據，不過設計上我們要遵循：不要通過共享來通信，而要通過通信來共享。

> runtime.Gosched()表示讓CPU把時間片讓給別人,下次某個時候繼續恢復執行該goroutine。

>默認情況下，調度器僅使用單線程，也就是說只實現了併發。想要發揮多核處理器的並行，需要在我們的程序中顯式調用 runtime.GOMAXPROCS(n) 告訴調度器同時使用多個線程。GOMAXPROCS 設置了同時運行邏輯代碼的系統線程的最大數量，並返回之前的設置。如果n < 1，不會改變當前設置。以後Go的新版本中調度得到改進後，這將被移除。這裡有一篇Rob介紹的關於併發和並行的文章：http://concur.rspace.googlecode.com/hg/talk/concur.html#landing-slide

## channels
goroutine運行在相同的地址空間，因此訪問共享內存必須做好同步。那麼goroutine之間如何進行數據的通信呢，Go提供了一個很好的通信機制channel。channel可以與Unix shell 中的雙向管道做類比：可以通過它發送或者接收值。這些值只能是特定的類型：channel類型。定義一個channel時，也需要定義發送到channel的值的類型。注意，必須使用make 創建channel：

	ci := make(chan int)
	cs := make(chan string)
	cf := make(chan interface{})

channel通過操作符`<-`來接收和發送數據

	ch <- v    // 發送v到channel ch.
	v := <-ch  // 從ch中接收數據，並賦值給v

我們把這些應用到我們的例子中來：

	package main

	import "fmt"

	func sum(a []int, c chan int) {
		total := 0
		for _, v := range a {
			total += v
		}
		c <- total  // send total to c
	}

	func main() {
		a := []int{7, 2, 8, -9, 4, 0}

		c := make(chan int)
		go sum(a[:len(a)/2], c)
		go sum(a[len(a)/2:], c)
		x, y := <-c, <-c  // receive from c

		fmt.Println(x, y, x + y)
	}

默認情況下，channel接收和發送數據都是阻塞的，除非另一端已經準備好，這樣就使得Goroutines同步變的更加的簡單，而不需要顯式的lock。所謂阻塞，也就是如果讀取（value := <-ch）它將會被阻塞，直到有數據接收。其次，任何發送（ch<-5）將會被阻塞，直到數據被讀出。無緩衝channel是在多個goroutine之間同步很棒的工具。

## Buffered Channels
上面我們介紹了默認的非緩存類型的channel，不過Go也允許指定channel的緩衝大小，很簡單，就是channel可以存儲多少元素。ch:= make(chan bool, 4)，創建了可以存儲4個元素的bool 型channel。在這個channel 中，前4個元素可以無阻塞的寫入。當寫入第5個元素時，代碼將會阻塞，直到其他goroutine從channel 中讀取一些元素，騰出空間。

	ch := make(chan type, value)

當 value = 0 時，channel 是無緩衝阻塞讀寫的，當value > 0 時，channel 有緩衝、是非阻塞的，直到寫滿 value 個元素才阻塞寫入。

我們看一下下面這個例子，你可以在自己本機測試一下，修改相應的value值


	package main

	import "fmt"

	func main() {
		c := make(chan int, 2)//修改2為1就報錯，修改2為3可以正常運行
		c <- 1
		c <- 2
		fmt.Println(<-c)
		fmt.Println(<-c)
	}
        //修改為1報如下的錯誤:
        //fatal error: all goroutines are asleep - deadlock!

## Range和Close
上面這個例子中，我們需要讀取兩次c，這樣不是很方便，Go考慮到了這一點，所以也可以通過range，像操作slice或者map一樣操作緩存類型的channel，請看下面的例子

	package main

	import (
		"fmt"
	)

	func fibonacci(n int, c chan int) {
		x, y := 1, 1
		for i := 0; i < n; i++ {
			c <- x
			x, y = y, x + y
		}
		close(c)
	}

	func main() {
		c := make(chan int, 10)
		go fibonacci(cap(c), c)
		for i := range c {
			fmt.Println(i)
		}
	}

`for i := range c`能夠不斷的讀取channel裡面的數據，直到該channel被顯式的關閉。上面代碼我們看到可以顯式的關閉channel，生產者通過內置函數`close`關閉channel。關閉channel之後就無法再發送任何數據了，在消費方可以通過語法`v, ok := <-ch`測試channel是否被關閉。如果ok返回false，那麼說明channel已經沒有任何數據並且已經被關閉。

>記住應該在生產者的地方關閉channel，而不是消費的地方去關閉它，這樣容易引起panic

>另外記住一點的就是channel不像文件之類的，不需要經常去關閉，只有當你確實沒有任何發送數據了，或者你想顯式的結束range循環之類的

## Select
我們上面介紹的都是隻有一個channel的情況，那麼如果存在多個channel的時候，我們該如何操作呢，Go裡面提供了一個關鍵字`select`，通過`select`可以監聽channel上的數據流動。

`select`默認是阻塞的，只有當監聽的channel中有發送或接收可以進行時才會運行，當多個channel都準備好的時候，select是隨機的選擇一個執行的。

	package main

	import "fmt"

	func fibonacci(c, quit chan int) {
		x, y := 1, 1
		for {
			select {
			case c <- x:
				x, y = y, x + y
			case <-quit:
				fmt.Println("quit")
				return
			}
		}
	}

	func main() {
		c := make(chan int)
		quit := make(chan int)
		go func() {
			for i := 0; i < 10; i++ {
				fmt.Println(<-c)
			}
			quit <- 0
		}()
		fibonacci(c, quit)
	}

在`select`裡面還有default語法，`select`其實就是類似switch的功能，default就是當監聽的channel都沒有準備好的時候，默認執行的（select不再阻塞等待channel）。

	select {
	case i := <-c:
		// use i
	default:
		// 當c阻塞的時候執行這裡
	}

## 超時
有時候會出現goroutine阻塞的情況，那麼我們如何避免整個程序進入阻塞的情況呢？我們可以利用select來設置超時，通過如下的方式實現：

	func main() {
		c := make(chan int)
		o := make(chan bool)
		go func() {
			for {
				select {
					case v := <- c:
						println(v)
					case <- time.After(5 * time.Second):
						println("timeout")
						o <- true
						break
				}
			}
		}()
		<- o
	}


## runtime goroutine
runtime包中有幾個處理goroutine的函數：

- Goexit

	退出當前執行的goroutine，但是defer函數還會繼續調用
	
- Gosched

	讓出當前goroutine的執行權限，調度器安排其他等待的任務運行，並在下次某個時候從該位置恢復執行。

- NumCPU

	返回 CPU 核數量
	
- NumGoroutine

	返回正在執行和排隊的任務總數
	
- GOMAXPROCS

	用來設置可以並行計算的CPU核數的最大值，並返回之前的值。
	
	

## links
   * [目錄](<preface.md>)
   * 上一章: [interface](<02.6.md>)
   * 下一節: [總結](<02.8.md>)

---

# 2.8 總結

這一章我們主要介紹了Go語言的一些語法，通過語法我們可以發現Go是多麼的簡單，只有二十五個關鍵字。讓我們再來回顧一下這些關鍵字都是用來幹什麼的。

	break    default      func    interface    select
	case     defer        go      map          struct
	chan     else         goto    package      switch
	const    fallthrough  if      range        type
	continue for          import  return       var

- var和const參考2.2Go語言基礎裡面的變量和常量申明
- package和import已經有過短暫的接觸
- func 用於定義函數和方法
- return 用於從函數返回
- defer 用於類似析構函數
- go 用於併發
- select 用於選擇不同類型的通訊
- interface 用於定義接口，參考2.6小節
- struct 用於定義抽象數據類型，參考2.5小節
- break、case、continue、for、fallthrough、else、if、switch、goto、default這些參考2.3流程介紹裡面
- chan用於channel通訊
- type用於聲明自定義類型
- map用於聲明map類型數據
- range用於讀取slice、map、channel數據

上面這二十五個關鍵字記住了，那麼Go你也已經差不多學會了。

## links
   * [目錄](<preface.md>)
   * 上一節: [併發](<02.7.md>)
   * 下一章: [Web基礎](<03.0.md>)
