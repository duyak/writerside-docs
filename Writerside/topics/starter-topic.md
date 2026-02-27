# Hướng dẫn JUnit 5 cho Java

## Mục tiêu
Tài liệu này giúp bạn:
- Cài JUnit 5 (Maven/Gradle)
- Viết test đầu tiên
- Nắm annotations, assertions, lifecycle
- Làm quen parameterized test và kiểm tra ngoại lệ

## Yêu cầu
- Java 17+ (khuyến nghị)
- Maven hoặc Gradle
- IDE bất kỳ (IntelliJ IDEA/Eclipse/VS Code)

## Cài đặt

### Maven
Thêm dependencies vào `pom.xml`:

```xml
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.2.5</version>
        </plugin>
    </plugins>
</build>
```

### Gradle (Kotlin DSL)
Thêm vào `build.gradle.kts`:

```kotlin
dependencies {
    testImplementation("org.junit.jupiter:junit-jupiter:5.10.2")
}

tasks.test {
    useJUnitPlatform()
}
```

## Viết test đầu tiên
Ví dụ lớp cần test:

```java
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
}
```

Test đơn giản:

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;

class CalculatorTest {
    @Test
    void add_twoNumbers_returnsSum() {
        Calculator calculator = new Calculator();
        assertEquals(5, calculator.add(2, 3));
    }
}
```

## Annotations thường dùng
- `@Test`: đánh dấu một test case.
- `@BeforeEach` / `@AfterEach`: chạy trước/sau mỗi test.
- `@BeforeAll` / `@AfterAll`: chạy một lần trước/sau tất cả test (phải `static`).
- `@Disabled`: tạm thời bỏ qua test.

Ví dụ:

```java
import org.junit.jupiter.api.*;

class LifecycleTest {
    @BeforeAll
    static void beforeAll() {
        // Khởi tạo tài nguyên dùng chung
    }

    @BeforeEach
    void setUp() {
        // Chuẩn bị dữ liệu cho từng test
    }

    @Test
    void sampleTest() {
        Assertions.assertTrue(true);
    }

    @AfterEach
    void tearDown() {
        // Dọn dẹp sau mỗi test
    }

    @AfterAll
    static void afterAll() {
        // Giải phóng tài nguyên
    }
}
```

## Assertions cơ bản
Các hàm hay dùng:
- `assertEquals(expected, actual)`
- `assertTrue(condition)` / `assertFalse(condition)`
- `assertNotNull(value)` / `assertNull(value)`
- `assertThrows(Exception.class, executable)`

Ví dụ kiểm tra ngoại lệ:

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertThrows;

class ExceptionTest {
    @Test
    void divide_byZero_throwsException() {
        assertThrows(ArithmeticException.class, () -> {
            int result = 10 / 0;
        });
    }
}
```

## Parameterized Tests
Dùng khi muốn chạy cùng một test với nhiều bộ dữ liệu.

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;

import static org.junit.jupiter.api.Assertions.assertEquals;

class ParameterizedCalculatorTest {
    @ParameterizedTest
    @CsvSource({
        "1, 2, 3",
        "2, 3, 5",
        "3, 5, 8"
    })
    void add_manyCases(int a, int b, int expected) {
        Calculator calculator = new Calculator();
        assertEquals(expected, calculator.add(a, b));
    }
}
```

## Chạy test
- Trong IDE: click Run ở class test.
- Maven: `mvn test`
- Gradle: `./gradlew test`

## Lỗi thường gặp
- Quên `useJUnitPlatform()` trong Gradle.
- Không dùng `static` cho `@BeforeAll/@AfterAll`.
- Lẫn JUnit 4 và JUnit 5 (imports sai).

## Tham khảo
- https://junit.org/junit5/docs/current/user-guide/
- https://maven.apache.org/surefire/maven-surefire-plugin/
- https://docs.gradle.org/current/userguide/java_testing.html
