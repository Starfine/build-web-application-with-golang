# 11 錯誤處理，調試和測試
我們經常會看到很多程序員大部分的"編程"時間都花費在檢查bug和修復bug上。無論你是在編寫修改代碼還是重構系統，幾乎都是花費大量的時間在進行故障排除和測試，外界都覺得我們程序員是設計師，能夠把一個系統從無做到有，是一項很偉大的工作，而且是相當有趣的工作，但事實上我們每天都是徘徊在排錯、調試、測試之間。當然如果你有良好的習慣和技術方案來直面這些問題，那麼你就有可能將排錯時間減到最少，而儘可能的將時間花費在更有價值的事情上。

但是遺憾的是很多程序員不願意在錯誤處理、調試和測試能力上下工夫，導致後面應用上線之後查找錯誤、定位問題花費更多的時間。所以我們在設計應用之前就做好錯誤處理規劃、測試用例等，那麼將來修改代碼、升級系統都將變得簡單。

開發Web應用過程中，錯誤自然難免，那麼如何更好的找到錯誤原因，解決問題呢？11.1小節將介紹Go語言中如何處理錯誤，如何設計自己的包、函數的錯誤處理，11.2小節將介紹如何使用GDB來調試我們的程序，動態運行情況下各種變量信息，運行情況的監控和調試。

11.3小節將對Go語言中的單元測試進行深入的探討，並示例如何來編寫單元測試，Go的單元測試規則規範如何定義，以保證以後升級修改運行相應的測試代碼就可以進行最小化的測試。

長期以來，培養良好的調試、測試習慣一直是很多程序員逃避的事情，所以現在你不要再逃避了，就從你現在的項目開發，從學習Go Web開發開始養成良好的習慣。

## 目錄
 
![](images/navi11.png?raw=true)

## links
   * [目錄](<preface.md>)
   * 上一章: [第十章總結](<10.4.md>)
   * 下一節: [錯誤處理](<11.1.md>)

---
 
# 11.1 錯誤處理
Go語言主要的設計準則是：簡潔、明白，簡潔是指語法和C類似，相當的簡單，明白是指任何語句都是很明顯的，不含有任何隱含的東西，在錯誤處理方案的設計中也貫徹了這一思想。我們知道在C語言裡面是通過返回-1或者NULL之類的信息來表示錯誤，但是對於使用者來說，不查看相應的API說明文檔，根本搞不清楚這個返回值究竟代表什麼意思，比如:返回0是成功，還是失敗,而Go定義了一個叫做error的類型，來顯式表達錯誤。在使用時，通過把返回的error變量與nil的比較，來判定操作是否成功。例如`os.Open`函數在打開文件失敗時將返回一個不為nil的error變量

	func Open(name string) (file *File, err error)

下面這個例子通過調用`os.Open`打開一個文件，如果出現錯誤，那麼就會調用`log.Fatal`來輸出錯誤信息：

	f, err := os.Open("filename.ext")
	if err != nil {
		log.Fatal(err)
	}

類似於`os.Open`函數，標準包中所有可能出錯的API都會返回一個error變量，以方便錯誤處理，這個小節將詳細地介紹error類型的設計，和討論開發Web應用中如何更好地處理error。
## Error類型
error類型是一個接口類型，這是它的定義：

	type error interface {
		Error() string
	}

error是一個內置的接口類型，我們可以在/builtin/包下面找到相應的定義。而我們在很多內部包裡面用到的 error是errors包下面的實現的私有結構errorString

	// errorString is a trivial implementation of error.
	type errorString struct {
		s string
	}

	func (e *errorString) Error() string {
		return e.s
	}
	
你可以通過`errors.New`把一個字符串轉化為errorString，以得到一個滿足接口error的對象，其內部實現如下：

	// New returns an error that formats as the given text.
	func New(text string) error {
		return &errorString{text}
	}

下面這個例子演示瞭如何使用`errors.New`:

	func Sqrt(f float64) (float64, error) {
		if f < 0 {
			return 0, errors.New("math: square root of negative number")
		}
		// implementation
	}
	
