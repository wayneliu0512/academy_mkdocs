# Class
 
## Class 的定義與宣告

Class 背後的基本概念是資料的抽象 (**data abstraction**) 與封裝 (**encapsulation**).

### 使用 `typedef` 讓 Class 更易閱讀

```cpp
class Screen {
public:
    // 成員函式的介面
private:
    std::string contents;
    std::string::size_type cursor;
    std::string::size_type height, width;
}
```

```cpp
class Screen {
public:
    typedef std::string::size_type index;
private:
    std::string contents;
    index cursor;
    index height, width;
}
``` 

### 成員函式可以被重載 (Overloaded Member Functions)

```cpp
class Screen {
public:
    typedef std::string::size_type index;

    char get() const { return contents[cursor]; }
    char get(index ht, index wd) const;

private:
    std::string contents;
    index cursor;
    index height, width;
}
```

### 前置宣告
宣告一個 class 但是不定義它，是可以的：

```cpp
class Screen; // 純粹宣告
```

這種宣告稱為**前置宣告** (forward declaration)，指出 `Screen` 代表某個 class 型別。在宣告之後，定義之前，`Screen` 是一個不完整型別 (incomplete type)，只知道 `Screen` 是個型別，不知道它含有什麼成員。

不完整型別只能用來定義 指向這個型別的 pointer 或是 reference，或是用來宣告以此型別為參數的函式。

## 如何使用 `this` pointer
Class `Screen` 就是個好例子，目前為止只有一對 `get()` 函式，很合理的我們該會要加上：
  
- 一對 `set()`，用來指定字元或游標所指字元設值。
- 一對 `move()`，給定兩個索引值，將 `cursor` 移至這個新位置。

理想上，我們會希望用戶可以把這些操作連成一列，成為單獨算式：

```cpp
myScreen.move(4,0).set('#');
```

我們希望上面等價於：

```cpp
myScreen.move(4,0);
myScree.set('#');
```

為了可以在同一個算式中呼叫 `move()` 和 `set()`，每個操作都必須回傳一個 reference 指向當前物件：

```cpp
class Screen {
public:
    Screen& set(char);
    Screen& move(index r, index c);
}

Screen& Screen::set(char c){
    content[cursor] = c;
    return *this;
}

Screen& Screen::move(index r, index c){
    index row = r * width;
    cursor = row + c;
    return *this;
}
```

### `const` 成員函式返回 `*this`
在 `const` 成員函式中，我們不能返回一個指向 class 物件的 reference，`const` 成員函式只能以 `const` reference 的形式返回 `*this`。

假設，我們為 class `Screen` 添加一個 `display()`，邏輯上這個函示應該是 `const` 成員，因為列印 `contents` 並不會改變物件內容。接著看以下程式碼:

```cpp
Screen mySreen;
//這段程式碼會失敗，display() 返回一個 const reference; 然後我們不能夠以一個 const 物件喚起 set()
myScreen.display().set('*');
```

為了解決這個問題，我們利用重載必須定義兩個 `display()`: 一個是 `const` 一個不是。

```cpp
class Screen {
public:
    Screen& display(std::iostream &os) { 
        do_display(os); 
        return *this;
    }

    const Screen& display(std::iostream &os) const {
        do_display(os); 
        return *this;
    }

private:
    void do_display(std::iostream &os) const {
        os << contents;
    }
}
```

現在把 `display()` 放入大算式中, nonconst 版本會被喚起。但是當我們要 `display()` 一個 `const` 物件時，`const` 版本會被喚起:

```cpp
Screen myScreen(5,3);
const Screen blank(5, 3);
myScreen.set('#').display(cout);
```

### 可變的 (Mutable) 成員函式
常常，即使在一個 `const` 成員函式中，我們也希望能夠修改 class 的某個成員。只要以關鍵字 `mutable` 修飾成員變數，就可以達到這樣的目的。

```cpp
class Screen {
public:

private:
    mutable size_t access_ctr;
    void do_display(std::iostream &os) const {
        ++access_ctr; // 計算成員函式被呼叫的次數
        os << contents;
    }
};
```

## 建構式 (Constructors)

### 建構是可以被重載

