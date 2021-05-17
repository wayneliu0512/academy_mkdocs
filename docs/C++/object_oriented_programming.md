# 物件導向編程 (OOP)

## Base Classes 和 Derived Classes

```cpp
class Item_base {
public:
    Item_base(const std::string &book = "", double sales_price = 0.0) :
        isbn(book), price(sales_price) {}
    std::string book() const { return isbn; }
    virtual double net_price(std::size_t n) const { 
        return n * price;
    }
    virtual ~Item_base() {}
private:
    std::string isbn;
protected:
    double price;
}
```

目前只要記住，繼承體系的 root class 一般都會定義一個 virtual 解構式。

`Item_base` class 定義兩個函式，其中一個但有關鍵字 `virtual`，其目的是啟動動態綁定。在預設情況下成員函式並非 virtual。對 non-virtual 函式的呼叫會在編譯時期被決議。

Base class 通常應該把 有必要由 derived class 重新定義 的函式定義為 virtual。

### 存取控制 (Access Control) 與繼承
Derived class 可存取其 base class 的 `public` 成員但不能存取其 `private` 成員。

有時候 base class 的成員會想讓 derived class 存取，但禁止被其他用戶存取。`protected` 存取級別是為此而生。`protected` 成員可被 derived 物件存取，但不能被一般用戶存取。

### Derived Classes (衍生類別)
定義一個 derived class，我們用 class derivation list (衍化列) 標示 base class。也就是列出一或多個 base class 名稱，形式如下:

*class classname : access-label base-class*

其中 *access-label* 是 `public`, `protected`, `private` 三者之一，*base-class* 是先前已定義好的 class 名稱。

```cpp
// 同一本書售出一定數量後可享有折扣
class Bulk_item : public Item_base {
public:
    // 將 base 版本重新定義，用以實現大量購書情況下的折扣策略
    double net_price(std::size_t) const override;
private:
    std::size_t min_qty; // 折扣所需的最低購買量
    double discount;     // 折扣率
}
```

每個 `Bulk_item` 物件含有 4 筆資料元素: 從 `Item_base` 繼承而得的 `isbn` 和 `price`，以及自己定義的 `min_qty` 和 `discount`。`Bulk_item` 還需要定義建構式，後面會提。

### Derived 轉換為 Base

```cpp
double print_total(const Item_base&, size_t);
Item_base item;
print_total(item, 10);

Item_base *p = &item;
Bulk_item bulk;

print_total(bulk, 10);
p = &bulk;
```

這段程式碼以同一個 base-type pointer 在不同時間分別指向 base-type 物件和 derived-type 物件。並且呼叫 接收一個 base-type reference 函式，不同時間分別傳入一個 base-type 物件和一個 derived-type 物件。兩種用法都沒有問題，因為每個 derived 物件都內含其 base 成分。

```cpp
void print_total(ostream &os, const Item_base &item, size_t n) {
    os << "ISBN: " << item.book()
       << "\tnumber sold: " << n << "\ttotal price: "
       // virtual call: 以下呼叫的 net_price() 在執行期才獲得決議
       << item.net_price(n) << endl;
}
```

```cpp
Item_base base;
bulk_item derived;
// print_total() 對 net_price() 進行 virtual call
print_total(cout, base, 10);    // 呼叫 Item_base::net_price
print_total(cout, derived, 10); // 呼叫 Bulk_item::net_price
```

### 關鍵觀念: C++ 多型 (Polymorphism)
Reference 和 Pointer 的靜態型別和動態型別可以不同，這個事實是 C++ 多型的基石。

當我們透過一個 base-class reference 或 pointer 呼叫定義於 base-class 內的函式時，我們並不知道該函式所處裡的物件的精確型別，因為該物件可以是 base type, 也可以是 derived type。

如果呼叫的是 virtual 函式，那麼 喚起哪個函式 的抉擇會延至執行期。

另一方面，物件(非 reference 或 pointer) 不帶多型性 (不是 polymorphic)，因為其型別已知且不可能改變。物件的動態型別永遠和靜態型別相同。

