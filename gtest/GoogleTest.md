# GoogleTest Primer

## Introduction: Why googletest?

> googletest helps you write better C++ tests.

`googletest` 是测试技术团队开发的一个测试框架，考虑到了谷歌的具体要求和限制。无论您是在 `Linux`、`Windows` 还是 `Mac` 上工作，如果您编写 `C++` 代码，`googletest` 都可以为您提供帮助。它支持任何类型的测试，而不仅仅是单元测试。

So what makes a good test, and how does googletest fit in? We believe:

1. Tests should be *independent* and *repeatable*. 调试由于其他测试而成功或失败的测试是一件痛苦的事情。 `googletest` 通过在不同的对象上运行每个测试来隔离测试。当测试失败时，`googletest` 允许您单独运行它以进行快速调试。
2. Tests should be well *organized* and reflect the structure of the tested code. `googletest` 将相关测试分组为可以共享数据和子程序的 test suites。这种常见模式很容易识别，并使测试易于维护。当人们切换项目并开始处理新的代码库时，这种一致性尤其有用。
3. Tests should be *portable* and *reusable*. `Google` 有很多与平台无关的代码；它的测试也应该是平台中立的。 `googletest` 适用于不同的操作系统，使用不同的编译器，有或没有例外，因此 `googletest` 测试可以使用各种配置。
4. When tests fail, they should provide as much *information* about the problem as possible.  `googletest` 不会在第一次测试失败时停止。 相反，它只会停止当前测试并继续下一个测试。 您还可以设置报告非致命故障的测试，之后当前测试继续。 因此，您可以在单个运行-编辑-编译周期中检测和修复多个错误。
5. The testing framework should liberate test writers from housekeeping chores and let them focus on the test *content*. `googletest` 自动跟踪定义的所有测试，并且不需要用户枚举它们来运行它们。
6. Tests should be *fast*.  使用 `googletest`，you can reuse shared resources across tests and pay for the set-up/tear-down only once，without making tests depend on each other.

## Beware of the nomenclature