```cpp
class Sales_item {
public:
    Sales_item(const std::string&);
    Sales_item(std::istream&);
    Sales_item(); // default 建構式
}
```

引數決定使用哪個建構式

```cpp
// 使用 default 建構式
Sales_item empty;
Sales_item primer_3rd("0-201-82470-1");
Sales_item primer_4th(cin);
```

### 建構式初值器 (Constructor Initializer)

```cpp
// 編寫建構式時，推薦使用建構式初值列
Sales_item::Sales_item(const string &book): isbn(book), units_sold(0),
                                            revenue(0.0) {}
```

```cpp
// 合法但執行速度較慢的建構式
// 沒有建構式初值器
Sales_item::Sales_item(const string &book) {
    isbn = book;
    units_sold = 0;
    revenue = 0.0;
}
```

概念上我們認為建構式分兩個階段執行:
1. 初始化階段
2. 一般計算階段

計算階段由建構式本體內的所有述式組成

上面兩版 `Sales_item` 建構式效果相同，唯一不同的是，使用建構式初始器的版本，是真正的**初始化**成員變數，而另一個版本是在建構式本體中為成員變數**賦值**

### 有時候就是需要建構式初值器
舉個例子，以下建構式有誤:

```cpp
class ConstRef{
public:
    ConstRef(int ii);
private:
    int i;
    const int ci;
    int &ri;
};

ConstRef::ConstRef(int ii)
{            // 賦值:
    i = ii;  // 沒問題
    ci = ii; // 錯誤: 不可對一個 const 賦值
    ri = i;  // 錯誤: 賦值給 ri，但後者尚未綁定至某物件
}
```

某些成員變數必須在建構式初值器中被初始化。Class 型別中凡是不帶 default 建構式的成員，`const` 成員，reference 成員，不論型別是什麼，都必須在建構初值器中完成初始化

```cpp
// 正確: 明確的初始化 reference 和 const 成員
ConstRef::ConstRef(int ii): i(ii), ci(i), ri(ii) {}
```

### 成員初始化的順序
初始化順序往往並不重要，然而如果某個成員相依於另一個成員，那麼初始化成員就變得重要了。

考慮以下情況:

```cpp
class X {
    int i;
    int j;
public:
    // 執行期錯誤:因為其實 i 在 j 之前被初始化。
    X(int val): j(val), i(j) {}
};
```

### Default 建構式
當我們定義一個物件卻沒有提供初值時，便會喚起 *default* 建構式。如果建構式的所有參數都有預設引數，那也成為一個 *default* 建構式。

如果 class 沒有定義建構式，編譯器會自動為它生成一個 *default* 建構式。

### 隱式型別轉換
**可被唯一引數喚起**的建構式，本身就定義了從其參數型別至 class 型別的一個隱式轉換。

```cpp
string null_book = "9-999-99999-9";
//沒問題: 建立一個 Sales_item，其中 units_sold 和 revenus 皆為 0，isbn 等於 null_book
item.same_isbn(null_book);
```

這個程式使用 `string` 物件作為 `Sales_item` 中的 `same_isbn()` 函式引數。然而該程式期望獲得一個 `Sales_item` 物件。於是編譯器喚起 `Sales_item` 中**帶一個 string**的建構式，以 `null_book` 生成一個新物件。新生成的 `Sales_item` 物件 (暫時物件) 被傳給 `same_isbn()`。

### 禁止建構式定義出隱式轉換
如果將建構式宣告為 `explicit`，就可以阻止編譯器在需要隱式轉換時喚起那個建構式。

```cpp
class Sales_item {
public:
    explicit Sales_item(const string &book = ""): isbn(book), units_sold(0), revenus(0.0) {}
    explicit Sales_item(std::istream &is);
};
```

加上關鍵字 `explicit` 之後，它們都不再可被用來隱式創建 `Sales_item` 物件。

```cpp
item.same_isbn(null_book); //錯誤: string 建構式是 explicit
item.same_isbn(cin);       //錯誤: istream 建構式是 explicit
```

通常應該將單引數建構式宣告為 `explicit`，除非有很明顯的理由需要定義一個隱式轉換。把建構式宣告為 `explicit` 可避免某些錯誤，而當用戶需要轉換時依然可以明確喚起他。