### 推翻虛擬機制 (Overriding Virtual Mechanism)
某種情況下我們會想推翻虛擬機制，強迫喚起 virtual 函式的某個特定版本。*scope* 運算子可以幫我們達到目的。

```cpp
Item_base *baseP = &derived;
// 無論 baseP 的動態型別是什麼，都希望喚起定義於 base class 的版本
double d = baseP->Item_base::net_price(42);
```

### `override` 關鍵字
derived class 如果定義了一個函數與 base class 中的虛擬函式的名子相同但是參數列不同，這仍然是合法行為。編譯器將認為新定義個這個函數與 base class 中原有的函式是相互獨立的。就實際的編程習慣而言，這種聲明往往意味著發生了錯誤，因為我們原本希望 derived class 能覆蓋掉 base class 中的虛擬函式，但是不小心把參數列寫錯了。

```cpp
struct B {
    virtual void f1(int) const;
    virtual void f2();
    void f3();
};

struct D1 : B {
    void f1(int) const override; // 正確: f1與 base class 中的 f1 匹配
    void f2(int) override;       // 錯誤: B 沒有形如 f2(int) 的函式
    void f3() override;          // 錯誤: f3 不是虛擬函式
    void f4() override;          // 錯誤: B 沒有名為 f4 的函式
}
```

### Public, Private, Protected 三種繼承

- `public` 繼承: base 成員的存取級別不變。
- `protected` 繼承: base class 的 `public` 和 `protected` 成員將變成 derived class 的 `protected` 成員。
- `private` 繼承: base class 的所有成員在 derived class 內部變成 private。

### 阻止繼承的發生
有時候我們在創造某個類別時，我們不希望其他類可以繼承它，C++11 提供一種阻止繼承發生的方法，就是在類名後加入關鍵字 `final`。

```cpp
class NoDerived final { /* */ }; // 不能作為基類。

struct D2 : B {
    void f1(int) const final; // 不允許後續的其他類覆蓋 f1(int)
}
```

### Friendship 和 繼承
`friend` class 可以直接存取 class 的 `private` 和 `protected` 資料。

`friend` 關係不會被繼承，base 的 friends 對於 derived classes 的成員並無特殊存取權。

### 繼承與 Static 成員
如果 base class 定義一個 `static` 成員，整個繼承體系也就只定義這麼一個成員。無論這個 base class 衍生多少 classes, `static` 成員只有一份實體。

## 繼承與轉換 (Conversions and Inheritance)
理解 base 與 derived 型別之間的轉換關係，對於理解 物件導向編程在 C++ 如何運作 十分重要。

### 轉換至 Reference 不同於 轉換至 Object
一如先前所見，我們可以傳遞 derived type 物件給預期接收一個 reference to base 的函式。當我們傳遞物件給接收 reference 的函式，該 reference 會被直接綁定於該物件身上。雖然看起來很像是在傳遞物件，其實引數是指向該物件的一個 reference。物件本身並未被複製，也沒有任何轉換動作改變 derived type 物件。

當我們傳遞 derived 物件給預期接收 base-type object (而非 reference) 的函式時，情況大為不同。如果我們傳入 derived type 物件，該物件中的 base 成分會被複製到參數身上。

理解 "derived 物件轉換為 base type reference" 和 "以 derived 物件初始化或賦值 base 物件" 之間的差異，是很重要的。

### Derived-to-Base 轉換的可用性 (Accessibility)
若想判斷是否可轉換至 base，可考慮 base class 的 public 成員可否存取。如果可以，轉換便可進行，否則無法。

### Base 轉換至 Derived
Base class 至 derived class 之間並無自發轉換關係。

```cpp
Item_base base;
Bulk_item* bulkP = &base;  // 錯誤: 無法將 base 轉換為 derived
Bulk_item& bulkRef = base; // 錯誤: 無法將 base 轉換為 derived
Bulk_item bulk = base;     // 錯誤: 無法將 base 轉換為 derived
```

