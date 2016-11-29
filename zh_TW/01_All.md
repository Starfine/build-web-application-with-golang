# 1 GO環境配置

歡迎來到Go的世界，讓我們開始探索吧！

Go是一種新的語言，一種併發的、帶垃圾回收的、快速編譯的語言。它具有以下特點：

- 它可以在一臺計算機上用幾秒鐘的時間編譯一個大型的Go程序。
- Go為軟件構造提供了一種模型，它使依賴分析更加容易，且避免了大部分C風格include文件與庫的開頭。
- Go是靜態類型的語言，它的類型系統沒有層級。因此用戶不需要在定義類型之間的關係上花費時間，這樣感覺起來比典型的面嚮對象語言更輕量級。
- Go完全是垃圾回收型的語言，併為併發執行與通信提供了基本的支持。
- 按照其設計，Go打算為多核機器上系統軟件的構造提供一種方法。

Go是一種編譯型語言，它結合瞭解釋型語言的遊刃有餘，動態類型語言的開發效率，以及靜態類型的安全性。它也打算成為現代的，支持網絡與多核計算的語言。要滿足這些目標，需要解決一些語言上的問題：一個富有表達能力但輕量級的類型系統，併發與垃圾回收機制，嚴格的依賴規範等等。這些無法通過庫或工具解決好，因此Go也就應運而生了。

在本章中，我們將講述Go的安裝方法，以及如何配置項目信息。

## 目錄
  
![](images/navi1.png?raw=true)

## links
  * [目錄](<preface.md>)
  * 下一節: [Go安裝](<01.1.md>)

---

# 1.1 Go 安裝

## Go的三種安裝方式
Go有多種安裝方式，你可以選擇自己喜歡的。這裡我們介紹三種最常見的安裝方式：

- Go源碼安裝：這是一種標準的軟件安裝方式。對於經常使用Unix類系統的用戶，尤其對於開發者來說，從源碼安裝可以自己定製。
- Go標準包安裝：Go提供了方便的安裝包，支持Windows、Linux、Mac等系統。這種方式適合快速安裝，可根據自己的系統位數下載好相應的安裝包，一路next就可以輕鬆安裝了。**推薦這種方式**
- 第三方工具安裝：目前有很多方便的第三方軟件包工具，例如Ubuntu的apt-get、Mac的homebrew等。這種安裝方式適合那些熟悉相應系統的用戶。

