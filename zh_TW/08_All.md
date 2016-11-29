# 8 Web服務
Web服務可以讓你在HTTP協議的基礎上通過XML或者JSON來交換信息。如果你想知道上海的天氣預報、中國石油的股價或者淘寶商家的一個商品信息，你可以編寫一段簡短的代碼，通過抓取這些信息然後通過標準的接口開放出來，就如同你調用一個本地函數並返回一個值。

Web服務背後的關鍵在於平臺的無關性，你可以運行你的服務在Linux系統，可以與其他Windows的asp.net程序交互，同樣的，也可以通過同一個接口和運行在FreeBSD上面的JSP無障礙地通信。

目前主流的有如下幾種Web服務：REST、SOAP。

REST請求是很直觀的，因為REST是基於HTTP協議的一個補充，他的每一次請求都是一個HTTP請求，然後根據不同的method來處理不同的邏輯，很多Web開發者都熟悉HTTP協議，所以學習REST是一件比較容易的事情。所以我們在8.3小節講詳細的講解如何在Go語言中來實現REST方式。

SOAP是W3C在跨網絡信息傳遞和遠程計算機函數調用方面的一個標準。但是SOAP非常複雜，其完整的規範篇幅很長，而且內容仍然在增加。Go語言是以簡單著稱，所以我們不會介紹SOAP這樣複雜的東西。而Go語言提供了一種天生性能很不錯，開發起來很方便的RPC機制，我們將會在8.4小節詳細介紹如何使用Go語言來實現RPC。

Go語言是21世紀的C語言，我們追求的是性能、簡單，所以我們在8.1小節裡面介紹如何使用Socket編程，很多遊戲服務都是採用Socket來編寫服務端，因為HTTP協議相對而言比較耗費性能，讓我們看看Go語言如何來Socket編程。目前隨著HTML5的發展，webSockets也逐漸的成為很多頁遊公司接下來開發的一些手段，我們將在8.2小節裡面講解Go語言如何編寫webSockets的代碼。

## 目錄
   ![](images/navi8.png?raw=true)

## links
   * [目錄](<preface.md>)
   * 上一章: [第七章總結](<07.7.md>)
   * 下一節: [Socket編程](<08.1.md>)

---

# 8.1 Socket編程
在很多底層網絡應用開發者的眼裡一切編程都是Socket，話雖然有點誇張，但卻也幾乎如此了，現在的網絡編程幾乎都是用Socket來編程。你想過這些情景麼？我們每天打開瀏覽器瀏覽網頁時，瀏覽器進程怎麼和Web服務器進行通信的呢？當你用QQ聊天時，QQ進程怎麼和服務器或者是你的好友所在的QQ進程進行通信的呢？當你打開PPstream觀看視頻時，PPstream進程如何與視頻服務器進行通信的呢？ 如此種種，都是靠Socket來進行通信的，以一斑窺全豹，可見Socket編程在現代編程中佔據了多麼重要的地位，這一節我們將介紹Go語言中如何進行Socket編程。

## 什麼是Socket？
Socket起源於Unix，而Unix基本哲學之一就是“一切皆文件”，都可以用“打開open –> 讀寫write/read –> 關閉close”模式來操作。Socket就是該模式的一個實現，網絡的Socket數據傳輸是一種特殊的I/O，Socket也是一種文件描述符。Socket也具有一個類似於打開文件的函數調用：Socket()，該函數返回一個整型的Socket描述符，隨後的連接建立、數據傳輸等操作都是通過該Socket實現的。

常用的Socket類型有兩種：流式Socket（SOCK_STREAM）和數據報式Socket（SOCK_DGRAM）。流式是一種面向連接的Socket，針對於面向連接的TCP服務應用；數據報式Socket是一種無連接的Socket，對應於無連接的UDP服務應用。
## Socket如何通信
網絡中的進程之間如何通過Socket通信呢？首要解決的問題是如何唯一標識一個進程，否則通信無從談起！在本地可以通過進程PID來唯一標識一個進程，但是在網絡中這是行不通的。其實TCP/IP協議族已經幫我們解決了這個問題，網絡層的“ip地址”可以唯一標識網絡中的主機，而傳輸層的“協議+端口”可以唯一標識主機中的應用程序（進程）。這樣利用三元組（ip地址，協議，端口）就可以標識網絡的進程了，網絡中需要互相通信的進程，就可以利用這個標誌在他們之間進行交互。請看下面這個TCP/IP協議結構圖

![](images/8.1.socket.png?raw=true)

圖8.1 七層網絡協議圖

使用TCP/IP協議的應用程序通常採用應用編程接口：UNIX BSD的套接字（socket）和UNIX System V的TLI（已經被淘汰），來實現網絡進程之間的通信。就目前而言，幾乎所有的應用程序都是採用socket，而現在又是網絡時代，網絡中進程通信是無處不在，這就是為什麼說“一切皆Socket”。

## Socket基礎知識
通過上面的介紹我們知道Socket有兩種：TCP Socket和UDP Socket，TCP和UDP是協議，而要確定一個進程的需要三元組，需要IP地址和端口。

### IPv4地址
目前的全球因特網所採用的協議族是TCP/IP協議。IP是TCP/IP協議中網絡層的協議，是TCP/IP協議族的核心協議。目前主要採用的IP協議的版本號是4(簡稱為IPv4)，發展至今已經使用了30多年。

IPv4的地址位數為32位，也就是最多有2的32次方的網絡設備可以聯到Internet上。近十年來由於互聯網的蓬勃發展，IP位址的需求量愈來愈大，使得IP位址的發放愈趨緊張，前一段時間，據報道IPV4的地址已經發放完畢，我們公司目前很多服務器的IP都是一個寶貴的資源。

地址格式類似這樣：127.0.0.1 172.122.121.111

