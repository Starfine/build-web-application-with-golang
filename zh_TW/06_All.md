# 6 session和數據存儲
Web開發中一個很重要的議題就是如何做好用戶的整個瀏覽過程的控制，因為HTTP協議是無狀態的，所以用戶的每一次請求都是無狀態的，我們不知道在整個Web操作過程中哪些連接與該用戶有關，我們應該如何來解決這個問題呢？Web裡面經典的解決方案是cookie和session，cookie機制是一種客戶端機制，把用戶數據保存在客戶端，而session機制是一種服務器端的機制，服務器使用一種類似於散列表的結構來保存信息，每一個網站訪客都會被分配給一個唯一的標誌符,即sessionID,它的存放形式無非兩種:要麼經過url傳遞,要麼保存在客戶端的cookies裡.當然,你也可以將Session保存到數據庫裡,這樣會更安全,但效率方面會有所下降。

6.1小節裡面講介紹session機制和cookie機制的關係和區別，6.2講解Go語言如何來實現session，裡面講實現一個簡易的session管理器，6.3小節講解如何防止session被劫持的情況，如何有效的保護session。我們知道session其實可以存儲在任何地方，6.3小節裡面實現的session是存儲在內存中的，但是如果我們的應用進一步擴展了，要實現應用的session共享，那麼我們可以把session存儲在數據庫中(memcache或者redis)，6.4小節將詳細的講解如何實現這些功能。

## 目錄
   ![](images/navi6.png?raw=true)

## links
   * [目錄](<preface.md>)
   * 上一章: [第五章總結](<05.7.md>)
   * 下一節: [session和cookie](<06.1.md>)

---

# 6.1 session和cookie
session和cookie是網站瀏覽中較為常見的兩個概念，也是比較難以辨析的兩個概念，但它們在瀏覽需要認證的服務頁面以及頁面統計中卻相當關鍵。我們先來了解一下session和cookie怎麼來的？考慮這樣一個問題：

如何抓取一個訪問受限的網頁？如新浪微博好友的主頁，個人微博頁面等。

顯然，通過瀏覽器，我們可以手動輸入用戶名和密碼來訪問頁面，而所謂的“抓取”，其實就是使用程序來模擬完成同樣的工作，因此我們需要了解“登陸”過程中到底發生了什麼。

當用戶來到微博登陸頁面，輸入用戶名和密碼之後點擊“登錄”後瀏覽器將認證信息POST給遠端的服務器，服務器執行驗證邏輯，如果驗證通過，則瀏覽器會跳轉到登錄用戶的微博首頁，在登錄成功後，服務器如何驗證我們對其他受限制頁面的訪問呢？因為HTTP協議是無狀態的，所以很顯然服務器不可能知道我們已經在上一次的HTTP請求中通過了驗證。當然，最簡單的解決方案就是所有的請求裡面都帶上用戶名和密碼，這樣雖然可行，但大大加重了服務器的負擔（對於每個request都需要到數據庫驗證），也大大降低了用戶體驗(每個頁面都需要重新輸入用戶名密碼，每個頁面都帶有登錄表單)。既然直接在請求中帶上用戶名與密碼不可行，那麼就只有在服務器或客戶端保存一些類似的可以代表身份的信息了，所以就有了cookie與session。

cookie，簡而言之就是在本地計算機保存一些用戶操作的歷史信息（當然包括登錄信息），並在用戶再次訪問該站點時瀏覽器通過HTTP協議將本地cookie內容發送給服務器，從而完成驗證，或繼續上一步操作。

![](images/6.1.cookie2.png?raw=true)

圖6.1 cookie的原理圖

session，簡而言之就是在服務器上保存用戶操作的歷史信息。服務器使用session id來標識session，session id由服務器負責產生，保證隨機性與唯一性，相當於一個隨機密鑰，避免在握手或傳輸中暴露用戶真實密碼。但該方式下，仍然需要將發送請求的客戶端與session進行對應，所以可以藉助cookie機制來獲取客戶端的標識（即session id），也可以通過GET方式將id提交給服務器。

![](images/6.1.session.png?raw=true)

圖6.2 session的原理圖

## cookie
Cookie是由瀏覽器維持的，存儲在客戶端的一小段文本信息，伴隨著用戶請求和頁面在Web服務器和瀏覽器之間傳遞。用戶每次訪問站點時，Web應用程序都可以讀取cookie包含的信息。瀏覽器設置裡面有cookie隱私數據選項，打開它，可以看到很多已訪問網站的cookies，如下圖所示：

