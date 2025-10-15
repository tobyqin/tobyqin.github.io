---
title: 防御性编程应该是一种习惯
categories: Tech
tags: programming
date: 2024-08-11
---

防御性编程是一种编程习惯和方法论，其核心思想是**预见并处理各种潜在的错误和异常情况**。它要求开发者在编写代码时，假设最坏的情况可能会发生——外部数据是不可信的、函数的调用者可能会传入非法参数、依赖的系统可能会失败等等。

这种编程方式的目标不是消除所有错误，而是确保当错误发生时，程序能够：

1.  **优雅地处理**：程序不会因为意外输入或内部状态错误而崩溃。
2.  **保证数据完整性**：防止错误数据污染系统。
3.  **提供清晰的错误信息**：帮助开发者快速定位和修复问题。
4.  **维持系统的安全性和稳定性**：防止因代码漏洞而被恶意利用。

简单来说，防御性编程就像是“代码的防御工事”，旨在抵御来自内部和外部的各种“攻击”和“意外”。

### 核心原则

防御性编程建立在几个关键原则之上，这些原则指导开发者如何构建健壮的软件。

#### 信任边界

明确划分系统的不同部分，并认识到跨越边界的数据是不可信的。外部输入（如用户输入、API 响应、文件内容）都必须经过严格校验。

关键实践：
- 输入验证（Validation）
- 数据清洗（Sanitization）
- 对所有外部数据源保持怀疑

#### 前置条件与后置条件

前置条件：一个方法或函数要正确执行，其调用者必须满足的条件（例如，传入的参数不能为空）。

后置条件： 一个方法或函数执行完毕后，必须满足的条件（例如，返回值不能为负数）。

关键实践：
- 断言（Assertions）
- 合约式设计（Design by Contract）
- 在方法入口检查参数，在方法出口检查结果

#### 最小权限原则

代码模块只应拥有完成其任务所必需的最小权限和信息。这限制了错误或攻击可能造成的损害范围。

关键实践：
- 避免使用全局变量
- 将成员变量设为 `private`
- 仅暴露必要的公共接口（API）

### 错误处理策略

预先定义好清晰、一致的错误处理机制。错误不应被忽略，而应被捕获、记录，并采取适当的行动。

关键实践：
- 使用异常处理（Exceptions）而非错误码
- 避免空的 `catch` 块
- 定义明确的自定义异常
- Fail-Fast（快速失败）策略

### 最佳实践与范例

下面我们将结合上述原则，通过 Java 代码示例来展示最佳实践。

#### 1\. 严格校验输入（信任边界）

永远不要相信来自外部的数据。这包括方法参数、配置文件、数据库查询结果和 API 响应。

**场景**：处理一个用户注册功能，用户名不能为空且长度必须在 6 到 20 个字符之间。

**反面范例 (Bad Practice):**

```java
public class UserService {
    public void register(String username, String password) {
        // 直接使用 username，如果为 null 会导致 NullPointerException
        // 如果为空字符串或长度不符，会写入非法数据
        System.out.println("Registering user: " + username.toLowerCase());
        // ... save to database
    }
}
```

**最佳实践 (Best Practice):**

```java
import org.apache.commons.lang3.StringUtils; // 使用 Apache Commons Lang 库简化校验

public class UserService {
    private static final int MIN_USERNAME_LENGTH = 6;
    private static final int MAX_USERNAME_LENGTH = 20;

    public void register(String username, String password) {
        // 1. 检查 null 和空字符串
        if (StringUtils.isBlank(username)) {
            throw new IllegalArgumentException("Username cannot be null or empty.");
        }
        // 2. 检查长度
        if (username.length() < MIN_USERNAME_LENGTH || username.length() > MAX_USERNAME_LENGTH) {
            throw new IllegalArgumentException("Username must be between " + MIN_USERNAME_LENGTH +
                                               " and " + MAX_USERNAME_LENGTH + " characters.");
        }
        // ... 其他校验，如密码强度

        // 只有在所有检查通过后，才继续执行核心逻辑
        System.out.println("Registering user: " + username.toLowerCase());
        // ... save to database
    }
}
```

**核心思想**：在方法的入口处设立“哨兵”，拒绝所有非法输入。这种“快速失败”（Fail-Fast）的策略可以防止非法数据污染系统深层。

#### 2\. 使用断言（Assertions）检查内部状态

断言用于检查程序中**永远不应该发生**的条件，它主要用于开发和测试阶段，以确保代码逻辑的正确性。

**场景**：一个计算折扣的方法，折扣率在逻辑上永远不应该是负数。