之所以如此，是因為 base 物件就只是一個 base，它並沒有 derived type 成員。

## 建構式和拷貝控制項 (Copy Control)
每個 derived 物件都會擁有 derived class 定義的成員，以及一個 (以上) 的 base class 子物件。這個事實影響了 derived 物件的建構, 拷貝, 賦值, 銷毀。當我們建構, 拷貝, 賦值, 銷毀一個 derived-type 物件，我們也同時建構, 拷貝, 賦值, 銷毀其中的 base class 子物件。

建構式和 copy-control 成員不會被繼承。每個 class 應該定義自己的建構式和 copy-control 成員。任何 class 如果沒有定義自己的 *default* 建構式和 copy-control 成員，就使用編譯器自動合成的版本。

### Base-class 建構式和拷貝控制項 (Copy Control)
自身不是 derived class 的 base class，其建構式和 copy control 多半不受繼承影響，唯一影響的是，當我們決定要提供那些建構式時，必須多考慮一種新用戶，derived class，當需要某些特殊建構式只給 derived class 使用，這種建構式應該設計為  `protected`

### Derived-class 建構式
Derived 建構式受到 "繼承自另一個 class" 的影響。每個 derived 建構式除了初始化自身成員變數外，還得初始化其 base class。

### 合成的 Derived class *default* 建構式
Derived class 的 *default* 建構式只有一個地方不同於 non-derived 建構式。它們不僅初始化 derived class 的成員變數，也初始化其物件的 base 成分。Base 成分由 base class 的 *default* 建構式初始化。

以我們的 `Bulk_item` class 而言，合成的 *default* 建構式會執行以下動作:
1. 喚起 `Item_base` 的 *default* 建構式，將 `isbn` 初始化為空字串，將 `price` 初始化為零。
2. 以變數初始化一班規則完成 `Bulk_item` 成員的初始化。

由於 `Bulk_item` 有內建型成員，我們應該定義自己的 *default* 建構式:
```cpp
class Bulk_item : public Item_base {
public:
    Bulk_item() : min_qty(0), discount(0.0) { }
    // ...
};
```

執行這個建構式的效果如下: 

首先以 `Item_base` 的 *default* 建構式完成 `Item_base` 成分的初始化，將 `isbn` 設為空字串並將 `price` 設為零。然後 `Bulk_item` 成分的成員被初始化，然後執行建構式主體。

### 傳遞引數給 Base class 建構式
Derived class 建構式的初值列 (initialize list) 只能初始化 derived class 成員，不能直接初始化繼承來的成員。derived 建構式必須藉由在建構式初值列含入 base class，才得以間接初始化繼承而來的成員:

```cpp
class Bulk_item : public Item_base {
public:
    Bulk_item(const std::string& book, double sales_price, 
              std::size_t qty = 0, double disc_rate = 0.0) :
              Item_base(book, sales_price), min_qty(0), discount(0.0) { }
    // ...
};
```

這個建構式以雙參數的 `Item_base` 建構式初始化其 base 子物件。我們可以這樣使用這個建構式:

```cpp
Bulk_item bulk("0-201-82470-1", 50, 5, .19);
```

建構式初始列 (Constructor initializer list) 提供 base class 和成員的初值。它們的排列次序並非就是初始化次序。Base class 一定先被初始化，然後 derived class 成員才以其宣告次序被初始化。

### 只有直接繼承的 Base class 可被初始化
如果 class C 衍生自 class B，class B 又衍生自 class A，則 B 是 C 的直接 base。即使每個 C 物件都有個 A 成分，C 的建構式仍然不能直接初始化 A 成分。必須 C 初始化 B，B 建構式在初始化 A。

舉個具體的例子，我們的書便可能有數種折扣策略。除了大量折扣外，也可以在購買一定數量內提供折扣，超過特定數量後便恢復原價。也可以在購買一定數量後才提供折扣，在那之前維持定價。

這些折扣策略的共通點是，都需要一個購買數量和一個折扣率。我們可以定義一個新的 `Disc_item` class 儲存購買量和折扣率，用以支持這些不同策略。