在下面的例子中，我們在調用Sqrt的時候傳遞的一個負數，然後就得到了non-nil的error對象，將此對象與nil比較，結果為true，所以fmt.Println(fmt包在處理error時會調用Error方法)被調用，以輸出錯誤，請看下面調用的示例代碼：

	f, err := Sqrt(-1)
    if err != nil {
        fmt.Println(err)
    }	

## 自定義Error
通過上面的介紹我們知道error是一個interface，所以在實現自己的包的時候，通過定義實現此接口的結構，我們就可以實現自己的錯誤定義，請看來自Json包的示例：

	type SyntaxError struct {
		msg    string // 錯誤描述
		Offset int64  // 錯誤發生的位置
	}

	func (e *SyntaxError) Error() string { return e.msg }

Offset字段在調用Error的時候不會被打印，但是我們可以通過類型斷言獲取錯誤類型，然後可以打印相應的錯誤信息，請看下面的例子:

	if err := dec.Decode(&val); err != nil {
		if serr, ok := err.(*json.SyntaxError); ok {
			line, col := findLine(f, serr.Offset)
			return fmt.Errorf("%s:%d:%d: %v", f.Name(), line, col, err)
		}
		return err
	}

需要注意的是，函數返回自定義錯誤時，返回值推薦設置為error類型，而非自定義錯誤類型，特別需要注意的是不應預聲明自定義錯誤類型的變量。例如：

	func Decode() *SyntaxError { // 錯誤，將可能導致上層調用者err!=nil的判斷永遠為true。
        var err *SyntaxError     // 預聲明錯誤變量
        if 出錯條件 {
            err = &SyntaxError{}
        }
        return err               // 錯誤，err永遠等於非nil，導致上層調用者err!=nil的判斷始終為true
    }
	
原因見 http://golang.org/doc/faq#nil_error

上面例子簡單的演示瞭如何自定義Error類型。但是如果我們還需要更復雜的錯誤處理呢？此時，我們來參考一下net包採用的方法：

	package net

	type Error interface {
	    error
	    Timeout() bool   // Is the error a timeout?
	    Temporary() bool // Is the error temporary?
	}

在調用的地方，通過類型斷言err是不是net.Error,來細化錯誤的處理，例如下面的例子，如果一個網絡發生臨時性錯誤，那麼將會sleep 1秒之後重試：

	if nerr, ok := err.(net.Error); ok && nerr.Temporary() {
		time.Sleep(1e9)
		continue
	}
	if err != nil {
		log.Fatal(err)
	}

## 錯誤處理
Go在錯誤處理上採用了與C類似的檢查返回值的方式，而不是其他多數主流語言採用的異常方式，這造成了代碼編寫上的一個很大的缺點:錯誤處理代碼的冗餘，對於這種情況是我們通過複用檢測函數來減少類似的代碼。

請看下面這個例子代碼：

	func init() {
		http.HandleFunc("/view", viewRecord)
	}

	func viewRecord(w http.ResponseWriter, r *http.Request) {
		c := appengine.NewContext(r)
		key := datastore.NewKey(c, "Record", r.FormValue("id"), 0, nil)
		record := new(Record)
		if err := datastore.Get(c, key, record); err != nil {
			http.Error(w, err.Error(), 500)
			return
		}
		if err := viewTemplate.Execute(w, record); err != nil {
			http.Error(w, err.Error(), 500)
		}
	}

上面的例子中獲取數據和模板展示調用時都有檢測錯誤，當有錯誤發生時，調用了統一的處理函數`http.Error`，返回給客戶端500錯誤碼，並顯示相應的錯誤數據。但是當越來越多的HandleFunc加入之後，這樣的錯誤處理邏輯代碼就會越來越多，其實我們可以通過自定義路由器來縮減代碼(實現的思路可以參考第三章的HTTP詳解)。

	type appHandler func(http.ResponseWriter, *http.Request) error

	func (fn appHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
		if err := fn(w, r); err != nil {
			http.Error(w, err.Error(), 500)
		}
	}

上面我們定義了自定義的路由器，然後我們可以通過如下方式來註冊函數：

	func init() {
		http.Handle("/view", appHandler(viewRecord))
	}

當請求/view的時候我們的邏輯處理可以變成如下代碼，和第一種實現方式相比較已經簡單了很多。

	func viewRecord(w http.ResponseWriter, r *http.Request) error {
		c := appengine.NewContext(r)
		key := datastore.NewKey(c, "Record", r.FormValue("id"), 0, nil)
		record := new(Record)
		if err := datastore.Get(c, key, record); err != nil {
			return err
		}
		return viewTemplate.Execute(w, record)
	}

