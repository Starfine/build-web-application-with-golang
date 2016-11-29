# 5 訪問數據庫
對許多Web應用程序而言，數據庫都是其核心所在。數據庫幾乎可以用來存儲你想查詢和修改的任何信息，比如用戶信息、產品目錄或者新聞列表等。

Go沒有內置的驅動支持任何的數據庫，但是Go定義了database/sql接口，用戶可以基於驅動接口開發相應數據庫的驅動，5.1小節裡面介紹Go設計的一些驅動，介紹Go是如何設計數據庫驅動接口的。5.2至5.4小節介紹目前使用的比較多的一些關係型數據驅動以及如何使用，5.5小節介紹我自己開發一個ORM庫，基於database/sql標準接口開發的，可以兼容幾乎所有支持database/sql的數據庫驅動，可以方便的使用Go style來進行數據庫操作。

目前NOSQL已經成為Web開發的一個潮流，很多應用採用了NOSQL作為數據庫，而不是以前的緩存，5.6小節將介紹MongoDB和Redis兩種NOSQL數據庫。

>[Go database/sql tutorial](http://go-database-sql.org/) 裡提供了慣用的範例及詳細的說明。

## 目錄
   ![](images/navi5.png?raw=true)

## links
   * [目錄](<preface.md>)
   * 上一章: [第四章總結](<04.6.md>)
   * 下一節: [database/sql接口](<05.1.md>)

---

# 5.1 database/sql接口
Go與PHP不同的地方是Go官方沒有提供數據庫驅動，而是為開發數據庫驅動定義了一些標準接口，開發者可以根據定義的接口來開發相應的數據庫驅動，這樣做有一個好處，只要是按照標準接口開發的代碼， 以後需要遷移數據庫時，不需要任何修改。那麼Go都定義了哪些標準接口呢？讓我們來詳細的分析一下   

## sql.Register
這個存在於database/sql的函數是用來註冊數據庫驅動的，當第三方開發者開發數據庫驅動時，都會實現init函數，在init裡面會調用這個`Register(name string, driver driver.Driver)`完成本驅動的註冊。

我們來看一下mymysql、sqlite3的驅動裡面都是怎麼調用的：

	//https://github.com/mattn/go-sqlite3驅動
	func init() {
		sql.Register("sqlite3", &SQLiteDriver{})
	}

	//https://github.com/mikespook/mymysql驅動
	// Driver automatically registered in database/sql
	var d = Driver{proto: "tcp", raddr: "127.0.0.1:3306"}
	func init() {
		Register("SET NAMES utf8")
		sql.Register("mymysql", &d)
	}

我們看到第三方數據庫驅動都是通過調用這個函數來註冊自己的數據庫驅動名稱以及相應的driver實現。在database/sql內部通過一個map來存儲用戶定義的相應驅動。

	var drivers = make(map[string]driver.Driver)

	drivers[name] = driver

因此通過database/sql的註冊函數可以同時註冊多個數據庫驅動，只要不重複。

>在我們使用database/sql接口和第三方庫的時候經常看到如下:

>		import (
>			"database/sql"
>		 	_ "github.com/mattn/go-sqlite3"
>		)

>新手都會被這個`_`所迷惑，其實這個就是Go設計的巧妙之處，我們在變量賦值的時候經常看到這個符號，它是用來忽略變量賦值的佔位符，那麼包引入用到這個符號也是相似的作用，這兒使用`_`的意思是引入後面的包名而不直接使用這個包中定義的函數，變量等資源。

>我們在2.3節流程和函數一節中介紹過init函數的初始化過程，包在引入的時候會自動調用包的init函數以完成對包的初始化。因此，我們引入上面的數據庫驅動包之後會自動去調用init函數，然後在init函數裡面註冊這個數據庫驅動，這樣我們就可以在接下來的代碼中直接使用這個數據庫驅動了。

## driver.Driver
Driver是一個數據庫驅動的接口，他定義了一個method： Open(name string)，這個方法返回一個數據庫的Conn接口。

	type Driver interface {
		Open(name string) (Conn, error)
	}

返回的Conn只能用來進行一次goroutine的操作，也就是說不能把這個Conn應用於Go的多個goroutine裡面。如下代碼會出現錯誤

	...
	go goroutineA (Conn)  //執行查詢操作
	go goroutineB (Conn)  //執行插入操作
	...

上面這樣的代碼可能會使Go不知道某個操作究竟是由哪個goroutine發起的,從而導致數據混亂，比如可能會把goroutineA裡面執行的查詢操作的結果返回給goroutineB從而使B錯誤地把此結果當成自己執行的插入數據。

第三方驅動都會定義這個函數，它會解析name參數來獲取相關數據庫的連接信息，解析完成後，它將使用此信息來初始化一個Conn並返回它。

## driver.Conn
Conn是一個數據庫連接的接口定義，他定義了一系列方法，這個Conn只能應用在一個goroutine裡面，不能使用在多個goroutine裡面，詳情請參考上面的說明。

	type Conn interface {
		Prepare(query string) (Stmt, error)
		Close() error
		Begin() (Tx, error)
	}

Prepare函數返回與當前連接相關的執行Sql語句的準備狀態，可以進行查詢、刪除等操作。

Close函數關閉當前的連接，執行釋放連接擁有的資源等清理工作。因為驅動實現了database/sql裡面建議的conn pool，所以你不用再去實現緩存conn之類的，這樣會容易引起問題。

Begin函數返回一個代表事務處理的Tx，通過它你可以進行查詢,更新等操作，或者對事務進行回滾、遞交。

## driver.Stmt
Stmt是一種準備好的狀態，和Conn相關聯，而且只能應用於一個goroutine中，不能應用於多個goroutine。

	type Stmt interface {
		Close() error
		NumInput() int
		Exec(args []Value) (Result, error)
		Query(args []Value) (Rows, error)
	}

Close函數關閉當前的鏈接狀態，但是如果當前正在執行query，query還是有效返回rows數據。

NumInput函數返回當前預留參數的個數，當返回>=0時數據庫驅動就會智能檢查調用者的參數。當數據庫驅動包不知道預留參數的時候，返回-1。

Exec函數執行Prepare準備好的sql，傳入參數執行update/insert等操作，返回Result數據

Query函數執行Prepare準備好的sql，傳入需要的參數執行select操作，返回Rows結果集


## driver.Tx
事務處理一般就兩個過程，遞交或者回滾。數據庫驅動裡面也只需要實現這兩個函數就可以

	type Tx interface {
		Commit() error
		Rollback() error
	}

這兩個函數一個用來遞交一個事務，一個用來回滾事務。

## driver.Execer
這是一個Conn可選擇實現的接口

	type Execer interface {
		Exec(query string, args []Value) (Result, error)
	}

如果這個接口沒有定義，那麼在調用DB.Exec,就會首先調用Prepare返回Stmt，然後執行Stmt的Exec，然後關閉Stmt。

## driver.Result
這個是執行Update/Insert等操作返回的結果接口定義

	type Result interface {
		LastInsertId() (int64, error)
		RowsAffected() (int64, error)
	}

LastInsertId函數返回由數據庫執行插入操作得到的自增ID號。

RowsAffected函數返回query操作影響的數據條目數。

## driver.Rows
Rows是執行查詢返回的結果集接口定義

	type Rows interface {
		Columns() []string
		Close() error
		Next(dest []Value) error
	}

Columns函數返回查詢數據庫表的字段信息，這個返回的slice和sql查詢的字段一一對應，而不是返回整個表的所有字段。

Close函數用來關閉Rows迭代器。

Next函數用來返回下一條數據，把數據賦值給dest。dest裡面的元素必須是driver.Value的值除了string，返回的數據裡面所有的string都必須要轉換成[]byte。如果最後沒數據了，Next函數最後返回io.EOF。


## driver.RowsAffected
RowsAffected其實就是一個int64的別名，但是他實現了Result接口，用來底層實現Result的表示方式

	type RowsAffected int64

	func (RowsAffected) LastInsertId() (int64, error)

	func (v RowsAffected) RowsAffected() (int64, error)

## driver.Value
Value其實就是一個空接口，他可以容納任何的數據

	type Value interface{}

drive的Value是驅動必須能夠操作的Value，Value要麼是nil，要麼是下面的任意一種

	int64
	float64
	bool
	[]byte
	string   [*]除了Rows.Next返回的不能是string.
	time.Time

## driver.ValueConverter
ValueConverter接口定義瞭如何把一個普通的值轉化成driver.Value的接口

	type ValueConverter interface {
		ConvertValue(v interface{}) (Value, error)
	}

在開發的數據庫驅動包裡面實現這個接口的函數在很多地方會使用到，這個ValueConverter有很多好處：

- 轉化driver.value到數據庫表相應的字段，例如int64的數據如何轉化成數據庫表uint16字段
- 把數據庫查詢結果轉化成driver.Value值
- 在scan函數裡面如何把driver.Value值轉化成用戶定義的值

## driver.Valuer
Valuer接口定義了返回一個driver.Value的方式

	type Valuer interface {
		Value() (Value, error)
	}

很多類型都實現了這個Value方法，用來自身與driver.Value的轉化。

通過上面的講解，你應該對於驅動的開發有了一個基本的瞭解，一個驅動只要實現了這些接口就能完成增刪查改等基本操作了，剩下的就是與相應的數據庫進行數據交互等細節問題了，在此不再贅述。

## database/sql
database/sql在database/sql/driver提供的接口基礎上定義了一些更高階的方法，用以簡化數據庫操作,同時內部還建議性地實現一個conn pool。

	type DB struct {
		driver 	 driver.Driver
		dsn    	 string
		mu       sync.Mutex // protects freeConn and closed
		freeConn []driver.Conn
		closed   bool
	}

我們可以看到Open函數返回的是DB對象，裡面有一個freeConn，它就是那個簡易的連接池。它的實現相當簡單或者說簡陋，就是當執行Db.prepare的時候會`defer db.putConn(ci, err)`,也就是把這個連接放入連接池，每次調用conn的時候會先判斷freeConn的長度是否大於0，大於0說明有可以複用的conn，直接拿出來用就是了，如果不大於0，則創建一個conn,然後再返回之。


## links
   * [目錄](<preface.md>)
   * 上一節: [訪問數據庫](<05.0.md>)
   * 下一節: [使用MySQL數據庫](<05.2.md>)

---

# 5.2 使用MySQL數據庫
目前Internet上流行的網站構架方式是LAMP，其中的M即MySQL, 作為數據庫，MySQL以免費、開源、使用方便為優勢成為了很多Web開發的後端數據庫存儲引擎。

## MySQL驅動
Go中支持MySQL的驅動目前比較多，有如下幾種，有些是支持database/sql標準，而有些是採用了自己的實現接口,常用的有如下幾種:

- https://github.com/go-sql-driver/mysql  支持database/sql，全部採用go寫。
- https://github.com/ziutek/mymysql   支持database/sql，也支持自定義的接口，全部採用go寫。
- https://github.com/Philio/GoMySQL 不支持database/sql，自定義接口，全部採用go寫。

接下來的例子我主要以第一個驅動為例(我目前項目中也是採用它來驅動)，也推薦大家採用它，主要理由：

- 這個驅動比較新，維護的比較好
- 完全支持database/sql接口
- 支持keepalive，保持長連接,雖然[星星](http://www.mikespook.com)fork的mymysql也支持keepalive，但不是線程安全的，這個從底層就支持了keepalive。

## 示例代碼
接下來的幾個小節裡面我們都將採用同一個數據庫表結構：數據庫test，用戶表userinfo，關聯用戶信息表userdetail。

	CREATE TABLE `userinfo` (
		`uid` INT(10) NOT NULL AUTO_INCREMENT,
		`username` VARCHAR(64) NULL DEFAULT NULL,
		`departname` VARCHAR(64) NULL DEFAULT NULL,
		`created` DATE NULL DEFAULT NULL,
		PRIMARY KEY (`uid`)
	)

	CREATE TABLE `userdetail` (
		`uid` INT(10) NOT NULL DEFAULT '0',
		`intro` TEXT NULL,
		`profile` TEXT NULL,
		PRIMARY KEY (`uid`)
	)

如下示例將示範如何使用database/sql接口對數據庫表進行增刪改查操作

	package main

	import (
		_ "github.com/go-sql-driver/mysql"
		"database/sql"
		"fmt"
		//"time"
	)

	func main() {
		db, err := sql.Open("mysql", "astaxie:astaxie@/test?charset=utf8")
		checkErr(err)

		//插入數據
		stmt, err := db.Prepare("INSERT userinfo SET username=?,departname=?,created=?")
		checkErr(err)

		res, err := stmt.Exec("astaxie", "研發部門", "2012-12-09")
		checkErr(err)

		id, err := res.LastInsertId()
		checkErr(err)

		fmt.Println(id)
		//更新數據
		stmt, err = db.Prepare("update userinfo set username=? where uid=?")
		checkErr(err)

		res, err = stmt.Exec("astaxieupdate", id)
		checkErr(err)

		affect, err := res.RowsAffected()
		checkErr(err)

		fmt.Println(affect)

		//查詢數據
		rows, err := db.Query("SELECT * FROM userinfo")
		checkErr(err)

		for rows.Next() {
			var uid int
			var username string
			var department string
			var created string
			err = rows.Scan(&uid, &username, &department, &created)
			checkErr(err)
			fmt.Println(uid)
			fmt.Println(username)
			fmt.Println(department)
			fmt.Println(created)
		}

		//刪除數據
		stmt, err = db.Prepare("delete from userinfo where uid=?")
		checkErr(err)

		res, err = stmt.Exec(id)
		checkErr(err)

		affect, err = res.RowsAffected()
		checkErr(err)

		fmt.Println(affect)

		db.Close()

	}

	func checkErr(err error) {
		if err != nil {
			panic(err)
		}
	}


通過上面的代碼我們可以看出，Go操作Mysql數據庫是很方便的。

關鍵的幾個函數我解釋一下：

sql.Open()函數用來打開一個註冊過的數據庫驅動，go-sql-driver中註冊了mysql這個數據庫驅動，第二個參數是DSN(Data Source Name)，它是go-sql-driver定義的一些數據庫鏈接和配置信息。它支持如下格式：

	user@unix(/path/to/socket)/dbname?charset=utf8
	user:password@tcp(localhost:5555)/dbname?charset=utf8
	user:password@/dbname
	user:password@tcp([de:ad:be:ef::ca:fe]:80)/dbname

db.Prepare()函數用來返回準備要執行的sql操作，然後返回準備完畢的執行狀態。

db.Query()函數用來直接執行Sql返回Rows結果。

stmt.Exec()函數用來執行stmt準備好的SQL語句

我們可以看到我們傳入的參數都是=?對應的數據，這樣做的方式可以一定程度上防止SQL注入。



## links
   * [目錄](<preface.md>)
   * 上一節: [database/sql接口](<05.1.md>)
   * 下一節: [使用SQLite數據庫](<05.3.md>)

---

# 5.3 使用SQLite數據庫

SQLite 是一個開源的嵌入式關係數據庫，實現自包容、零配置、支持事務的SQL數據庫引擎。其特點是高度便攜、使用方便、結構緊湊、高效、可靠。 與其他數據庫管理系統不同，SQLite 的安裝和運行非常簡單，在大多數情況下,只要確保SQLite的二進制文件存在即可開始創建、連接和使用數據庫。如果您正在尋找一個嵌入式數據庫項目或解決方案，SQLite是絕對值得考慮。SQLite可以是說開源的Access。

## 驅動
Go支持sqlite的驅動也比較多，但是好多都是不支持database/sql接口的

- https://github.com/mattn/go-sqlite3 支持database/sql接口，基於cgo(關於cgo的知識請參看官方文檔或者本書後面的章節)寫的
- https://github.com/feyeleanor/gosqlite3 不支持database/sql接口，基於cgo寫的
- https://github.com/phf/go-sqlite3  不支持database/sql接口，基於cgo寫的

目前支持database/sql的SQLite數據庫驅動只有第一個，我目前也是採用它來開發項目的。採用標準接口有利於以後出現更好的驅動的時候做遷移。

## 實例代碼
示例的數據庫表結構如下所示，相應的建表SQL：

	CREATE TABLE `userinfo` (
		`uid` INTEGER PRIMARY KEY AUTOINCREMENT,
		`username` VARCHAR(64) NULL,
		`departname` VARCHAR(64) NULL,
		`created` DATE NULL
	);

	CREATE TABLE `userdeatail` (
		`uid` INT(10) NULL,
		`intro` TEXT NULL,
		`profile` TEXT NULL,
		PRIMARY KEY (`uid`)
	);

看下面Go程序是如何操作數據庫表數據:增刪改查

	package main

	import (
		"database/sql"
		"fmt"
		"time"
		_ "github.com/mattn/go-sqlite3"
	)

	func main() {
		db, err := sql.Open("sqlite3", "./foo.db")
		checkErr(err)

		//插入數據
		stmt, err := db.Prepare("INSERT INTO userinfo(username, departname, created) values(?,?,?)")
		checkErr(err)

		res, err := stmt.Exec("astaxie", "研發部門", "2012-12-09")
		checkErr(err)

		id, err := res.LastInsertId()
		checkErr(err)

		fmt.Println(id)
		//更新數據
		stmt, err = db.Prepare("update userinfo set username=? where uid=?")
		checkErr(err)

		res, err = stmt.Exec("astaxieupdate", id)
		checkErr(err)

		affect, err := res.RowsAffected()
		checkErr(err)

		fmt.Println(affect)

		//查詢數據
		rows, err := db.Query("SELECT * FROM userinfo")
		checkErr(err)

		for rows.Next() {
			var uid int
			var username string
			var department string
			var created time.Time
			err = rows.Scan(&uid, &username, &department, &created)
			checkErr(err)
			fmt.Println(uid)
			fmt.Println(username)
			fmt.Println(department)
			fmt.Println(created)
		}

		//刪除數據
		stmt, err = db.Prepare("delete from userinfo where uid=?")
		checkErr(err)

		res, err = stmt.Exec(id)
		checkErr(err)

		affect, err = res.RowsAffected()
		checkErr(err)

		fmt.Println(affect)

		db.Close()

	}

	func checkErr(err error) {
		if err != nil {
			panic(err)
		}
	}


我們可以看到上面的代碼和MySQL例子裡面的代碼幾乎是一模一樣的，唯一改變的就是導入的驅動改變了，然後調用`sql.Open`是採用了SQLite的方式打開。


>sqlite管理工具：http://sqliteadmin.orbmu2k.de/

>可以方便的新建數據庫管理。

## links
   * [目錄](<preface.md>)
   * 上一節: [使用MySQL數據庫](<05.2.md>)
   * 下一節: [使用PostgreSQL數據庫](<05.4.md>)

---

# 5.4 使用PostgreSQL數據庫

PostgreSQL 是一個自由的對象-關係數據庫服務器(數據庫管理系統)，它在靈活的 BSD-風格許可證下發行。它提供了相對其他開放源代碼數據庫系統(比如 MySQL 和 Firebird)，和對專有系統比如 Oracle、Sybase、IBM 的 DB2 和 Microsoft SQL Server的一種選擇。

PostgreSQL和MySQL比較，它更加龐大一點，因為它是用來替代Oracle而設計的。所以在企業應用中採用PostgreSQL是一個明智的選擇。

MySQL被Oracle收購之後正在逐步的封閉（自MySQL 5.5.31以後的所有版本將不再遵循GPL協議），鑑於此，將來我們也許會選擇PostgreSQL而不是MySQL作為項目的後端數據庫。

## 驅動
Go實現的支持PostgreSQL的驅動也很多，因為國外很多人在開發中使用了這個數據庫。

- https://github.com/lib/pq 支持database/sql驅動，純Go寫的
- https://github.com/jbarham/gopgsqldriver 支持database/sql驅動，純Go寫的
- https://github.com/lxn/go-pgsql 支持database/sql驅動，純Go寫的

在下面的示例中我採用了第一個驅動，因為它目前使用的人最多，在github上也比較活躍。

## 實例代碼
數據庫建表語句：

	CREATE TABLE userinfo
	(
		uid serial NOT NULL,
		username character varying(100) NOT NULL,
		departname character varying(500) NOT NULL,
		Created date,
		CONSTRAINT userinfo_pkey PRIMARY KEY (uid)
	)
	WITH (OIDS=FALSE);

	CREATE TABLE userdeatail
	(
		uid integer,
		intro character varying(100),
		profile character varying(100)
	)
	WITH(OIDS=FALSE);

看下面這個Go如何操作數據庫表數據:增刪改查

package main

	import (
		"database/sql"
		"fmt"
		_ "https://github.com/lib/pq"
	)

	func main() {
		db, err := sql.Open("postgres", "user=astaxie password=astaxie dbname=test sslmode=disable")
		checkErr(err)

		//插入數據
		stmt, err := db.Prepare("INSERT INTO userinfo(username,departname,created) VALUES($1,$2,$3) RETURNING uid")
		checkErr(err)

		res, err := stmt.Exec("astaxie", "研發部門", "2012-12-09")
		checkErr(err)

		//pg不支持這個函數，因為他沒有類似MySQL的自增ID
		id, err := res.LastInsertId()
		checkErr(err)

		fmt.Println(id)

		//更新數據
		stmt, err = db.Prepare("update userinfo set username=$1 where uid=$2")
		checkErr(err)

		res, err = stmt.Exec("astaxieupdate", 1)
		checkErr(err)

		affect, err := res.RowsAffected()
		checkErr(err)

		fmt.Println(affect)

		//查詢數據
		rows, err := db.Query("SELECT * FROM userinfo")
		checkErr(err)

		for rows.Next() {
			var uid int
			var username string
			var department string
			var created string
			err = rows.Scan(&uid, &username, &department, &created)
			checkErr(err)
			fmt.Println(uid)
			fmt.Println(username)
			fmt.Println(department)
			fmt.Println(created)
		}

		//刪除數據
		stmt, err = db.Prepare("delete from userinfo where uid=$1")
		checkErr(err)

		res, err = stmt.Exec(1)
		checkErr(err)

		affect, err = res.RowsAffected()
		checkErr(err)

		fmt.Println(affect)

		db.Close()

	}

	func checkErr(err error) {
		if err != nil {
			panic(err)
		}
	}

從上面的代碼我們可以看到，PostgreSQL是通過`$1`,`$2`這種方式來指定要傳遞的參數，而不是MySQL中的`?`，另外在sql.Open中的dsn信息的格式也與MySQL的驅動中的dsn格式不一樣，所以在使用時請注意它們的差異。

還有pg不支持LastInsertId函數，因為PostgreSQL內部沒有實現類似MySQL的自增ID返回，其他的代碼幾乎是一模一樣。

## links
   * [目錄](<preface.md>)
   * 上一節: [使用SQLite數據庫](<05.3.md>)
   * 下一節: [使用beedb庫進行ORM開發](<05.5.md>)

---

# 5.5 使用beedb庫進行ORM開發
beedb是我開發的一個Go進行ORM操作的庫，它採用了Go style方式對數據庫進行操作，實現了struct到數據表記錄的映射。beedb是一個十分輕量級的Go ORM框架，開發這個庫的本意降低複雜的ORM學習曲線，儘可能在ORM的運行效率和功能之間尋求一個平衡，beedb是目前開源的Go ORM框架中實現比較完整的一個庫，而且運行效率相當不錯，功能也基本能滿足需求。但是目前還不支持關係關聯，這個是接下來版本升級的重點。

beedb是支持database/sql標準接口的ORM庫，所以理論上來說，只要數據庫驅動支持database/sql接口就可以無縫的接入beedb。目前我測試過的驅動包括下面幾個：

Mysql:github.com/ziutek/mymysql/godrv[*]

Mysql:code.google.com/p/go-mysql-driver[*]

PostgreSQL:github.com/bmizerany/pq[*]

SQLite:github.com/mattn/go-sqlite3[*]

MS ADODB: github.com/mattn/go-adodb[*]

ODBC: bitbucket.org/miquella/mgodbc[*]

## 安裝

beedb支持go get方式安裝，是完全按照Go Style的方式來實現的。

	go get github.com/astaxie/beedb

## 如何初始化
首先你需要import相應的數據庫驅動包、database/sql標準接口包以及beedb包，如下所示：

	import (
		"database/sql"
		"github.com/astaxie/beedb"
		_ "github.com/ziutek/mymysql/godrv"
	)

導入必須的package之後,我們需要打開到數據庫的鏈接，然後創建一個beedb對象（以MySQL為例)，如下所示

	db, err := sql.Open("mymysql", "test/xiemengjun/123456")
	if err != nil {
		panic(err)
	}
	orm := beedb.New(db)

beedb的New函數實際上應該有兩個參數，第一個參數標準接口的db，第二個參數是使用的數據庫引擎，如果你使用的數據庫引擎是MySQL/Sqlite,那麼第二個參數都可以省略。

如果你使用的數據庫是SQLServer，那麼初始化需要：

	orm = beedb.New(db, "mssql")

如果你使用了PostgreSQL，那麼初始化需要：

	orm = beedb.New(db, "pg")

目前beedb支持打印調試，你可以通過如下的代碼實現調試

	beedb.OnDebug=true

接下來我們的例子採用前面的數據庫表Userinfo，現在我們建立相應的struct

	type Userinfo struct {
		Uid     int `PK` //如果表的主鍵不是id，那麼需要加上pk註釋，顯式的說這個字段是主鍵
		Username    string
		Departname  string
		Created     time.Time
	}

>注意一點，beedb針對駝峰命名會自動幫你轉化成下劃線字段，例如你定義了Struct名字為`UserInfo`，那麼轉化成底層實現的時候是`user_info`，字段命名也遵循該規則。

## 插入數據
下面的代碼演示瞭如何插入一條記錄，可以看到我們操作的是struct對象，而不是原生的sql語句，最後通過調用Save接口將數據保存到數據庫。

	var saveone Userinfo
	saveone.Username = "Test Add User"
	saveone.Departname = "Test Add Departname"
	saveone.Created = time.Now()
	orm.Save(&saveone)

我們看到插入之後`saveone.Uid`就是插入成功之後的自增ID。Save接口會自動幫你存進去。

beedb接口提供了另外一種插入的方式，map數據插入。

	add := make(map[string]interface{})
	add["username"] = "astaxie"
	add["departname"] = "cloud develop"
	add["created"] = "2012-12-02"
	orm.SetTable("userinfo").Insert(add)

插入多條數據

	addslice := make([]map[string]interface{}, 0)
	add:=make(map[string]interface{})
	add2:=make(map[string]interface{})
	add["username"] = "astaxie"
	add["departname"] = "cloud develop"
	add["created"] = "2012-12-02"
	add2["username"] = "astaxie2"
	add2["departname"] = "cloud develop2"
	add2["created"] = "2012-12-02"
	addslice =append(addslice, add, add2)
	orm.SetTable("userinfo").InsertBatch(addslice)

上面的操作方式有點類似鏈式查詢，熟悉jquery的同學應該會覺得很親切，每次調用的method都會返回原orm對象，以便可以繼續調用該對象上的其他method。

上面我們調用的SetTable函數是顯式的告訴ORM，我要執行的這個map對應的數據庫表是`userinfo`。

## 更新數據
繼續上面的例子來演示更新操作，現在saveone的主鍵已經有值了，此時調用save接口，beedb內部會自動調用update以進行數據的更新而非插入操作。

	saveone.Username = "Update Username"
	saveone.Departname = "Update Departname"
	saveone.Created = time.Now()
	orm.Save(&saveone)  //現在saveone有了主鍵值，就執行更新操作

更新數據也支持直接使用map操作

	t := make(map[string]interface{})
	t["username"] = "astaxie"
	orm.SetTable("userinfo").SetPK("uid").Where(2).Update(t)

這裡我們調用了幾個beedb的函數

SetPK：顯式的告訴ORM，數據庫表`userinfo`的主鍵是`uid`。

Where:用來設置條件，支持多個參數，第一個參數如果為整數，相當於調用了Where("主鍵=?",值)。
Updata函數接收map類型的數據，執行更新數據。

## 查詢數據
beedb的查詢接口比較靈活，具體使用請看下面的例子

例子1，根據主鍵獲取數據：

	var user Userinfo
	//Where接受兩個參數，支持整形參數
	orm.Where("uid=?", 27).Find(&user)


例子2：

	var user2 Userinfo
	orm.Where(3).Find(&user2) // 這是上面版本的縮寫版，可以省略主鍵

例子3，不是主鍵類型的的條件：

	var user3 Userinfo
	//Where接受兩個參數，支持字符型的參數
	orm.Where("name	 = ?", "john").Find(&user3)
例子4，更加複雜的條件：

	var user4 Userinfo
	//Where支持三個參數
	orm.Where("name = ? and age < ?", "john", 88).Find(&user4)


可以通過如下接口獲取多條數據，請看示例

例子1，根據條件id>3，獲取20位置開始的10條數據的數據

	var allusers []Userinfo
	err := orm.Where("id > ?", "3").Limit(10,20).FindAll(&allusers)

例子2，省略limit第二個參數，默認從0開始，獲取10條數據

	var tenusers []Userinfo
	err := orm.Where("id > ?", "3").Limit(10).FindAll(&tenusers)

例子3，獲取全部數據

	var everyone []Userinfo
	err := orm.OrderBy("uid desc,username asc").FindAll(&everyone)

上面這些裡面裡面我們看到一個函數Limit，他是用來控制查詢結構條數的。

Limit:支持兩個參數，第一個參數表示查詢的條數，第二個參數表示讀取數據的起始位置，默認為0。

OrderBy:這個函數用來進行查詢排序，參數是需要排序的條件。

上面這些例子都是將獲取的的數據直接映射成struct對象，如果我們只是想獲取一些數據到map，以下方式可以實現：

	a, _ := orm.SetTable("userinfo").SetPK("uid").Where(2).Select("uid,username").FindMap()

上面和這個例子裡面又出現了一個新的接口函數Select，這個函數用來指定需要查詢多少個字段。默認為全部字段`*`。

FindMap()函數返回的是`[]map[string][]byte`類型，所以你需要自己作類型轉換。

## 刪除數據
beedb提供了豐富的刪除數據接口，請看下面的例子

例子1，刪除單條數據

	//saveone就是上面示例中的那個saveone
	orm.Delete(&saveone)

例子2，刪除多條數據

	//alluser就是上面定義的獲取多條數據的slice
	orm.DeleteAll(&alluser)

例子3，根據sql刪除數據

	orm.SetTable("userinfo").Where("uid>?", 3).DeleteRow()


## 關聯查詢
目前beedb還不支持struct的關聯關係，但是有些應用卻需要用到連接查詢，所以現在beedb提供了一個簡陋的實現方案：

	a, _ := orm.SetTable("userinfo").Join("LEFT", "userdeatail", "userinfo.uid=userdeatail.uid").Where("userinfo.uid=?", 1).Select("userinfo.uid,userinfo.username,userdeatail.profile").FindMap()

上面代碼中我們看到了一個新的接口Join函數，這個函數帶有三個參數

- 第一個參數可以是：INNER, LEFT, OUTER, CROSS等
- 第二個參數表示連接的表
- 第三個參數表示連接的條件


## Group By和Having
針對有些應用需要用到group by和having的功能，beedb也提供了一個簡陋的實現

	a, _ := orm.SetTable("userinfo").GroupBy("username").Having("username='astaxie'").FindMap()

上面的代碼中出現了兩個新接口函數

GroupBy:用來指定進行groupby的字段

Having:用來指定having執行的時候的條件

## 進一步的發展
目前beedb已經獲得了很多來自國內外用戶的反饋，我目前也正在考慮重構，接下來會在幾個方面進行改進

- 實現interface設計，類似databse/sql/driver的設計，設計beedb的接口，然後去實現相應數據庫的CRUD操作
- 實現關聯數據庫設計，支持一對一，一對多，多對多的實現，示例代碼如下：


	type Profile struct{
		Nickname	string
		Mobile		string
	}

	type Userinfo struct {
		Uid     int `PK`
		Username    string
		Departname  string
		Created     time.Time
		Profile     `HasOne`
	}

- 自動建庫建表建索引
- 實現連接池的實現，採用goroutine

## links
   * [目錄](<preface.md>)
   * 上一節: [使用PostgreSQL數據庫](<05.4.md>)
   * 下一節: [NOSQL數據庫操作](<05.6.md>)

---

# 5.6 NOSQL數據庫操作
NoSQL(Not Only SQL)，指的是非關係型的數據庫。隨著Web2.0的興起，傳統的關係數據庫在應付Web2.0網站，特別是超大規模和高併發的SNS類型的Web2.0純動態網站已經顯得力不從心，暴露了很多難以克服的問題，而非關係型的數據庫則由於其本身的特點得到了非常迅速的發展。

而Go語言作為21世紀的C語言，對NOSQL的支持也是很好，目前流行的NOSQL主要有redis、mongoDB、Cassandra和Membase等。這些數據庫都有高性能、高併發讀寫等特點，目前已經廣泛應用於各種應用中。我接下來主要講解一下redis和mongoDB的操作。

## redis
redis是一個key-value存儲系統。和Memcached類似，它支持存儲的value類型相對更多，包括string(字符串)、list(鏈表)、set(集合)和zset(有序集合)。

目前應用redis最廣泛的應該是新浪微博平臺，其次還有Facebook收購的圖片社交網站instagram。以及其他一些有名的[互聯網企業](http://redis.io/topics/whos-using-redis)

Go目前支持redis的驅動有如下
- https://github.com/alphazero/Go-Redis
- http://code.google.com/p/tideland-rdc/
- https://github.com/simonz05/godis
- https://github.com/hoisie/redis.go

目前我fork了最後一個驅動，更新了一些bug，目前應用在我自己的短域名服務項目中(每天200W左右的PV值)

https://github.com/astaxie/goredis

接下來的以我自己fork的這個redis驅動為例來演示如何進行數據的操作

	package main

	import (
		"github.com/astaxie/goredis"
		"fmt"
	)

	func main() {
		var client goredis.Client
		// 設置端口為redis默認端口
		client.Addr = "127.0.0.1:6379"
		
		//字符串操作
		client.Set("a", []byte("hello"))
		val, _ := client.Get("a")
		fmt.Println(string(val))
		client.Del("a")

		//list操作
		vals := []string{"a", "b", "c", "d", "e"}
		for _, v := range vals {
			client.Rpush("l", []byte(v))
		}
		dbvals,_ := client.Lrange("l", 0, 4)
		for i, v := range dbvals {
			println(i,":",string(v))
		}
		client.Del("l")
	}

我們可以看到操作redis非常的方便，而且我實際項目中應用下來性能也很高。client的命令和redis的命令基本保持一致。所以和原生態操作redis非常類似。

## mongoDB

MongoDB是一個高性能，開源，無模式的文檔型數據庫，是一個介於關係數據庫和非關係數據庫之間的產品，是非關係數據庫當中功能最豐富，最像關係數據庫的。他支持的數據結構非常鬆散，採用的是類似json的bjson格式來存儲數據，因此可以存儲比較複雜的數據類型。Mongo最大的特點是他支持的查詢語言非常強大，其語法有點類似於面向對象的查詢語言，幾乎可以實現類似關係數據庫單表查詢的絕大部分功能，而且還支持對數據建立索引。

下圖展示了mysql和mongoDB之間的對應關係，我們可以看出來非常的方便，但是mongoDB的性能非常好。

![](images/5.6.mongodb.png?raw=true)

圖5.1 MongoDB和Mysql的操作對比圖

目前Go支持mongoDB最好的驅動就是[mgo](http://labix.org/mgo)，這個驅動目前最有可能成為官方的pkg。

下面我將演示如何通過Go來操作mongoDB：

	package main

	import (
		"fmt"
		"labix.org/v2/mgo"
		"labix.org/v2/mgo/bson"
	)

	type Person struct {
		Name string
		Phone string
	}

	func main() {
		session, err := mgo.Dial("server1.example.com,server2.example.com")
		if err != nil {
			panic(err)
		}
		defer session.Close()

		session.SetMode(mgo.Monotonic, true)

		c := session.DB("test").C("people")
		err = c.Insert(&Person{"Ale", "+55 53 8116 9639"},
			&Person{"Cla", "+55 53 8402 8510"})
		if err != nil {
			panic(err)
		}

		result := Person{}
		err = c.Find(bson.M{"name": "Ale"}).One(&result)
		if err != nil {
			panic(err)
		}

		fmt.Println("Phone:", result.Phone)
	}

我們可以看出來mgo的操作方式和beedb的操作方式幾乎類似，都是基於struct的操作方式，這個就是Go Style。



## links
   * [目錄](<preface.md>)
   * 上一節: [使用beedb庫進行ORM開發](<05.5.md>)
   * 下一節: [小結](<05.7.md>)

---

# 5.7 小結
這一章我們講解了Go如何設計database/sql接口，然後介紹了各種第三方關係型數據庫驅動的使用。接著介紹了beedb，一種基於關係型數據庫的ORM庫，如何對數據庫進行簡單的操作。最後介紹了NOSQL的一些知識，目前Go對於NOSQL支持還是不錯，因為Go作為21世紀的C語言，那麼對於21世紀的數據庫也是支持的相當好。

通過這一章的學習，我們學會了如何操作各種數據庫，那麼就解決了我們數據存儲的問題，這是Web裡面最重要的一部分，所以希望大家能夠深入的去了解database/sql的設計思想。

>[Go database/sql tutorial](http://go-database-sql.org/) 裡提供了慣用的範例及詳細的說明。

## links
   * [目錄](<preface.md>)
   * 上一節: [NOSQL數據庫操作](<05.6.md>)
   * 下一章: [session和數據存儲](<06.0.md>)