最後，如果你想在同一個系統中安裝多個版本的Go，你可以參考第三方工具[GVM](https://github.com/moovweb/gvm)，這是目前在這方面做得最好的工具，除非你知道怎麼處理。

## Go源碼安裝
在Go的源代碼中，有些部分是用Plan 9 C和AT&T彙編寫的，因此假如你要想從源碼安裝，就必須安裝C的編譯工具。

在Mac系統中，只要你安裝了Xcode，就已經包含了相應的編譯工具。

在類Unix系統中，需要安裝gcc等工具。例如Ubuntu系統可通過在終端中執行`sudo apt-get install gcc libc6-dev`來安裝編譯工具。

在Windows系統中，你需要安裝MinGW，然後通過MinGW安裝gcc，並設置相應的環境變量。

你可以直接去官網[下載源碼](http://golang.org/dl/)，找相應的`goVERSION.src.tar.gz`的文件下載，下載之後解壓縮到`$HOME`目錄，執行如下代碼：

	cd go/src
	./all.bash

運行all.bash後出現"ALL TESTS PASSED"字樣時才算安裝成功。

上面是Unix風格的命令，Windows下的安裝方式類似，只不過是運行`all.bat`，調用的編譯器是MinGW的gcc。

如果是Mac或者Unix用戶需要設置幾個環境變量，如果想重啟之後也能生效的話把下面的命令寫到`.bashrc`或者`.zshrc`裡面，

	export GOPATH=$HOME/gopath
	export PATH=$PATH:$HOME/go/bin:$GOPATH/bin

如果你是寫入文件的，記得執行`bash .bashrc`或者`bash .zshrc`使得設置立馬生效。

如果是window系統，就需要設置環境變量，在path裡面增加相應的go所在的目錄，設置gopath變量。

當你設置完畢之後在命令行裡面輸入`go`，看到如下圖片即說明你已經安裝成功

![](images/1.1.mac.png?raw=true)

圖1.1 源碼安裝之後執行Go命令的圖

如果出現Go的Usage信息，那麼說明Go已經安裝成功了；如果出現該命令不存在，那麼可以檢查一下自己的PATH環境變中是否包含了Go的安裝目錄。

> 關於上面的GOPATH將在下面小節詳細講解

## Go標準包安裝

Go提供了每個平臺打好包的一鍵安裝，這些包默認會安裝到如下目錄：/usr/local/go (Windows系統：c:\Go)，當然你可以改變他們的安裝位置，但是改變之後你必須在你的環境變量中設置如下信息：

	export GOROOT=$HOME/go  
	export GOPATH=$HOME/gopath
	export PATH=$PATH:$GOROOT/bin:$GOPATH/bin

上面這些命令對於Mac和Unix用戶來說最好是寫入`.bashrc`或者`.zshrc`文件，對於windows用戶來說當然是寫入環境變量。	

### 如何判斷自己的操作系統是32位還是64位？

我們接下來的Go安裝需要判斷操作系統的位數，所以這小節我們先確定自己的系統類型。

Windows系統用戶請按Win+R運行cmd，輸入`systeminfo`後回車，稍等片刻，會出現一些系統信息。在“系統類型”一行中，若顯示“x64-based PC”，即為64位系統；若顯示“X86-based PC”，則為32位系統。

Mac系統用戶建議直接使用64位的，因為Go所支持的Mac OS X版本已經不支持純32位處理器了。

Linux系統用戶可通過在Terminal中執行命令`arch`(即`uname -m`)來查看系統信息：

64位系統顯示

	x86_64

32位系統顯示

	i386

### Mac 安裝

訪問[下載地址][downlink]，32位系統下載go1.4.2.darwin-386-osx10.8.pkg，64位系統下載go1.4.2.darwin-amd64-osx10.8.pkg，雙擊下載文件，一路默認安裝點擊下一步，這個時候go已經安裝到你的系統中，默認已經在PATH中增加了相應的`~/go/bin`,這個時候打開終端，輸入`go`

看到類似上面源碼安裝成功的圖片說明已經安裝成功

如果出現go的Usage信息，那麼說明go已經安裝成功了；如果出現該命令不存在，那麼可以檢查一下自己的PATH環境變中是否包含了go的安裝目錄。

### Linux 安裝

訪問[下載地址][downlink]，32位系統下載go1.4.2.linux-386.tar.gz，64位系統下載go1.4.2.linux-amd64.tar.gz，

假定你想要安裝Go的目錄為 `$GO_INSTALL_DIR`，後面替換為相應的目錄路徑。

解壓縮`tar.gz`包到安裝目錄下：`tar zxvf go1.4.2.linux-amd64.tar.gz -C $GO_INSTALL_DIR`。

設置PATH，`export PATH=$PATH:$GO_INSTALL_DIR/go/bin`

然後執行`go`

![](images/1.1.linux.png?raw=true)

圖1.2 Linux系統下安裝成功之後執行go顯示的信息

如果出現go的Usage信息，那麼說明go已經安裝成功了；如果出現該命令不存在，那麼可以檢查一下自己的PATH環境變中是否包含了go的安裝目錄。

### Windows 安裝 ###

訪問[Google Code 下載頁][downlink]，32 位請選擇名稱中包含 windows-386 的 msi 安裝包，64 位請選擇名稱中包含 windows-amd64 的。下載好後運行，不要修改默認安裝目錄 C:\Go\，若安裝到其他位置會導致不能執行自己所編寫的 Go 代碼。安裝完成後默認會在環境變量 Path 後添加 Go 安裝目錄下的 bin 目錄 `C:\Go\bin\`，並添加環境變量 GOROOT，值為 Go 安裝根目錄 `C:\Go\` 。

**驗證是否安裝成功**

在運行中輸入 `cmd` 打開命令行工具，在提示符下輸入 `go`，檢查是否能看到 Usage 信息。輸入 `cd %GOROOT%`，看是否能進入 Go 安裝目錄。若都成功，說明安裝成功。

不能的話請檢查上述環境變量 Path 和 GOROOT 的值。若不存在請卸載後重新安裝，存在請重啟計算機後重試以上步驟。

## 第三方工具安裝

### GVM

gvm是第三方開發的Go多版本管理工具，類似ruby裡面的rvm工具。使用起來相當的方便，安裝gvm使用如下命令：

	bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)

安裝完成後我們就可以安裝go了：

	gvm install go1.4.2
	gvm use go1.4.2

也可以使用下面的命令，省去每次調用gvm use的麻煩：
        gvm use go1.4.2 --default
        
執行完上面的命令之後GOPATH、GOROOT等環境變量會自動設置好，這樣就可以直接使用了。

### apt-get
Ubuntu是目前使用最多的Linux桌面系統，使用`apt-get`命令來管理軟件包，我們可以通過下面的命令來安裝Go，為了以後方便，應該把 `git` `mercurial` 也安裝上：

	sudo apt-get install python-software-properties
	sudo add-apt-repository ppa:gophers/go
	sudo apt-get update
	sudo apt-get install golang-stable git-core mercurial

### homebrew
homebrew是Mac系統下面目前使用最多的管理軟件的工具，目前已支持Go，可以通過命令直接安裝Go，為了以後方便，應該把 `git` `mercurial` 也安裝上：

	brew update && brew upgrade
	brew install go
	brew install git
	brew install mercurial


## links
   * [目錄](<preface.md>)
   * 上一節: [Go環境配置](<01.0.md>)
   * 下一節: [GOPATH 與工作空間](<01.2.md>)

[downlink]:http://golang.org/dl/ "Go安裝包下載"

---

# 1.2 GOPATH與工作空間

前面我們在安裝Go的時候看到需要設置GOPATH變量，Go從1.1版本開始必須設置這個變量，而且不能和Go的安裝目錄一樣，這個目錄用來存放Go源碼，Go的可運行文件，以及相應的編譯之後的包文件。所以這個目錄下面有三個子目錄：src、bin、pkg

## GOPATH設置
  go 命令依賴一個重要的環境變量：$GOPATH

  Windows系統中環境變量的形式為`%GOPATH%`，本書主要使用Unix形式，Windows用戶請自行替換。

  *（注：這個不是Go安裝目錄。下面以筆者的工作目錄為示例，如果你想不一樣請把GOPATH替換成你的工作目錄。）*

  在類似 Unix 環境大概這樣設置：
```sh
export GOPATH=/home/apple/mygo
```
  為了方便，應該新建以上文件夾，並且上一行加入到 `.bashrc` 或者 `.zshrc` 或者自己的 `sh` 的配置文件中。

  Windows 設置如下，新建一個環境變量名稱叫做GOPATH：
```sh
	GOPATH=c:\mygo
```
GOPATH允許多個目錄，當有多個目錄時，請注意分隔符，多個目錄的時候Windows是分號，Linux系統是冒號，當有多個GOPATH時，默認會將go get的內容放在第一個目錄下。


以上 $GOPATH 目錄約定有三個子目錄：

- src 存放源代碼（比如：.go .c .h .s等）
- pkg 編譯後生成的文件（比如：.a）
- bin 編譯後生成的可執行文件（為了方便，可以把此目錄加入到 $PATH 變量中，如果有多個gopath，那麼使用`${GOPATH//://bin:}/bin`添加所有的bin目錄）

以後我所有的例子都是以mygo作為我的gopath目錄


## 代碼目錄結構規劃
GOPATH下的src目錄就是接下來開發程序的主要目錄，所有的源碼都是放在這個目錄下面，那麼一般我們的做法就是一個目錄一個項目，例如: $GOPATH/src/mymath 表示mymath這個應用包或者可執行應用，這個根據package是main還是其他來決定，main的話就是可執行應用，其他的話就是應用包，這個會在後續詳細介紹package。


所以當新建應用或者一個代碼包時都是在src目錄下新建一個文件夾，文件夾名稱一般是代碼包名稱，當然也允許多級目錄，例如在src下面新建了目錄$GOPATH/src/github.com/astaxie/beedb 那麼這個包路徑就是"github.com/astaxie/beedb"，包名稱是最後一個目錄beedb

下面我就以mymath為例來講述如何編寫應用包，執行如下代碼
```sh
cd $GOPATH/src
mkdir mymath
```

新建文件sqrt.go，內容如下
```go
// $GOPATH/src/mymath/sqrt.go源碼如下：
package mymath

func Sqrt(x float64) float64 {
	z := 0.0
	for i := 0; i < 1000; i++ {
		z -= (z*z - x) / (2 * x)
	}
	return z
}
```
這樣我的應用包目錄和代碼已經新建完畢，注意：一般建議package的名稱和目錄名保持一致

## 編譯應用
上面我們已經建立了自己的應用包，如何進行編譯安裝呢？有兩種方式可以進行安裝

1、只要進入對應的應用包目錄，然後執行`go install`，就可以安裝了

2、在任意的目錄執行如下代碼`go install mymath`

安裝完之後，我們可以進入如下目錄
```sh
cd $GOPATH/pkg/${GOOS}_${GOARCH}
//可以看到如下文件
mymath.a
```
這個.a文件是應用包，那麼我們如何進行調用呢？

接下來我們新建一個應用程序來調用這個應用包

新建應用包mathapp
```sh
cd $GOPATH/src
mkdir mathapp
cd mathapp
vim main.go
```

`$GOPATH/src/mathapp/main.go`源碼：
```go
package main

import (
	  "mymath"
	  "fmt"
)

func main() {
	  fmt.Printf("Hello, world.  Sqrt(2) = %v\n", mymath.Sqrt(2))
}
```

可以看到這個的package是`main`，import裡面調用的包是`mymath`,這個就是相對於`$GOPATH/src`的路徑，如果是多級目錄，就在import裡面引入多級目錄，如果你有多個GOPATH，也是一樣，Go會自動在多個`$GOPATH/src`中尋找。

如何編譯程序呢？進入該應用目錄，然後執行`go build`，那麼在該目錄下面會生成一個mathapp的可執行文件
```sh
./mathapp
```

輸出如下內容
```sh
Hello, world.  Sqrt(2) = 1.414213562373095
```

如何安裝該應用，進入該目錄執行`go install`,那麼在$GOPATH/bin/下增加了一個可執行文件mathapp, 還記得前面我們把`$GOPATH/bin`加到我們的PATH裡面了，這樣可以在命令行輸入如下命令就可以執行

```sh
mathapp
```
	
也是輸出如下內容

	Hello, world.  Sqrt(2) = 1.414213562373095
	
這裡我們展示如何編譯和安裝一個可運行的應用，以及如何設計我們的目錄結構。

## 獲取遠程包
   go語言有一個獲取遠程包的工具就是`go get`，目前go get支持多數開源社區(例如：github、googlecode、bitbucket、Launchpad)

	go get github.com/astaxie/beedb
	
>go get -u 參數可以自動更新包，而且當go get的時候會自動獲取該包依賴的其他第三方包	

通過這個命令可以獲取相應的源碼，對應的開源平臺採用不同的源碼控制工具，例如github採用git、googlecode採用hg，所以要想獲取這些源碼，必須先安裝相應的源碼控制工具

通過上面獲取的代碼在我們本地的源碼相應的代碼結構如下

	$GOPATH
	  src
	   |--github.com
			  |-astaxie
				  |-beedb
	   pkg
		|--相應平臺
			 |-github.com
				   |--astaxie
						|beedb.a

go get本質上可以理解為首先第一步是通過源碼工具clone代碼到src下面，然後執行`go install`

在代碼中如何使用遠程包，很簡單的就是和使用本地包一樣，只要在開頭import相應的路徑就可以

	import "github.com/astaxie/beedb"

## 程序的整體結構
通過上面建立的我本地的mygo的目錄結構如下所示

	bin/
		mathapp
	pkg/
		平臺名/ 如：darwin_amd64、linux_amd64
			 mymath.a
			 github.com/
				  astaxie/
					   beedb.a
	src/
		mathapp
			  main.go
		mymath/
			  sqrt.go
		github.com/
			   astaxie/
					beedb/
						beedb.go
						util.go

從上面的結構我們可以很清晰的看到，bin目錄下面存的是編譯之後可執行的文件，pkg下面存放的是應用包，src下面保存的是應用源代碼


## links
  * [目錄](<preface.md>)
  * 上一節: [GO安裝](<01.1.md>)
  * 下一節: [GO 命令](<01.3.md>)

---

# 1.3 Go 命令

## Go 命令

  Go語言自帶有一套完整的命令操作工具，你可以通過在命令行中執行`go`來查看它們：

  ![](images/1.1.mac.png?raw=true)

圖1.3 Go命令顯示詳細的信息

  這些命令對於我們平時編寫的代碼非常有用，接下來就讓我們瞭解一些常用的命令。

## go build

  這個命令主要用於編譯代碼。在包的編譯過程中，若有必要，會同時編譯與之相關聯的包。

  - 如果是普通包，就像我們在1.2節中編寫的`mymath`包那樣，當你執行`go build`之後，它不會產生任何文件。如果你需要在`$GOPATH/pkg`下生成相應的文件，那就得執行`go install`。

  - 如果是`main`包，當你執行`go build`之後，它就會在當前目錄下生成一個可執行文件。如果你需要在`$GOPATH/bin`下生成相應的文件，需要執行`go install`，或者使用`go build -o 路徑/a.exe`。

  - 如果某個項目文件夾下有多個文件，而你只想編譯某個文件，就可在`go build`之後加上文件名，例如`go build a.go`；`go build`命令默認會編譯當前目錄下的所有go文件。

  - 你也可以指定編譯輸出的文件名。例如1.2節中的`mathapp`應用，我們可以指定`go build -o astaxie.exe`，默認情況是你的package名(非main包)，或者是第一個源文件的文件名(main包)。

  （注：實際上，package名在[Go語言規範](https://golang.org/ref/spec)中指代碼中“package”後使用的名稱，此名稱可以與文件夾名不同。默認生成的可執行文件名是文件夾名。）

  - go build會忽略目錄下以“_”或“.”開頭的go文件。

  - 如果你的源代碼針對不同的操作系統需要不同的處理，那麼你可以根據不同的操作系統後綴來命名文件。例如有一個讀取數組的程序，它對於不同的操作系統可能有如下幾個源文件：

	array_linux.go
	array_darwin.go
	array_windows.go
	array_freebsd.go

  `go build`的時候會選擇性地編譯以系統名結尾的文件（Linux、Darwin、Windows、Freebsd）。例如Linux系統下面編譯只會選擇array_linux.go文件，其它系統命名後綴文件全部忽略。

參數的介紹

- `-o` 指定輸出的文件名，可以帶上路徑，例如 `go build -o a/b/c`
- `-i` 安裝相應的包，編譯+`go install`
- `-a` 更新全部已經是最新的包的，但是對標準包不適用
- `-n` 把需要執行的編譯命令打印出來，但是不執行，這樣就可以很容易的知道底層是如何運行的
- `-p n` 指定可以並行可運行的編譯數目，默認是CPU數目
- `-race` 開啟編譯的時候自動檢測數據競爭的情況，目前只支持64位的機器
- `-v` 打印出來我們正在編譯的包名
- `-work` 打印出來編譯時候的臨時文件夾名稱，並且如果已經存在的話就不要刪除
- `-x` 打印出來執行的命令，其實就是和`-n`的結果類似，只是這個會執行
- `-ccflags 'arg list'` 傳遞參數給5c, 6c, 8c 調用
- `-compiler name` 指定相應的編譯器，gccgo還是gc
- `-gccgoflags 'arg list'` 傳遞參數給gccgo編譯連接調用
- `-gcflags 'arg list'` 傳遞參數給5g, 6g, 8g 調用
- `-installsuffix suffix` 為了和默認的安裝包區別開來，採用這個前綴來重新安裝那些依賴的包，`-race`的時候默認已經是`-installsuffix race`,大家可以通過`-n`命令來驗證
- `-ldflags 'flag list'` 傳遞參數給5l, 6l, 8l 調用
- `-tags 'tag list'` 設置在編譯的時候可以適配的那些tag，詳細的tag限制參考裡面的 [Build Constraints](http://golang.org/pkg/go/build/)

## go clean

  這個命令是用來移除當前源碼包和關聯源碼包裡面編譯生成的文件。這些文件包括

	_obj/            舊的object目錄，由Makefiles遺留
	_test/           舊的test目錄，由Makefiles遺留
	_testmain.go     舊的gotest文件，由Makefiles遺留
	test.out         舊的test記錄，由Makefiles遺留
	build.out        舊的test記錄，由Makefiles遺留
	*.[568ao]        object文件，由Makefiles遺留

	DIR(.exe)        由go build產生
	DIR.test(.exe)   由go test -c產生
	MAINFILE(.exe)   由go build MAINFILE.go產生
	*.so             由 SWIG 產生

  我一般都是利用這個命令清除編譯文件，然後github遞交源碼，在本機測試的時候這些編譯文件都是和系統相關的，但是對於源碼管理來說沒必要。

	$ go clean -i -n
	cd /Users/astaxie/develop/gopath/src/mathapp
	rm -f mathapp mathapp.exe mathapp.test mathapp.test.exe app app.exe
	rm -f /Users/astaxie/develop/gopath/bin/mathapp

參數介紹

- `-i` 清除關聯的安裝的包和可運行文件，也就是通過go install安裝的文件
- `-n` 把需要執行的清除命令打印出來，但是不執行，這樣就可以很容易的知道底層是如何運行的
- `-r` 循環的清除在import中引入的包
- `-x` 打印出來執行的詳細命令，其實就是`-n`打印的執行版本

## go fmt

  有過C/C++經驗的讀者會知道,一些人經常為代碼採取K&R風格還是ANSI風格而爭論不休。在go中，代碼則有標準的風格。由於之前已經有的一些習慣或其它的原因我們常將代碼寫成ANSI風格或者其它更合適自己的格式，這將為人們在閱讀別人的代碼時添加不必要的負擔，所以go強制了代碼格式（比如左大括號必須放在行尾），不按照此格式的代碼將不能編譯通過，為了減少浪費在排版上的時間，go工具集中提供了一個`go fmt`命令 它可以幫你格式化你寫好的代碼文件，使你寫代碼的時候不需要關心格式，你只需要在寫完之後執行`go fmt <文件名>.go`，你的代碼就被修改成了標準格式，但是我平常很少用到這個命令，因為開發工具裡面一般都帶了保存時候自動格式化功能，這個功能其實在底層就是調用了`go fmt`。接下來的一節我將講述兩個工具，這兩個工具都自帶了保存文件時自動化`go fmt`功能。

使用go fmt命令，其實是調用了gofmt，而且需要參數-w，否則格式化結果不會寫入文件。gofmt -w -l src，可以格式化整個項目。

所以go fmt是gofmt的上層一個包裝的命令，我們想要更多的個性化的格式化可以參考 [gofmt](http://golang.org/cmd/gofmt/)

gofmt的參數介紹

- `-l` 顯示那些需要格式化的文件
- `-w` 把改寫後的內容直接寫入到文件中，而不是作為結果打印到標準輸出。
- `-r` 添加形如“a[b:len(a)] -> a[b:]”的重寫規則，方便我們做批量替換
- `-s` 簡化文件中的代碼
- `-d` 顯示格式化前後的diff而不是寫入文件，默認是false
- `-e` 打印所有的語法錯誤到標準輸出。如果不使用此標記，則只會打印不同行的前10個錯誤。
- `-cpuprofile` 支持調試模式，寫入相應的cpufile到指定的文件

## go get

  這個命令是用來動態獲取遠程代碼包的，目前支持的有BitBucket、GitHub、Google Code和Launchpad。這個命令在內部實際上分成了兩步操作：第一步是下載源碼包，第二步是執行`go install`。下載源碼包的go工具會自動根據不同的域名調用不同的源碼工具，對應關係如下：

	BitBucket (Mercurial Git)
	GitHub (Git)
	Google Code Project Hosting (Git, Mercurial, Subversion)
	Launchpad (Bazaar)

  所以為了`go get` 能正常工作，你必須確保安裝了合適的源碼管理工具，並同時把這些命令加入你的PATH中。其實`go get`支持自定義域名的功能，具體參見`go help remote`。

參數介紹：

- `-d` 只下載不安裝
- `-f` 只有在你包含了`-u`參數的時候才有效，不讓`-u`去驗證import中的每一個都已經獲取了，這對於本地fork的包特別有用
- `-fix` 在獲取源碼之後先運行fix，然後再去做其他的事情
- `-t` 同時也下載需要為運行測試所需要的包
- `-u` 強制使用網絡去更新包和它的依賴包
- `-v` 顯示執行的命令

## go install

  這個命令在內部實際上分成了兩步操作：第一步是生成結果文件(可執行文件或者.a包)，第二步會把編譯好的結果移到`$GOPATH/pkg`或者`$GOPATH/bin`。

參數支持`go build`的編譯參數。大家只要記住一個參數`-v`就好了，這個隨時隨地的可以查看底層的執行信息。

## go test

  執行這個命令，會自動讀取源碼目錄下面名為`*_test.go`的文件，生成並運行測試用的可執行文件。輸出的信息類似

	ok   archive/tar   0.011s
	FAIL archive/zip   0.022s
	ok   compress/gzip 0.033s
	...

  默認的情況下，不需要任何的參數，它會自動把你源碼包下面所有test文件測試完畢，當然你也可以帶上參數，詳情請參考`go help testflag`

這裡我介紹幾個我們常用的參數：

- `-bench regexp` 執行相應的benchmarks，例如 `-bench=.`
- `-cover` 開啟測試覆蓋率
- `-run regexp` 只運行regexp匹配的函數，例如 `-run=Array` 那麼就執行包含有Array開頭的函數
- `-v` 顯示測試的詳細命令

## go tool
`go tool`下面下載聚集了很多命令，這裡我們只介紹兩個，fix和vet

- `go tool fix .` 用來修復以前老版本的代碼到新版本，例如go1之前老版本的代碼轉化到go1,例如API的變化
- `go tool vet directory|files` 用來分析當前目錄的代碼是否都是正確的代碼,例如是不是調用fmt.Printf裡面的參數不正確，例如函數裡面提前return瞭然後出現了無用代碼之類的。

## go generate
這個命令是從Go1.4開始才設計的，用於在編譯前自動化生成某類代碼。`go generate`和`go build`是完全不一樣的命令，通過分析源碼中特殊的註釋，然後執行相應的命令。這些命令都是很明確的，沒有任何的依賴在裡面。而且大家在用這個之前心裡面一定要有一個理念，這個`go generate`是給你用的，不是給使用你這個包的人用的，是方便你來生成一些代碼的。

這裡我們來舉一個簡單的例子，例如我們經常會使用`yacc`來生成代碼，那麼我們常用這樣的命令：

	go tool yacc -o gopher.go -p parser gopher.y

-o 指定了輸出的文件名， -p指定了package的名稱，這是一個單獨的命令，如果我們想讓`go generate`來觸發這個命令，那麼就可以在當然目錄的任意一個`xxx.go`文件裡面的任意位置增加一行如下的註釋：

	//go:generate go tool yacc -o gopher.go -p parser gopher.y

這裡我們注意了，`//go:generate`是沒有任何空格的，這其實就是一個固定的格式，在掃描源碼文件的時候就是根據這個來判斷的。

所以我們可以通過如下的命令來生成，編譯，測試。如果`gopher.y`文件有修改，那麼就重新執行`go generate`重新生成文件就好。

	$ go generate
	$ go build
	$ go test


## godoc

在Go1.2版本之前還支持`go doc`命令，但是之後全部移到了godoc這個命令下，需要這樣安裝`go get golang.org/x/tools/cmd/godoc`

  很多人說go不需要任何的第三方文檔，例如chm手冊之類的（其實我已經做了一個了，[chm手冊](https://github.com/astaxie/godoc)），因為它內部就有一個很強大的文檔工具。

  如何查看相應package的文檔呢？
  例如builtin包，那麼執行`godoc builtin`
  如果是http包，那麼執行`godoc net/http`
  查看某一個包裡面的函數，那麼執行`godoc fmt Printf`
  也可以查看相應的代碼，執行`godoc -src fmt Printf`

  通過命令在命令行執行 godoc -http=:端口號 比如`godoc -http=:8080`。然後在瀏覽器中打開`127.0.0.1:8080`，你將會看到一個golang.org的本地copy版本，通過它你可以查詢pkg文檔等其它內容。如果你設置了GOPATH，在pkg分類下，不但會列出標準包的文檔，還會列出你本地`GOPATH`中所有項目的相關文檔，這對於經常被牆的用戶來說是一個不錯的選擇。

## 其它命令

  go還提供了其它很多的工具，例如下面的這些工具

	go version 查看go當前的版本
	go env 查看當前go的環境變量
	go list 列出當前全部安裝的package
	go run 編譯並運行Go程序

以上這些工具還有很多參數沒有一一介紹，用戶可以使用`go help 命令`獲取更詳細的幫助信息。


## links
   * [目錄](<preface.md>)
   * 上一節: [GOPATH與工作空間](<01.2.md>)
   * 下一節: [Go開發工具](<01.4.md>)

---

# 1.4 Go開發工具

本節我將介紹幾個開發工具，它們都具有自動化提示，自動化fmt功能。因為它們都是跨平臺的，所以安裝步驟之類的都是通用的。

## LiteIDE

  LiteIDE是一款專門為Go語言開發的跨平臺輕量級集成開發環境（IDE），由visualfc編寫。

  ![](images/1.4.liteide.png?raw=true)

圖1.4 LiteIDE主界面

**LiteIDE主要特點：**

* 支持主流操作系統
	* Windows 
	* Linux 
	* MacOS X
* Go編譯環境管理和切換
	* 管理和切換多個Go編譯環境
	* 支持Go語言交叉編譯
* 與Go標準一致的項目管理方式
	* 基於GOPATH的包瀏覽器
	* 基於GOPATH的編譯系統
	* 基於GOPATH的Api文檔檢索
* Go語言的編輯支持
	* 類瀏覽器和大綱顯示
	* Gocode(代碼自動完成工具)的完美支持
	* Go語言文檔查看和Api快速檢索
	* 代碼表達式信息顯示`F1`
	* 源代碼定義跳轉支持`F2`
	* Gdb斷點和調試支持
	* gofmt自動格式化支持
* 其他特徵
	* 支持多國語言界面顯示
	* 完全插件體系結構
	* 支持編輯器配色方案
	* 基於Kate的語法顯示支持
	* 基於全文的單詞自動完成
	* 支持鍵盤快捷鍵綁定方案
	* Markdown文檔編輯支持
		* 實時預覽和同步顯示
		* 自定義CSS顯示
		* 可導出HTML和PDF文檔
		* 批量轉換/合併為HTML/PDF文檔

**LiteIDE安裝配置**

* LiteIDE安裝
	* 下載地址 <http://sourceforge.net/projects/liteide/files>
	* 源碼地址 <https://github.com/visualfc/liteide>
	
	首先安裝好Go語言環境，然後根據操作系統下載LiteIDE對應的壓縮文件直接解壓即可使用。

* 編譯環境設置

	根據自身系統要求切換和配置LiteIDE當前使用的環境變量。
	
	以Windows操作系統，64位Go語言為例，
	工具欄的環境配置中選擇win64，點`編輯環境`，進入LiteIDE編輯win64.env文件
	
		GOROOT=c:\go
		GOBIN=
		GOARCH=amd64
		GOOS=windows
		CGO_ENABLED=1
		
		PATH=%GOBIN%;%GOROOT%\bin;%PATH%
		。。。
	
	將其中的`GOROOT=c:\go`修改為當前Go安裝路徑，存盤即可，如果有MinGW64，可以將`c:\MinGW64\bin`加入PATH中以便go調用gcc支持CGO編譯。

	以Linux操作系統，64位Go語言為例，
	工具欄的環境配置中選擇linux64，點`編輯環境`，進入LiteIDE編輯linux64.env文件
	
		GOROOT=$HOME/go
		GOBIN=
		GOARCH=amd64
		GOOS=linux
		CGO_ENABLED=1
		
		PATH=$GOBIN:$GOROOT/bin:$PATH	
		。。。
		
	將其中的`GOROOT=$HOME/go`修改為當前Go安裝路徑，存盤即可。

* GOPATH設置

	Go語言的工具鏈使用GOPATH設置，是Go語言開發的項目路徑列表，在命令行中輸入(在LiteIDE中也可以`Ctrl+,`直接輸入)`go help gopath`快速查看GOPATH文檔。
	
	在LiteIDE中可以方便的查看和設置GOPATH。通過`菜單－查看－GOPATH`設置，可以查看系統中已存在的GOPATH列表，
	同時可根據需要添加項目目錄到自定義GOPATH列表中。

## Sublime Text

  這裡將介紹Sublime Text 2（以下簡稱Sublime）+GoSublime的組合，那麼為什麼選擇這個組合呢？

  - 自動化提示代碼,如下圖所示

	![](images/1.4.sublime1.png?raw=true)

	圖1.5 sublime自動化提示界面

  - 保存的時候自動格式化代碼，讓您編寫的代碼更加美觀，符合Go的標準。
  - 支持項目管理
	
	![](images/1.4.sublime2.png?raw=true)
	
	圖1.6 sublime項目管理界面
	
  - 支持語法高亮
  - Sublime Text 2可免費使用，只是保存次數達到一定數量之後就會提示是否購買，點擊取消繼續用，和正式註冊版本沒有任何區別。


接下來就開始講如何安裝，下載[Sublime](http://www.sublimetext.com/)

  根據自己相應的系統下載相應的版本，然後打開Sublime，對於不熟悉Sublime的同學可以先看一下這篇文章[Sublime Text 2 入門及技巧](http://lucifr.com/139225/sublime-text-2-tricks-and-tips/)

  1. 打開之後安裝 Package Control：Ctrl+` 打開命令行，執行如下代碼：

		import urllib2,os; pf='Package Control.sublime-package'; ipp=sublime.installed_packages_path(); os.makedirs(ipp) if not os.path.exists(ipp) else None; urllib2.install_opener(urllib2.build_opener(urllib2.ProxyHandler())); open(os.path.join(ipp,pf),'wb').write(urllib2.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read()); print 'Please restart Sublime Text to finish installation'

   這個時候重啟一下Sublime，可以發現在在菜單欄多了一個如下的欄目，說明Package Control已經安裝成功了。

  ![](images/1.4.sublime3.png?raw=true)

	圖1.7 sublime包管理


  2. 安裝完之後就可以安裝Sublime的插件了。需安裝GoSublime、SidebarEnhancements和Go Build，安裝插件之後記得重啟Sublime生效，Ctrl+Shift+p打開Package Controll 輸入`pcip`（即“Package Control: Install Package”的縮寫）。

  這個時候看左下角顯示正在讀取包數據，完成之後出現如下界面

  ![](images/1.4.sublime4.png?raw=true)

	圖1.8 sublime安裝插件界面

  這個時候輸入GoSublime，按確定就開始安裝了。同理應用於SidebarEnhancements和Go Build。

  3. 驗證是否安裝成功，你可以打開Sublime，打開main.go，看看語法是不是高亮了，輸入`import`是不是自動化提示了，`import "fmt"`之後，輸入`fmt.`是不是自動化提示有函數了。

  如果已經出現這個提示，那說明你已經安裝完成了，並且完成了自動提示。

  如果沒有出現這樣的提示，一般就是你的`$PATH`沒有配置正確。你可以打開終端，輸入gocode，是不是能夠正確運行，如果不行就說明`$PATH`沒有配置正確。
  (針對XP)有時候在終端能運行成功,但sublime無提示或者編譯解碼錯誤,請安裝sublime text3和convert utf8插件試一試

  4. MacOS下已經設置了$GOROOT, $GOPATH, $GOBIN，還是沒有自動提示怎麼辦。
  
  請在sublime中使用command + 9， 然後輸入env檢查$PATH, GOROOT, $GOPATH, $GOBIN等變量， 如果沒有請採用下面的方法。
  
  首先建立下面的連接， 然後從Terminal中直接啟動sublime
  
  ln -s /Applications/Sublime\ Text\ 2.app/Contents/SharedSupport/bin/subl /usr/local/bin/sublime


## Vim
Vim是從vi發展出來的一個文本編輯器, 代碼補全、編譯及錯誤跳轉等方便編程的功能特別豐富，在程序員中被廣泛使用。

![](images/1.4.vim.png?raw=true)

圖1.9 VIM編輯器自動化提示Go界面

 1. 配置vim高亮顯示

		cp -r $GOROOT/misc/vim/* ~/.vim/

 2. 在~/.vimrc文件中增加語法高亮顯示

		filetype plugin indent on
		syntax on

 3. 安裝[Gocode](https://github.com/nsf/gocode/)

		go get -u github.com/nsf/gocode

	gocode默認安裝到`$GOPATH/bin`下面。

 4. 配置[Gocode](https://github.com/nsf/gocode/)

		~ cd $GOPATH/src/github.com/nsf/gocode/vim
		~ ./update.bash
		~ gocode set propose-builtins true
		propose-builtins true
		~ gocode set lib-path "/home/border/gocode/pkg/linux_amd64"
		lib-path "/home/border/gocode/pkg/linux_amd64"
		~ gocode set
		propose-builtins true
		lib-path "/home/border/gocode/pkg/linux_amd64"

	>gocode set裡面的兩個參數的含意說明：
	>
	>propose-builtins：是否自動提示Go的內置函數、類型和常量，默認為false，不提示。
	>
	>lib-path:默認情況下，gocode只會搜索**$GOPATH/pkg/$GOOS_$GOARCH** 和 **$GOROOT/pkg/$GOOS_$GOARCH**目錄下的包，當然這個設置就是可以設置我們額外的lib能訪問的路徑


 5. 恭喜你，安裝完成，你現在可以使用`:e main.go`體驗一下開發Go的樂趣。

更多VIM 設定, 可參考[鏈接](http://monnand.me/p/vim-golang-environment/zhCN/)

## Emacs
Emacs傳說中的神器，她不僅僅是一個編輯器，它是一個整合環境，或可稱它為集成開發環境，這些功能如讓使用者置身於全功能的操作系統中。

  ![](images/1.4.emacs.png?raw=true)

圖1.10 Emacs編輯Go主界面

1. 配置Emacs高亮顯示

		cp $GOROOT/misc/emacs/* ~/.emacs.d/

2. 安裝[Gocode](https://github.com/nsf/gocode/)

		go get -u github.com/nsf/gocode

	gocode默認安裝到`$GOBIN`裡面下面。

3. 配置[Gocode](https://github.com/nsf/gocode/)


		~ cd $GOPATH/src/github.com/nsf/gocode/emacs
		~ cp go-autocomplete.el ~/.emacs.d/
		~ gocode set propose-builtins true
		propose-builtins true
		~ gocode set lib-path "/home/border/gocode/pkg/linux_amd64" // 換為你自己的路徑
		lib-path "/home/border/gocode/pkg/linux_amd64"
		~ gocode set
		propose-builtins true
		lib-path "/home/border/gocode/pkg/linux_amd64"

4. 需要安裝 [Auto Completion](http://www.emacswiki.org/emacs/AutoComplete)

   下載AutoComplete並解壓

	~ make install DIR=$HOME/.emacs.d/auto-complete

   配置~/.emacs文件

		;;auto-complete
		(require 'auto-complete-config)
		(add-to-list 'ac-dictionary-directories "~/.emacs.d/auto-complete/ac-dict")
		(ac-config-default)
		(local-set-key (kbd "M-/") 'semantic-complete-analyze-inline)
		(local-set-key "." 'semantic-complete-self-insert)
		(local-set-key ">" 'semantic-complete-self-insert)

   詳細信息參考: http://www.emacswiki.org/emacs/AutoComplete

5. 配置.emacs

		;; golang mode
		(require 'go-mode-load)
		(require 'go-autocomplete)
		;; speedbar
		;; (speedbar 1)
		(speedbar-add-supported-extension ".go")
		(add-hook
		'go-mode-hook
		'(lambda ()
			;; gocode
			(auto-complete-mode 1)
			(setq ac-sources '(ac-source-go))
			;; Imenu & Speedbar
			(setq imenu-generic-expression
				'(("type" "^type *\\([^ \t\n\r\f]*\\)" 1)
				("func" "^func *\\(.*\\) {" 1)))
			(imenu-add-to-menubar "Index")
			;; Outline mode
			(make-local-variable 'outline-regexp)
			(setq outline-regexp "//\\.\\|//[^\r\n\f][^\r\n\f]\\|pack\\|func\\|impo\\|cons\\|var.\\|type\\|\t\t*....")
			(outline-minor-mode 1)
			(local-set-key "\M-a" 'outline-previous-visible-heading)
			(local-set-key "\M-e" 'outline-next-visible-heading)
			;; Menu bar
			(require 'easymenu)
			(defconst go-hooked-menu
				'("Go tools"
				["Go run buffer" go t]
				["Go reformat buffer" go-fmt-buffer t]
				["Go check buffer" go-fix-buffer t]))
			(easy-menu-define
				go-added-menu
				(current-local-map)
				"Go tools"
				go-hooked-menu)

			;; Other
			(setq show-trailing-whitespace t)
			))
		;; helper function
		(defun go ()
			"run current buffer"
			(interactive)
			(compile (concat "go run " (buffer-file-name))))

		;; helper function
		(defun go-fmt-buffer ()
			"run gofmt on current buffer"
			(interactive)
			(if buffer-read-only
			(progn
				(ding)
				(message "Buffer is read only"))
			(let ((p (line-number-at-pos))
			(filename (buffer-file-name))
			(old-max-mini-window-height max-mini-window-height))
				(show-all)
				(if (get-buffer "*Go Reformat Errors*")
			(progn
				(delete-windows-on "*Go Reformat Errors*")
				(kill-buffer "*Go Reformat Errors*")))
				(setq max-mini-window-height 1)
				(if (= 0 (shell-command-on-region (point-min) (point-max) "gofmt" "*Go Reformat Output*" nil "*Go Reformat Errors*" t))
			(progn
				(erase-buffer)
				(insert-buffer-substring "*Go Reformat Output*")
				(goto-char (point-min))
				(forward-line (1- p)))
			(with-current-buffer "*Go Reformat Errors*"
			(progn
				(goto-char (point-min))
				(while (re-search-forward "<standard input>" nil t)
				(replace-match filename))
				(goto-char (point-min))
				(compilation-mode))))
				(setq max-mini-window-height old-max-mini-window-height)
				(delete-windows-on "*Go Reformat Output*")
				(kill-buffer "*Go Reformat Output*"))))
		;; helper function
		(defun go-fix-buffer ()
			"run gofix on current buffer"
			(interactive)
			(show-all)
			(shell-command-on-region (point-min) (point-max) "go tool fix -diff"))

6. 恭喜你，你現在可以體驗在神器中開發Go的樂趣。默認speedbar是關閉的，如果打開需要把 ;; (speedbar 1) 前面的註釋去掉，或者也可以通過 *M-x speedbar* 手動開啟。

## Eclipse
Eclipse也是非常常用的開發利器，以下介紹如何使用Eclipse來編寫Go程序。

  ![](images/1.4.eclipse1.png?raw=true)

圖1.11 Eclipse編輯Go的主界面

1. 首先下載並安裝好[Eclipse](http://www.eclipse.org/)

2. 下載[goclipse](https://code.google.com/p/goclipse/)插件

	http://code.google.com/p/goclipse/wiki/InstallationInstructions

3. 下載gocode，用於go的代碼補全提示

	gocode的github地址：

		https://github.com/nsf/gocode

	在windows下要安裝git，通常用[msysgit](https://code.google.com/p/msysgit/)
	
	再在cmd下安裝：
	
		go get -u github.com/nsf/gocode
	
	也可以下載代碼，直接用go build來編譯，會生成gocode.exe

4. 下載[MinGW](http://sourceforge.net/projects/mingw/files/MinGW/)並按要求裝好

5. 配置插件

	Windows->Reference->Go

  (1).配置Go的編譯器

  ![](images/1.4.eclipse2.png?raw=true)

  圖1.12 設置Go的一些基礎信息


  (2).配置Gocode（可選，代碼補全），設置Gocode路徑為之前生成的gocode.exe文件

  ![](images/1.4.eclipse3.png?raw=true)

  圖1.13 設置gocode信息

  (3).配置GDB（可選，做調試用），設置GDB路徑為MingW安裝目錄下的gdb.exe文件

  ![](images/1.4.eclipse4.png?raw=true)
  
  圖1.14 設置GDB信息

6. 測試是否成功

	新建一個go工程，再建立一個hello.go。如下圖：
	
	  ![](images/1.4.eclipse5.png?raw=true)
	
	  圖1.15 新建項目編輯文件
	
	調試如下（要在console中用輸入命令來調試）：
	
	  ![](images/1.4.eclipse6.png?raw=true)
	  
	  圖1.16 調試Go程序

## IntelliJ IDEA
熟悉Java的讀者應該對於idea不陌生，idea是通過一個插件來支持go語言的高亮語法,代碼提示和重構實現。

1. 先下載idea，idea支持多平臺：win,mac,linux，如果有錢就買個正式版，如果不行就使用社區免費版，對於只是開發Go語言來說免費版足夠用了

	![](images/1.4.idea1.png?raw=true)

2. 安裝Go插件，點擊菜單File中的Setting，找到Plugins,點擊,Broswer repo按鈕。國內的用戶可能會報錯，自己解決哈。

	![](images/1.4.idea3.png?raw=true)

3. 這時候會看見很多插件，搜索找到Golang,雙擊,download and install。等到golang那一行後面出現Downloaded標誌後,點OK。

	![](images/1.4.idea4.png?raw=true)
	
	然後點 Apply .這時候IDE會要求你重啟。
	
4. 	重啟完畢後,創建新項目會發現已經可以創建golang項目了：

	![](images/1.4.idea5.png?raw=true)

	下一步,會要求你輸入 go sdk的位置,一般都安裝在C:\Go，linux和mac根據自己的安裝目錄設置，選中目錄確定,就可以了。

## links
   * [目錄](<preface.md>)
   * 上一節: [Go 命令](<01.3.md>)
   * 下一節: [總結](<01.5.md>)

---

# 1.5 總結

這一章中我們主要介紹瞭如何安裝Go，Go可以通過三種方式安裝：源碼安裝、標準包安裝、第三方工具安裝，安裝之後我們需要配置我們的開發環境，然後介紹瞭如何配置本地的`$GOPATH`，通過設置`$GOPATH`之後讀者就可以創建項目，接著介紹瞭如何來進行項目編譯、應用安裝等問題，這些需要用到很多Go命令，所以接著就介紹了一些Go的常用命令工具，包括編譯、安裝、格式化、測試等命令，最後介紹了Go的開發工具，目前有很多Go的開發工具：LiteIDE、sublime、VIM、Emacs、Eclipse、Idea等工具，讀者可以根據自己熟悉的工具進行配置，希望能夠通過方便的工具快速的開發Go應用。

## links
   * [目錄](<preface.md>)
   * 上一節: [Go開發工具](<01.4.md>)
   * 下一章: [Go語言基礎](<02.0.md>)