上面的例子錯誤處理的時候所有的錯誤返回給用戶的都是500錯誤碼，然後打印出來相應的錯誤代碼，其實我們可以把這個錯誤信息定義的更加友好，調試的時候也方便定位問題，我們可以自定義返回的錯誤類型：

	type appError struct {
		Error   error
		Message string
		Code    int
	}

這樣我們的自定義路由器可以改成如下方式：

	type appHandler func(http.ResponseWriter, *http.Request) *appError

	func (fn appHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
		if e := fn(w, r); e != nil { // e is *appError, not os.Error.
			c := appengine.NewContext(r)
			c.Errorf("%v", e.Error)
			http.Error(w, e.Message, e.Code)
		}
	}

這樣修改完自定義錯誤之後，我們的邏輯處理可以改成如下方式：

	func viewRecord(w http.ResponseWriter, r *http.Request) *appError {
		c := appengine.NewContext(r)
		key := datastore.NewKey(c, "Record", r.FormValue("id"), 0, nil)
		record := new(Record)
		if err := datastore.Get(c, key, record); err != nil {
			return &appError{err, "Record not found", 404}
		}
		if err := viewTemplate.Execute(w, record); err != nil {
			return &appError{err, "Can't display record", 500}
		}
		return nil
	}

如上所示，在我們訪問view的時候可以根據不同的情況獲取不同的錯誤碼和錯誤信息，雖然這個和第一個版本的代碼量差不多，但是這個顯示的錯誤更加明顯，提示的錯誤信息更加友好，擴展性也比第一個更好。

## 總結
在程序設計中，容錯是相當重要的一部分工作，在Go中它是通過錯誤處理來實現的，error雖然只是一個接口，但是其變化卻可以有很多，我們可以根據自己的需求來實現不同的處理，最後介紹的錯誤處理方案，希望能給大家在如何設計更好Web錯誤處理方案上帶來一點思路。

## links
   * [目錄](<preface.md>)
   * 上一節: [錯誤處理，調試和測試](<11.0.md>)
   * 下一節: [使用GDB調試](<11.2.md>)

---

# 11.2 使用GDB調試
開發程序過程中調試代碼是開發者經常要做的一件事情，Go語言不像PHP、Python等動態語言，只要修改不需要編譯就可以直接輸出，而且可以動態的在運行環境下打印數據。當然Go語言也可以通過Println之類的打印數據來調試，但是每次都需要重新編譯，這是一件相當麻煩的事情。我們知道在Python中有pdb/ipdb之類的工具調試，Javascript也有類似工具，這些工具都能夠動態的顯示變量信息，單步調試等。不過慶幸的是Go也有類似的工具支持：GDB。Go內部已經內置支持了GDB，所以，我們可以通過GDB來進行調試，那麼本小節就來介紹一下如何通過GDB來調試Go程序。

## GDB調試簡介
GDB是FSF(自由軟件基金會)發佈的一個強大的類UNIX系統下的程序調試工具。使用GDB可以做如下事情：

1. 啟動程序，可以按照開發者的自定義要求運行程序。
2. 可讓被調試的程序在開發者設定的調置的斷點處停住。（斷點可以是條件表達式）
3. 當程序被停住時，可以檢查此時程序中所發生的事。
4. 動態的改變當前程序的執行環境。

目前支持調試Go程序的GDB版本必須大於7.1。

編譯Go程序的時候需要注意以下幾點

1. 傳遞參數-ldflags "-s"，忽略debug的打印信息
2. 傳遞-gcflags "-N -l" 參數，這樣可以忽略Go內部做的一些優化，聚合變量和函數等優化，這樣對於GDB調試來說非常困難，所以在編譯的時候加入這兩個參數避免這些優化。 

## 常用命令
GDB的一些常用命令如下所示