![](images/6.1.cookie.png?raw=true)

圖6.3 瀏覽器端保存的cookie信息

cookie是有時間限制的，根據生命期不同分成兩種：會話cookie和持久cookie；

如果不設置過期時間，則表示這個cookie生命週期為從創建到瀏覽器關閉止，只要關閉瀏覽器窗口，cookie就消失了。這種生命期為瀏覽會話期的cookie被稱為會話cookie。會話cookie一般不保存在硬盤上而是保存在內存裡。

如果設置了過期時間(setMaxAge(60*60*24))，瀏覽器就會把cookie保存到硬盤上，關閉後再次打開瀏覽器，這些cookie依然有效直到超過設定的過期時間。存儲在硬盤上的cookie可以在不同的瀏覽器進程間共享，比如兩個IE窗口。而對於保存在內存的cookie，不同的瀏覽器有不同的處理方式。
　　

### Go設置cookie
Go語言中通過net/http包中的SetCookie來設置：

	http.SetCookie(w ResponseWriter, cookie *Cookie)

w表示需要寫入的response，cookie是一個struct，讓我們來看一下cookie對象是怎麼樣的

	type Cookie struct {
		Name       string
		Value      string
		Path       string
		Domain     string
		Expires    time.Time
		RawExpires string

	// MaxAge=0 means no 'Max-Age' attribute specified.
	// MaxAge<0 means delete cookie now, equivalently 'Max-Age: 0'
	// MaxAge>0 means Max-Age attribute present and given in seconds
		MaxAge   int
		Secure   bool
		HttpOnly bool
		Raw      string
		Unparsed []string // Raw text of unparsed attribute-value pairs
	}

我們來看一個例子，如何設置cookie

	expiration := time.Now()
	expiration = expiration.AddDate(1, 0, 0)
	cookie := http.Cookie{Name: "username", Value: "astaxie", Expires: expiration}
	http.SetCookie(w, &cookie)

　　
### Go讀取cookie
上面的例子演示瞭如何設置cookie數據，我們這裡來演示一下如何讀取cookie

	cookie, _ := r.Cookie("username")
	fmt.Fprint(w, cookie)

還有另外一種讀取方式

	for _, cookie := range r.Cookies() {
		fmt.Fprint(w, cookie.Name)
	}

可以看到通過request獲取cookie非常方便。

## session

session，中文經常翻譯為會話，其本來的含義是指有始有終的一系列動作/消息，比如打電話是從拿起電話撥號到掛斷電話這中間的一系列過程可以稱之為一個session。然而當session一詞與網絡協議相關聯時，它又往往隱含了“面向連接”和/或“保持狀態”這樣兩個含義。

session在Web開發環境下的語義又有了新的擴展，它的含義是指一類用來在客戶端與服務器端之間保持狀態的解決方案。有時候Session也用來指這種解決方案的存儲結構。

session機制是一種服務器端的機制，服務器使用一種類似於散列表的結構(也可能就是使用散列表)來保存信息。

但程序需要為某個客戶端的請求創建一個session的時候，服務器首先檢查這個客戶端的請求裡是否包含了一個session標識－稱為session id，如果已經包含一個session id則說明以前已經為此客戶創建過session，服務器就按照session id把這個session檢索出來使用(如果檢索不到，可能會新建一個，這種情況可能出現在服務端已經刪除了該用戶對應的session對象，但用戶人為地在請求的URL後面附加上一個JSESSION的參數)。如果客戶請求不包含session id，則為此客戶創建一個session並且同時生成一個與此session相關聯的session id，這個session id將在本次響應中返回給客戶端保存。

session機制本身並不複雜，然而其實現和配置上的靈活性卻使得具體情況複雜多變。這也要求我們不能把僅僅某一次的經驗或者某一個瀏覽器，服務器的經驗當作普遍適用的。

## 小結

如上文所述，session和cookie的目的相同，都是為了克服http協議無狀態的缺陷，但完成的方法不同。session通過cookie，在客戶端保存session id，而將用戶的其他會話消息保存在服務端的session對象中，與此相對的，cookie需要將所有信息都保存在客戶端。因此cookie存在著一定的安全隱患，例如本地cookie中保存的用戶名密碼被破譯，或cookie被其他網站收集（例如：1. appA主動設置域B cookie，讓域B cookie獲取；2. XSS，在appA上通過javascript獲取document.cookie，並傳遞給自己的appB）。


