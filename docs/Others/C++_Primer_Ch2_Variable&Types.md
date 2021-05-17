
# Chapter 2 - Variables and Basic Types

###### tags: `C++` `C++11` `C++primer`

---

## 2.1 Primitive Built-in Types

### 2.1.1 Arithmetic Types
- `char` 保證大小足夠儲存裝置的 basic character set 
- `wchar_t`, `char16_t`, `char32_t` 用於儲存 extended character set, `wchar_t` 保證大小足夠儲存裝置最大的 extended character set, 而 `char16_t`, `char32_t` 則是用於 Unicode character
- `int` 保證可儲存的大小 >= `short`
- `long` 保證可儲存的大小 >= `int`
- `long long` 保證可儲存的大小 >= `long`

    | Type     | Meaning  | Minimum Size |
    | -------- | -------- | ------------ |
    | `bool`       | boolean            | NA   |
    | `char`       | character          | 8 bits |
    | `wchar_t`    | wide character     | 16 bits |
    | `char16_t`   | Unicode character  | 16 bits |
    | `char32_t`   | Unicode character  | 32 bits |
    | `short`      | short integer      | 16 bits |
    | `int`        | integer            | 16 bits |
    | `long`       | long integer       | 32 bits |
    | `long long`  | long integer       | 64 bits |
    | `float`      | single-precision floating-point | 6 significant digits |
    | `double`     | double-precision floating-point | 10 significant digits |
    | `long double`| extended-precision floating-point | 10 significant digits |

    > ### Advice: Deciding Which Type to use
    > C++ 的設計讓程式設計師在必要的時候可以更貼近硬體, 這些 arithmetic types 被設計成可以迎合各式各樣不同的硬體, 某些的 arithmetic types 或許會讓開發者覺得複雜, 所以以下提供幾種簡單的方法去選擇使用的型別:
    > - 當你知道這個值不可能是負的時候, 請使用 `unsigned`
    > - 整數請使用 `int`, `short` 通常太小, `long` 則往往跟 `int` 佔用一樣的大小, 如果你要使用的值大於 `int` 的 minimum size, 則請改用 `long long`
    > - 浮點數運算請使用 `double`, `float` 通常不夠精準, double-precision floating-point 與 single-precision floating-point 的所耗的計算量差異並不大, 事實上, 在有些裝置 double-precision floating-point 的運算速度甚至快於 single-precision floating-point

### 2.1.2 Type Conversions
- 型別轉換通常取決於此型態可容許的大小範圍
    ```cpp
    bool b = 42;          // b is true
    int i = b;            // i has value 1
    i = 3.14;             // i has value 3
    double pi = i;        // pi has value 3.0
    unsigned char c = -1; // assuming 8-bits char, c has value 255
    signed char c2 = 256; // assuming 8-bits char, the value of c is undefined
    ```
    > 隱式型別轉換的細節規則, 可以在書上查

- 當我們在一個 expression 裡面使用了一種 arithmetic types 的變數, 但 expression 卻期望得到另一種 arithmetic types 的變數時, compiler 會自動做型別轉換
    ```cpp
    int i = 42;
    if (i)  // condition will evaluate as true
        i = 0;
    ```
- 當涉及到 `unsigned` 型別時必須小心
    ```cpp
    unsigned u = 10;
    int i = -42;
    std::cout << i + i << std::endl; // print -84 
    std::cout << u + i << std::endl; // if 32-bits int, print 4294967264
    ```
    可以看到上面第二個 expression `i` 被隱式型別轉換成 `unsigned`所以造成問題
    ```cpp
    // WRONG: u can never be less than 0; the condition will always succeed
    for (unsigned u = 10; u >= 0; --u)
        std::cout << u << std::endl;
    ```
    可以改成使用 `while` 來解決這種問題
    ```cpp
    unsigned u = 11;
    while (u > 0) {
        --u;
        std::cout << u << std::endl;
    }
    ```

    > #### Caution: Don't Mix Signed and Unsigned Types
    > 當 expression 同時包含 `signed` 跟 `unsigned` 型別的變數而 `signed` 變數是負值時, 可能會產生意外的後果, 簡單來說, 避免將這兩種型別混合使用

### 2.1.3 Literals
- 每個字面常數都有一種型別, 以下為常見幾種寫法, 若想要強制決定字面常數的型別, 可以參考 **Character and Character String Literals** 介紹
    ```cpp
    20        // decimal
    024       // octal
    0x14      // hexadecimal
    3.14159   // double
    3.14159E0 // E exponential
    .001      // double
    'h'       // char
    "h"       // string, '\0'
    ```
    #### Character and Character String Literals
    | Prefix | Meaning | Type |
    | -------- | -------- | -------- |
    | `u` | Unicode 16 character | `char16_t` |
    | `U` | Unicode 32 character | `char32_t` |
    | `L` | wide character | `wchar_t` |
    | `u8`| utf-8(string literals only) | `char` |

    #### Integer Literals
    | Suffix | Minimum Type |
    | -------- | -------- | 
    | `u` or `U` | unsigned |
    | `l` or `L` | long |
    | `ll` or `LL` | long long |

    #### Floating-point Literals
    | Suffix | Minimum Type |
    | -------- | -------- | 
    | `f` or `F` | float |
    | `l` or `L` | long double |

