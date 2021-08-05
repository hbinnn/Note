# Advanced googletest Topics

## More Assertions

### Explicit Success and Failure

The assertions in this section generate a success or failure directly instead of testing a value or expression. These are useful when control flow, rather than a Boolean expression, determines the test’s success or failure, as shown by the following example:

```c++
switch(expression) {
  case 1:
    ... some checks ...
  case 2:
    ... some other checks ...
  default:
    FAIL() << "We shouldn't get here.";
}
```

#### SUCCEED

`SUCCEED()`

Generates a success. This *does not* make the overall test succeed. A test is considered successful only if none of its assertions fail during its execution.

The `SUCCEED` assertion is purely documentary and currently doesn’t generate any user-visible output. However, we may add `SUCCEED` messages to GoogleTest output in the future.

#### FAIL

`FAIL()`

Generates a fatal failure, which returns from the current function.

Can only be used in functions that return `void`. See [Assertion Placement](https://google.github.io/googletest/advanced.html#assertion-placement) for more information.

#### ADD_FAILURE

`ADD_FAILURE()`

Generates a nonfatal failure, which allows the current function to continue running.

#### ADD_FAILURE_AT

`ADD_FAILURE_AT(`*`file_path`*`,`*`line_number`*`)`

Generates a nonfatal failure at the file and line number specified.

### Exception Assertions

The following assertions verify that a piece of code throws, or does not throw, an exception. Usage requires exceptions to be enabled in the build environment.

Note that the piece of code under test can be a compound statement, for example:

```
EXPECT_NO_THROW({
  int n = 5;
  DoSomething(&n);
});
```

#### EXPECT_THROW

`EXPECT_THROW(`*`statement`*`,`*`exception_type`*`)`

`ASSERT_THROW(`*`statement`*`,`*`exception_type`*`)`

Verifies that *`statement`* throws an exception of type *`exception_type`*.

#### EXPECT_ANY_THROW

`EXPECT_ANY_THROW(`*`statement`*`)`

`ASSERT_ANY_THROW(`*`statement`*`)`

Verifies that *`statement`* throws an exception of any type.

#### EXPECT_NO_THROW

`EXPECT_NO_THROW(`*`statement`*`)`

`EXPECT_NO_THROW(`*`statement`*`)`

Verifies that *`statement`* does not throw any exception.

### Predicate Assertions for Better Error Messages

Even though googletest has a rich set of assertions, they can never be complete, as it’s impossible (nor a good idea) to anticipate all scenarios a user might run into. Therefore, sometimes a user has to use `EXPECT_TRUE()` to check a complex expression, for lack of a better macro. This has the problem of not showing you the values of the parts of the expression, making it hard to understand what went wrong. As a workaround, some users choose to construct the failure message by themselves, streaming it into `EXPECT_TRUE()`. However, this is awkward especially when the expression has side-effects or is expensive to evaluate.

googletest gives you three different options to solve this problem:

#### Using an Existing Boolean Function

If you already have a function or functor that returns `bool` (or a type that can be implicitly converted to `bool`), you can use it in a *predicate assertion* to get the function arguments printed for free. See [`EXPECT_PRED*`](https://google.github.io/googletest/reference/assertions.html#EXPECT_PRED) in the Assertions Reference for details.

##### EXPECT_PRED*

`EXPECT_PRED1(pred,val1)`
`EXPECT_PRED2(pred,val1,val2)`
`EXPECT_PRED3(pred,val1,val2,val3)`
`EXPECT_PRED4(pred,val1,val2,val3,val4)`
`EXPECT_PRED5(pred,val1,val2,val3,val4,val5)`

`ASSERT_PRED1(pred,val1)`
`ASSERT_PRED2(pred,val1,val2)`
`ASSERT_PRED3(pred,val1,val2,val3)`

`ASSERT_PRED4(pred,val1,val2,val3,val4)`
`ASSERT_PRED5(pred,val1,val2,val3,val4,val5)`

Verifies that the predicate *`pred`* returns `true` when passed the given values as arguments.

The parameter *`pred`* is a function or functor that accepts as many arguments as the corresponding macro accepts values. If *`pred`* returns `true` for the given arguments, the assertion succeeds, otherwise the assertion fails.

When the assertion fails, it prints the value of each argument. Arguments are always evaluated exactly once.

As an example, see the following code:

```c++
// Returns true if m and n have no common divisors except 1.
bool MutuallyPrime(int m, int n) { ... }
...
const int a = 3;
const int b = 4;
const int c = 10;
...
EXPECT_PRED2(MutuallyPrime, a, b);  // Succeeds
EXPECT_PRED2(MutuallyPrime, b, c);  // Fails
```

In the above example, the first assertion succeeds, and the second fails with the following message:

```c++
MutuallyPrime(b, c) is false, where
b is 4
c is 10
```

Note that if the given predicate is an overloaded function or a function template, the assertion macro might not be able to determine which version to use, and it might be necessary to explicitly specify the type of the function. For example, for a Boolean function `IsPositive()` overloaded to take either a single `int` or `double` argument, it would be necessary to write one of the following:

```c++
EXPECT_PRED1(static_cast<bool (*)(int)>(IsPositive), 5);
EXPECT_PRED1(static_cast<bool (*)(double)>(IsPositive), 3.14);
```

Writing simply `EXPECT_PRED1(IsPositive, 5);` would result in a compiler error. Similarly, to use a template function, specify the template arguments:

```c++
template <typename T>
bool IsNegative(T x) {
  return x < 0;
}
...
EXPECT_PRED1(IsNegative<int>, -5);  // Must specify type for IsNegative
```

If a template has multiple parameters, wrap the predicate in parentheses so the macro arguments are parsed correctly:

```c++
ASSERT_PRED2((MyPredicate<int, int>), 5, 0);
```

#### Using a Function That Returns an AssertionResult

