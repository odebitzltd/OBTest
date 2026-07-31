# OBTest — Black-Box Automated JUnit Testing Framework

[![License](https://img.shields.io/badge/License-MIT_with_Commons_Clause-blue.svg)](LICENSE)
[![Java Version](https://img.shields.io/badge/Java-8%2B-orange.svg)](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)

**OBTest** is a lightweight, zero-dependency Java library designed to automate the process of black-box testing for public and protected methods. By analyzing method signatures, OBTest dynamically generates comprehensive sets of positive, negative, and edge-case inputs to test methods up to their breaking point.

Everything runs entirely **locally and offline** with zero external network calls.

---

## Key Features

1. **Fully Automated Input Generation**: Generates type-specific boundary values, null injections, special character strings, SQL/XSS injections, and extreme numbers.
2. **Combinatorial Testing & Pairwise Reduction**: Intelligently scales from full Cartesian products to pairwise coverage (all-pairs testing) to prevent combinatorial explosion.
3. **No Reflection Mutation**: Restricts testing to `public` and `protected` classes/methods. It never modifies field or method access modifiers (`setAccessible(true)` is only used to invoke otherwise visible `protected` methods).
4. **Zero Runtime Dependencies**: The compiled JAR has zero runtime dependencies, preventing classpath pollution in your main project.
5. **Java 8+ Compatibility**: Built to run on any Java version from Java 8 all the way to modern Java versions.
6. **Robust Sandboxed Execution**: Includes built-in execution isolation, per-scenario timeouts, and unchecked exception reporting.

---

## Installation

Add OBTest as a test-scoped dependency to your project.

### Maven (`pom.xml`)

```xml
<dependency>
    <groupId>com.odebitz.oss</groupId>
    <artifactId>obtest</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <scope>test</scope>
</dependency>
```

### Gradle (`build.gradle`)

```groovy
testImplementation 'com.odebitz.oss:obtest:1.0.0-SNAPSHOT'
```

---

## Compatibility

OBTest is designed to run seamlessly across legacy and modern Java environments.

### Java Version Matrix

| Java Version | Support Status | Notes |
| :--- | :--- | :--- |
| **Java 8 (LTS)** | ✅ Fully Supported | Baseline compiled target (`-target 1.8`) |
| **Java 11 (LTS)** | ✅ Fully Supported | Fully compatible |
| **Java 17 (LTS)** | ✅ Fully Supported | Fully compatible |
| **Java 21 (LTS)** | ✅ Fully Supported | Fully compatible |
| **Java 22+** | ✅ Fully Supported | Compatible with modern JDK releases |

* **Zero Runtime Dependencies**: No external runtime libraries, preventing classpath and version conflicts.
* **Module System Friendly (Java 9+ JPMS)**: Tests `public` and `protected` APIs without illegal reflective access into internal JDK modules.
* **Code Coverage (JaCoCo)**: Fully compatible with JaCoCo and other JVM code coverage tools. Executing tests through `OBTest` records complete line and branch coverage in JaCoCo reports.

### Transitive Dependencies

OBTest maintains a **100% clean dependency tree** with **zero runtime transitive dependencies**.

```
com.odebitz.oss:obtest:jar:1.0.0-SNAPSHOT
└── (0 runtime dependencies)
```

* **Runtime Scope**: `0` dependencies. Adding `obtest` to your project will never pull in third-party libraries (e.g. Jackson, Guava, Commons-Lang, ByteBuddy, ASM, or SLF4J), guaranteeing zero classpath pollution or version collision issues.
* **Provided / Test Scope**: JUnit 4 (scope `provided`) is referenced solely for interface compatibility; consumers bring their own test runner (JUnit 4, JUnit 5, or TestNG).

---

## Usage Guide

### 1. Basic Instance Method Testing
For methods that do not have external dependencies, instantiate the class and pass it to `OBTest.forInstance()`.

```java
import com.odebitz.oss.obtest.OBTest;
import org.junit.Test;

public class CalculatorTest {

    @Test
    public void testAdd() {
        OBTest.forInstance(new Calculator())
            .method("add", int.class, int.class)
            .returnsType(int.class)
            .run();
    }
}
```

### 2. Static Method Testing
Use `OBTest.forClass()` to test static utility methods.

```java
import com.odebitz.oss.obtest.OBTest;
import org.junit.Test;

public class StringUtilsTest {

    @Test
    public void testReverse() {
        OBTest.forClass(StringUtils.class)
            .method("reverse", String.class)
            .returnsType(String.class)
            .run();
    }
}
```

### 3. Testing with Mocked Dependencies (Recommended)
If the class under test requires database repositories, HTTP clients, or other services, create the mock objects using JUnit/Mockito, inject them into the target class constructor, and pass the resulting instance to `OBTest`.

```java
import com.odebitz.oss.obtest.OBTest;
import org.junit.Before;
import org.junit.Test;
import org.mockito.Mockito;
import static org.mockito.Mockito.*;

public class UserServiceTest {

    private UserRepository mockRepo;
    private UserService userService;

    @Before
    public void setUp() {
        mockRepo = mock(UserRepository.class);
        // Wire up stubbing behavior for positive flow
        when(mockRepo.save(any(User.class))).thenReturn(new User("saved-user"));
        
        // Construct the object under test
        userService = new UserService(mockRepo);
    }

    @Test
    public void testCreateUser() {
        OBTest.forInstance(userService)
            .method("createUser", String.class, String.class) // username, email
            .returnsType(User.class)
            .run();
    }
}
```

### 4. Registering Custom Value Providers
For complex domain objects (e.g. custom classes), register a `ValueProvider` to generate test data.

```java
import com.odebitz.oss.obtest.OBTest;
import com.odebitz.oss.obtest.generator.TestValue;
import com.odebitz.oss.obtest.generator.provider.ValueProvider;
import org.junit.Test;
import java.util.Arrays;
import java.util.List;

public class AccountServiceTest {

    @Test
    public void testProcessAccount() {
        OBTest.forInstance(new AccountService())
            .method("process", Account.class)
            .withValueProvider(Account.class, new ValueProvider<Account>() {
                @Override
                public List<TestValue<Account>> generate() {
                    return Arrays.asList(
                        TestValue.positive(new Account("savings", 1000)),
                        TestValue.negative(new Account("invalid", -100)),
                        TestValue.edgeCase(null)
                    );
                }

                @Override
                public Class<Account> getTargetType() {
                    return Account.class;
                }
            })
            .run();
    }
}
```

### 5. Configuring Test Rules
You can customize the test execution behavior, such as maximum generated combinations, timeouts, or strict failure modes.

```java
OBTest.forInstance(new Calculator())
    .method("divide", double.class, double.class)
    .config(cfg -> cfg
        .maxCombinations(500)            // switch to pairwise if combinations exceed 500
        .timeout(2000)                   // fail individual scenario if it takes > 2000ms
        .failOnUncheckedExceptions(true) // treat unexpected RuntimeExceptions as failures
        .verbose(true)                   // print all successful test scenarios to the console
    )
    .run();
```

---

## Execution Reports

OBTest generates styled console output showing the classification of each test execution.

* **PASSED (Green)**: The method completed normally, or threw a declared checked exception.
* **WARNING (Yellow)**: The method threw an undeclared RuntimeException (e.g. `NullPointerException`, `IndexOutOfBoundsException`). These represent potential bugs but do not fail the build by default unless `failOnUncheckedExceptions(true)` is set.
* **FAILED (Red)**: The method execution timed out, or strict mode is enabled and it threw an unexpected exception.

### Sample Console Output

```
==================================================================
  OBTest Report
  SampleStringUtils.reverse(String) → String
==================================================================

  Total: 16  |  ✅ Passed: 15  |  ⚠️ Warnings: 1  |  Time: 4ms

  ------------------------------------------------------------
  ✅ POSITIVE   ("hello") -> olleh [1ms]
  ✅ POSITIVE   ("test123") -> 321tset [0ms]
  ✅ POSITIVE   ("Hello World") -> dlroW olleH [0ms]
  ✅ POSITIVE   ("a") -> a [0ms]
  ⚠️ NEGATIVE   (null) -> threw NullPointerException: Input string cannot be null [0ms]
  ✅ NEGATIVE   ("") ->  [0ms]
  ✅ NEGATIVE   ("   ") ->     [0ms]
  
  RESULT: PASSED WITH WARNINGS
==================================================================
```

---

## Design and Philosophy

### 1. Primitive Null Filtering
In Java, primitive variables (such as `int`, `double`, `boolean`) can never hold `null`. OBTest automatically detects primitive parameter types and filters out `null` arguments from the test suite to prevent invalid reflection dispatch failures. Wrapper classes (such as `Integer`, `Double`, `Boolean`) continue to be tested with `null` values.

### 2. Combinatorial Explosion Management
For methods with many parameters, executing a complete Cartesian product of all boundary values is unfeasible. 
* If the total number of combinations is below `maxCombinations` (default: 1000), a full **Cartesian product** is generated.
* If it exceeds the limit, OBTest automatically switches to a greedy **pairwise combination** algorithm (All-Pairs testing) to guarantee that every possible pair of parameter values is tested together in at least one scenario.

---

## License

This project is licensed under the MIT License with Commons Clause 1.0. Commercial reselling or offering this work as a paid service is prohibited. See the `LICENSE` file for details.
