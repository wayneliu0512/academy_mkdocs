# Template and Generic Programming

所謂泛型編程 (generic programming)，意味以 "獨立於任何特定目標型別" 的方式撰寫程式。使用這種程式時用戶必須提供欲操作的目標型會目標值。

泛型編程 (GP)，一如物件導向編程 (OOP)，亦倚賴某種形式的多型。OOP 的多型乃是在執行期施行 "以繼承關係相連的 classes"。泛型編程則是讓我們所寫的 classes 和 functions 在編譯期於彼此不相干的目標型別之間展現多型。單一 class 或 function 可用來操控各型物件。C++ 標準庫的容器，疊代器和演算法都是泛型編程的極佳例子。

Template (模板) 是泛型編程的基石。所謂 template 是 "建立 class 或 function 的藍圖或方程式"。

例如 C++ 標準庫定義了一個 class template 表現出 `vector` 意義，這個 template 可被用來產生任意數量的 "某目標型別專屬" 的 `vector` classes，像是 `vector<int>` 或 `vector<string>`。

### Template 參數列

```cpp
template <typename T>
int compare(const T &v1, const T &v2) {
    if (v1 < v2)
        return -1;
    if (v2 < v1)
        return 1;
    return 0;
}
```

Template 參數代表 "可在 class 或 function 定義式中使用的型別或是數值"。例如我們的 `compare` 函式宣告了一個 `T` 的型別參數。`compare` 定義式內可以使用 `T`指稱某個型別。至於 `T` 所代表的實際型別由編譯器根據用戶指定而定。

Template 參數可以是代表型別的所謂 ***type* 參數**，或是代表常數算式的所謂 ***nontype* 參數**。

### 使用 Function Template
使用 function template 時，編譯器會推導將尚未確定的 **template 引數**綁定於 template 參數上。一旦判別出 template 引數，編譯器就為我們將 **function template 具現 (instantiate)** 出一份實體。

推導出實際的 templates 引數後，編譯器就將那些引數至於對應的 template 參數位置然後產出該函式的一個版本並編譯之。換句話說編譯器接管了 "為所用的每一個目標型別 (再一次) 撰寫函式" 這樣的乏味工作。

```cpp
int main()
{
    // T 是 int
    // 編譯器具現出 int compare(const int&, const int&)
    cout << compare(1, 0) << endl;

    string str_1("hi");
    string str_2("world");

    // T 是 string
    // 編譯器具現出 int compare(const string&, const string&)
    cout << compare(str_1, str_2) << endl;

    return 0;
}
```

### 使用 Class Template
和呼叫 function template 不同，使用 class template 時我們必須明確指出 template 參數的引數:

```cpp
Queue<int> q1;
Queue<vector<double>> qc;
Queue<string> qs;
```

### Template 參數
就像是函式參數一樣，程式員為 template 參數所選的名稱並無本質上的意義。先前例子中我們將 `compare()` 的 template *type* 參數命名為 `T`，但也可以取其他任何命名:

```cpp
template <typename Glorp>
int compare(const Glorp &v1, const Glorp &v2) {
    if (v1 < v2)
        return -1;
    if (v2 < v1)
        return 1;
    return 0;
}
```

這段程式碼定義的 `compare` template 與先前完全相同。

### Template 參數的作用域 (scope)
Template 參數遵循一般名稱的遮蔽規則。如果它與 global 作用域內的物件，函式，或型別的名稱相同，會遮蔽 global 作用域內的名稱:

```cpp
typedef double T;
template <class T> T clac(const T &a, const T &b) 
{
    // tmp 的型別是 template 參數 T
    // 而非 global 作用域內的那個 typedef
    T tmp = a;
    // ...
    return tmp;
}
```

### Template 宣告式
如同其他 class 或函式，我們可以宣告一個 template，而不定義它。

Template 參數的名稱再同一個 template 的 (多個) 宣告式和定義式之間不需相同。

```cpp
template <class T> T calc(const T&, const T&);
template <class U> U calc(const U&, const U&);

template <class Type> 
Type calc(const Type &a, const Type &b) {
    /*...*/
}
```

### 關鍵字 `typename` 和 `class` 之間區別
關鍵字 `typename` 和 `class` 的意義相同，可互換使用。

`typename` 的意義可能比 `class` 更直覺一些，不過這個關鍵字直到標準 C++ 才加入，舊程式可能只會使用關鍵字 `class`。