通過上面的一些簡單介紹我們瞭解了cookie和session的一些基礎知識，知道他們之間的聯繫和區別，做web開發之前，有必要將一些必要知識瞭解清楚，才不會在用到時捉襟見肘，或是在調bug時候如無頭蒼蠅亂轉。接下來的幾小節我們將詳細介紹session相關的知識。

## links
   * [目錄](<preface.md>)
   * 上一節: [session和數據存儲](<06.0.md>)
   * 下一節: [Go如何使用session](<06.2.md>)

---

# 6.2 Go如何使用session
通過上一小節的介紹，我們知道session是在服務器端實現的一種用戶和服務器之間認證的解決方案，目前Go標準包沒有為session提供任何支持，這小節我們將會自己動手來實現go版本的session管理和創建。

## session創建過程
session的基本原理是由服務器為每個會話維護一份信息數據，客戶端和服務端依靠一個全局唯一的標識來訪問這份數據，以達到交互的目的。當用戶訪問Web應用時，服務端程序會隨需要創建session，這個過程可以概括為三個步驟：

- 生成全局唯一標識符（sessionid）；
- 開闢數據存儲空間。一般會在內存中創建相應的數據結構，但這種情況下，系統一旦掉電，所有的會話數據就會丟失，如果是電子商務類網站，這將造成嚴重的後果。所以為了解決這類問題，你可以將會話數據寫到文件裡或存儲在數據庫中，當然這樣會增加I/O開銷，但是它可以實現某種程度的session持久化，也更有利於session的共享；
- 將session的全局唯一標示符發送給客戶端。

以上三個步驟中，最關鍵的是如何發送這個session的唯一標識這一步上。考慮到HTTP協議的定義，數據無非可以放到請求行、頭域或Body裡，所以一般來說會有兩種常用的方式：cookie和URL重寫。

1. Cookie
服務端通過設置Set-cookie頭就可以將session的標識符傳送到客戶端，而客戶端此後的每一次請求都會帶上這個標識符，另外一般包含session信息的cookie會將失效時間設置為0(會話cookie)，即瀏覽器進程有效時間。至於瀏覽器怎麼處理這個0，每個瀏覽器都有自己的方案，但差別都不會太大(一般體現在新建瀏覽器窗口的時候)；
2. URL重寫
所謂URL重寫，就是在返回給用戶的頁面裡的所有的URL後面追加session標識符，這樣用戶在收到響應之後，無論點擊響應頁面裡的哪個鏈接或提交表單，都會自動帶上session標識符，從而就實現了會話的保持。雖然這種做法比較麻煩，但是，如果客戶端禁用了cookie的話，此種方案將會是首選。

## Go實現session管理
通過上面session創建過程的講解，讀者應該對session有了一個大體的認識，但是具體到動態頁面技術裡面，又是怎麼實現session的呢？下面我們將結合session的生命週期（lifecycle），來實現go語言版本的session管理。

### session管理設計
我們知道session管理涉及到如下幾個因素

- 全局session管理器
- 保證sessionid 的全局唯一性
- 為每個客戶關聯一個session
- session 的存儲(可以存儲到內存、文件、數據庫等)
- session 過期處理

接下來我將講解一下我關於session管理的整個設計思路以及相應的go代碼示例：

### Session管理器

定義一個全局的session管理器

	type Manager struct {
		cookieName  string     //private cookiename
		lock        sync.Mutex // protects session
		provider    Provider
		maxlifetime int64
	}

	func NewManager(provideName, cookieName string, maxlifetime int64) (*Manager, error) {
		provider, ok := provides[provideName]
		if !ok {
			return nil, fmt.Errorf("session: unknown provide %q (forgotten import?)", provideName)
		}
		return &Manager{provider: provider, cookieName: cookieName, maxlifetime: maxlifetime}, nil
	}

Go實現整個的流程應該也是這樣的，在main包中創建一個全局的session管理器

	var globalSessions *session.Manager
	//然後在init函數中初始化
	func init() {
		globalSessions, _ = NewManager("memory","gosessionid",3600)
	}

