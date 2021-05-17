# Effective C++

## 1. View C++ as a federation of languages.

In the beginning, C++ was just C with some object-oriented features tacked on. Even C++'s original name, "C with Classes", reflected this simple heritage.

As the language matured, it grew bolder and more adventurous. Today C++ is a *multiparadigm programming language*, one supporting a combination of procedural, object-oriented, functional, generic, and metaprogramming features. This is power and flexibility make C++ a tool without equal, but can also cause some confusion.

The easiest way is to view C++ not as a single language but as a federation of related languages. To make sense of C++, you have to recognize its primary sub-languages.

- **C**

    Blocks, statements, the preprocessor, build-in data types, arrays, pointers, etc...

- **Object-Oriented C++**

    Classes, encapsulation, inheritance, polymorphism, virtual functions, etc...

- **Template C++**

    This is the generic programming part of C++.

- **The STL**

    The STL is a template library. Its conventions regarding containers, iterators, algorithm, and function objects, etc...

**Things to Remember**

- Rules for effective C++ programming vary, depending on the part of C++ you are using.

## 2. Prefer `const` , `enum` , and `inline` to `#define` .

This item might better be called "prefer the compiler to the preprocessor", because `#define` may be treated as if it's not part of the language. For example:

    #define ASPECT_RATIO 1.653

The symbolic name `ASPECT_RATIO` may never be seen by compiler; This can be confusing if you get an error during compilation involving the use of the constant, because the error message may refer to `1.653`, not `ASPECT_RATIO` .

The solution is to replace the macro with a constant:

    const double kAspectRatio = 1.653;

The second special case concerns class-specific constants. To limit the scope of a constant to a class, you must make it a member, and to ensure there's at most one copy of the constant, you must make it a `static` member:

```cpp
class CostEstimate{
private:
    static const double FudgeFactor; // declaration; goes in header file
    ...
};
    
const double CostEstimate::FudgeFactor = 1.35; //definition; goes in impl. file
```

There is a exception when you need the value of a class constant during compilation of the class, you can use `enum` defined like this:

```cpp
class GamePlayer {
private:
    enum {NumTurns = 5}; // enum class is better.

    int scores[NumTurns];
    ...
};
``` 

Getting back to preprocessor, another common (mis)use of the `#define` directive is using it to implement macros that look like functions but that don't incur the overhead of a function call.

```cpp
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))
```

Look at the weird things that can happen:

```cpp
int a = 5, b = 0;
CALL_WITH_MAX(++a, b);    // a is incremented twice
CALL_WITH_MAX(++a, b+10); // a is incremented once
```

The solution is:

```cpp
template<typename T>   // template <class T>
inline void CallWithMax(const T &a, const T &b)
{
    f(a > b ? a : b);
}
```

**Things to Remember**

- For simple constants, prefer `const` objects or `enum` to `#define`.
- For function-like macros, prefer inline functions to `#define`.

## 3. Use `const` whenever possible.

The wonderful thing about `const` is that it allows you to specify a semantic constraint - a particular object should not be modified - and compilers will enforce that constraint.

這個章節希望大家每當有可能就使用 `const`, 所以前半部很多內容是在複習 `const` 的作用, 這部分我就只用 sample code 帶過, 大家也順便複習一下, 看是不是知道這些 `const` 的用法以及差異, 如果有不懂的地方可以提出來.

```cpp
char greeting[] = "Hello";
char *p = greeting;              // non-const pointer,
    							 // non-const data
const char *p = greeting;        // non-const pointer,
								 // const data
char * const p = greeting;       // const pointer,
                                 // non-const data
const char * const p = greeting; // const pointer, 
    							 // const data
```

```cpp
void f1(const Widget *pw); // f1 takes a pointer to a constant Widget object
void f2(Widget const *pw); // so does f2
```

```cpp
std::vector<int> vec;
...
const std::vector<int>::iterator iter = vec.begin(); // iter acts like a T* const
*iter = 10;                                          // OK, changes what iter points to
++iter;                                              // error! iter is const
    
std::vector<int>::const_iterator c_iter = vec.begin(); // c_iter acts like a const T*
*c_iter = 10;                                          // error! *c_iter is const
++c_iter;                                              // fine, changes c_iter
```