```java
public class PriceCalculator {
    public double applyDiscount(double price, double discountRate) {
        // 前置条件检查：对外来的参数使用异常
        if (price < 0 || discountRate < 0 || discountRate > 1) {
            throw new IllegalArgumentException("Invalid price or discount rate.");
        }

        double finalPrice = price * (1 - discountRate);

        // 后置条件断言：内部逻辑保证了 finalPrice 不可能为负
        // 如果这个断言失败，说明内部计算逻辑有严重错误
        assert finalPrice >= 0 : "Final price cannot be negative. Calculation error!";

        return finalPrice;
    }
}
```

**注意**：默认情况下 JVM 会禁用断言。你需要通过 `-enableassertions` 或 `-ea` 标志来启用它。断言不应该用于处理可预期的错误（如用户输入错误），那是异常处理的范畴。

#### 3\. 避免返回 `null`

方法返回 `null` 常常是 `NullPointerException` 的根源。调用者很容易忘记检查 `null`。

**场景**：通过 ID 查询用户信息。

**反面范例 (Bad Practice):**

```java
public class UserRepository {
    public User findById(String id) {
        // ... query database
        if (userNotFound) {
            return null; // 调用者必须处理 null
        }
        return user;
    }
}

// 调用方
User user = userRepository.findById("123");
System.out.println(user.getName()); // 如果 user 为 null，这里会抛出 NullPointerException
```

**最佳实践 (Best Practice):** 使用 `Optional` (Java 8+)

```java
import java.util.Optional;

public class UserRepository {
    public Optional<User> findById(String id) {
        // ... query database
        if (userNotFound) {
            return Optional.empty(); // 清晰地表示“值不存在”
        }
        return Optional.of(user);
    }
}

// 调用方
userRepository.findById("123")
    .ifPresent(user -> System.out.println(user.getName())); // 安全地处理存在的情况

// 或者，如果需要一个默认值
User user = userRepository.findById("123").orElse(User.GUEST_USER);
```

**核心思想**：`Optional` 强迫调用者思考“值不存在”的情况，从而使代码更加健壮。对于集合，即使没有结果也应返回空集合（如 `Collections.emptyList()`）而不是 `null`。

#### 4\. 创建不可变对象（Immutability）

不可变对象的状态在创建后无法修改，这大大降低了多线程编程的复杂性，并减少了因意外状态改变而导致的错误。

**场景**：一个表示二维坐标点的类。

**反面范例 (Mutable Class):**

```java
public class Point {
    public int x;
    public int y;
    // ... constructor
}
// 任何地方都可以修改 point 实例的状态，难以追踪和推理
```

**最佳实践 (Immutable Class):**

```java
public final class Point { // 1. 类声明为 final，防止被继承
    private final int x;   // 2. 成员变量声明为 private final
    private final int y;

    public Point(int x, int y) { // 3. 在构造函数中初始化所有字段
        this.x = x;
        this.y = y;
    }

    // 4. 只提供 getter 方法，不提供 setter
    public int getX() {
        return x;
    }

    public int getY() {
        return y;
    }

    // 5. 如果需要修改，返回一个新的对象
    public Point move(int dx, int dy) {
        return new Point(this.x + dx, this.y + dy);
    }
}
```

#### 5\. 正确处理异常

不要“吞掉”异常，也不要捕获过于宽泛的异常。

**反面范例 (Bad Practice):**

```java
try {
    // ... 一些可能抛出 IOException 或 SQLException 的代码 ...
} catch (Exception e) { // 捕获了过于宽泛的 Exception
    // 吞掉了异常，问题发生时没有任何日志，排查极其困难
}
```

**最佳实践 (Best Practice):**

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

// ...

private static final Logger logger = LoggerFactory.getLogger(MyClass.class);

try {
    // ... I/O 操作 ...
} catch (IOException e) {
    // 1. 记录详细的错误信息
    logger.error("Failed to read the configuration file.", e);
    // 2. 向上抛出自定义的、更具体的异常，让上层决定如何处理
    throw new ConfigurationException("Unable to load configuration", e);
} catch (SQLException e) {
    logger.error("Database error occurred while fetching user data.", e);
    throw new DataAccessException("Failed to access user data", e);
}
```

**核心思想**：具体地捕获异常，清晰地记录日志，并根据业务场景决定是恢复执行还是向上抛出，让调用栈上层来处理。

### 总结

防御性编程不是一项特定的技术，而是一种思维模式和编程哲学。它要求开发者保持警惕，对代码的运行环境和输入数据持怀疑态度。虽然它可能会增加一些前期的代码量（如参数校验、异常处理），但从长远来看，它能极大地提高软件的质量、稳定性和可维护性，显著减少调试和修复线上问题的时间。

将这些原则和实践融入日常开发，变成习惯，是成为一名优秀软件工程师的关键一步。