### IPv6地址
IPv6是下一版本的互聯網協議，也可以說是下一代互聯網的協議，它是為了解決IPv4在實施過程中遇到的各種問題而被提出的，IPv6採用128位地址長度，幾乎可以不受限制地提供地址。按保守方法估算IPv6實際可分配的地址，整個地球的每平方米麵積上仍可分配1000多個地址。在IPv6的設計過程中除了一勞永逸地解決了地址短缺問題以外，還考慮了在IPv4中解決不好的其它問題，主要有端到端IP連接、服務質量（QoS）、安全性、多播、移動性、即插即用等。

地址格式類似這樣：2002:c0e8:82e7:0:0:0:c0e8:82e7

### Go支持的IP類型
在Go的`net`包中定義了很多類型、函數和方法用來網絡編程，其中IP的定義如下：

	type IP []byte

在`net`包中有很多函數來操作IP，但是其中比較有用的也就幾個，其中`ParseIP(s string) IP`函數會把一個IPv4或者IPv6的地址轉化成IP類型，請看下面的例子:

	package main
	import (
		"net"
		"os"
		"fmt"
	)
	func main() {
		if len(os.Args) != 2 {
			fmt.Fprintf(os.Stderr, "Usage: %s ip-addr\n", os.Args[0])
			os.Exit(1)
		}
		name := os.Args[1]
		addr := net.ParseIP(name)
		if addr == nil {
			fmt.Println("Invalid address")
		} else {
			fmt.Println("The address is ", addr.String())
		}
		os.Exit(0)
	}

執行之後你就會發現只要你輸入一個IP地址就會給出相應的IP格式

## TCP Socket
當我們知道如何通過網絡端口訪問一個服務時，那麼我們能夠做什麼呢？作為客戶端來說，我們可以通過向遠端某臺機器的的某個網絡端口發送一個請求，然後得到在機器的此端口上監聽的服務反饋的信息。作為服務端，我們需要把服務綁定到某個指定端口，並且在此端口上監聽，當有客戶端來訪問時能夠讀取信息並且寫入反饋信息。

在Go語言的`net`包中有一個類型`TCPConn`，這個類型可以用來作為客戶端和服務器端交互的通道，他有兩個主要的函數：

	func (c *TCPConn) Write(b []byte) (n int, err os.Error)
	func (c *TCPConn) Read(b []byte) (n int, err os.Error)

`TCPConn`可以用在客戶端和服務器端來讀寫數據。

還有我們需要知道一個`TCPAddr`類型，他表示一個TCP的地址信息，他的定義如下：

	type TCPAddr struct {
		IP IP
		Port int
	}
在Go語言中通過`ResolveTCPAddr`獲取一個`TCPAddr`

	func ResolveTCPAddr(net, addr string) (*TCPAddr, os.Error)

- net參數是"tcp4"、"tcp6"、"tcp"中的任意一個，分別表示TCP(IPv4-only),TCP(IPv6-only)或者TCP(IPv4,IPv6的任意一個).
- addr表示域名或者IP地址，例如"www.google.com:80" 或者"127.0.0.1:22".


### TCP client
Go語言中通過net包中的`DialTCP`函數來建立一個TCP連接，並返回一個`TCPConn`類型的對象，當連接建立時服務器端也創建一個同類型的對象，此時客戶端和服務器段通過各自擁有的`TCPConn`對象來進行數據交換。一般而言，客戶端通過`TCPConn`對象將請求信息發送到服務器端，讀取服務器端響應的信息。服務器端讀取並解析來自客戶端的請求，並返回應答信息，這個連接只有當任一端關閉了連接之後才失效，不然這連接可以一直在使用。建立連接的函數定義如下：

	func DialTCP(net string, laddr, raddr *TCPAddr) (c *TCPConn, err os.Error)

- net參數是"tcp4"、"tcp6"、"tcp"中的任意一個，分別表示TCP(IPv4-only)、TCP(IPv6-only)或者TCP(IPv4,IPv6的任意一個)
- laddr表示本機地址，一般設置為nil
- raddr表示遠程的服務地址

接下來我們寫一個簡單的例子，模擬一個基於HTTP協議的客戶端請求去連接一個Web服務端。我們要寫一個簡單的http請求頭，格式類似如下：

	"HEAD / HTTP/1.0\r\n\r\n"

從服務端接收到的響應信息格式可能如下：

	HTTP/1.0 200 OK
	ETag: "-9985996"
	Last-Modified: Thu, 25 Mar 2010 17:51:10 GMT
	Content-Length: 18074
	Connection: close
	Date: Sat, 28 Aug 2010 00:43:48 GMT
	Server: lighttpd/1.4.23

我們的客戶端代碼如下所示：

	package main

	import (
		"fmt"
		"io/ioutil"
		"net"
		"os"
	)

	func main() {
		if len(os.Args) != 2 {
			fmt.Fprintf(os.Stderr, "Usage: %s host:port ", os.Args[0])
			os.Exit(1)
		}
		service := os.Args[1]
		tcpAddr, err := net.ResolveTCPAddr("tcp4", service)
		checkError(err)
		conn, err := net.DialTCP("tcp", nil, tcpAddr)
		checkError(err)
		_, err = conn.Write([]byte("HEAD / HTTP/1.0\r\n\r\n"))
		checkError(err)
		result, err := ioutil.ReadAll(conn)
		checkError(err)
		fmt.Println(string(result))
		os.Exit(0)
	}
	func checkError(err error) {
		if err != nil {
			fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
			os.Exit(1)
		}
	}

通過上面的代碼我們可以看出：首先程序將用戶的輸入作為參數`service`傳入`net.ResolveTCPAddr`獲取一個tcpAddr,然後把tcpAddr傳入DialTCP後創建了一個TCP連接`conn`，通過`conn`來發送請求信息，最後通過`ioutil.ReadAll`從`conn`中讀取全部的文本，也就是服務端響應反饋的信息。

