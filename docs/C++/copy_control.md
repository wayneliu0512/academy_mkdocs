# 拷貝控制項 (Copy Control)

定義一個新型別時，我們可以明白或是隱晦 (隱式或顯式) 的指出此型物件被複製 (copied)，被賦值 (assigned)，被銷毀 (destoryed)時會發生什麼事。只要定義這些特殊成員函式: *copy* 建構式，*assignment* 運算子，解構式 就可以完成控制。

- *copy* 建構式 (copy constructor)
- 解構式 (destructor)
- *assignment* 運算子
  
以上合稱 Copy Control (拷貝控制項)，如果沒有明確定義這些特殊成員函式，編譯器往往會幫我們定義一份，但 classes 也可以定義自己的版本。

編譯器合成的 *copy-control* 函式常常是好的。但是對於某些 classes，倚賴這些預設定義可能會導致災難。實作這些 *copy-control* 操作的最困難的部分是: 分辨什麼時候需要改寫這些預設版本。最常見的情況是: 當 class 擁有 pointer 成員，它就應該要定義自己的 *copy-control* 成員。

## *Copy* 建構式
這種建構式用來:
- 以同型物件初始化另一個物件。
- 複製 (拷貝) 一個物件，將復件做為引數傳給一個函式。
- 複製一個物件，從函式中返回復件。
- 在循序容器中將元素初始化。
- 根據一系列元素出值設定值來初始化 array 內的元素。

### 物件定義式 (Object Definition) 的形式
C++ 支援兩種初始化形式: 直接形式和拷貝形式。拷貝初始化 (copy-initialization) 使用符號 `=`，直接初始化 (direct-initialization) 則是把初值放在小括號裡。

```cpp
string null_book = "9-999-99999-9"; // copy-initialization
string dots(10, '.');               // direct-initialization
string empty_direct;                // direct-initialization
```

這兩種形式用在 classes 物件身上有些微的不同。直接初始化會直接喚起與引數吻合的建構式。拷貝初始化總是喚起 *copy* 建構式。拷貝初始化首先使用被指出的建構式創建一個暫時物件，然後再使用 *copy* 建構式把暫時物件拷貝到新物件。

### 參數和返回值
我們已經知道，如果參數不是 reference 型別，引數會被拷貝傳遞。同樣道理，返回值如果不是 reference 型別，`return` 數據中的值會先被複製在被返回。

當參數或返回型別是 class 型別，拷貝行為就由 *copy* 建構式完成。

```cpp
string make_plural(const string&, const string&);
```

函式返回值不是 reference，所以會使用 *copy* 建構式返回給定值。
參數都是 `const` reference，因此不會發生拷貝行為。

### 將容器元素初始化
*copy* 建構式可用來初始化循序是容器的元素。

```cpp
// 喚起 string 的 default 建構式 1 次，和 string 的 copy 建構式 5 次
vector<string> svec(5);
```

### 建構式與 array 元素
如果我們以 大括弧圍住的 array 初值列 提供明確的元素初值，那麼每個元素都會採取拷貝初始化。

```cpp
Sales_item primer_eds[] = { string("0-201-16487-6"),
                            string("0-201-54848-8"),
                            string("0-201-82470-1"),
                            Sales_item()
                          }
```

### 定義 *Copy* 建構式

```cpp
Sales_item {
    Sales_item(const Sales_item &);
private:
    std::string isbn;
    int units_sold;
    double revenue;
};

Sales_item::Sales_item(const Sales_item &orig) : 
    isbn(orig.isbn),             // 使用 string 的 copy 建構式
    units_sold(orig.units_sold), // 複製 orig.units_sold
    revenue(orig.revenue)        // 複製 orig.revenue
    { }
```

對很多 classes 而言，合成的 *copy* 建構式已經除夠完成所有必要工作。然而有些 classes 有必要控制複製物件的行為。這些 classes 一般都有 pointer 成員。

### 阻止拷貝 (Preventing Copies)
某些 classes 需要阻止被拷貝，那就不該寫出 *copy* 建構式，然而如果我們不定義 *copy* 建構式，編譯器會自動幫我們合成。

為了阻止拷貝，有以下兩種方式:
- 將 *copy* 建構式宣告為 `private`。
- 將 *copy* 建構式 ` = delete`。

```cpp
Sales_item {
    // ...
private:
    Sales_item(const Sales_item &);
};
```

```cpp
Sales_item {
    Sales_item(const Sales_item &) = delete;
private:
    // ...
};
```

## 賦值 (Assignment) 運算子
就像 classes 可以控制物件初始化一樣，他們也可以定義 當賦值操作出現時會發生什麼事:

```cpp
Sales_item trans, accum;
trans = accum;
```

和 *copy* 建構式一樣，如果 class 沒有定義自己的 *assignment* 運算子，編譯器會自動合成一個。

例如 `Sales_item` 的 *assignment* 運算子可以宣告為:

```cpp
class Sales_item {
public:
    // 以下等價於編譯器合成的 assignment 運算子
    Sales_item& operator=(const Sales_item &);
}
```

### 合成的 *Assignment* 運算子
合成的 *assignment* 運算子與合成的 *copy* 建構子操作形式相似。

```cpp
Sales_item& Sales_item::operator=(const Sales_item &rhs)
{
    isbn = rhs.isbn;             // 喚起 string::operator=
    units_sold = rhs.units_sold; // 使用內建的 int 賦值操作
    revenue = rhs.revenue;       // 使用內建的 double 賦值操作
    return *this;
}
```

### *Copy* 和 *Assign* 往往結伴而行
事實上，這兩個操作可被視為一體。如果我們需要其中一個，幾乎就可以確定也需要另一個。

## 解構式 (Destructor)
解構式是一種特殊的成員函式，可以用來解除任何資源配置。他就像建構式的互補操作。

### 什麼時候呼叫解構式
當 class 物件被銷毀 (destroyed)，解構式就會被編譯器自動喚起。

### 何時才需明白寫出一個解構式
一個有著建構式的 class 不一定需要解構式。當有需要放棄建構式中或物件生命中曾經索取的資源時，我們才定義建構式。

一個很有用的原則是，如果 class 需要解構式，他也會需要 *assignment* 運算子和 *copy* 建構式。這個規則常被稱為 **Rule of Three**。也就是說如果你需要一個解構式，你也將需要所有三個 *copy-control* (拷貝控制項) 成員。

### 合成的解構式
不同於 *copy* 建構式或 *assignment* 運算子，編譯器無論如何總是會為我們合成一個解構式。這個合成版本會按物件創建時的相反順序銷毀每一個 non`static` 成員，但是合成版解構式並不會將 pointer 成員所指向的物件 `delete` 掉。

### 移動建構子(Move contructor)
C++11 新功能，目前不會用到，待續...

### 訊息處理範例 (A Message-Handling Example)
待續...

## QObject No Copy Constructor or Assignment Operator

QObject has neither a copy constructor nor an assignment operator. This is by design. Actually, they are declared, but in a private section with the macro `Q_DISABLE_COPY()`. In fact, all Qt classes derived from QObject (direct or indirect) use this macro to declare their copy constructor and assignment operator to be private. The reasoning is found in the discussion on Identity vs Value on the Qt Object Model page.

The main consequence is that you should use pointers to QObject (or to your QObject subclass) where you might otherwise be tempted to use your QObject subclass as a value. For example, without a copy constructor, you can't use a subclass of QObject as the value to be stored in one of the container classes. You must store pointers.