### `const` Member Functions

The purpose of `const` on member functions is to identify which member functions may be invoked on `const` objects. Such member functions are important for two reasons. First, they make the interface of a class easier to understand. It's important to know which functions may modify an object and which may not. Seconds, they make it possible to work with `const` objects. That's a critical aspect of writing efficient code, because, as Item 20 explains, one of the fundamental ways to improve a C++ program's performance is to pass objects by reference-to-const.

```cpp
class TextBlock {
public:
    ...
    const char& operator[](const std::size_t position) const // operator[] for
    { return text[position]; }                               // const objects
    char& operator[](const std::size_t position)             // operator[] for
    { return text[position]; }                               // non-const objects
    
private:
    std::string text;
};
```

`TextBlock` 's `operator[]` s can be used like this:

```cpp
TextBlock tb("Hello");
std::cout << tb[0];           // calls non-const TextBlock::operator[]
    
const TextBlock ctb("World");
std::cout << ctb[0];          // calls const TextBlock::operator[]
```

Incidentally, `const` objects most often arise in real programs as a result of being passed by pointer- or reference-to-const.

```cpp
void print(const TextBlock& ctb)  // in this function, ctb is const
{
    std::cout << ctb[0];            // call const TextBlock::operator[]
    ...
}
```

By overloading `operator[]` and giving the different versions different return types, you can have `const` and non- `const` `TextBlock` s handled differently:

```cpp
std::cout << tb[0];  // fine - reading a non-const TextBlock
tb[0] = 'x';         // fine - writing a non-const TextBlock
std::cout << ctb[0]; // fine - reading a const TextBlock
ctb[0] = 'x';        // error! - writing a const TextBlock
```

Sometimes `const` member function might modify some of the bits in the object on which it's invoked, but only in ways that clients cannot detect. For example, your `CTextBlock` class might want to cache the length of the textbook whenever it's requested:

```cpp
class CTextBlock {
public:
    ...
    std::size_t length() const;

private:
    char *pText;
    std::size_t textLength;  // last calculated length of textblock
    bool lengthIsValid;      // whether length is currently valid
}

std::size_t CTextBlock::length() const
{
    if (!lengthIsValid) {
        textLength = std::strlen(pText); // error! can't assign to textLength and lengthIsValid in a const member function
        lengthIsValid = true;
    }

    return textLength;
}
```

This implementation of `length` is certainly not bitwise `const` - both `textLength` and `lengthIsValid` may be modified. Compilers disagree. They insist on bitwise constness. What to do?

The solution is simple: take advantage of C++'s `const`-related wiggle room known as `mutable`. `mutable` frees non-static data members from the constraints of bitwise constness:

```cpp
class CTextBlock {
public:
    ...
    std::size_t length() const;

private:
    char *pText;
    mutable std::size_t textLength;  // these data members may
    mutable bool lengthIsValid;      // always be modified, even
                                     // in const member function
}
```

### Avoiding Duplication in `const` and Non-`const` Member Functions

For example, suppose that `operator[]` in `TextBlock` not only returned a reference to the appropriate character, it also performed bounds checking, logged access information, maybe even did data integrity validation. Putting all this in both the `const` and the non-`const` `operator[]` functions yields this kind of monstrosity:

```cpp
class TextBlock {
public:
    ...
    const char& operator[](const std::size_t position) const
    {
        ... // do bounds checking
        ... // log access data
        ... // verify data integrity
        return text[position];
    }

    char& operator[](const std::size_t position)
    {
        ... // do bounds checking
        ... // log access data
        ... // verify data integrity
        return text[position];
    }

private:
    std::string text;
};
```

Ouch! Can you say code duplication, along with its attendant compilation time, maintenance, and code-bloat headaches? Sure, So what you really want to do is implement `operator[]` functionality once and use it twice. That is, you want to have one version of `operator[]` call the other one. And that bring us to casting away constness.