### TCP server
上面我們編寫了一個TCP的客戶端程序，也可以通過net包來創建一個服務器端程序，在服務器端我們需要綁定服務到指定的非激活端口，並監聽此端口，當有客戶端請求到達的時候可以接收到來自客戶端連接的請求。net包中有相應功能的函數，函數定義如下：

	func ListenTCP(net string, laddr *TCPAddr) (l *TCPListener, err os.Error)
	func (l *TCPListener) Accept() (c Conn, err os.Error)

參數說明同DialTCP的參數一樣。下面我們實現一個簡單的時間同步服務，監聽7777端口

	package main

	import (
		"fmt"
		"net"
		"os"
		"time"
	)

	func main() {
		service := ":7777"
		tcpAddr, err := net.ResolveTCPAddr("tcp4", service)
		checkError(err)
		listener, err := net.ListenTCP("tcp", tcpAddr)
		checkError(err)
		for {
			conn, err := listener.Accept()
			if err != nil {
				continue
			}
			daytime := time.Now().String()
			conn.Write([]byte(daytime)) // don't care about return value
			conn.Close()                // we're finished with this client
		}
	}
	func checkError(err error) {
		if err != nil {
			fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
			os.Exit(1)
		}
	}

上面的服務跑起來之後，它將會一直在那裡等待，直到有新的客戶端請求到達。當有新的客戶端請求到達並同意接受`Accept`該請求的時候他會反饋當前的時間信息。值得注意的是，在代碼中`for`循環裡，當有錯誤發生時，直接continue而不是退出，是因為在服務器端跑代碼的時候，當有錯誤發生的情況下最好是由服務端記錄錯誤，然後當前連接的客戶端直接報錯而退出，從而不會影響到當前服務端運行的整個服務。

上面的代碼有個缺點，執行的時候是單任務的，不能同時接收多個請求，那麼該如何改造以使它支持多併發呢？Go裡面有一個goroutine機制，請看下面改造後的代碼

	package main

	import (
		"fmt"
		"net"
		"os"
		"time"
	)

	func main() {
		service := ":1200"
		tcpAddr, err := net.ResolveTCPAddr("tcp4", service)
		checkError(err)
		listener, err := net.ListenTCP("tcp", tcpAddr)
		checkError(err)
		for {
			conn, err := listener.Accept()
			if err != nil {
				continue
			}
			go handleClient(conn)
		}
	}

	func handleClient(conn net.Conn) {
		defer conn.Close()
		daytime := time.Now().String()
		conn.Write([]byte(daytime)) // don't care about return value
		// we're finished with this client
	}
	func checkError(err error) {
		if err != nil {
			fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
			os.Exit(1)
		}
	}

通過把業務處理分離到函數`handleClient`，我們就可以進一步地實現多併發執行了。看上去是不是很帥，增加`go`關鍵詞就實現了服務端的多併發，從這個小例子也可以看出goroutine的強大之處。

有的朋友可能要問：這個服務端沒有處理客戶端實際請求的內容。如果我們需要通過從客戶端發送不同的請求來獲取不同的時間格式，而且需要一個長連接，該怎麼做呢？請看：

	package main

	import (
		"fmt"
		"net"
		"os"
		"time"
		"strconv"
		"strings"
	)

	func main() {
		service := ":1200"
		tcpAddr, err := net.ResolveTCPAddr("tcp4", service)
		checkError(err)
		listener, err := net.ListenTCP("tcp", tcpAddr)
		checkError(err)
		for {
			conn, err := listener.Accept()
			if err != nil {
				continue
			}
			go handleClient(conn)
		}
	}

	func handleClient(conn net.Conn) {
		conn.SetReadDeadline(time.Now().Add(2 * time.Minute)) // set 2 minutes timeout
		request := make([]byte, 128) // set maxium request length to 128B to prevent flood attack
		defer conn.Close()  // close connection before exit
		for {
			read_len, err := conn.Read(request)

			if err != nil {
				fmt.Println(err)
				break
			}

    		if read_len == 0 {
    			break // connection already closed by client
    		} else if strings.TrimSpace(string(request[:read_len])) == "timestamp" {
    			daytime := strconv.FormatInt(time.Now().Unix(), 10)
    			conn.Write([]byte(daytime))
    		} else {
    			daytime := time.Now().String()
    			conn.Write([]byte(daytime))
    		}

    		request = make([]byte, 128) // clear last read content
		}
	}

	func checkError(err error) {
		if err != nil {
			fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
			os.Exit(1)
		}
	}

在上面這個例子中，我們使用`conn.Read()`不斷讀取客戶端發來的請求。由於我們需要保持與客戶端的長連接，所以不能在讀取完一次請求後就關閉連接。由於`conn.SetReadDeadline()`設置了超時，當一定時間內客戶端無請求發送，`conn`便會自動關閉，下面的for循環即會因為連接已關閉而跳出。需要注意的是，`request`在創建時需要指定一個最大長度以防止flood attack；每次讀取到請求處理完畢後，需要清理request，因為`conn.Read()`會將新讀取到的內容append到原內容之後。

### 控制TCP連接
TCP有很多連接控制函數，我們平常用到比較多的有如下幾個函數：

	func DialTimeout(net, addr string, timeout time.Duration) (Conn, error)

設置建立連接的超時時間，客戶端和服務器端都適用，當超過設置時間時，連接自動關閉。

	func (c *TCPConn) SetReadDeadline(t time.Time) error
	func (c *TCPConn) SetWriteDeadline(t time.Time) error

用來設置寫入/讀取一個連接的超時時間。當超過設置時間時，連接自動關閉。

	func (c *TCPConn) SetKeepAlive(keepalive bool) os.Error

設置客戶端是否和服務器端保持長連接，可以降低建立TCP連接時的握手開銷，對於一些需要頻繁交換數據的應用場景比較適用。