我們知道session是保存在服務器端的數據，它可以以任何的方式存儲，比如存儲在內存、數據庫或者文件中。因此我們抽象出一個Provider接口，用以表徵session管理器底層存儲結構。

	type Provider interface {
		SessionInit(sid string) (Session, error)
		SessionRead(sid string) (Session, error)
		SessionDestroy(sid string) error
		SessionGC(maxLifeTime int64)
	}

- SessionInit函數實現Session的初始化，操作成功則返回此新的Session變量
- SessionRead函數返回sid所代表的Session變量，如果不存在，那麼將以sid為參數調用SessionInit函數創建並返回一個新的Session變量
- SessionDestroy函數用來銷燬sid對應的Session變量
- SessionGC根據maxLifeTime來刪除過期的數據

那麼Session接口需要實現什麼樣的功能呢？有過Web開發經驗的讀者知道，對Session的處理基本就 設置值、讀取值、刪除值以及獲取當前sessionID這四個操作，所以我們的Session接口也就實現這四個操作。

	type Session interface {
		Set(key, value interface{}) error //set session value
		Get(key interface{}) interface{}  //get session value
		Delete(key interface{}) error     //delete session value
		SessionID() string                //back current sessionID
	}

>以上設計思路來源於database/sql/driver，先定義好接口，然後具體的存儲session的結構實現相應的接口並註冊後，相應功能這樣就可以使用了，以下是用來隨需註冊存儲session的結構的Register函數的實現。

	var provides = make(map[string]Provider)

	// Register makes a session provide available by the provided name.
	// If Register is called twice with the same name or if driver is nil,
	// it panics.
	func Register(name string, provider Provider) {
		if provider == nil {
			panic("session: Register provide is nil")
		}
		if _, dup := provides[name]; dup {
			panic("session: Register called twice for provide " + name)
		}
		provides[name] = provider
	}

### 全局唯一的Session ID

Session ID是用來識別訪問Web應用的每一個用戶，因此必須保證它是全局唯一的（GUID），下面代碼展示瞭如何滿足這一需求：

	func (manager *Manager) sessionId() string {
		b := make([]byte, 32)
		if _, err := io.ReadFull(rand.Reader, b); err != nil {
			return ""
		}
		return base64.URLEncoding.EncodeToString(b)
	}

### session創建
我們需要為每個來訪用戶分配或獲取與他相關連的Session，以便後面根據Session信息來驗證操作。SessionStart這個函數就是用來檢測是否已經有某個Session與當前來訪用戶發生了關聯，如果沒有則創建之。

	func (manager *Manager) SessionStart(w http.ResponseWriter, r *http.Request) (session Session) {
		manager.lock.Lock()
		defer manager.lock.Unlock()
		cookie, err := r.Cookie(manager.cookieName)
		if err != nil || cookie.Value == "" {
			sid := manager.sessionId()
			session, _ = manager.provider.SessionInit(sid)
			cookie := http.Cookie{Name: manager.cookieName, Value: url.QueryEscape(sid), Path: "/", HttpOnly: true, MaxAge: int(manager.maxlifetime)}
			http.SetCookie(w, &cookie)
		} else {
			sid, _ := url.QueryUnescape(cookie.Value)
			session, _ = manager.provider.SessionRead(sid)
		}
		return
	}

我們用前面login操作來演示session的運用：

	func login(w http.ResponseWriter, r *http.Request) {
		sess := globalSessions.SessionStart(w, r)
		r.ParseForm()
		if r.Method == "GET" {
			t, _ := template.ParseFiles("login.gtpl")
			w.Header().Set("Content-Type", "text/html")
			t.Execute(w, sess.Get("username"))
		} else {
			sess.Set("username", r.Form["username"])
			http.Redirect(w, r, "/", 302)
		}
	}

### 操作值：設置、讀取和刪除
SessionStart函數返回的是一個滿足Session接口的變量，那麼我們該如何用他來對session數據進行操作呢？

