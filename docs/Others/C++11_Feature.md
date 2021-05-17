# C++ 11 new feature
###### tags:`C++11`, `C++`

## Chapter 2: Variable and Basic Types
- **List Initialization**
- `nullptr`
- `constexpr`
- **Alias Declaration**:
    -  `using SI = Sales_item;`
- `auto`
- `decltype`

## Chapter 3: Strings, Vectors, and Arrays
- `for(`*declaration* `:` *expression*`)`:
    - *expression* -> `vector`, `array[]`, `string`
- **List Initialization**: 
    ``` cpp
    vector<string> articles = {"a", "an", "the"};
    ```
- The `.cbegin()` and `.cend()` Operations in Iterator:
    ``` cpp
    vector<int> v;
    auto it = v.cbegin(); // it has type vector<int>::const_iterator
    ```
- The Library `begin()` and `end()` Functions:
    ``` cpp
    int ia[] = {0,1,2,3,4,5,6,7,8,9}; // ia is an array of ten ints 
    int *beg = begin(ia); // pointer to the first element in ia
    int *last = end(ia); // pointer one past the last element in ia
    ```
- Multidimension`for` loop application:
    ```cpp
    size_t cnt = 0;
    for (auto &row : ia) // for every element in the outer array 
        for (auto &col : row) { // for every element in the inner array col = cnt; // give this element the next value
            ++cnt; // increment cnt 
        }
    ```

## Pointer:
- Function pointer: `std::bind()`,`std::function<>` 
    ```cpp
    #include<function>

    using namespace std;
    using namespace placeholder;

    int add(int a, int b){
        return a + b;
    }

    auto func = bind(add, _1, 20);

    cout << func(10) << endl;
    // result: 30
    ``` 
- Smart pointer: `unique_ptr`
    ```cpp
    #include<iostream>
    #include<memory>

    using namespace std;

    class Test {
    public:
        Test() {
            cout << "created" << endl;
        }

        ~Test() {
            cout << "destroy" << endl;
        }

        void greet() {
            cout << "Hello" << endl;
        }
    };

    class Temp {
    private:
        unique_ptr<Test[]> pTest_;
    public:
        Temp() : pTest_(new Test[2]) {}
    };

    int main()
    {
        Temp temp;

        cout << "finished" << endl;

        return 0;
    }
    // pTest_ will be delete when temp out of the scope.
    ```
- Smart pointer: `shared_ptr`
    ```cpp
    int main()
    {
        shared_ptr<Test> test2(nullptr);

        {
            shared_ptr<Test> test1 = make_shared<Test>();
            test2 = test1;
        }

        cout << "finished" << endl;

        return 0;
    }
    // result: 
    // "created"
    // "finished"
    // "destroy"

    // shared_ptr will be delete until all of same address pointer out of scope.
    ```