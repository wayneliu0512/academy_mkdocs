# Google Mock

The key to using a mock object successfully is to set the right expectations on it. If you set the expectations too strict, your test will fail as the result of unrelated changes. If you set them too loose, bugs can slip through. You want to do it just right such that your test can catch exactly the kind of bugs you intend it to catch. Google Mock provides the necessary means for you to do it "just right."

## General Syntax

In Google Mock we use the `EXPECT_CALL()` macro to set an expectation on a mock method. The general syntax is:

```cpp
EXPECT_CALL(mock_object, method(matchers))
    .Times(cardinality)
    .WillOnce(action)
    .WillRepeatedly(action);
```

This syntax is designed to make an expectation read like English. For example, you can probably guess that

```cpp
using ::testing::Return;
...
EXPECT_CALL(turtle, GetX())
    .Times(5)
    .WillOnce(Return(100))
    .WillOnce(Return(150))
    .WillRepeatedly(Return(200));
```

## Key

- **Matchers**: What Arguments Do We Expect?
- **Cardinalities**: How Many Times Will It Be Called?
- **Actions**: What Should It Do?

## Using Multiple Expectations

By default, when a mock method is invoked, Google Mock will search the expectations in the reverse order they are defined, and stop when an active expectation that matches the arguments is found (you can think of it as "newer rules override older ones.").

```cpp
using ::testing::_;
...
EXPECT_CALL(turtle, Forward(_));  // #1
EXPECT_CALL(turtle, Forward(10))  // #2
    .Times(2);
```

## Ordered vs Unordered Calls

By default, an expectation can match a call even though an earlier expectation hasn't been satisfied. In other words, the calls don't have to occur in the order the expectations are specified.

Sometimes, you may want all the expected calls to occur in a strict order. To say this in Google Mock is easy:

```cpp
using ::testing::InSequence;
...
TEST(FooTest, DrawsLineSegment) {
  ...
  {
    InSequence dummy;

    EXPECT_CALL(turtle, PenDown());
    EXPECT_CALL(turtle, Forward(100));
    EXPECT_CALL(turtle, PenUp());
  }
  Foo();
}
```

## The Nice, the Strict, and the Naggy

If a mock method has no `EXPECT_CALL` spec but is called, Google Mock will print a warning about the "uninteresting call". The rationale is:

- New methods may be added to an interface after a test is written. We shouldn't fail a test just because a method it doesn't know about is called.
- However, this may also mean there's a bug in the test, so Google Mock shouldn't be silent either. If the user believes these calls are harmless, they can add an `EXPECT_CALL()` to suppress the warning.

However, sometimes you may want to suppress all "uninteresting call" warnings, while sometimes you may want the opposite, i.e. to treat all of them as errors. Google Mock lets you make the decision on a per-mock-object basis.

```cpp
using ::testing::NiceMock;

TEST(...) {
  NiceMock<MockFoo> mock_foo;
  EXPECT_CALL(mock_foo, DoThis());
  ... code that uses mock_foo ...
}
```

```cpp
using ::testing::StrictMock;

TEST(...) {
  StrictMock<MockFoo> mock_foo;
  EXPECT_CALL(mock_foo, DoThis());
  ... code that uses mock_foo ...

  // The test will fail if a method of mock_foo other than DoThis()
  // is called.
}
```

## Delegating Calls to a Fake

Some times you have a non-trivial fake implementation of an interface. For example:

```cpp
class Foo {
 public:
  virtual ~Foo() {}
  virtual char DoThis(int n) = 0;
  virtual void DoThat(const char* s, int* p) = 0;
};

class FakeFoo : public Foo {
 public:
  virtual char DoThis(int n) {
    return (n > 0) ? '+' :
        (n < 0) ? '-' : '0';
  }

  virtual void DoThat(const char* s, int* p) {
    *p = strlen(s);
  }
};
```