更多的內容請查看`net`包的文檔。
## UDP Socket
Go語言包中處理UDP Socket和TCP Socket不同的地方就是在服務器端處理多個客戶端請求數據包的方式不同,UDP缺少了對客戶端連接請求的Accept函數。其他基本幾乎一模一樣，只有TCP換成了UDP而已。UDP的幾個主要函數如下所示：

	func ResolveUDPAddr(net, addr string) (*UDPAddr, os.Error)
	func DialUDP(net string, laddr, raddr *UDPAddr) (c *UDPConn, err os.Error)
	func ListenUDP(net string, laddr *UDPAddr) (c *UDPConn, err os.Error)
	func (c *UDPConn) ReadFromUDP(b []byte) (n int, addr *UDPAddr, err os.Error)
	func (c *UDPConn) WriteToUDP(b []byte, addr *UDPAddr) (n int, err os.Error)

一個UDP的客戶端代碼如下所示,我們可以看到不同的就是TCP換成了UDP而已：

	package main

	import (
		"fmt"
		"net"
		"os"
	)

	func main() {
		if len(os.Args) != 2 {
			fmt.Fprintf(os.Stderr, "Usage: %s host:port", os.Args[0])
			os.Exit(1)
		}
		service := os.Args[1]
		udpAddr, err := net.ResolveUDPAddr("udp4", service)
		checkError(err)
		conn, err := net.DialUDP("udp", nil, udpAddr)
		checkError(err)
		_, err = conn.Write([]byte("anything"))
		checkError(err)
		var buf [512]byte
		n, err := conn.Read(buf[0:])
		checkError(err)
		fmt.Println(string(buf[0:n]))
		os.Exit(0)
	}
	func checkError(err error) {
		if err != nil {
			fmt.Fprintf(os.Stderr, "Fatal error ", err.Error())
			os.Exit(1)
		}
	}

我們來看一下UDP服務器端如何來處理：

	package main

	import (
		"fmt"
		"net"
		"os"
		"time"
	)

	func main() {
		service := ":1200"
		udpAddr, err := net.ResolveUDPAddr("udp4", service)
		checkError(err)
		conn, err := net.ListenUDP("udp", udpAddr)
		checkError(err)
		for {
			handleClient(conn)
		}
	}
	func handleClient(conn *net.UDPConn) {
		var buf [512]byte
		_, addr, err := conn.ReadFromUDP(buf[0:])
		if err != nil {
			return
		}
		daytime := time.Now().String()
		conn.WriteToUDP([]byte(daytime), addr)
	}
	func checkError(err error) {
		if err != nil {
			fmt.Fprintf(os.Stderr, "Fatal error ", err.Error())
			os.Exit(1)
		}
	}

## 總結
通過對TCP和UDP Socket編程的描述和實現，可見Go已經完備地支持了Socket編程，而且使用起來相當的方便，Go提供了很多函數，通過這些函數可以很容易就編寫出高性能的Socket應用。


## links
   * [目錄](<preface.md>)
   * 上一節: [Web服務](<08.0.md>)
   * 下一節: [WebSocket](<08.2.md>)

---

# 8.2 WebSocket
WebSocket是HTML5的重要特性，它實現了基於瀏覽器的遠程socket，它使瀏覽器和服務器可以進行全雙工通信，許多瀏覽器（Firefox、Google Chrome和Safari）都已對此做了支持。

在WebSocket出現之前，為了實現即時通信，採用的技術都是“輪詢”，即在特定的時間間隔內，由瀏覽器對服務器發出HTTP Request，服務器在收到請求後，返回最新的數據給瀏覽器刷新，“輪詢”使得瀏覽器需要對服務器不斷髮出請求，這樣會佔用大量帶寬。

WebSocket採用了一些特殊的報頭，使得瀏覽器和服務器只需要做一個握手的動作，就可以在瀏覽器和服務器之間建立一條連接通道。且此連接會保持在活動狀態，你可以使用JavaScript來向連接寫入或從中接收數據，就像在使用一個常規的TCP Socket一樣。它解決了Web實時化的問題，相比傳統HTTP有如下好處：

- 一個Web客戶端只建立一個TCP連接
- Websocket服務端可以推送(push)數據到web客戶端.
- 有更加輕量級的頭，減少數據傳送量

WebSocket URL的起始輸入是ws://或是wss://（在SSL上）。下圖展示了WebSocket的通信過程，一個帶有特定報頭的HTTP握手被髮送到了服務器端，接著在服務器端或是客戶端就可以通過JavaScript來使用某種套接口（socket），這一套接口可被用來通過事件句柄異步地接收數據。

![](images/8.2.websocket.png?raw=true)

圖8.2 WebSocket原理圖

## WebSocket原理
WebSocket的協議頗為簡單，在第一次handshake通過以後，連接便建立成功，其後的通訊數據都是以”\x00″開頭，以”\xFF”結尾。在客戶端，這個是透明的，WebSocket組件會自動將原始數據“掐頭去尾”。

瀏覽器發出WebSocket連接請求，然後服務器發出迴應，然後連接建立成功，這個過程通常稱為“握手” (handshaking)。請看下面的請求和反饋信息：

![](images/8.2.websocket2.png?raw=true)

圖8.3 WebSocket的request和response信息

在請求中的"Sec-WebSocket-Key"是隨機的，對於整天跟編碼打交到的程序員，一眼就可以看出來：這個是一個經過base64編碼後的數據。服務器端接收到這個請求之後需要把這個字符串連接上一個固定的字符串：

	258EAFA5-E914-47DA-95CA-C5AB0DC85B11

即：`f7cb4ezEAl6C3wRaU6JORA==`連接上那一串固定字符串，生成一個這樣的字符串：

	f7cb4ezEAl6C3wRaU6JORA==258EAFA5-E914-47DA-95CA-C5AB0DC85B11

對該字符串先用 sha1安全散列算法計算出二進制的值，然後用base64對其進行編碼，即可以得到握手後的字符串：

	rE91AJhfC+6JdVcVXOGJEADEJdQ=

將之作為響應頭`Sec-WebSocket-Accept`的值反饋給客戶端。

## Go實現WebSocket
Go語言標準包裡面沒有提供對WebSocket的支持，但是在由官方維護的go.net子包中有對這個的支持，你可以通過如下的命令獲取該包：

	go get code.google.com/p/go.net/websocket