想要實現這個設計，必須先定義 `Disc_item` class:

```cpp
class Disc_item : public Item_base {
public:
    Disc_item(const std::string& book = "", double sales_price = 0.0,
              std::size_t qty = 0, double disc_rate = 0.0) :
              Item_base(book, sales_price), quantity(qty),
              discount(disc_rate) { }
protected:
    std::size_t quantity; // 得享受折扣的臨界購買數
    double discount       // 折扣率
}
```

接下來重新實作 `Bulk_item`，令它繼承自 `Disc_item` 而不再直接繼承 `Item_base`:

```cpp
class Bulk_item : public Disc_item {
public:
    Bulk_item(const std::string& book = "", double sales_price = 0.0, 
              std::size_t qty = 0, double disc_rate = 0.0) :
              Disc_item(book, sales_price, qty, disc_rate) { }
    // 重新定義 base 版本，以實作大量採購的折扣策略
    double net_price(std::size_t) const;
};
```
### 關鍵概念: 遵守 Base class 介面規範
當我們定義 `Disc_item`，我們定義其建構式，指明如何初始化 `Disc_item`。一旦 class 定義其介面，所有與該 class 物件的任何互動都應該透過其介面進行，即便這些物件是 derived 物件的一部分。

基於類似原因，derived class 不能對其 base class 成員進行初始化，也不該對它們賦值。如果這些成員是 `public` 或 `protected`，derived 建構式就語法而言可在函式主體內賦值給其 base class 成員。然而這樣做會違反 base 介面。Derived classes 應該改用建構式，遵從其 base classes 的初始化方式，而非在自己的建構式主體內直接賦值給那些成員。

### *Copy Control* 與繼承
如同其他 class，derived class 可使用編譯器合成的 copy-control 成員。這些合成操作可用來對 "物件的 base 成分和 derived 成分內的成員" 進行拷貝, 賦值, 銷毀。Base 成分是以 base class 的 *copy* 建構式, *assignment* 運算子及解構式分別進行拷貝, 賦值, 銷毀。

### 定義 Derived 的 *Copy* 建構式
如果 derived class 定義自己的 *copy* 建構式，這個建構式通常應該明確使用 base class 的 *copy* 建構式將物件的 base 成分初始化:

```cpp
class Base { /*...*/ }

class Derived : public Base {
public:
    // Base::Base(const Base&) 不會自動被喚起
    Derived(const Derived& d): Base(d) /* 初始化其它成員 */ { /*...*/ }
};
```

上面的 `Base(d)` 會將 derived 物件 `d` 轉換為一個 reference 指向 base 成分，然後喚起 base class 的 *copy* 建構式。如果略而不寫 `Base(d)`:

```cpp
// Derived 的 copy 建構式定義，可能有誤。
Derived(const Derived& d) /* 初始化 Derived 的成員 */ { /*...*/ }
```

上面這樣寫的效果是喚起 `Base` 的 *default* 建構式將 base 成分初始化。假設 `Derived` 成員初始化方式是將 `d` 的對應成員複製過來，那麼新建物件將處於奇怪的狀態:其 Base 成分放置預設值，而其 `Derived` 成員是另一物件的複製品。

### Derived Class 的 *Assignment* 運算子

```cpp
// Base::operator=(const Base&) 不會被自動喚起
Derived &Derived::operator=(const Derived &rhs)
{
    if (this != &rhs) {
        Base::operator=(rhs); // 賦值給 Base 成分
        // 執行 "清理 derived 成分舊值" 所需的任何動作
        // 賦值給 derived 成員
    }
    return *this;
}
```

### Derived Class 的解構式 (Destructor)
解釋的運作方式不同於 *copy* 建構式和 *assignment* 運算子: 它不必負責摧毀其 base 物件成員。編譯器總是會隱寓喚起 derived 物件的 base 成分的解構式。每個解構式只需要清理自己的成員:

```cpp
class Derived: public Base {
public:
    // Base::~Base 會被自動喚起
    ~Derived() { /* 清理 derived 成員 */ }
}
```