- list

	簡寫命令`l`，用來顯示源代碼，默認顯示十行代碼，後面可以帶上參數顯示的具體行，例如：`list 15`，顯示十行代碼，其中第15行在顯示的十行裡面的中間，如下所示。

		10	        time.Sleep(2 * time.Second)
		11	        c <- i
		12	    }
		13	    close(c)
		14	}
		15	
		16	func main() {
		17	    msg := "Starting main"
		18	    fmt.Println(msg)
		19	    bus := make(chan int)

	
- break

	簡寫命令 `b`,用來設置斷點，後面跟上參數設置斷點的行數，例如`b 10`在第十行設置斷點。
	
- delete
	簡寫命令 `d`,用來刪除斷點，後面跟上斷點設置的序號，這個序號可以通過`info breakpoints`獲取相應的設置的斷點序號，如下是顯示的設置斷點序號。

		Num     Type           Disp Enb Address            What
		2       breakpoint     keep y   0x0000000000400dc3 in main.main at /home/xiemengjun/gdb.go:23
		breakpoint already hit 1 time

- backtrace
	
	簡寫命令 `bt`,用來打印執行的代碼過程，如下所示：

		#0  main.main () at /home/xiemengjun/gdb.go:23
		#1  0x000000000040d61e in runtime.main () at /home/xiemengjun/go/src/pkg/runtime/proc.c:244
		#2  0x000000000040d6c1 in schedunlock () at /home/xiemengjun/go/src/pkg/runtime/proc.c:267
		#3  0x0000000000000000 in ?? ()
- info

	info命令用來顯示信息，後面有幾種參數，我們常用的有如下幾種：
		
	- `info locals`

		顯示當前執行的程序中的變量值
	- `info breakpoints`

		顯示當前設置的斷點列表
	- `info goroutines`

		顯示當前執行的goroutine列表，如下代碼所示,帶*的表示當前執行的

			* 1  running runtime.gosched
			* 2  syscall runtime.entersyscall
			  3  waiting runtime.gosched
			  4 runnable runtime.gosched
- print

	簡寫命令`p`，用來打印變量或者其他信息，後面跟上需要打印的變量名，當然還有一些很有用的函數$len()和$cap()，用來返回當前string、slices或者maps的長度和容量。

- whatis 
	
	用來顯示當前變量的類型，後面跟上變量名，例如`whatis msg`,顯示如下：

		type = struct string
- next

	簡寫命令 `n`,用來單步調試，跳到下一步，當有斷點之後，可以輸入`n`跳轉到下一步繼續執行
- coutinue

	簡稱命令 `c`，用來跳出當前斷點處，後面可以跟參數N，跳過多少次斷點

- set variable

	該命令用來改變運行過程中的變量值，格式如：`set variable <var>=<value>`

## 調試過程
我們通過下面這個代碼來演示如何通過GDB來調試Go程序，下面是將要演示的代碼：

	package main

	import (
		"fmt"
		"time"
	)

	func counting(c chan<- int) {
		for i := 0; i < 10; i++ {
			time.Sleep(2 * time.Second)
			c <- i
		}
		close(c)
	}

	func main() {
		msg := "Starting main"
		fmt.Println(msg)
		bus := make(chan int)
		msg = "starting a gofunc"
		go counting(bus)
		for count := range bus {
			fmt.Println("count:", count)
		}
	}

編譯文件，生成可執行文件gdbfile:

	go build -gcflags "-N -l" gdbfile.go

通過gdb命令啟動調試：

	gdb gdbfile
	
啟動之後首先看看這個程序是不是可以運行起來，只要輸入`run`命令回車後程序就開始運行，程序正常的話可以看到程序輸出如下，和我們在命令行直接執行程序輸出是一樣的：

	(gdb) run
	Starting program: /home/xiemengjun/gdbfile 
	Starting main
	count: 0
	count: 1
	count: 2
	count: 3
	count: 4
	count: 5
	count: 6
	count: 7
	count: 8
	count: 9
	[LWP 2771 exited]
	[Inferior 1 (process 2771) exited normally]	
好了，現在我們已經知道怎麼讓程序跑起來了，接下來開始給代碼設置斷點：

	(gdb) b 23
	Breakpoint 1 at 0x400d8d: file /home/xiemengjun/gdbfile.go, line 23.
	(gdb) run
	Starting program: /home/xiemengjun/gdbfile 
	Starting main
	[New LWP 3284]
	[Switching to LWP 3284]

	Breakpoint 1, main.main () at /home/xiemengjun/gdbfile.go:23
	23	        fmt.Println("count:", count)

上面例子`b 23`表示在第23行設置了斷點，之後輸入`run`開始運行程序。現在程序在前面設置斷點的地方停住了，我們需要查看斷點相應上下文的源碼，輸入`list`就可以看到源碼顯示從當前停止行的前五行開始：

	(gdb) list
	18	    fmt.Println(msg)
	19	    bus := make(chan int)
	20	    msg = "starting a gofunc"
	21	    go counting(bus)
	22	    for count := range bus {
	23	        fmt.Println("count:", count)
	24	    }
	25	}