WebSocket分為客戶端和服務端，接下來我們將實現一個簡單的例子:用戶輸入信息，客戶端通過WebSocket將信息發送給服務器端，服務器端收到信息之後主動Push信息到客戶端，然後客戶端將輸出其收到的信息，客戶端的代碼如下：

	<html>
	<head></head>
	<body>
		<script type="text/javascript">
			var sock = null;
			var wsuri = "ws://127.0.0.1:1234";

			window.onload = function() {

				console.log("onload");

				sock = new WebSocket(wsuri);

				sock.onopen = function() {
					console.log("connected to " + wsuri);
				}

				sock.onclose = function(e) {
					console.log("connection closed (" + e.code + ")");
				}

				sock.onmessage = function(e) {
					console.log("message received: " + e.data);
				}
			};

			function send() {
				var msg = document.getElementById('message').value;
				sock.send(msg);
			};
		</script>
		<h1>WebSocket Echo Test</h1>
		<form>
			<p>
				Message: <input id="message" type="text" value="Hello, world!">
			</p>
		</form>
		<button onclick="send();">Send Message</button>
	</body>
	</html>


可以看到客戶端JS，很容易的就通過WebSocket函數建立了一個與服務器的連接sock，當握手成功後，會觸發WebScoket對象的onopen事件，告訴客戶端連接已經成功建立。客戶端一共綁定了四個事件。

- 1）onopen 建立連接後觸發
- 2）onmessage 收到消息後觸發
- 3）onerror 發生錯誤時觸發
- 4）onclose 關閉連接時觸發

我們服務器端的實現如下：

	package main

	import (
		"golang.org/x/net/websocket"
		"fmt"
		"log"
		"net/http"
	)

	func Echo(ws *websocket.Conn) {
		var err error

		for {
			var reply string

			if err = websocket.Message.Receive(ws, &reply); err != nil {
				fmt.Println("Can't receive")
				break
			}

			fmt.Println("Received back from client: " + reply)

			msg := "Received:  " + reply
			fmt.Println("Sending to client: " + msg)

			if err = websocket.Message.Send(ws, msg); err != nil {
				fmt.Println("Can't send")
				break
			}
		}
	}

	func main() {
		http.Handle("/", websocket.Handler(Echo))

		if err := http.ListenAndServe(":1234", nil); err != nil {
			log.Fatal("ListenAndServe:", err)
		}
	}

當客戶端將用戶輸入的信息Send之後，服務器端通過Receive接收到了相應信息，然後通過Send發送了應答信息。

![](images/8.2.websocket3.png?raw=true)

圖8.4 WebSocket服務器端接收到的信息

通過上面的例子我們看到客戶端和服務器端實現WebSocket非常的方便，Go的源碼net分支中已經實現了這個的協議，我們可以直接拿來用，目前隨著HTML5的發展，我想未來WebSocket會是Web開發的一個重點，我們需要儲備這方面的知識。


## links
   * [目錄](<preface.md>)
   * 上一節: [Socket編程](<08.1.md>)
   * 下一節: [REST](<08.3.md>)

---

# 8.3 REST
RESTful，是目前最為流行的一種互聯網軟件架構。因為它結構清晰、符合標準、易於理解、擴展方便，所以正得到越來越多網站的採用。本小節我們將來學習它到底是一種什麼樣的架構？以及在Go裡面如何來實現它。
## 什麼是REST
REST(REpresentational State Transfer)這個概念，首次出現是在 2000年Roy Thomas Fielding（他是HTTP規範的主要編寫者之一）的博士論文中，它指的是一組架構約束條件和原則。滿足這些約束條件和原則的應用程序或設計就是RESTful的。

要理解什麼是REST，我們需要理解下面幾個概念:

- 資源（Resources）
  REST是"表現層狀態轉化"，其實它省略了主語。"表現層"其實指的是"資源"的"表現層"。

  那麼什麼是資源呢？就是我們平常上網訪問的一張圖片、一個文檔、一個視頻等。這些資源我們通過URI來定位，也就是一個URI表示一個資源。

- 表現層（Representation）

  資源是做一個具體的實體信息，他可以有多種的展現方式。而把實體展現出來就是表現層，例如一個txt文本信息，他可以輸出成html、json、xml等格式，一個圖片他可以jpg、png等方式展現，這個就是表現層的意思。

  URI確定一個資源，但是如何確定它的具體表現形式呢？應該在HTTP請求的頭信息中用Accept和Content-Type字段指定，這兩個字段才是對"表現層"的描述。

- 狀態轉化（State Transfer）

  訪問一個網站，就代表了客戶端和服務器的一個互動過程。在這個過程中，肯定涉及到數據和狀態的變化。而HTTP協議是無狀態的，那麼這些狀態肯定保存在服務器端，所以如果客戶端想要通知服務器端改變數據和狀態的變化，肯定要通過某種方式來通知它。

  客戶端能通知服務器端的手段，只能是HTTP協議。具體來說，就是HTTP協議裡面，四個表示操作方式的動詞：GET、POST、PUT、DELETE。它們分別對應四種基本操作：GET用來獲取資源，POST用來新建資源（也可以用於更新資源），PUT用來更新資源，DELETE用來刪除資源。

綜合上面的解釋，我們總結一下什麼是RESTful架構：

- （1）每一個URI代表一種資源；
- （2）客戶端和服務器之間，傳遞這種資源的某種表現層；
- （3）客戶端通過四個HTTP動詞，對服務器端資源進行操作，實現"表現層狀態轉化"。


Web應用要滿足REST最重要的原則是:客戶端和服務器之間的交互在請求之間是無狀態的,即從客戶端到服務器的每個請求都必須包含理解請求所必需的信息。如果服務器在請求之間的任何時間點重啟，客戶端不會得到通知。此外此請求可以由任何可用服務器回答，這十分適合雲計算之類的環境。因為是無狀態的，所以客戶端可以緩存數據以改進性能。