### 在 Template 定義式中表明型別
當我們想在 function template 中使用某種型別，我們必須告訴編譯器我們所寫的名稱代表一個型別。因為編譯器 (以及閱讀程式碼的人) 無法檢驗 "由 *type* 參數所定義的名稱" 式個型別還是個數值。例如:

```cpp
template <class Parm, class U>
Parm fcn(Parm *array, U value)
{
    Parm::size_type * p; // 如果 Parm::size_type 是個型別，這就是個宣告
                         // 如果 Parm::size_type 是個物件，這就是個乘法運算
}
```

我們不知道 `size_type` 是個型別名稱還是個成員變數。預設情況下編譯器會假設他是個成員變數而不是型別。

如果我們想讓編譯器把 `size_type` 視為型別，必須明確告訴編譯器:

```cpp
template <class Parm, class U>
Parm fcn(Parm *array, U value)
{
    typename Parm::size_type * p; // OK: 將 p 宣告為指標
}
```

在成員名稱之前加上關鍵字 `typename`，便告訴編譯器將該成員視為一個型別。

### Template 的 *non-type* 參數
當函式被呼叫，*non-type* 參數會被數值替代。該數值的型別被具體指明於 template 參數列。例如以下的 function template 宣告 `array_init()` 擁有一個 *type* 參數和一個 *non-type* 參數，並接收唯一參數，是個 reference to array:

```cpp
template <class T, size_t N> void array_init(T (&parm)[N])
{
    for (size_t i = 0; i != N; ++i) {
        parm[i] = 0;
    }
}
```

### 編寫泛型程式 (Generic Programs)
如果我們在不支援 `<` 運算子的物件身上呼叫 `compare()`，呼叫動作就不合法:

```cpp
Sales_item item1, item2;
// 錯誤: Sales_item 不支援 <
cout << compare(item1, item2) << endl;
```

`Sales_item` 未定義 `<` 運算子，程式無法通過編譯。

"function template" 內執行那些操作" 會造成目標型別的範圍受到侷限，程式員應該保證，作為目標型別的那個引數，其型別必須支援 function template 內使用的任何操作，而且這些操作在 (template 內的) 使用語境中均有正常行為。

### 撰寫型別獨立碼 (type-independent code)
撰寫優秀泛型碼的技術，已經超出本課程的範疇。然而有個整體性準則值得一提。

我們的 `compare()` 雖然簡單，卻也展示了泛型碼的兩個重要的撰寫原則:
- 這個 function template 的參數是 `const` reference。
- 主體內的測試只使用 `<` 進行大小比較。

有些讀者可能會認為大小比較動作如果以 `<` 和 `>` 運算子執行，會更自然:

```cpp
if (v1 < v2)
    return -1;
if (v1 > v2)
    return 1;
return 0;
```

然而若寫成這樣:

```cpp
if (v1 < v2)
    return -1;
if (v2 < v1)
    return 1;
return 0;
```

就是減少 "對引數型別的需求量"。在這，引數型別必須支援 `<` 但不必同時支援 `>`。

#### 重點
撰寫 template 程式碼時，若能盡量減少 "對引數型別的需求"，將會十分有利。

## 具現化 (Instantiation)
Template 是一份藍圖，他本身並非 class 或 function。編譯器會根據 template 產出對應之 class 或 function 的特定版本。為 template 產出一份針對特定目標型別的實體，這個過程便是所謂的具現化 (instantiation)。

### Template 引數推導 (Argument Deduction)
第一個呼叫式 `compare(1,0)` 的引數型別為 `int`，第二個呼叫式 `compare(3.14, 2.7)` 的引數型別為 `double`。"以函式引數決定 template 引數的型別和值" 的過程稱為 **template 引數推導**

### "以 *type* 參數為型別" 的引數身上的有限轉換

```cpp
short s1, s2;
int i1, i2;
compare(i1, i2); // 沒問題: 具現出 compare(int, int)
compare(s1, s2); // 沒問題: 具現出 compare(short, short)
```

如果 `compare(int, int)` 是一個一般的 non-template 函式，那麼能夠滿足上述第二個呼叫式，因為 `short` 引數會被晉升為 `int`。但因為 `compare()` 是個 template，所以上述第二個呼叫式會具現出新函式，其 *type* 參數被綁於 `short`。

一般而言函式引數並不會為了 "與某個現有具現體契合" 而被轉換。實際發生的是另外產生一個新的具現體。只有兩種轉換會誘使編譯器執行而不產生新具現體。
- `const` 轉換: 
  - 函式如果接收 reference to const 或 pointer to const，用戶可以用 reference to non-const 或 pointer to non-const 呼叫之，不會產生新具現體。如果函式接收的是 non-reference 型別，那麼參數型別和引數的 const 都會被忽略。換句話說如果函式被定義為接收 non-reference，無論我們傳入 const 或 non-const 物件，使用的是同一份具現體。