```cpp
class TextBlock {
public:
    ...
    const char& operator[](const std::size_t position) const // same as before
    {
        ... 
        ... 
        ... 
        return text[position];
    }

    char& operator[](const std::size_t position)
    {
        return 
          const_cast<char&>(                     // cast away const on op[]'s return type;
            static_cast<const TextBlock&>(*this) // add const to *this's type;
              [position]                         // call const version of op[]
        );
        
    }

private:
    std::string text;
};
```

**Things to Remember**

- Declaring something `const` helps compilers detect usage errors. `const` can be applied to objects at any scope, to function parameters and return types, and to member functions as a whole.
- Compilers enforce bitwise constness, but you should program using logical constness.
- When `const` and non-`const` member functions have essentially identical implementations, code duplication can be avoided by having the non-`const` version call the `const` version.  

## 4. Make sure that objects are initialized before they're used.

C++ can seem rather fickle about initializing the values of objects. For example, if you say this,

```cpp
int x;
```

in some constexts, `x` is guaranteed to be initialized(to zero), but in others, it's not. If you say this,

```cpp
class Point {
    int x, y
}
...
Point p;
```

`p`'s data members are sometimes guaranteed to be initialized(to zero), but sometimes they're not. Reading uninitialized values yields undefined behavior.

Now, there are rules that describe when object initialization is guaranteed to take place and when it isn't. Unfortunately, the rules are complicated - too complicated to be memorizing.

The best way to deal with this seemingly indeterminate state of affairs is to *always* initialize your objects before you use them.

```cpp
int x = 0;                             // manual initialization of an int
const char *text = "A C-style string"; // manual initialization of a pointer

double d;                              // "initialization" by 
std::cin >> d;                         // reading from an input stream
```

For almost everything else, the responsibility for initialization falls on constructors. The rule there is simple: make sure that all constructors initialize everything in the object.

The rule is easy to follow, but it's important not to confuse assignment with initialization.

```cpp
class PhoneNumber {...};

class ABEntry{   // ABEntry = "Address Book Entry"
public:
    ABEntry(const std::string& name, const std::string &address,
            const std::list<PhoneNumber>& phone);

private:
    std::string theName;
    std::string theAddress;
    std::list<PhoneNumber> thePhones;
    int numTimesConsulted;
};

ABEntry::ABEntry(const std::string& name, const std::string &address,
                 const std::list<PhoneNumber>& phone)
{
    theName = name;        // these are all assignments,
    theAddress = address;  // not initializations
    thePhones = phones;
    numTimesConsulted = 0;
}
```

This will yield `ABEntry` objects with the values you expect, but it's still not the best approach. They aren't being initialized, they're being assigned. Initialization took place earlier.

A better way to write the `ABEntry` constructor is to use the member initialization list instead of assignments:

```cpp
ABEntry::ABEntry(const std::string& name, const std::string &address,
                 const std::list<PhoneNumber>& phone)
: theName(name),
  theAddress(address),   // these are now all initializations
  thePhones(phones),
  numTimesConsulted(0)
{}
```

Sometimes the initialization list must be used, even for built-in types. For example, data member that are `const` or are references must be initialized. They cant't be assigned.

One aspect of C++ that isn't fickle is the order in which an object's data is initialized. This order is always the same: base classes are initialized before derived classes, and within a class, data members are initialized in the order in which they are declared. This is true even if they are listed in a different order on the member initialization list. To avoid reader confusion, always list members in the initialization list in the same order as they're declared in the class.

**Things to Remember**

- Manually initialize objects of built-in type, because C++ only sometimes initializes them itself.
- In a constructor, prefer use of the member initialization list to assignment inside the body of the constructor. List data members in the initialization list in the same order ther're declared in the class.
- Avoid initialization order problem across translation units by replacing non-local statics objects with local static objects.

## 5. Know what function C++ silently writes and calls.

When is an empty class not an empty class? When C++ gets through with it. If you don't declare them yourself, compiler will declare their own versions of a copy constructor, a copy assignment operator, and a destructor. As a result, if you write 

```cpp
class Empty{};
```

it's essentially the same as if you'd written this:

```cpp
class Empty{
public:
    Empty(){...}                  // default constructor
    Empty(const Empty& rhs){...}  // copy constructor
    ~Empty(){...}                 // destructor

    Empty& operator=(const Empty& rhs){...} // copy assignment operator
};
```

These functions are generated only if they are needed. The following code will cause each function to be generated:

```cpp
Empty e1;     // default constructor
              // destructor
Empty e2(e1); // copy constructor
e2 = e1;      // copy assignment operator
```

Let's consider next example:

```cpp
template<typename T>
class NamedObject {
public:
    NamedObject(const char *name, const T &value);
    NamedObject(const std::string &name, const T &value);
    ...

private:
    std::string nameValue;
    T objectValue;
}
```

Because a constructor is declared in `NamedObject`, compilers won't generate a default constructor. This is important.

`NamedObject` declares neither copy constructor nor copy assignment operator. so compilers will generate those functions. Look, then, at this use of the copy constructor:

```cpp
NamedObject<int> no1("Small prime number", 2);
NamedObject<int> no2(no1);  // calls copy constructor
```

The copy constructor generated by compilers must initialize `no2.nameValue` and `no2.objectValue` using `no1.nameValue` and `no1.objectValue`, respectively. The type of `nameValue` is `string`, so `no2.nameValue` will be initialized by calling the `string` copy constructor with `no1.nameValue` as its argument. On the other hand, `int` is a built-in type, so `no2.objectValue` will be initialized by copying the bits in `no1.objectValue`.

The compiler-generated copy assignment operator for `NameObject<int>` would behave essentially the same way.

Next example, suppose `NameObject` were defined like this, where `nameValue` is a reference to a string and `objectValue` is a `const T`:

```cpp
template<typename T>
class NamedObject {
public:
    NamedObject(const std::string &name, const T &value);
    ...

private:
    std::string &nameValue; // this is now a reference
    const T objectValue;    // this is now const
}
```

Now consider what should happen here:

```cpp
std::string newDog("Persephone");
std::string oldDog("Satch");

NameObject<int> p (newDog, 2);

NameObject<int> s (oldDog, 36);

p = s;  // what should happen to the data members in p ?
```

After the assignment, should `p.nameValue` refer to the `string` referred to by `s.nameValue`. Should the reference itself be modified? If so, that breaks new ground, because C++ doesn't provide a way to make a reference refer to a different object.

Faced with this conundrum, C++ refuse to compile the code. If you want to support copy assignment in a class containing a reference member, you must define the copy assignment operator yourself. Compilers behave similarly for classes containing `const` members.

**Things to Remember**

- Compilers may implicitly generate a class's default constructor, copy constructor, copy assignment operator, and destructor.

## 6. Explicitly disallow the use of compiler-generated functions you do not want.

這邊書是以房產做舉例, 假設我們把房產寫成一個類:

```cpp
class HomeForSale{...};
```

我們可以知道現實中任何一個房產都是獨一無二的, 如果它可以複製, 那肯定是不合理的, 所以對於以下情況, 我們應該讓它編譯失敗:

```cpp
HomeForSale h1;
HomeForSale h2;
HomeForSale h3(h1);  // attempt to copy h1 - should not compile!

h1 = h2              // attempt to copy h2 - should not compile!
```

然而要作到這樣的效果並不直觀, 通常我們直覺會認為如果我們不想要一個 function 的功能, 我們只要不宣告這個 function 就好了, 但這個作法在 copy constructor, copy assignment operator 不管用, 因為根據第 5 點我們提到, 如果你不宣告, compiler 會幫你自動宣告.

所以如果我們要解決這個問題, 可以用以下方法:

```cpp
class HomeForSale{
public:
    ...
    HomeForSale(const HomeForSale&) = delete;
    HomeForSale& operator=(const HomeForSale&) = delete;
};
```

**Things to Remember**

- To disallow functionality automatically provided by compilers, declare the corresponding member functions `= delete` and give no implementations.

## 7. Declare destructors virtual in polymorphic base classes.

There are lots of ways to keep track of time, so it would be resonable to create a `TimeKeeper` base class along with derived classes for different approaches to timekeeping:

```cpp
class TimeKeeper {
public:
    TimeKeeper();
    ~TimeKeeper();
    ...
};

class AtomicClock: public TimeKeeper {...};
class WaterClock: public TimeKeeper {...};
class WristWatch: public TimeKeeper {...};
```

Many clients will want access to the time without worrying about the details of how it's calculated, so a factory function - a function that return a base class pointer to a newly-created derived class object - can be used to return a pointer to a timekeeping object:

```cpp
TimeKeeper *getTimeKeeper(); // returns a pointer to a dynamically allocated object of a class derived from TimeKeeper
```

Consider next situation:

```cpp
TimeKeeper *ptk = getTimeKeeper(); // get dynamically allocated object from TimeKeeper hierachy
...                                // use it
delete ptk;                        // release it to avoid resource leak
```

The problem is that `getTimeKeeper` returns a pointer to a derived class object(e.g. `AtomClock`), that object is being deleted via a base class pointer(i.e., a `TimeKeeper*` pointer), and the base class (`TimeKeeper`) has a non-virtual destructor. This is a recipe for disaster, result are undefined. What typically  happens at runtime is that the derived part of the object is never destroyed.

Eliminating the problem is simple: give the base class a virtual destructor. Then deleting the derived class object will do exactly what you want. It will destory the entire object, including all its derived class parts:

```cpp
class TimeKeeper{
public:
    TimeKeeper();
    virtual ~TimeKeeper();
    ...
};
TimeKeeper *ptk = getTimeKeeper();
...
delete ptk;
```

**Things to Remember**

- Polymorphic base classes should declare virtual destructors. If a class has any virtual functions, it should have a virtual destructor.
- Classes not designed to be base classes or not designed to be used polymorphically should not declare virtual destructors.

## 8. Prevent exceptions from leaving destructors.

Consider:

```cpp
class Widget{
public:
    ...
    ~Widget(){...}  // assume this might emit an exception
};

void doSomething()
{
    std::vector<Widget> v;
}   // v is automatically destroyed here
```

When the `vector v` is destroyed, suppose `v` has ten `Widget`s in it, and during destruction of the first one, an exception is thrown. The other nine `Widget`s still have to be destroyed, so `v` should invoke their destructors. But suppose that during those calls, a second `Widget` destructor throws an exception. Now there are two simultaneously active exceptions, and that's one too many for C++. Program execution either terminates or yields undefined behavior.

That's easy enough to understand, but what should you do if your destructor needs to perform an operation that may fail by throwing an exception? For example:

```cpp
class DBConnection{
public:
    ...
    static DBConnection create();

    void close(); // close connection; throw an exception if closing fails
};
```

To ensure that clients don't forget to call `close` on `DBConnetion` objects, a reasonable idea would be create a resourse-managing class for `DBConnection` that calls `close` in its destructor.

```cpp
class DBConn {
public:
    ...
    ~DBConn()
    {
        db.close();
    }
private:
    DBConnection db;
};
```

That allows clients to program like this:

```cpp
{
    DBConn dbc(DBConnection::create());
    ...

} // at the end of block, the DBConn object is destroyed, thus automatically calling close on the DBConnection object
```

If destructor yields an exception, allow it to leave the destructor. That's a problem.

There are two primary ways to avoid the trouble:

- **Terminate the program** if `close` throws, typically by calling `abort`:

    ```cpp
    DBConn::~DBConn()
    {
        try{ db.close(); }
        catch(...){
            // make log entry that the call to close failed;
            std::abort();
        }
    }
    ```

- **Swallow the exception** arising from the call to `close`:

    ```cpp
    DBConn::~DBConn()
    {
        try{ db.close(); }
        catch(...){
            // make log entry that the call to close failed;
        }
    }
    ```

    In general, swallowing exception is a bad idea, because it suppresses important information. Sometimes, however, swallowing exception is preferable to running the risk of premature program termination or undefined behavior.

**Things to Remember**

- Destructors should never emit exceptions. If functions called in a destructor may throw, the destructor should catch any exceptions, then swallow them or terminate the program.
- If class clients need to be able to react to exceptions thrown during an operation, the class should provide a regular function that performs the operation.