現在GDB在運行當前的程序的環境中已經保留了一些有用的調試信息，我們只需打印出相應的變量，查看相應變量的類型及值：

	(gdb) info locals
	count = 0
	bus = 0xf840001a50
	(gdb) p count
	$1 = 0
	(gdb) p bus
	$2 = (chan int) 0xf840001a50
	(gdb) whatis bus
	type = chan int

接下來該讓程序繼續往下執行，請繼續看下面的命令

	(gdb) c
	Continuing.
	count: 0
	[New LWP 3303]
	[Switching to LWP 3303]

	Breakpoint 1, main.main () at /home/xiemengjun/gdbfile.go:23
	23 fmt.Println("count:", count)
	(gdb) c
	Continuing.
	count: 1
	[Switching to LWP 3302]

	Breakpoint 1, main.main () at /home/xiemengjun/gdbfile.go:23
	23 fmt.Println("count:", count)

每次輸入`c`之後都會執行一次代碼，又跳到下一次for循環，繼續打印出來相應的信息。

設想目前需要改變上下文相關變量的信息，跳過一些過程，並繼續執行下一步，得出修改後想要的結果：

	(gdb) info locals
	count = 2
	bus = 0xf840001a50
	(gdb) set variable count=9
	(gdb) info locals
	count = 9
	bus = 0xf840001a50
	(gdb) c
	Continuing.
	count: 9
	[Switching to LWP 3302]

	Breakpoint 1, main.main () at /home/xiemengjun/gdbfile.go:23
	23 fmt.Println("count:", count)		
	
最後稍微思考一下，前面整個程序運行的過程中到底創建了多少個goroutine，每個goroutine都在做什麼：

	(gdb) info goroutines
	* 1 running runtime.gosched
	* 2 syscall runtime.entersyscall 
	3 waiting runtime.gosched 
	4 runnable runtime.gosched
	(gdb) goroutine 1 bt
	#0 0x000000000040e33b in runtime.gosched () at /home/xiemengjun/go/src/pkg/runtime/proc.c:927
	#1 0x0000000000403091 in runtime.chanrecv (c=void, ep=void, selected=void, received=void)
	at /home/xiemengjun/go/src/pkg/runtime/chan.c:327
	#2 0x000000000040316f in runtime.chanrecv2 (t=void, c=void)
	at /home/xiemengjun/go/src/pkg/runtime/chan.c:420
	#3 0x0000000000400d6f in main.main () at /home/xiemengjun/gdbfile.go:22
	#4 0x000000000040d0c7 in runtime.main () at /home/xiemengjun/go/src/pkg/runtime/proc.c:244
	#5 0x000000000040d16a in schedunlock () at /home/xiemengjun/go/src/pkg/runtime/proc.c:267
	#6 0x0000000000000000 in ?? ()

通過查看goroutines的命令我們可以清楚地瞭解goruntine內部是怎麼執行的，每個函數的調用順序已經明明白白地顯示出來了。

## 小結
本小節我們介紹了GDB調試Go程序的一些基本命令，包括`run`、`print`、`info`、`set variable`、`coutinue`、`list`、`break`	等經常用到的調試命令，通過上面的例子演示，我相信讀者已經對於通過GDB調試Go程序有了基本的理解，如果你想獲取更多的調試技巧請參考官方網站的GDB調試手冊，還有GDB官方網站的手冊。	
	
## links
   * [目錄](<preface.md>)
   * 上一節: [錯誤處理](<11.1.md>)
   * 下一節: [Go怎麼寫測試用例](<11.3.md>)

---