物件的銷毀將以 "建構之相反次序" 進行，也就是 derived 解構式先執行，然後喚起 base 解構式，並向上回朔整個繼承體系。

### Virtual 解構式
"Base 解構式會自動被喚起"，此一事實對 base class 的設計帶來重要影響。

當我們對著一個指向動態配置物的 pointer 施行 `delete` 時，該物件的記憶體被釋放前，解構式會先起來清理物件。處理繼承體系的物件時，pointer 的靜態型別可能不同於正被刪除的物件的動態型別。我們可能施行 `delete` 於一個 pointer to base-type，而後者實際指向一個 derived 物件。

如果我們對 pointer to base 施行 `delete`，那麼 base class 解構式會被喚起，負責清理 base 成員。如果所指物件實際上是 derived 型別，則上述行為將不明確 (undefined)。若要保證喚起適當之解構式，base class的解構式一定必須是 virtual:

```cpp
class Item_base {
public:
    virtual ~Item_base() {  }
}
```

如果解構式是 virtual，當它經由 pointer 被喚起，執行起來的解構式會依 pointer 所指物的型別而不同:

```cpp
Item_base *itemP = new Item_base; // 靜態型別和動態型別相同
delete itemP;                     // 沒問題: 呼叫 Item_base 的解構式
itemP = new Bulk_item;            // 沒問題: 靜態型別和動態型別不同
delete itemP;                     // 沒問題: 呼叫 Bulk_item 的解構式
```

重點: 繼承體系的 root class 應該定義一個 virtual 解構式，即使該解構式無事可做。

### 建構式和 *Assignment* 運算子不是 virtual
將 *assignment* 運算子設為 virtual 可能會引起疑惑，因為 virtual 函式在 base class 和 derived class 內必須擁有相同的參數型別。Base class 的 *assignment* 運算子接收的參數是個 reference 指向自身 class 型別。如果此運算子為 virtual，那麼每個 derived classes 都將獲得一個 virtual 成員，定義出 "以 base 物件為參數" 的 `operator=`，但那並不等同於 derived class 的 *assignment* 運算子。

### 當 Virtuals 出現在建構式和解構式內
重點: 如果在建構或解構式中呼叫 virtual 函式，喚起的將是建構或解構式所在型別所定義的版本。

為了要了解上述行為，試考慮 virtual 函式的 derived class 版本被 base class 建構式(或解構式)呼叫。此 virtual 函式的 derived 版本可能存取 derived 物件成員，畢竟如果 derived class 版本不需要使用 derived 物件成員，這個 derived class 可直接使用 base class 的 virtual 函式。然而當 base 建構式 (或解構式) 執行時，物件的衍生部的成員並未初始化。現實中如果允許這種存取動作，程式有可能當掉。

## 繼承下的 Class 作用域
繼承機制使得 class 作用域被階層式地嵌套堆置，當我們寫:

```cpp
Bulk_item bulk;
cout << bulk.book();
```

對於名稱 `book` 的使用將這麼決議 (resolved):
1. `bulk` 是個 `Bulk_item` 物件。編譯器於是在 `Bulk_item` class 內搜尋 `book`，但找不到。
2. 由於 `Bulk_item` 衍生自 `Item_base`，編譯器於是搜尋 `Item_base` class，找到了名稱 `book`，決議成功。

### 名稱衝突 (Name Collision) 和繼承
重點: Derived class 成員如果與 base class 成員同名，會對後者產生遮蔽效果，使後者無法被直接存取。

```cpp
struct Base {
    Base(): mem(0) {}
protected:
    int mem; // 會被遮蔽
};

struct Derived : Base {
    Derived(int i): mem(i) { }
    int get_mem() { return mem; }
protected:
    int mem;
};
```

一旦在 `get_mem()` 中指涉 `mem`，會被編譯器視為 `Derived` 內的名稱。

```cpp
Derived d(42);
cout << g.get_mem() << endl; // 印出 42
```