另一個重要的REST原則是系統分層，這表示組件無法瞭解除了與它直接交互的層次以外的組件。通過將系統知識限制在單個層，可以限制整個系統的複雜性，從而促進了底層的獨立性。

下圖即是REST的架構圖：

![](images/8.3.rest2.png?raw=true)

圖8.5 REST架構圖

當REST架構的約束條件作為一個整體應用時，將生成一個可以擴展到大量客戶端的應用程序。它還降低了客戶端和服務器之間的交互延遲。統一界面簡化了整個系統架構，改進了子系統之間交互的可見性。REST簡化了客戶端和服務器的實現，而且對於使用REST開發的應用程序更加容易擴展。

下圖展示了REST的擴展性：

![](images/8.3.rest.png?raw=true)

圖8.6 REST的擴展性

## RESTful的實現
Go沒有為REST提供直接支持，但是因為RESTful是基於HTTP協議實現的，所以我們可以利用`net/http`包來自己實現，當然需要針對REST做一些改造，REST是根據不同的method來處理相應的資源，目前已經存在的很多自稱是REST的應用，其實並沒有真正的實現REST，我暫且把這些應用根據實現的method分成幾個級別，請看下圖：

![](images/8.3.rest3.png?raw=true)

圖8.7 REST的level分級

上圖展示了我們目前實現REST的三個level，我們在應用開發的時候也不一定全部按照RESTful的規則全部實現他的方式，因為有些時候完全按照RESTful的方式未必是可行的，RESTful服務充分利用每一個HTTP方法，包括`DELETE`和`PUT`。可有時，HTTP客戶端只能發出`GET`和`POST`請求：

- HTML標準只能通過鏈接和表單支持`GET`和`POST`。在沒有Ajax支持的網頁瀏覽器中不能發出`PUT`或`DELETE`命令

- 有些防火牆會擋住HTTP `PUT`和`DELETE`請求要繞過這個限制，客戶端需要把實際的`PUT`和`DELETE`請求通過 POST 請求穿透過來。RESTful 服務則要負責在收到的 POST 請求中找到原始的 HTTP 方法並還原。

我們現在可以通過`POST`裡面增加隱藏字段`_method`這種方式可以來模擬`PUT`、`DELETE`等方式，但是服務器端需要做轉換。我現在的項目裡面就按照這種方式來做的REST接口。當然Go語言裡面完全按照RESTful來實現是很容易的，我們通過下面的例子來說明如何實現RESTful的應用設計。

	package main

	import (
		"fmt"
		"github.com/drone/routes"
		"net/http"
	)

	func getuser(w http.ResponseWriter, r *http.Request) {
		params := r.URL.Query()
		uid := params.Get(":uid")
		fmt.Fprintf(w, "you are get user %s", uid)
	}

	func modifyuser(w http.ResponseWriter, r *http.Request) {
		params := r.URL.Query()
		uid := params.Get(":uid")
		fmt.Fprintf(w, "you are modify user %s", uid)
	}

	func deleteuser(w http.ResponseWriter, r *http.Request) {
		params := r.URL.Query()
		uid := params.Get(":uid")
		fmt.Fprintf(w, "you are delete user %s", uid)
	}

	func adduser(w http.ResponseWriter, r *http.Request) {
		uid := r.FormValue("uid")
		fmt.Fprint(w, "you are add user %s", uid)
	}

	func main() {
		mux := routes.New()
		mux.Get("/user/:uid", getuser)
		mux.Post("/user/", adduser)
		mux.Del("/user/:uid", deleteuser)
		mux.Put("/user/:uid", modifyuser)
		http.Handle("/", mux)
		http.ListenAndServe(":8088", nil)
	}

上面的代碼演示瞭如何編寫一個REST的應用，我們訪問的資源是用戶，我們通過不同的method來訪問不同的函數，這裡使用了第三方庫`github.com/drone/routes`，在前面章節我們介紹過如何實現自定義的路由器，這個庫實現了自定義路由和方便的路由規則映射，通過它，我們可以很方便的實現REST的架構。通過上面的代碼可知，REST就是根據不同的method訪問同一個資源的時候實現不同的邏輯處理。

## 總結
REST是一種架構風格，汲取了WWW的成功經驗：無狀態，以資源為中心，充分利用HTTP協議和URI協議，提供統一的接口定義，使得它作為一種設計Web服務的方法而變得流行。在某種意義上，通過強調URI和HTTP等早期Internet標準，REST是對大型應用程序服務器時代之前的Web方式的迴歸。目前Go對於REST的支持還是很簡單的，通過實現自定義的路由規則，我們就可以為不同的method實現不同的handle，這樣就實現了REST的架構。

## links
   * [目錄](<preface.md>)
   * 上一節: [WebSocket](<08.2.md>)
   * 下一節: [RPC](<08.4.md>)

---

# 8.4 RPC
前面幾個小節我們介紹瞭如何基於Socket和HTTP來編寫網絡應用，通過學習我們瞭解了Socket和HTTP採用的是類似"信息交換"模式，即客戶端發送一條信息到服務端，然後(一般來說)服務器端都會返回一定的信息以表示響應。客戶端和服務端之間約定了交互信息的格式，以便雙方都能夠解析交互所產生的信息。但是很多獨立的應用並沒有採用這種模式，而是採用類似常規的函數調用的方式來完成想要的功能。

RPC就是想實現函數調用模式的網絡化。客戶端就像調用本地函數一樣，然後客戶端把這些參數打包之後通過網絡傳遞到服務端，服務端解包到處理過程中執行，然後執行的結果反饋給客戶端。

RPC（Remote Procedure Call Protocol）——遠程過程調用協議，是一種通過網絡從遠程計算機程序上請求服務，而不需要了解底層網絡技術的協議。它假定某些傳輸協議的存在，如TCP或UDP，以便為通信程序之間攜帶信息數據。通過它可以使函數調用模式網絡化。在OSI網絡通信模型中，RPC跨越了傳輸層和應用層。RPC使得開發包括網絡分佈式多程序在內的應用程序更加容易。