# 11.3 Go怎麼寫測試用例
開發程序其中很重要的一點是測試，我們如何保證代碼的質量，如何保證每個函數是可運行，運行結果是正確的，又如何保證寫出來的代碼性能是好的，我們知道單元測試的重點在於發現程序設計或實現的邏輯錯誤，使問題及早暴露，便於問題的定位解決，而性能測試的重點在於發現程序設計上的一些問題，讓線上的程序能夠在高併發的情況下還能保持穩定。本小節將帶著這一連串的問題來講解Go語言中如何來實現單元測試和性能測試。

Go語言中自帶有一個輕量級的測試框架`testing`和自帶的`go test`命令來實現單元測試和性能測試，`testing`框架和其他語言中的測試框架類似，你可以基於這個框架寫針對相應函數的測試用例，也可以基於該框架寫相應的壓力測試用例，那麼接下來讓我們一一來看一下怎麼寫。

## 如何編寫測試用例
由於`go test`命令只能在一個相應的目錄下執行所有文件，所以我們接下來新建一個項目目錄`gotest`,這樣我們所有的代碼和測試代碼都在這個目錄下。

接下來我們在該目錄下面創建兩個文件：gotest.go和gotest_test.go

1. gotest.go:這個文件裡面我們是創建了一個包，裡面有一個函數實現了除法運算:

		package gotest
		
		import (
			"errors"
		)
		
		func Division(a, b float64) (float64, error) {
			if b == 0 {
				return 0, errors.New("除數不能為0")
			}
		
			return a / b, nil
		}

2. gotest_test.go:這是我們的單元測試文件，但是記住下面的這些原則：

	- 文件名必須是`_test.go`結尾的，這樣在執行`go test`的時候才會執行到相應的代碼
	- 你必須import `testing`這個包
	- 所有的測試用例函數必須是`Test`開頭
	- 測試用例會按照源代碼中寫的順序依次執行
	- 測試函數`TestXxx()`的參數是`testing.T`，我們可以使用該類型來記錄錯誤或者是測試狀態
	- 測試格式：`func TestXxx (t *testing.T)`,`Xxx`部分可以為任意的字母數字的組合，但是首字母不能是小寫字母[a-z]，例如`Testintdiv`是錯誤的函數名。
	- 函數中通過調用`testing.T`的`Error`, `Errorf`, `FailNow`, `Fatal`, `FatalIf`方法，說明測試不通過，調用`Log`方法用來記錄測試的信息。
	
	下面是我們的測試用例的代碼：
	
		package gotest
		
		import (
			"testing"
		)
		
		func Test_Division_1(t *testing.T) {
			if i, e := Division(6, 2); i != 3 || e != nil { //try a unit test on function
				t.Error("除法函數測試沒通過") // 如果不是如預期的那麼就報錯
			} else {
				t.Log("第一個測試通過了") //記錄一些你期望記錄的信息
			}
		}
		
		func Test_Division_2(t *testing.T) {
			t.Error("就是不通過")
		}

	我們在項目目錄下面執行`go test`,就會顯示如下信息：

		--- FAIL: Test_Division_2 (0.00 seconds)
			gotest_test.go:16: 就是不通過
		FAIL
		exit status 1
		FAIL	gotest	0.013s
	從這個結果顯示測試沒有通過，因為在第二個測試函數中我們寫死了測試不通過的代碼`t.Error`，那麼我們的第一個函數執行的情況怎麼樣呢？默認情況下執行`go test`是不會顯示測試通過的信息的，我們需要帶上參數`go test -v`，這樣就會顯示如下信息：
	
		=== RUN Test_Division_1
		--- PASS: Test_Division_1 (0.00 seconds)
			gotest_test.go:11: 第一個測試通過了
		=== RUN Test_Division_2
		--- FAIL: Test_Division_2 (0.00 seconds)
			gotest_test.go:16: 就是不通過
		FAIL
		exit status 1
		FAIL	gotest	0.012s
	上面的輸出詳細的展示了這個測試的過程，我們看到測試函數1`Test_Division_1`測試通過，而測試函數2`Test_Division_2`測試失敗了，最後得出結論測試不通過。接下來我們把測試函數2修改成如下代碼：
	
		func Test_Division_2(t *testing.T) {
			if _, e := Division(6, 0); e == nil { //try a unit test on function
				t.Error("Division did not work as expected.") // 如果不是如預期的那麼就報錯
			} else {
				t.Log("one test passed.", e) //記錄一些你期望記錄的信息
			}
		}	
	然後我們執行`go test -v`，就顯示如下信息，測試通過了：
	
		=== RUN Test_Division_1
		--- PASS: Test_Division_1 (0.00 seconds)
			gotest_test.go:11: 第一個測試通過了
		=== RUN Test_Division_2
		--- PASS: Test_Division_2 (0.00 seconds)
			gotest_test.go:20: one test passed. 除數不能為0
		PASS
		ok  	gotest	0.013s