上面的例子中的代碼`session.Get("uid")`已經展示了基本的讀取數據的操作，現在我們再來看一下詳細的操作:

	func count(w http.ResponseWriter, r *http.Request) {
		sess := globalSessions.SessionStart(w, r)
		createtime := sess.Get("createtime")
		if createtime == nil {
			sess.Set("createtime", time.Now().Unix())
		} else if (createtime.(int64) + 360) < (time.Now().Unix()) {
			globalSessions.SessionDestroy(w, r)
			sess = globalSessions.SessionStart(w, r)
		}
		ct := sess.Get("countnum")
		if ct == nil {
			sess.Set("countnum", 1)
		} else {
			sess.Set("countnum", (ct.(int) + 1))
		}
		t, _ := template.ParseFiles("count.gtpl")
		w.Header().Set("Content-Type", "text/html")
		t.Execute(w, sess.Get("countnum"))
	}

通過上面的例子可以看到，Session的操作和操作key/value數據庫類似:Set、Get、Delete等操作

因為Session有過期的概念，所以我們定義了GC操作，當訪問過期時間滿足GC的觸發條件後將會引起GC，但是當我們進行了任意一個session操作，都會對Session實體進行更新，都會觸發對最後訪問時間的修改，這樣當GC的時候就不會誤刪除還在使用的Session實體。

### session重置
我們知道，Web應用中有用戶退出這個操作，那麼當用戶退出應用的時候，我們需要對該用戶的session數據進行銷燬操作，上面的代碼已經演示瞭如何使用session重置操作，下面這個函數就是實現了這個功能：

	//Destroy sessionid
	func (manager *Manager) SessionDestroy(w http.ResponseWriter, r *http.Request){
		cookie, err := r.Cookie(manager.cookieName)
		if err != nil || cookie.Value == "" {
			return
		} else {
			manager.lock.Lock()
			defer manager.lock.Unlock()
			manager.provider.SessionDestroy(cookie.Value)
			expiration := time.Now()
			cookie := http.Cookie{Name: manager.cookieName, Path: "/", HttpOnly: true, Expires: expiration, MaxAge: -1}
			http.SetCookie(w, &cookie)
		}
	}


### session銷燬
我們來看一下Session管理器如何來管理銷燬,只要我們在Main啟動的時候啟動：

	func init() {
		go globalSessions.GC()
	}

	func (manager *Manager) GC() {
		manager.lock.Lock()
		defer manager.lock.Unlock()
		manager.provider.SessionGC(manager.maxlifetime)
		time.AfterFunc(time.Duration(manager.maxlifetime), func() { manager.GC() })
	}

我們可以看到GC充分利用了time包中的定時器功能，當超時`maxLifeTime`之後調用GC函數，這樣就可以保證`maxLifeTime`時間內的session都是可用的，類似的方案也可以用於統計在線用戶數之類的。

## 總結
至此 我們實現了一個用來在Web應用中全局管理Session的SessionManager，定義了用來提供Session存儲實現Provider的接口,下一小節，我們將會通過接口定義來實現一些Provider,供大家參考學習。

## links
   * [目錄](<preface.md>)
   * 上一節: [session和cookie](<06.1.md>)
   * 下一節: [session存儲](<06.3.md>)

---