---

## 2.2 Variables

### 2.2.1 Variable Definitions
- 一個簡單的變數定義是要由 type specifier 和名稱所組成, 一個定義可能可以由一個或多個初始值和名稱所組成
    ```cpp
    int sum = 0, value, units_sold = 0;
    Sales_item item;
    std:string book("0-201-78345-X");
    ```
    #### Initializers
    一個物件被創建時, 同時給定一個特定的值, 稱為初始化
    > !! Initialization is not assignment. 初始化是當變數在被創建時給定一個值, 賦值是物件現有的值被新的值所取代

    #### List Initialization 
    ```cpp
    int val = 0;
    int val = {0};
    int val{0};     // C++ 11
    int val(0);

    long double ld = 3.1415926536;
    int a{ld}, b = {ld};    // error: narrowing conversion required
    int c(ld), d = ld;      // ok: but value will be truncated
    ```

    #### Default Initialization
    ```cpp
    std::string empty;    // 隱式初始化空的string
    Sales_item item;      // 預設初始化
    ```

### 2.2.2 Variable Declaration and Definitions
- 一個變數只能被定義一次(只能有一個定義)，但是可以被宣告很多次。
- 一個變數的定義也是一種宣告。
    ```cpp
    extern int i; // declares but doesn't defines i
    int j;        // declares and defines j
    ```

### 2.2.3 Identifiers

### 2.2.4 Scope of a Name

---

## 2.3 Compound Types
- 複合型態，是一種根據另一種型態定義而來的型態。C++ 有很多種複合型態，在本章節會提到其中 - **reference and pointer**。

### 2.3.1 Reference 
- > 附註：在之後的章節會提到一種新的標準的reference - "rvalue reference"，它主要是用於 class 裡面，技術上我們通常說的reference 指的是 - "lvalue reference"。

- reference 是對一個物件定義一個別名。
- 通常我們初始化一個變數，是將物件的值**複製**到我們要創建的變數。但是當我們定義一個reference並初始化時，不是複製物件的值到reference，而是 連結(bind) reference 跟這個物件。一但初始化過後，reference 就跟這個初始化物件連結在一起了，且不能重新連結到其他的物件
    ```cpp
    int ival = 1024;
    int &refVal = ival; // refVal refers to(is another name for) ival
    int &refVal2;       // error: a reference must be initialized
    int &refVal3 = 10;  // error;
    ```

### 2.3.2 Pointers

### 2.3.3 Understanding Compound Type Declaration
- Compound Type (複合型態) 指的就是常見的 `*`, `&` 字符 , 此種型態的定義取決於多種型態的組合

    #### Defining Multiple Variable
    ```cpp
    int* p1, p2;  // p1 is and pointer, p2 is an int
    int *p1, *p2; // p1, p2 are pointers to int 
    ```
    #### Pointers to Pointers
    通常, 在宣告上對於 type modifiers 使用的數量是沒有限制的, 當我們使用超過一個 type modifiers, 邏輯上是ok的, 但是並不直觀
    ```cpp
    int ival = 1024;
    int *pi = &ival; // pi point to an int
    int **ppi = &pi; // ppi points to a pointer to an int
    ```
    #### Reference to Pointers
    reference 不是一個 object, 所以 pointer 是不能指向 reference 的, 但是 pointer 是一個 object, 所以 reference 是可以參考 pointer 的:
    ```cpp
    int i = 42;
    int *p;      // p 是一個 pointer 指向 int
    int *&r = p; // r 是一個 reference 參考 pointer p

    r = &i; // r 參考一個 pointer, 將 &i assigning to r 使得 p 指向到 i
    *r = 0; // dereference r, p 所指向的 object i, 值被改變成 0
    ```
    最簡單去閱讀理解 r 型別的方式, 就是由右讀到左的方式去閱讀
    最靠近變數的 symbol (`&` in `&r`) 是最早被作動的 symbol

---

## 2.4 `const` Qualifier
- 型態 `const`, 我們可以用來定義一個不能被改變的變數

    ```cpp
    const int bufsize = 512; // input buffer size
    bufsize = 512; // error:attempt to write to const object
    ```
    因為我們create `const` object 過後, 就不能改變他的值了, 所以 `const` 變數一定要初始化
    ```cpp
    const int i = get_size(); // ok:initialized at run time
    const int j = 42;         // ok:initialized at compile time
    const int k;              // error:k is uninitialized const
    ```

    #### Initialization and `const`
    ```cpp
    int i = 42;
    const int ci = i; // ok:the value in i is copied into ci
    const j = ci;     // ok:the value in ci is copied into j
    ```
    #### By Default, `const` Object Are Local to a File
    ```cpp
    const int bufSize = 512;
    ```
    如果在編譯時期初始化的 `const` 常數, compiler 在編譯過程中會將對應的值直接取代變數, 以上面為例: compiler 會產生 `512` 取代 `bufSize`

### 2.4.1 Reference to `const`
- 待續，等有時間再繼續寫。