### 委派建構式 (Delegating Constructors)
C++11 提供了新的設定建構式初始值的功能, 稱為委派建構式, 範例如下:

```cpp
class Sales_data {
public:
    // nondelegating constructor initializes members from corresponding arguments
    Sales_data(std::string s, unsigned cnt, double price): bookNo(s), 
    units_sold(cnt), revenue(cnt*price) {}
    // remaining constructors all delegate to another constructor
    Sales_data(): Sales_data("", 0, 0) {}
    Sales_data(std::string s): Sales_data(s, 0,0) {}
    Sales_data(std::istream &is): Sales_data() { read(is, *this); }
    // other members as before
};

```

## Friends (友元)
有時候，令**特定的某些非成員函式**得以取用 **class 的 private 成員**，而又仍能阻止其他的一般性存取，可帶來很大的方便。

```cpp
class Screen {
    // Window_Mgr 的成員函式可以存取 class Screen 的 private 成分
    friend class Window_Mgr;
    //...
}
```

```cpp
Window_Mgr& Window_Mgr::relocate(Screen::index r, Screen::index c, Screen& s) {
    // 直接取用 height 和 width 沒問題
    s.height += r;
    s.width += c;
    return *this;
}
```

### 讓另一個 Class 的成員函式成為 friend
```cpp
class Screen {
    // Window_Mgr 必須在 class Screen 之前先定義好
    friend Window_Mgr& Window_Mgr::relocate(Screen::index, Screen::index, Screen&);
    //...
};
```

## `static` 成員
一般的所謂 nonstatic 成員變數存在於 class 的每個物件中。然而 `static` 成員卻是獨立於 class 的任何物件而存在；每一個 `static` 成員變數都是 **與 class 相關聯** 而非 **與 object 相關聯**。

### 使用 `static` 成員而不使用 globals 好處

1. `static` 成員名稱落在 class 作用域中，這可以避免和其他 classes 的成員或是和 global 物件發生名稱上的衝突。
2. 厲行封裝。`static` 成員可以是 `private`，global 物件卻不能夠。
3. 閱讀程式碼可以輕鬆看出一個 `static` 成員和其 class 之間有關連性。這可增加程式可讀性。

### 定義 `static` 成員

舉個例子，一個銀行帳戶的簡單 class。每個帳戶都有餘額和戶名，每個帳戶都有利息，但每個帳戶的利率總是相同的。

```cpp
class Account {
public:
    void applyint() { amount += amount * interestRate; }
    static double rate() { return interestRate; }
    static void rate(double); //設定新利率
private:
    std::string owner;
    double amount;
    static double interestRate;
    static double initRate();
};
```

物件裡面並沒有哪一個成員相應於 `static` 成員變數，而是存在唯一的 `interestRate` 物件被 `Account` 的所有物件共用。

### 使用 `static` 成員

```cpp
Account ac1;
Account *ac2 = &ac1;
//以下是呼叫 static rate() 三種等價寫法
double rate;
rate = ac1.rate();
rate = ac2->rate();
rate = Account::rate();
```

### `static` 成員函式不帶 `this`
`static` 成員是 class 的一部分，不是任何物件的一部分。因此 `static` 成員函式沒有 `this` 指標。在 `static` 成員函式中取用 `this` 指標，編譯會錯誤。

由於 `static` 成員不是任何物件的一部分，所以 `static` 成員函式不能宣告為 `const`。 `static` 成員函式也不該宣告為 `virtual`。

### `static` 成員變數
你必須在 class 本體外對 `static` 成員變數定義一次。和一般成員變數不同，`static` 成員變數並非經由 class 建構式來初始化，而是在定義時被初始化。

### `static` 成員不隸屬於 class 物件
因為 `static` 成員獨立於任何物件而存在，所以 `static` 成員變數可以是其所在 class。但 nonstatic 成員變數只能是以 pointer 或 reference 指向其所在的 class:

```cpp
class Bar {
public:
    // ...
private:
    static Bar mem1; //正確
    Bar *mem2;       //正確
    Bar mem3;        //錯誤
};
```

類似情況，`static` 成員變數可以被當作一個預設引數:

```cpp
class Screen {
public:
    Screen& clear(char = bkground);
private:
    static const char bkground;
};
```