# 6.3 session存儲
上一節我們介紹了Session管理器的實現原理，定義了存儲session的接口，這小節我們將示例一個基於內存的session存儲接口的實現，其他的存儲方式，讀者可以自行參考示例來實現，內存的實現請看下面的例子代碼

	package memory

	import (
		"container/list"
		"github.com/astaxie/session"
		"sync"
		"time"
	)

	var pder = &Provider{list: list.New()}

	type SessionStore struct {
		sid          string                      //session id唯一標示
		timeAccessed time.Time                   //最後訪問時間
		value        map[interface{}]interface{} //session裡面存儲的值
	}

	func (st *SessionStore) Set(key, value interface{}) error {
		st.value[key] = value
		pder.SessionUpdate(st.sid)
		return nil
	}

	func (st *SessionStore) Get(key interface{}) interface{} {
		pder.SessionUpdate(st.sid)
		if v, ok := st.value[key]; ok {
			return v
		} else {
			return nil
		}
		return nil
	}

	func (st *SessionStore) Delete(key interface{}) error {
		delete(st.value, key)
		pder.SessionUpdate(st.sid)
		return nil
	}

	func (st *SessionStore) SessionID() string {
		return st.sid
	}

	type Provider struct {
		lock     sync.Mutex               //用來鎖
		sessions map[string]*list.Element //用來存儲在內存
		list     *list.List               //用來做gc
	}

	func (pder *Provider) SessionInit(sid string) (session.Session, error) {
		pder.lock.Lock()
		defer pder.lock.Unlock()
		v := make(map[interface{}]interface{}, 0)
		newsess := &SessionStore{sid: sid, timeAccessed: time.Now(), value: v}
		element := pder.list.PushBack(newsess)
		pder.sessions[sid] = element
		return newsess, nil
	}

	func (pder *Provider) SessionRead(sid string) (session.Session, error) {
		if element, ok := pder.sessions[sid]; ok {
			return element.Value.(*SessionStore), nil
		} else {
			sess, err := pder.SessionInit(sid)
			return sess, err
		}
		return nil, nil
	}

	func (pder *Provider) SessionDestroy(sid string) error {
		if element, ok := pder.sessions[sid]; ok {
			delete(pder.sessions, sid)
			pder.list.Remove(element)
			return nil
		}
		return nil
	}

	func (pder *Provider) SessionGC(maxlifetime int64) {
		pder.lock.Lock()
		defer pder.lock.Unlock()

		for {
			element := pder.list.Back()
			if element == nil {
				break
			}
			if (element.Value.(*SessionStore).timeAccessed.Unix() + maxlifetime) < time.Now().Unix() {
				pder.list.Remove(element)
				delete(pder.sessions, element.Value.(*SessionStore).sid)
			} else {
				break
			}
		}
	}

	func (pder *Provider) SessionUpdate(sid string) error {
		pder.lock.Lock()
		defer pder.lock.Unlock()
		if element, ok := pder.sessions[sid]; ok {
			element.Value.(*SessionStore).timeAccessed = time.Now()
			pder.list.MoveToFront(element)
			return nil
		}
		return nil
	}

	func init() {
		pder.sessions = make(map[string]*list.Element, 0)
		session.Register("memory", pder)
	}

上面這個代碼實現了一個內存存儲的session機制。通過init函數註冊到session管理器中。這樣就可以方便的調用了。我們如何來調用該引擎呢？請看下面的代碼

	import (
		"github.com/astaxie/session"
		_ "github.com/astaxie/session/providers/memory"
	)

當import的時候已經執行了memory函數裡面的init函數，這樣就已經註冊到session管理器中，我們就可以使用了，通過如下方式就可以初始化一個session管理器：

	var globalSessions *session.Manager

	//然後在init函數中初始化
	func init() {
		globalSessions, _ = session.NewManager("memory", "gosessionid", 3600)
		go globalSessions.GC()
	}


## links
   * [目錄](<preface.md>)
   * 上一節: [Go如何使用session](<06.2.md>)
   * 下一節: [預防session劫持](<06.4.md>)

---

# 6.4 預防session劫持
session劫持是一種廣泛存在的比較嚴重的安全威脅，在session技術中，客戶端和服務端通過session的標識符來維護會話， 但這個標識符很容易就能被嗅探到，從而被其他人利用.它是中間人攻擊的一種類型。

本節將通過一個實例來演示會話劫持，希望通過這個實例，能讓讀者更好地理解session的本質。
## session劫持過程
我們寫了如下的代碼來展示一個count計數器：

	func count(w http.ResponseWriter, r *http.Request) {
		sess := globalSessions.SessionStart(w, r)
		ct := sess.Get("countnum")
		if ct == nil {
			sess.Set("countnum", 1)
		} else {
			sess.Set("countnum", (ct.(int) + 1))
		}
		t, _ := template.ParseFiles("count.gtpl")
		w.Header().Set("Content-Type", "text/html")
		t.Execute(w, sess.Get("countnum"))
	}


count.gtpl的代碼如下所示：

	Hi. Now count:{{.}}

然後我們在瀏覽器裡面刷新可以看到如下內容：

![](images/6.4.hijack.png?raw=true)

圖6.4 瀏覽器端顯示count數