Now you want to mock this interface such that you can set expectations on it. However, you also want to use `FakeFoo` for the default behavior, as duplicating it in the mock object is, well, a lot of work.

When you define the mock class using Google Mock, you can have it delegate its default action to a fake class you already have, using this pattern:

```cpp
using ::testing::_;
using ::testing::Invoke;

class MockFoo : public Foo {
 public:
  // Normal mock method definitions using Google Mock.
  MOCK_METHOD1(DoThis, char(int n));
  MOCK_METHOD2(DoThat, void(const char* s, int* p));

  // Delegates the default actions of the methods to a FakeFoo object.
  // This must be called *before* the custom ON_CALL() statements.
  void DelegateToFake() {
    ON_CALL(*this, DoThis(_))
        .WillByDefault(Invoke(&fake_, &FakeFoo::DoThis));
    ON_CALL(*this, DoThat(_, _))
        .WillByDefault(Invoke(&fake_, &FakeFoo::DoThat));
  }
 private:
  FakeFoo fake_;  // Keeps an instance of the fake in the mock.
};
```

## Delegating Calls to a Real Object

You can use the delegating-to-real technique to ensure that your mock has the same behavior as the real object while retaining the ability to validate calls. This technique is very similar to the delegating-to-fake technique, the difference being that we use a real object instead of a fake. Here's an example:

```cpp
using ::testing::_;
using ::testing::AtLeast;
using ::testing::Invoke;

class MockFoo : public Foo {
 public:
  MockFoo() {
    // By default, all calls are delegated to the real object.
    ON_CALL(*this, DoThis())
        .WillByDefault(Invoke(&real_, &Foo::DoThis));
    ON_CALL(*this, DoThat(_))
        .WillByDefault(Invoke(&real_, &Foo::DoThat));
    ...
  }
  MOCK_METHOD0(DoThis, ...);
  MOCK_METHOD1(DoThat, ...);
  ...
 private:
  Foo real_;
};
...

  MockFoo mock;

  EXPECT_CALL(mock, DoThis())
      .Times(3);
  EXPECT_CALL(mock, DoThat("Hi"))
      .Times(AtLeast(1));
  ... use mock in test ...
```

## Combining Matchers

You can build complex matchers from existing ones using `AllOf()`, `AnyOf()`, and `Not()`:

```cpp
using ::testing::AllOf;
using ::testing::Gt;
using ::testing::HasSubstr;
using ::testing::Ne;
using ::testing::Not;
...
  // The argument must be > 5 and != 10.
  EXPECT_CALL(foo, DoThis(AllOf(Gt(5),
                                Ne(10))));

  // The first argument must not contain sub-string "blah".
  EXPECT_CALL(foo, DoThat(Not(HasSubstr("blah")),
                          NULL));
```

## Casting Matchers

Google Mock matchers are statically typed, meaning that the compiler can catch your mistake if you use a matcher of the wrong type (for example, if you use `Eq(5)` to match a `string` argument). Good for you!

Sometimes, however, you know what you're doing and want the compiler to give you some slack. One example is that you have a matcher for `long` and the argument you want to match is `int`. While the two types aren't exactly the same, there is nothing really wrong with using a `Matcher<long>` to match an `int` - after all, we can first convert the `int` argument to a `long` before giving it to the matcher.

```cpp
using ::testing::SafeMatcherCast;

// A base class and a child class.
class Base { ... };
class Derived : public Base { ... };

class MockFoo : public Foo {
 public:
  MOCK_METHOD1(DoThis, void(Derived* derived));
};
...

  MockFoo foo;
  // m is a Matcher<Base*> we got from somewhere.
  EXPECT_CALL(foo, DoThis(SafeMatcherCast<Derived*>(m)));
```

## Selecting Between Overloaded Functions

## Matching Multiple Arguments as a Whole

## Using Matchers as Predicates

## Matching Arguments that Are Not Copyable