Historically, googletest started to use the term *Test Case* for grouping related tests, whereas current publications, including International Software Testing Qualifications Board ([ISTQB](http://www.istqb.org/)) materials and various textbooks on software quality, use the term *[Test Suite](http://glossary.istqb.org/en/search/test suite)* for this.

The related term *Test*, as it is used in googletest, corresponds to the term *[Test Case](http://glossary.istqb.org/en/search/test case)* of ISTQB and others.

The term *Test* is commonly of broad enough sense, including ISTQB’s definition of *Test Case*, so it’s not much of a problem here. But the term *Test Case* as was used in Google Test is of contradictory sense and thus confusing.

googletest recently started replacing the term *Test Case* with *Test Suite*. The preferred API is *TestSuite*. The older TestCase API is being slowly deprecated and refactored away.

So please be aware of the different definitions of the terms:

| Meaning                                                      | googletest Term                                              | [ISTQB](http://www.istqb.org/) Term                        |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :--------------------------------------------------------- |
| Exercise a particular program path with specific input values and verify the results | [TEST()](http://google.github.io/googletest/primer.html#simple-tests) | [Test Case](http://glossary.istqb.org/en/search/test case) |

## Basic Concepts

使用 `googletest` 时，首先要编写断言，断言是检查条件是否为真的语句。断言的结果可以是成功、非致命失败或致命失败。如果发生致命故障，则中止当前功能；否则程序将继续正常运行。

测试使用断言来验证被测试代码的行为。如果测试崩溃或断言失败，则它失败；否则它会成功。

一个 *test suite* 包含一个或多个测试。您应该将测试分组到反映测试代码结构的 *test suite* 中。当 *test suite* 中的多个测试需要共享公共对象和子例程时，您可以将它们放入一个 *test fixture* class 中。

一个测试程序可以包含多个 test suites。

我们现在将解释如何编写测试程序，从单个断言级别开始，然后构建测试和测试套件。

## Assertions

`googletest` 断言是类似于函数调用的宏。您可以通过对其行为进行断言来测试类或函数。当断言失败时，`googletest` 会打印断言的源文件和行号位置，以及一条失败消息。您还可以提供自定义失败消息，该消息将附加到 `googletest` 的消息中。

The assertions come in pairs that test the same thing but have different effects on the current function. `ASSERT_*` 版本在失败时会产生致命的失败，并中止当前函数。 `EXPECT_*` 版本生成非致命故障，不会中止当前函数。通常首选 `EXPECT_*`，因为它们允许在测试中报告多个失败。但是，如果在相关断言失败时继续操作没有意义，您应该使用 `ASSERT_*`。

由于失败的 `ASSERT_*` 立即从当前函数返回，可能会跳过后面的清理代码，因此可能会导致空间泄漏。Depending on the nature of the leak, it may or may not be worth fixing - so keep this in mind if you get a heap checker error in addition to assertion errors.

 要提供自定义失败消息，只需使用 &lt;&lt; 运算符或一系列此类运算符将其流式传输到宏中。 请参阅以下示例，使用 [`ASSERT_EQ` and `EXPECT_EQ`](http://google.github.io/googletest/reference/assertions.html#EXPECT_EQ) 宏来验证值相等：

```c++
ASSERT_EQ(x.size(), y.size()) << "Vectors x and y are of unequal length";

for (int i = 0; i < x.size(); ++i) {
  EXPECT_EQ(x[i], y[i]) << "Vectors x and y differ at index " << i;
}
```

Anything that can be streamed to an `ostream` can be streamed to an assertion macro–in particular, C strings and `string` objects.  If a wide string (`wchar_t*`, `TCHAR*` in `UNICODE` mode on Windows, or `std::wstring`) is streamed to an assertion, it will be translated to UTF-8 when printed.

`GoogleTest` 提供了一系列断言，用于以各种方式验证代码的行为。您可以检查布尔条件、基于关系运算符比较值、验证字符串值、浮点值等等。甚至还有断言使您能够通过提供自定义谓词来验证更复杂的状态。有关 `GoogleTest` 提供的断言的完整列表，请参阅[Assertions Reference](http://google.github.io/googletest/reference/assertions.html).

## Simple Tests

To create a test:

1. 使用 `TEST()` 宏来定义和命名测试函数。这些是不返回值的普通 `C++` 函数。
2. 在此函数中，连同您想要包含的任何有效 `C++` 语句，使用各种 `googletest` 断言来检查值。
3. 测试的结果由断言决定；如果测试中的任何断言失败（致命或非致命），或者如果测试崩溃，则整个测试失败。否则，它成功。

```c++
TEST(TestSuiteName, TestName) {
  ... test body ...
}
```

`TEST()` 参数从一般到具体。第一个参数是 test suite 的名称，第二个参数是test suite 中的测试名称。两个名称都必须是有效的 `C++` 标识符，并且不应包含任何下划线 (`_`)。 A test’s *full name* consists of its containing test suite and its individual name. Tests from different test suites can have the same individual name.

For example, let’s take a simple integer function:

```c++
int Factorial(int n);  // Returns the factorial of n
```

A test suite for this function might look like:

```c++
// Tests factorial of 0.
TEST(FactorialTest, HandlesZeroInput) {
  EXPECT_EQ(Factorial(0), 1);
}

// Tests factorial of positive numbers.
TEST(FactorialTest, HandlesPositiveInput) {
  EXPECT_EQ(Factorial(1), 1);
  EXPECT_EQ(Factorial(2), 2);
  EXPECT_EQ(Factorial(3), 6);
  EXPECT_EQ(Factorial(8), 40320);
}
```

googletest groups the test results by test suites, so logically related tests should be in the same test suite; in other words, the first argument to their `TEST()` should be the same. In the above example, we have two tests, `HandlesZeroInput` and `HandlesPositiveInput`, that belong to the same test suite `FactorialTest`.

When naming your test suites and tests, you should follow the same convention as for [naming functions and classes](https://google.github.io/styleguide/cppguide.html#Function_Names).

可用性：Linux、Windows、Mac。

## Test Fixtures: Using the Same Data Configuration for Multiple Tests

如果您发现自己编写了两个或多个对类似数据进行操作的测试，则可以使用 *test fixture*。This allows you to reuse the same configuration of objects for several different tests.

To create a fixture:

1. 从 `::testing::Test` 派生一个类。以 `protected:` 开始它的主体，as we’ll want to access fixture members from sub-classes.
2. Inside the class, declare any objects you plan to use.
3. 如有必要，编写一个默认构造函数或 `SetUp()` 函数来为每个测试准备对象。一个常见的错误是将 `SetUp() `拼写为带有小 `u` 的 `Setup()` - Use `override` in C++11 to make sure you spelled it correctly.
4. 如有必要，编写一个析构函数或 `TearDown()` 函数来释放您在 `SetUp()` 中分配的任何资源。要了解何时应该使用构造函数/析构函数以及何时应该使用 `SetUp()/TearDown()`，请阅读[FAQ](http://google.github.io/googletest/faq.html#CtorVsSetUp).。
5. If needed, define subroutines for your tests to share.

When using a fixture, use `TEST_F()` instead of `TEST()` as it allows you to access objects and subroutines in the test fixture:

```c++
TEST_F(TestFixtureName, TestName) {
  ... test body ...
}
```

与 `TEST()` 一样，the first argument is the test suite name, but for `TEST_F()` this must be the name of the test fixture class. You’ve probably guessed: `_F` is for fixture.

Unfortunately, the C++ macro system does not allow us to create a single macro that can handle both types of tests. Using the wrong macro causes a compiler error.

Also, you must first define a test fixture class before using it in a `TEST_F()`, or you’ll get the compiler error “`virtual outside class declaration`”.

For each test defined with `TEST_F()`, googletest will create a *fresh* test fixture at runtime, immediately initialize it via `SetUp()`, run the test, clean up by calling `TearDown()`, and then delete the test fixture. Note that different tests in the same test suite have different test fixture objects, and googletest always deletes a test fixture before it creates the next one. googletest does **not** reuse the same test fixture for multiple tests. Any changes one test makes to the fixture do not affect other tests.

作为 例如，让我们为名为 `Queue` 的 `FIFO` 队列类编写测试，该类具有以下接口：

```c++
template <typename E>  // E is the element type.
class Queue {
 public:
  Queue();
  void Enqueue(const E& element);
  E* Dequeue();  // Returns NULL if the queue is empty.
  size_t size() const;
  ...
};
```

First, define a fixture class. By convention, you should give it the name `FooTest` where `Foo` is the class being tested.

```c++
class QueueTest : public ::testing::Test {
 protected:
  void SetUp() override {
     q1_.Enqueue(1);
     q2_.Enqueue(2);
     q2_.Enqueue(3);
  }

  // void TearDown() override {}

  Queue<int> q0_;
  Queue<int> q1_;
  Queue<int> q2_;
};
```

在这种情况下，不需要 `TearDown()`，因为我们不必在每次测试后进行清理，除了析构函数已经完成的操作。

Now we’ll write tests using `TEST_F()` and this fixture.

```c++
TEST_F(QueueTest, IsEmptyInitially) {
  EXPECT_EQ(q0_.size(), 0);
}

TEST_F(QueueTest, DequeueWorks) {
  int* n = q0_.Dequeue();
  EXPECT_EQ(n, nullptr);

  n = q1_.Dequeue();
  ASSERT_NE(n, nullptr);
  EXPECT_EQ(*n, 1);
  EXPECT_EQ(q1_.size(), 0);
  delete n;

  n = q2_.Dequeue();
  ASSERT_NE(n, nullptr);
  EXPECT_EQ(*n, 2);
  EXPECT_EQ(q2_.size(), 1);
  delete n;
}
```

以上使用了 `ASSERT_*` 和 `EXPECT_*` 断言。经验法则是，当您希望测试在断言失败后继续显示更多错误时使用 `EXPECT_*`，并在失败后继续运行没有意义时使用 `ASSERT_*`。例如，`Dequeue` 测试中的第二个断言是 `ASSERT_NE(n, nullptr)`，因为我们稍后需要取消对指针`n` 的引用，这会在 `n` 为 `NULL` 时导致段错误。

When these tests run, the following happens:

1. googletest constructs a `QueueTest` object (let’s call it `t1`).
2. `t1.SetUp()` initializes `t1`.
3. The first test (`IsEmptyInitially`) runs on `t1`.
4. `t1.TearDown()` cleans up after the test finishes.
5. `t1` is destructed.
6. The above steps are repeated on another `QueueTest` object, this time running the `DequeueWorks` test.

可用性：Linux、Windows、Mac。

## Invoking the Tests

`TEST()` and `TEST_F()` implicitly register their tests with googletest. So, unlike with many other C++ testing frameworks, you don’t have to re-list all your defined tests in order to run them.

After defining your tests, you can run them with `RUN_ALL_TESTS()`, which returns `0` if all the tests are successful, or `1` otherwise. Note that `RUN_ALL_TESTS()` runs *all tests* in your link unit–they can be from different test suites, or even different source files.

When invoked, the `RUN_ALL_TESTS()` macro:

* Saves the state of all googletest flags.
* Creates a test fixture object for the first test.
* Initializes it via `SetUp()`.
* Runs the test on the fixture object.
* Cleans up the fixture via `TearDown()`.
* Deletes the fixture.
* Restores the state of all googletest flags.
* Repeats the above steps for the next test, until all tests have run.

如果发生致命故障，将跳过后续步骤。

> 重要提示：您不能忽略 RUN_ALL_TESTS() 的返回值，否则您将收到编译器错误。这种设计的基本原理是自动化测试服务根据其退出代码而不是其 stdout/stderr 输出来确定测试是否通过；因此您的 main() 函数必须返回 RUN_ALL_TESTS() 的值。
>
> 此外，您应该只调用 RUN_ALL_TESTS() 一次。多次调用它会与一些高级 googletest 功能（例如，线程安全的死亡测试）发生冲突，因此不受支持。

## Writing the main() Function

Most users should *not* need to write their own `main` function and instead link with `gtest_main` (as opposed to with `gtest`), which defines a suitable entry point. See the end of this section for details. The remainder of this section should only apply when you need to do something custom before the tests run that cannot be expressed within the framework of fixtures and test suites.

If you write your own `main` function, it should return the value of `RUN_ALL_TESTS()`.

You can start from this boilerplate:

```c++
#include "this/package/foo.h"

#include "gtest/gtest.h"

namespace my {
namespace project {
namespace {

// The fixture for testing class Foo.
class FooTest : public ::testing::Test {
 protected:
  // You can remove any or all of the following functions if their bodies would
  // be empty.

  FooTest() {
     // You can do set-up work for each test here.
  }

  ~FooTest() override {
     // You can do clean-up work that doesn't throw exceptions here.
  }

  // If the constructor and destructor are not enough for setting up
  // and cleaning up each test, you can define the following methods:

  void SetUp() override {
     // Code here will be called immediately after the constructor (right
     // before each test).
  }

  void TearDown() override {
     // Code here will be called immediately after each test (right
     // before the destructor).
  }

  // Class members declared here can be used by all tests in the test suite
  // for Foo.
};

// Tests that the Foo::Bar() method does Abc.
TEST_F(FooTest, MethodBarDoesAbc) {
  const std::string input_filepath = "this/package/testdata/myinputfile.dat";
  const std::string output_filepath = "this/package/testdata/myoutputfile.dat";
  Foo f;
  EXPECT_EQ(f.Bar(input_filepath, output_filepath), 0);
}

// Tests that Foo does Xyz.
TEST_F(FooTest, DoesXyz) {
  // Exercises the Xyz feature of Foo.
}

}  // namespace
}  // namespace project
}  // namespace my

int main(int argc, char **argv) {
  ::testing::InitGoogleTest(&argc, argv);
  return RUN_ALL_TESTS();
}
```

The `::testing::InitGoogleTest()` function parses the command line for googletest flags, and removes all recognized flags. This allows the user to control a test program’s behavior via various flags, which we’ll cover in the [AdvancedGuide](http://google.github.io/googletest/advanced.html). You **must** call this function before calling `RUN_ALL_TESTS()`, or the flags won’t be properly initialized.

On Windows, `InitGoogleTest()` also works with wide strings, so it can be used in programs compiled in `UNICODE` mode as well.

But maybe you think that writing all those `main` functions is too much work? We agree with you completely, and that’s why Google Test provides a basic implementation of main(). If it fits your needs, then just link your test with the `gtest_main` library and you are good to go.

> NOTE: `ParseGUnitFlags()` is deprecated in favor of `InitGoogleTest()`.

## Known Limitations

Google Test 被设计为线程安全的。该实现在 `pthreads` 库可用的系统上是线程安全的。目前在其他系统（例如 `Windows`）上同时使用来自两个线程的 Google 测试断言是不安全的。在大多数测试中，这不是问题，因为通常断言是在主线程中完成的。If you want to help, you can volunteer to implement the necessary synchronization primitives in `gtest-port.h` for your platform.