隨著刷新，數字將不斷增長，當數字顯示為6的時候，打開瀏覽器(以chrome為例）的cookie管理器，可以看到類似如下的信息：


![](images/6.4.cookie.png?raw=true)

圖6.5 獲取瀏覽器端保存的cookie

下面這個步驟最為關鍵: 打開另一個瀏覽器(這裡我打開了firefox瀏覽器),複製chrome地址欄裡的地址到新打開的瀏覽器的地址欄中。然後打開firefox的cookie模擬插件，新建一個cookie，把按上圖中cookie內容原樣在firefox中重建一份:

![](images/6.4.setcookie.png?raw=true)

圖6.6 模擬cookie

回車後，你將看到如下內容：

![](images/6.4.hijacksuccess.png?raw=true)

圖6.7 劫持session成功

可以看到雖然換了瀏覽器，但是我們卻獲得了sessionID，然後模擬了cookie存儲的過程。這個例子是在同一臺計算機上做的，不過即使換用兩臺來做，其結果仍然一樣。此時如果交替點擊兩個瀏覽器裡的鏈接你會發現它們其實操縱的是同一個計數器。不必驚訝，此處firefox盜用了chrome和goserver之間的維持會話的鑰匙，即gosessionid，這是一種類型的“會話劫持”。在goserver看來，它從http請求中得到了一個gosessionid，由於HTTP協議的無狀態性，它無法得知這個gosessionid是從chrome那裡“劫持”來的，它依然會去查找對應的session，並執行相關計算。與此同時 chrome也無法得知自己保持的會話已經被“劫持”。
## session劫持防範
### cookieonly和token
通過上面session劫持的簡單演示可以瞭解到session一旦被其他人劫持，就非常危險，劫持者可以假裝成被劫持者進行很多非法操作。那麼如何有效的防止session劫持呢？

其中一個解決方案就是sessionID的值只允許cookie設置，而不是通過URL重置方式設置，同時設置cookie的httponly為true,這個屬性是設置是否可通過客戶端腳本訪問這個設置的cookie，第一這個可以防止這個cookie被XSS讀取從而引起session劫持，第二cookie設置不會像URL重置方式那麼容易獲取sessionID。

第二步就是在每個請求裡面加上token，實現類似前面章節裡面講的防止form重複遞交類似的功能，我們在每個請求裡面加上一個隱藏的token，然後每次驗證這個token，從而保證用戶的請求都是唯一性。

	h := md5.New()
	salt:="astaxie%^7&8888"
	io.WriteString(h,salt+time.Now().String())
	token:=fmt.Sprintf("%x",h.Sum(nil))
	if r.Form["token"]!=token{
		//提示登錄
	}
	sess.Set("token",token)


### 間隔生成新的SID
還有一個解決方案就是，我們給session額外設置一個創建時間的值，一旦過了一定的時間，我們銷燬這個sessionID，重新生成新的session，這樣可以一定程度上防止session劫持的問題。

	createtime := sess.Get("createtime")
	if createtime == nil {
		sess.Set("createtime", time.Now().Unix())
	} else if (createtime.(int64) + 60) < (time.Now().Unix()) {
		globalSessions.SessionDestroy(w, r)
		sess = globalSessions.SessionStart(w, r)
	}

session啟動後，我們設置了一個值，用於記錄生成sessionID的時間。通過判斷每次請求是否過期(這裡設置了60秒)定期生成新的ID，這樣使得攻擊者獲取有效sessionID的機會大大降低。

上面兩個手段的組合可以在實踐中消除session劫持的風險，一方面，	由於sessionID頻繁改變，使攻擊者難有機會獲取有效的sessionID；另一方面，因為sessionID只能在cookie中傳遞，然後設置了httponly，所以基於URL攻擊的可能性為零，同時被XSS獲取sessionID也不可能。最後，由於我們還設置了MaxAge=0，這樣就相當於session cookie不會留在瀏覽器的歷史記錄裡面。


## links
   * [目錄](<preface.md>)
   * 上一節: [session存儲](<06.3.md>)
   * 下一節: [小結](<06.5.md>)

---

# 6.5 小結
這章我們學習了什麼是session，什麼是cookie，以及他們兩者之間的關係。但是目前Go官方標準包裡面不支持session，所以我們設計了一個session管理器，實現了session從創建到銷燬的整個過程。然後定義了Provider的接口，使得可以支持各種後端的session存儲，然後我們在第三小節裡面介紹瞭如何使用內存存儲來實現session的管理。第四小節我們講解了session劫持的過程，以及我們如何有效的來防止session劫持。通過這一章的講解，希望能夠讓讀者瞭解整個sesison的執行原理以及如何實現，而且是如何更加安全的使用session。
## links
   * [目錄](<preface.md>)
   * 上一節: [session存儲](<06.4.md>)
   * 下一章: [文本處理](<07.0.md>)