## RPC工作原理

![](images/8.4.rpc.png?raw=true)

圖8.8 RPC工作流程圖

運行時,一次客戶機對服務器的RPC調用,其內部操作大致有如下十步：

- 1.調用客戶端句柄；執行傳送參數
- 2.調用本地系統內核發送網絡消息
- 3.消息傳送到遠程主機
- 4.服務器句柄得到消息並取得參數
- 5.執行遠程過程
- 6.執行的過程將結果返回服務器句柄
- 7.服務器句柄返回結果，調用遠程系統內核
- 8.消息傳回本地主機
- 9.客戶句柄由內核接收消息
- 10.客戶接收句柄返回的數據

## Go RPC
Go標準包中已經提供了對RPC的支持，而且支持三個級別的RPC：TCP、HTTP、JSONRPC。但Go的RPC包是獨一無二的RPC，它和傳統的RPC系統不同，它只支持Go開發的服務器與客戶端之間的交互，因為在內部，它們採用了Gob來編碼。

Go RPC的函數只有符合下面的條件才能被遠程訪問，不然會被忽略，詳細的要求如下：

- 函數必須是導出的(首字母大寫)
- 必須有兩個導出類型的參數，
- 第一個參數是接收的參數，第二個參數是返回給客戶端的參數，第二個參數必須是指針類型的
- 函數還要有一個返回值error

舉個例子，正確的RPC函數格式如下：

	func (t *T) MethodName(argType T1, replyType *T2) error

T、T1和T2類型必須能被`encoding/gob`包編解碼。

任何的RPC都需要通過網絡來傳遞數據，Go RPC可以利用HTTP和TCP來傳遞數據，利用HTTP的好處是可以直接複用`net/http`裡面的一些函數。詳細的例子請看下面的實現

### HTTP RPC
http的服務端代碼實現如下：

	package main

	import (
		"errors"
		"fmt"
		"net/http"
		"net/rpc"
	)

	type Args struct {
		A, B int
	}

	type Quotient struct {
		Quo, Rem int
	}

	type Arith int

	func (t *Arith) Multiply(args *Args, reply *int) error {
		*reply = args.A * args.B
		return nil
	}

	func (t *Arith) Divide(args *Args, quo *Quotient) error {
		if args.B == 0 {
			return errors.New("divide by zero")
		}
		quo.Quo = args.A / args.B
		quo.Rem = args.A % args.B
		return nil
	}

	func main() {

		arith := new(Arith)
		rpc.Register(arith)
		rpc.HandleHTTP()

		err := http.ListenAndServe(":1234", nil)
		if err != nil {
			fmt.Println(err.Error())
		}
	}

通過上面的例子可以看到，我們註冊了一個Arith的RPC服務，然後通過`rpc.HandleHTTP`函數把該服務註冊到了HTTP協議上，然後我們就可以利用http的方式來傳遞數據了。

請看下面的客戶端代碼：

	package main

	import (
		"fmt"
		"log"
		"net/rpc"
		"os"
	)

	type Args struct {
		A, B int
	}

	type Quotient struct {
		Quo, Rem int
	}

	func main() {
		if len(os.Args) != 2 {
			fmt.Println("Usage: ", os.Args[0], "server")
			os.Exit(1)
		}
		serverAddress := os.Args[1]

		client, err := rpc.DialHTTP("tcp", serverAddress+":1234")
		if err != nil {
			log.Fatal("dialing:", err)
		}
		// Synchronous call
		args := Args{17, 8}
		var reply int
		err = client.Call("Arith.Multiply", args, &reply)
		if err != nil {
			log.Fatal("arith error:", err)
		}
		fmt.Printf("Arith: %d*%d=%d\n", args.A, args.B, reply)

		var quot Quotient
		err = client.Call("Arith.Divide", args, &quot)
		if err != nil {
			log.Fatal("arith error:", err)
		}
		fmt.Printf("Arith: %d/%d=%d remainder %d\n", args.A, args.B, quot.Quo, quot.Rem)

	}

我們把上面的服務端和客戶端的代碼分別編譯，然後先把服務端開啟，然後開啟客戶端，輸入代碼，就會輸出如下信息：

	$ ./http_c localhost
	Arith: 17*8=136
	Arith: 17/8=2 remainder 1

通過上面的調用可以看到參數和返回值是我們定義的struct類型，在服務端我們把它們當做調用函數的參數的類型，在客戶端作為`client.Call`的第2，3兩個參數的類型。客戶端最重要的就是這個Call函數，它有3個參數，第1個要調用的函數的名字，第2個是要傳遞的參數，第3個要返回的參數(注意是指針類型)，通過上面的代碼例子我們可以發現，使用Go的RPC實現相當的簡單，方便。
### TCP RPC
上面我們實現了基於HTTP協議的RPC，接下來我們要實現基於TCP協議的RPC，服務端的實現代碼如下所示：

	package main

	import (
		"errors"
		"fmt"
		"net"
		"net/rpc"
		"os"
	)

	type Args struct {
		A, B int
	}

	type Quotient struct {
		Quo, Rem int
	}

	type Arith int

	func (t *Arith) Multiply(args *Args, reply *int) error {
		*reply = args.A * args.B
		return nil
	}

	func (t *Arith) Divide(args *Args, quo *Quotient) error {
		if args.B == 0 {
			return errors.New("divide by zero")
		}
		quo.Quo = args.A / args.B
		quo.Rem = args.A % args.B
		return nil
	}

	func main() {

		arith := new(Arith)
		rpc.Register(arith)

		tcpAddr, err := net.ResolveTCPAddr("tcp", ":1234")
		checkError(err)

		listener, err := net.ListenTCP("tcp", tcpAddr)
		checkError(err)

		for {
			conn, err := listener.Accept()
			if err != nil {
				continue
			}
			rpc.ServeConn(conn)
		}

	}

	func checkError(err error) {
		if err != nil {
			fmt.Println("Fatal error ", err.Error())
			os.Exit(1)
		}
	}

