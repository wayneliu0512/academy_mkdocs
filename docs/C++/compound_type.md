# 複合型別 (Compound type)
複合型別是指基於其他型別定義的型別，C++有幾種複合型別，本章將介紹其中兩種: 參照 (Reference) 和 指標 (Pointer)。

## 參照 (Reference)
Reference 是個複合型別 (compound type)，藉由在變數名稱之前加上 `&` 符號而定義出來。就 reference 而言，每個 reference 型別都 "參照，引用" 其他型別。我們不能定義參照另一個 reference 型別的 reference，但可以讓 reference 參照其他任何資料型別。

```cpp
int ival = 1024;
int &refVal = ival;  // refVal refers to (is another name for) ival
int &refVal2;        // error: a reference must be initialized
int &refVal3 = 10;   // 錯誤: 初始值必須是一個 object
```

### Reference 是個別名

```cpp
refVal += 2;     // 其實是把 2 加入 ival
int ii = refVal; // 其實是把 ival 的值賦予 ii
```

一旦 reference 被初始化，只要他仍然存在，就會持續綁定 (bind) 原先那個物件。沒有任何辦法可以將 reference 重新綁定 (rebind) 至另一個不同物件。

### 定義多個 Reference

```cpp
int i = 1024, i2 = 2048;
int &r = i, r2 = i2;     // r 是個 reference, r2 是個 int
int i3 = 1024, &ri = i3; // 定義一個 object 和一個 reference
int &r3 = i3, &r4 = i2;  // 定義兩個 reference
```

### `const` Reference
所謂 `const` reference 是一個 "用以參照 `const` 物件" 的 reference :

```cpp
const int ival = 1024;
const int &refVal = ival; // 沒問題: reference 和 object 都是 const
int &ref2 = ival;         // 錯誤: nonconst reference 卻參照至一個 const object
```

`const` reference 可以不同型別的物件或以一個右值 (rvalue) 初始化，例如字面常數

```cpp
int i = 42;
// 以下只有使用 const reference 才合法
const int &r = 42;
const int &r2 = r + i;
```

此種初始化對 non`const` 不合法，會導致編譯錯誤。

## Pointer
概念上 Pointer 很簡單:指向一個物件。具體而言， pointer 內部存放 另一個物件的位址:

```cpp
string s("hello world");
string *sp = &s; // sp 裡存放的是 s 的位址
```

第二個敘述式裡把 `sp` 定義為指向 `string` 的一個 pointer。式中的 `*` 表示 `sp` 是個 pointer, `&` 則是所謂 *address-off* (取址) 運算子

```cpp
double dval;
double *pd = &dval; // 正確: 初始值是 double 型別對象的地址
double *pd2 = pd;   // 正確: 初始值是指向 double 對象的指標

int *pi = pd;       // 錯誤: 指標 pi 的型別和 pd 的型別不匹配
pi = &dval;         // 錯誤: 試圖把 double 型別對象的地址賦值給 int 型別指標
```

### 不同的 Pointer 宣告風格

```cpp
string* ps; //合法，但容易造成誤解
```

因為讓人有以下想法:  `string*` 是個型別，此定義式定義的任何變數都是 pointer to `string` 的想法，然而

```cpp
string* ps1, ps2; // ps1 是個 pointer to string, ps2 是個 string
```

### 避免使用未初始化的 Pointers
未初始化的 pointers 是一種常見的執行期錯誤。

### 操作 Pointer
我們可以藉由提領 (dereference) pointers 來存取該物件。*dereference* 運算子 (`*`) 傳回 pointer 所指的物件:

```cpp
string s("hello world");
string *sp = &s; // sp 存放著 s 的位址
cout << *sp;     // 印出 hello world
```

### 提領 (Dereference) 產生左值
*Dereference* 運算子傳回底層物件的左值，所以我們可以用它改變 pointer 所指的物件值:

```cpp
*sp = "goodbye";
```

因為我們是對 `*sp` 賦值，所以這個述句讓 `sp` 仍然指向 `s`，但改變了 `s` 的值。

我們也可以賦予新值給 `sp` 本身，這會造成 `sp` 指向不同的物件:

```cpp
string s2 = "some value";
sp = &s2; // sp 現在指向 s2
```

### 關鍵概念
像 `&` 和 `*` 這樣的符號，既能做表達式裡面的運算符，也能作為宣告的一部分，符號的上下文決定了符號的涵義:

```cpp
int i = 42;
int &r = i;   // & 在型別名後出現，因此是宣告的一部分，r 是一個參照
int *p;       // * 在型別名後出現，因此是宣告的一部分，p 是一個指標
p = &i;       // & 出現在表達式中，是一個取址運算子 (address-off)
*p = i;       // * 出現在表達式中，是一個提領運算子 (dereference)
int &r2 = *p; // & 是宣告的一部份，* 是一個提領運算子
```

### 空指標
空指標 (null pointer) 不指向任何對象，在試圖使用一個指標之前程式碼可以首先檢查他是否為空。

```cpp
int *p1 = nullptr; // 等價於 int *p1 = 0, C++11 新標準
int *p2 = 0;       // 直接將 p2 初始化為字面常數 0
// 需要首先 #include cstdlib
int *p3 = NULL;    // 等價於 int *p3 = 0
```

### Pointers 與 `const`
pointer 與 `const` 有兩種相關連: 我們可以擁有 指向 `const` 物件 和 本身為 `const` 兩種 pointers。

### Pointers to `const` (指向常數的指標)

```cpp
const double *cptr; // cptr 可指向一個 const double
*cptr = 42;         // 錯誤: *cptr 可能是個 const
```

將一個 `const` 物件的位址賦予 non-`const` pointer 會造成編譯錯誤:

```cpp
const double pi = 3.14;
double *ptr = &pi;        // 錯誤: ptr 是一般的 pointer
const double *cptr = &pi; // 沒問題: cptr 指向 const
```

一個 pointers to `const` 可被賦予一個 non`const` 物件位址:

```cpp
double dval = 3.14; // dval 是個 double: 其值可以被改變
cptr = &dval;       // 沒問題: 但不能透過 cptr 改變 dval
```

### `const` Pointers
除了 pointer to `const`，還有所謂 `const` pointer，那是一個我們不能改變其本身值的 pointer:

```cpp
int errNumb = 0;
int *const curErr = &errNumb; // curErr 是個 const pointer
curErr = curErr               // 錯誤: curErr 是個 const
```

像面對任何 `const` 一樣，我們必須在建立 `const` pointer 時把它初始化。
Pointer 本身是 `const` 這個事實並不代表我們不能透過它改變它所指的值。

```cpp
if (*curErr) {
    errorHandler();
    *curErr = 0; // 沒問題: 重設 curErr 所綁定的物件值
}
```
補充：可以藉由 `const` 在 `*` 左邊還是右邊來分辨是 `const` pointer 還是 pointer to `const`