## 如何編寫壓力測試
壓力測試用來檢測函數(方法）的性能，和編寫單元功能測試的方法類似,此處不再贅述，但需要注意以下幾點：

- 壓力測試用例必須遵循如下格式，其中XXX可以是任意字母數字的組合，但是首字母不能是小寫字母

		func BenchmarkXXX(b *testing.B) { ... }
		
- `go test`不會默認執行壓力測試的函數，如果要執行壓力測試需要帶上參數`-test.bench`，語法:`-test.bench="test_name_regex"`,例如`go test -test.bench=".*"`表示測試全部的壓力測試函數
- 在壓力測試用例中,請記得在循環體內使用`testing.B.N`,以使測試可以正常的運行
- 文件名也必須以`_test.go`結尾

下面我們新建一個壓力測試文件webbench_test.go，代碼如下所示：

	package gotest
	
	import (
		"testing"
	)
	
	func Benchmark_Division(b *testing.B) {
		for i := 0; i < b.N; i++ { //use b.N for looping 
			Division(4, 5)
		}
	}
	
	func Benchmark_TimeConsumingFunction(b *testing.B) {
		b.StopTimer() //調用該函數停止壓力測試的時間計數
	
		//做一些初始化的工作,例如讀取文件數據,數據庫連接之類的,
		//這樣這些時間不影響我們測試函數本身的性能
	
		b.StartTimer() //重新開始時間
		for i := 0; i < b.N; i++ {
			Division(4, 5)
		}
	}


我們執行命令`go test -file webbench_test.go -test.bench=".*"`，可以看到如下結果：

	PASS
	Benchmark_Division	500000000	         7.76 ns/op
	Benchmark_TimeConsumingFunction	500000000	         7.80 ns/op
	ok  	gotest	9.364s	

上面的結果顯示我們沒有執行任何`TestXXX`的單元測試函數，顯示的結果只執行了壓力測試函數，第一條顯示了`Benchmark_Division`執行了500000000次，每次的執行平均時間是7.76納秒，第二條顯示了`Benchmark_TimeConsumingFunction`執行了500000000，每次的平均執行時間是7.80納秒。最後一條顯示總共的執行時間。

## 小結
通過上面對單元測試和壓力測試的學習，我們可以看到`testing`包很輕量，編寫單元測試和壓力測試用例非常簡單，配合內置的`go test`命令就可以非常方便的進行測試，這樣在我們每次修改完代碼,執行一下go test就可以簡單的完成迴歸測試了。


## links
   * [目錄](<preface.md>)
   * 上一節: [使用GDB調試](<11.2.md>)
   * 下一節: [小結](<11.4.md>)

---

# 11.4 小結
本章我們通過三個小節分別介紹了Go語言中如何處理錯誤，如何設計錯誤處理，然後第二小節介紹瞭如何通過GDB來調試程序，通過GDB我們可以單步調試、可以查看變量、修改變量、打印執行過程等，最後我們介紹瞭如何利用Go語言自帶的輕量級框架`testing`來編寫單元測試和壓力測試，使用`go test`就可以方便的執行這些測試，使得我們將來代碼升級修改之後很方便的進行迴歸測試。這一章也許對於你編寫程序邏輯沒有任何幫助，但是對於你編寫出來的程序代碼保持高質量是至關重要的，因為一個好的Web應用必定有良好的錯誤處理機制(錯誤提示的友好、可擴展性)、有好的單元測試和壓力測試以保證上線之後代碼能夠保持良好的性能和按預期的運行。

## links
   * [目錄](<preface.md>)
   * 上一節: [Go怎麼寫測試用例](<11.3.md>)
   * 下一節: [部署與維護](<12.0.md>)