### 作用域 (scope) 與成員函式
如果 base 和 derived class 擁有相同名稱的成員函式，其運作方式如同成員變數:
derived 作用域內的成員會遮蔽 base class 的成員。

```cpp
struct Base {
    int memfcn();
};

struct Derived : Base {
    int memfcn(int);
}
Derived d;
Base b;
b.memfcn();       // 呼叫 Base::memfcn
d.memfcn(10);     // 呼叫 Derived::memfcn
d.memfcn();       // 錯誤 : 無引數的 memfcn 已被遮蔽
d.Base::memfcn(); // 沒問題 : 呼叫 Base::memfcn
```

重要觀念: 回憶一下，宣告於 local 作用域的函式，並不會與定義於 global 作用域的函式形成重載。同樣道理，定義於 derived class 的函式不會與 base 定義的函式形成重載。

#### 關鍵概念: 名稱搜尋 (Name Lookup) 和繼承:
了解 "函式呼叫如何被決議" 對於了解 C++ 繼承體系十分重要。決議動作依以下四步驟進行:
1. 首先決定呼叫者 (object, reference 或 pointer) 的靜態型別。
2. 在上述 class 內搜尋該函式。如果沒有找到，就往最接近的 (immediate) base class 尋找，如此上朔 classes 繼承鏈，直到發現該函式，或直至搜尋到最後一個 class。如果都沒有發現，表示呼叫有誤。
3. 一旦找到該名稱，就根據找到的定義式進行一般的型別檢驗，檢查呼叫動作是否合法。
4. 若呼叫合法，編譯器變產出二進制碼。如果函式是 virtual，且係透過 reference 或 pointer 被呼叫，那麼編譯器產生的碼會根據物件的動態型別決定喚起哪個版本。否則編譯器產生的碼會直接喚起該函式。

## Pure Virtual (純虛擬) 函式
之前所寫的 `Disc_item` class 呈現一個有趣的問題: 這個 class 從 `Item_base` 繼承到 `net_price()`，但並未重新定義，因為沒必要。這個 class 的存在只是為了讓其他 classes 繼承。

```cpp
class Disc_item : public Item_base {
public:
    Disc_item(const std::string& book = "", double sales_price = 0.0,
              std::size_t qty = 0, double disc_rate = 0.0) :
              Item_base(book, sales_price), quantity(qty),
              discount(disc_rate) { }
protected:
    std::size_t quantity; // 得享受折扣的臨界購買數
    double discount;      // 折扣率
}
```

```cpp
class Bulk_item : public Disc_item {
public:
    Bulk_item(const std::string& book = "", double sales_price = 0.0, 
              std::size_t qty = 0, double disc_rate = 0.0) :
              Disc_item(book, sales_price, qty, disc_rate) { }
    // 重新定義 base 版本，以實作大量採購的折扣策略
    double net_price(std::size_t) const;
};
```

我們不希望用戶創建任何 `Disc_item` 物件。然而我們無法阻止用戶定義一個 `Disc_item` 物件。如果用戶真的建立一個 `Disc_item` 物件並喚起 `net_price()`，會計算出未打折的價格。

結論是我們根本不希望用戶建立這種物件，如果將 `net_price()` 設為 pure virtual 函式就可以達到目的。

```cpp
class Disc_item : public Item_base {
public:
    double net_price(std::size_t) const = 0;
};
```

這裡將 virtual 函式定義為 pure virtual，意味此函式提供一個介面，可讓後續 (衍生) 型別覆寫，而此 class 內的版本永遠不會被喚起。而且用戶無法建立 `Disc_item` 物件。

#### 重點:
如果 class 內含 (或繼承) 一個以上的 pure virtual 函式，便稱為所謂抽象的 (abstract) base class。我們不能為 abstract 型別建立任何物件。

```cpp
// Disc_item 內宣告有 pure virtual 函式
Disc_item discounted; // 錯誤: 不能定義 (建立) Disc_item 物件
Bulk_item bulk;       // 沒問題: Bulk_item 內含 Disc_item 子物件
```