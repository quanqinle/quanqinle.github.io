---
layout:       post
title:        "JUnit | Parameterized test in JUnit5"
subtitle:     "`@ParameterizedTest` can use different parameters and run test case multiple times in a loop"
date:         2020-08-14
updated:      2021-08-23
author:       "Quan Qinle"
lang:         en
catalog:      true
tags:
    - junit

---

In JUnit5, we can replace `@Test` with `@ParameterizedTest`, so that the test case can use different parameters and be executed several times in a loop.

<!-- more -->

# `@ValueSource`

```Java
@DisplayName("this is demo 1")
@ParameterizedTest
@ValueSource(ints = { 1, 2, 3 })
public void test_Demo1(int arg) {
    int expected = 2;
    assertEquals(expected, arg, "You are not such 2");
}

@DisplayName("this is demo 2")
@ParameterizedTest(name = "{index} ==> the testcase is running")
@ValueSource(strings = {"1", "2", "3,4"})
public void test_Demo2(String arg) {
    assertNull(arg, "Can NOT be NULL");

    String expected = "2";
    assertEquals(expected, arg, "You are not such 2");
}
```

In `@ValueSource`, the parameter name can be strings, ints, etc. However, I personally prefer to use `strings`, because the String type can be implicitly converted to many other types which makes passing arguments more flexible.

```Java
@ParameterizedTest
@ValueSource(strings = {"true", "false"})
public void test_Demo3(boolean arg) {
    assertTrue(arg);
}

@ParameterizedTest
@ValueSource(strings = {"0", "1.1", "2.2"})
public void test_Demo4(BigDecimal arg) {
    assertEquals(BigDecimal.ZERO, arg);
}
```

[Which types can a String be converted to automatically?](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests-argument-conversion-implicit)


# `@CsvSource`

```Java
@ParameterizedTest
@CsvSource({
    "  85,        体育,   true",
    "99.5,  '语,数,外',   false",
    })
public void test_Demo5(BigDecimal score, String subject, boolean isWin) {

}
```

## `ArgumentsAccessor`, the parameter aggregator

By `ArgumentsAccessor`, multiple parameters can be received at once.

```java
@ParameterizedTest
@CsvSource({
    "  85,        体育,   true",
    "99.5,  '语,数,外',   false",
    })
public void test_Demo6(ArgumentsAccessor args) {
    args.get(0 , BigDecimal.class);
    args.get(1, String.class); // OR args.getString(1);
    args.get(2, Boolean.class);
}
```

[ArgumentsAccessor doc api](https://junit.org/junit5/docs/current/api/org.junit.jupiter.params/org/junit/jupiter/params/aggregator/ArgumentsAccessor.html)


# `@CsvFileSource`

The difference with '@csvsource' is reading test data from CSV file, however, the same is how to pass parameters and others.

```java
@ParameterizedTest
@CsvFileSource(resources = "/login-data.csv", numLinesToSkip = 1)
void test_Demo7(ArgumentsAccessor args) {
}
```