上面這個代碼和http的服務器相比，不同在於:在此處我們採用了TCP協議，然後需要自己控制連接，當有客戶端連接上來後，我們需要把這個連接交給rpc來處理。

如果你留心了，你會發現這它是一個阻塞型的單用戶的程序，如果想要實現多併發，那麼可以使用goroutine來實現，我們前面在socket小節的時候已經介紹過如何處理goroutine。
下面展現了TCP實現的RPC客戶端：

	package main

	import (
		"fmt"
		"log"
		"net/rpc"
		"os"
	)

	type Args struct {
		A, B int
	}

	type Quotient struct {
		Quo, Rem int
	}

	func main() {
		if len(os.Args) != 2 {
			fmt.Println("Usage: ", os.Args[0], "server:port")
			os.Exit(1)
		}
		service := os.Args[1]

		client, err := rpc.Dial("tcp", service)
		if err != nil {
			log.Fatal("dialing:", err)
		}
		// Synchronous call
		args := Args{17, 8}
		var reply int
		err = client.Call("Arith.Multiply", args, &reply)
		if err != nil {
			log.Fatal("arith error:", err)
		}
		fmt.Printf("Arith: %d*%d=%d\n", args.A, args.B, reply)

		var quot Quotient
		err = client.Call("Arith.Divide", args, &quot)
		if err != nil {
			log.Fatal("arith error:", err)
		}
		fmt.Printf("Arith: %d/%d=%d remainder %d\n", args.A, args.B, quot.Quo, quot.Rem)

	}

這個客戶端代碼和http的客戶端代碼對比，唯一的區別一個是DialHTTP，一個是Dial(tcp)，其他處理一模一樣。

### JSON RPC
JSON RPC是數據編碼採用了JSON，而不是gob編碼，其他和上面介紹的RPC概念一模一樣，下面我們來演示一下，如何使用Go提供的json-rpc標準包，請看服務端代碼的實現：

	package main

	import (
		"errors"
		"fmt"
		"net"
		"net/rpc"
		"net/rpc/jsonrpc"
		"os"
	)

	type Args struct {
		A, B int
	}

	type Quotient struct {
		Quo, Rem int
	}

	type Arith int

	func (t *Arith) Multiply(args *Args, reply *int) error {
		*reply = args.A * args.B
		return nil
	}

	func (t *Arith) Divide(args *Args, quo *Quotient) error {
		if args.B == 0 {
			return errors.New("divide by zero")
		}
		quo.Quo = args.A / args.B
		quo.Rem = args.A % args.B
		return nil
	}

	func main() {

		arith := new(Arith)
		rpc.Register(arith)

		tcpAddr, err := net.ResolveTCPAddr("tcp", ":1234")
		checkError(err)

		listener, err := net.ListenTCP("tcp", tcpAddr)
		checkError(err)

		for {
			conn, err := listener.Accept()
			if err != nil {
				continue
			}
			jsonrpc.ServeConn(conn)
		}

	}

	func checkError(err error) {
		if err != nil {
			fmt.Println("Fatal error ", err.Error())
			os.Exit(1)
		}
	}

通過示例我們可以看出 json-rpc是基於TCP協議實現的，目前它還不支持HTTP方式。

請看客戶端的實現代碼：

	package main

	import (
		"fmt"
		"log"
		"net/rpc/jsonrpc"
		"os"
	)

	type Args struct {
		A, B int
	}

	type Quotient struct {
		Quo, Rem int
	}

	func main() {
		if len(os.Args) != 2 {
			fmt.Println("Usage: ", os.Args[0], "server:port")
			log.Fatal(1)
		}
		service := os.Args[1]

		client, err := jsonrpc.Dial("tcp", service)
		if err != nil {
			log.Fatal("dialing:", err)
		}
		// Synchronous call
		args := Args{17, 8}
		var reply int
		err = client.Call("Arith.Multiply", args, &reply)
		if err != nil {
			log.Fatal("arith error:", err)
		}
		fmt.Printf("Arith: %d*%d=%d\n", args.A, args.B, reply)

		var quot Quotient
		err = client.Call("Arith.Divide", args, &quot)
		if err != nil {
			log.Fatal("arith error:", err)
		}
		fmt.Printf("Arith: %d/%d=%d remainder %d\n", args.A, args.B, quot.Quo, quot.Rem)

	}

## 總結
Go已經提供了對RPC的良好支持，通過上面HTTP、TCP、JSON RPC的實現,我們就可以很方便的開發很多分佈式的Web應用，我想作為讀者的你已經領會到這一點。但遺憾的是目前Go尚未提供對SOAP RPC的支持，欣慰的是現在已經有第三方的開源實現了。



## links
   * [目錄](<preface.md>)
   * 上一節: [REST](<08.3.md>)
   * 下一節: [小結](<08.5.md>)

---

# 8.5 小結
這一章我們介紹了目前流行的幾種主要的網絡應用開發方式，第一小節介紹了網絡編程中的基礎:Socket編程，因為現在網絡正在朝雲的方向快速進化，作為這一技術演進的基石的的socket知識，作為開發者的你，是必須要掌握的。第二小節介紹了正愈發流行的HTML5中一個重要的特性WebSocket，通過它,服務器可以實現主動的push消息，以簡化以前ajax輪詢的模式。第三小節介紹了REST編寫模式，這種模式特別適合來開發網絡應用API，目前移動應用的快速發展，我覺得將來會是一個潮流。第四小節介紹了Go實現的RPC相關知識，對於上面四種開發方式，Go都已經提供了良好的支持，net包及其子包,是所有涉及到網絡編程的工具的所在地。如果你想更加深入的瞭解相關實現細節，可以嘗試閱讀這個包下面的源碼。
## links
   * [目錄](<preface.md>)
   * 上一節: [RPC](<08.4.md>)
   * 下一章: [安全與加密](<09.0.md>)
