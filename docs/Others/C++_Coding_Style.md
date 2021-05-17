# C++ Coding Style

## Class

#### 1. 建構子(Constructor)
- 不要在建構子中進行複雜的初始化 (尤其是那些有可能失敗或者需要呼叫虛函式的初始化).

#### 2. 初始化
- 如果類中定義了成員變數, 則必須在類中為每個類提供初始化函式或定義一個建構子. 若未宣告建構函式, 則編譯器會生成一個默認的構造函式, 這有可能導致某些成員未被初始化或被初始化為不恰當的值.

#### 3. 顯示(`explicit`) 建構子(Constructor)
- 對單個參數的建構子使用 C++ 關鍵字 `explicit`.

#### 4. 可拷貝類型和可移動類型 (Copy contructor and Move contructor)
- 如果你的類型需要, 就讓它們支持拷貝 / 移動. 否則, 就把隱式產生的拷貝和移動函式禁用.
[what's `default`, `delete` copy , move contructor](http://talesofcpp.fusionfenix.com/post-24/episode-eleven-to-kill-a-move-constructor)

#### 5. 
     