- array(或 function) 對 pointer 的轉換: 
  - 如果 template 參數不是 reference 型別，那麼 "一般的 pointer 轉換" 會施行於 "array 型別或 function 型別" 的引數身上。array 引數會被視為一個 pointer 指向其第一元素，function 引數則被視為一個 pointer 指向該函式。

```cpp
template <typename T> T fobj(T, T); // 引數會被複製
template <typename T> T fref(const T&, const T&); // reference 引數

string s1("a value");
const string s2("another value");
fobj(s1, s2); // 沒問題: 呼叫 f(string, string)，const 被忽略
fref(s1, s2); // 沒問題: non-const 物件 s1 被轉換為 const reference 

int a[10], b[42];
fobj(a, b); // 沒問題: 呼叫 f(int*, int*)
fref(a, b); // 錯誤: array型別未契合，引數未被轉換為 pointer
```

### Function-Template 的顯式引數 (Explicit Arguments)
考慮以下問題，我們想定義一個名為 `sum` 的 function template，接收兩個不同型別的引數。我們想讓返回型別的大小足夠容納 "以任意順序傳入的任意兩個型別值的合"。

```cpp
// 返回型別該寫 T 還是 U ?
template <class T, class U> ??? sum(T, U);
```

答案是: 採用其中任何一個，都會在某些時候出問題:

```cpp
sum(3, 4L); // 第二個型別比較大: 我們希望寫出 U sum(T, U)
sum(3L, 4); // 第一個型別比較大: 我們希望寫出 T sum(T, U)
```

解決方法有兩種，第一種是強迫 `sum()` 用戶將較小的型別強制轉型至我們想要的結果型別:

```cpp
int i, short s;
sum(static_cast<int>(s), i) // 沒問題: 具現化 int sum(int, int)
```

第二種解決方法式引進第三個 template 參數，此參數必須由呼叫者明確指定:

```cpp
template <class T1, class T2, class T3>
T1 sum(T2, T3);
```

這唯一的缺點是: 沒有任何函式引數可以用來推導 `T1` 型別。因此用戶每次呼叫 `sum` 時都必須為 `T1` 參數明確提供引數。

```cpp
long val3 = sum<long>(i, lng); // 沒問題: 呼叫的是 long sum(int, long)
```

### 顯式引數和 Pointer to Function Template

```cpp
template <typename T> int compare(const T&, const T&);
// func 重載版本: 各自接收不同的 function pointer 型別
void func(int(*)(const string&, const string&));
void func(int(*)(const int&, const int&));

func(compare<int>); // 沒問題: 明確指出要哪個 compare 版本
```

## Template 的編譯模型(Compilation Models)
當我們呼叫函式時，編譯器通常只需在當時見過函式宣告即可。同樣道理，當我們定義 class 物件時，該 class 的定義式必須可見，但其成員函式的定義式不需在場。因此我們把 class 定義式和函式宣告放在標頭檔內，把一般函式和 class 成員函式的定義放在源碼檔 (source files) 內。

但是 template 就不同了，欲產生一個具現體，編譯器必須能夠取得 template 的源碼。當我們呼叫 function template 或 "class template 的成員函式" 時，編譯器需要那些函式的定義，而那些定義式一般都被至於源碼檔。

為了編譯 template 程式碼，有以下三種方法:
1. 可將 class 定義式和函式宣告，函式和成員的定義全部都放到標頭檔
2. "置入式編譯模型"(inclusion compilation model)
3. "分離式編譯模型"(separate compilation model)

2, 3種程式的架構方式大體相同: class 定義式和函式宣告式至於標頭檔，函式和成員定義放在源碼檔。幾乎所有編譯器都支援第 2 種，第 3 種只有少數編譯器支援。

### 置入式編譯模型(Inclusion Compilation Model)

```cpp
// 標頭檔 utilities.h
#ifndef UTILITIES_H // header guard
#define UTILITIES_H

template <class T> int compare(const T&, const T&);
//...
#include "utilities.cpp" // 取得 compare 和其他函式的定義 
#endif
```

```cpp
// 實作檔 utilities.cpp
template <class T> int compare(const T &v1, const T &v2)
{
    if (v1 < v2)
        return -1;
    if (v2 < v1)
        return 1;
    return 0;
}
```