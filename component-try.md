# Try 组件使用规则

> 强制性约束 · 组件使用规范 · 全局适用

---

## 1. 规则目的

本规则定义了 Try 组件的使用规范。Try 是 gonbay-core 项目中提供的一个实用工具类，位于 `net.gonbay.common.exception` 包中，主要用于简化异常处理流程，提供了一种优雅的方式来处理可能抛出异常的操作，避免了大量的 try-catch 语句块。AI 编码助手在处理可能抛出异常的操作时，必须严格遵守本规则，使用 Try 工具类进行异常处理。

---

## 2. 核心约束

### 2.1 必须遵守的规则

**必须使用**
- ✅ 所有可能抛出异常的操作必须使用 `Try` 工具类处理
- ✅ 所有异常处理必须使用 `Try.execute()` 或 `Try.get()` 方法
- ✅ 所有需要回退机制的操作必须使用 `Try.getOr()` 方法
- ✅ 所有断言验证必须使用 `Try.predicate()` 方法

**禁止行为**
- ❌ 禁止使用 try-catch 语句块处理异常（应使用 Try 工具类）
- ❌ 禁止捕获异常后不处理（空的 catch 块）
- ❌ 禁止使用通用的 Exception 而不是具体异常类型
- ❌ 禁止在业务代码中直接抛出异常（应使用 Try 工具类统一处理）

---

## 3. Maven 依赖配置

### 3.1 依赖引入

```xml
<dependency>
    <groupId>net.gonbay</groupId>
    <artifactId>gonbay-common</artifactId>
    <version>最新版本</version>
</dependency>
```

### 3.2 导入类

```java
import net.gonbay.common.exception.Try;
import net.gonbay.common.exception.ErrorCode;
```

**规则**
- ✅ 必须引入 `gonbay-common` 依赖
- ✅ 必须导入 `Try` 和 `ErrorCode` 类

---

## 4. 方法使用规范

### 4.1 execute 方法 - 执行操作

#### 4.1.1 基础用法

```java
public <T> void execute(ExecuteAction action, ErrorCode code)
```

**功能**：执行某个可能抛出异常的操作，如果发生异常则根据指定的错误码抛出 SystemException。

**参数**：
- `action`: ExecuteAction 接口，包含可能抛出异常的操作
- `code`: ErrorCode，异常时使用的错误码

**使用场景**：适用于不需要返回值的可能抛异常操作。

**示例代码**：

```java
// 安全执行可能抛出异常的文件操作
Try.execute(
    () -> { 
        // 可能抛出异常的操作
        someRiskyOperation(); 
    },
    BasicErrorCode.SERVER_ERROR
);
```

#### 4.1.2 带错误携带参数

```java
public <T> void execute(ExecuteAction action, ErrorCode code, Object errorTakeObj)
```

**功能**：执行某个可能抛出异常的操作，如果发生异常则根据指定的错误码和携带参数抛出 SystemException。

**参数**：
- `action`: ExecuteAction 接口，包含可能抛出异常的操作
- `code`: ErrorCode，异常时使用的错误码
- `errorTakeObj`: Object，异常时携带的附加信息

**示例代码**：

```java
// 执行操作并携带额外错误信息
String fileName = "example.txt";
Try.execute(
    () -> { 
        // 可能抛出异常的操作
        processFile(fileName); 
    },
    BasicErrorCode.SERVER_ERROR,
    fileName // 错误时携带文件名信息
);
```

**规则**
- ✅ 所有可能抛出异常且不需要返回值的操作必须使用 `Try.execute()`
- ✅ 必须提供明确的错误码
- ✅ 需要传递额外错误信息时使用带 `errorTakeObj` 参数的方法

---

### 4.2 get 方法 - 获取结果

#### 4.2.1 基础用法

```java
public <T> T get(ResultAction<T> action, ErrorCode code)
```

**功能**：执行可能抛出异常的操作并获取结果，如果发生异常则根据指定错误码抛出 SystemException。

**参数**：
- `action`: ResultAction<T>，可能抛出异常的操作
- `code`: ErrorCode，异常时使用的错误码

**返回值**：操作的结果

**使用场景**：适用于需要返回值且有明确异常处理的场景。

**示例代码**：

```java
// 在 FormConditionUtil 中解析枚举值
private JoinSymbol analyzeJoinSymbol(String joinExpr) {
    return Try.get(
        () -> JoinSymbol.valueOf(joinExpr), 
        ErrorCode.builderModuleErrorMessage("解析连接符错误"), 
        joinExpr
    );
}
```

#### 4.2.2 带错误携带参数

```java
public <T> T get(ResultAction<T> action, ErrorCode code, Object errorTakeObj)
```

**功能**：执行可能抛出异常的操作并获取结果，如果发生异常则根据指定错误码和携带参数抛出 SystemException。

**示例代码**：

```java
// 使用自定义错误码
ErrorCode customError = ErrorCode.builder("CUSTOM_ERROR", "自定义错误信息", 400);

String result = Try.get(
    () -> riskyOperation(),
    customError,
    "附加错误信息"
);
```

**规则**
- ✅ 所有可能抛出异常且需要返回值的操作必须使用 `Try.get()`
- ✅ 必须提供明确的错误码
- ✅ 需要传递额外错误信息时使用带 `errorTakeObj` 参数的方法

---

### 4.3 getOr 方法 - 回退机制

#### 4.3.1 方法签名

```java
public <T> T getOr(ResultAction<T> primry, FailBackResultAction<T> fallback)
```

**功能**：执行主操作，如果主操作失败则执行回退操作。

**参数**：
- `primry`: ResultAction<T>，主要操作，可能抛出异常
- `fallback`: FailBackResultAction<T>，回退操作，在主操作失败时执行

**返回值**：主操作或回退操作的结果

**使用场景**：适用于需要提供备选方案的场景。

**示例代码**：

```java
// 在 VerifyUtil 中用于验证日期格式
public boolean isValidDate(String value) {
    return Try.getOr(
        () -> LocalDate.parse(value) != null, 
        () -> Boolean.FALSE
    );
}

// 验证字符串是否为正数
public boolean isPositive(String value) {
    return Try.getOr(
        () -> Double.parseDouble(value) > 0, 
        () -> Boolean.FALSE
    );
}

// 验证字符串是否为整数
public boolean isInteger(String value) {
    return Try.getOr(
        () -> (Long) Long.parseLong(value) != null, 
        () -> Boolean.FALSE
    );
}
```

**规则**
- ✅ 所有需要回退机制的操作必须使用 `Try.getOr()`
- ✅ 回退操作必须返回与主操作相同类型的值
- ✅ 回退操作不应该抛出异常

---

### 4.4 predicate 方法 - 断言验证

#### 4.4.1 方法签名

```java
public Boolean predicate(ResultAction<Boolean> action, ErrorCode code, Object errorTakeObj)
```

**功能**：执行断言操作，如果结果为 true 则抛出指定异常。

**参数**：
- `action`: ResultAction<Boolean>，返回布尔值的断言操作
- `code`: ErrorCode，需要抛出异常时使用的错误码
- `errorTakeObj`: Object，异常时携带的附加信息

**返回值**：断言操作的结果（非 true 情况下）

**使用场景**：适用于验证条件并抛出异常的场景。

**示例代码**：

```java
// 在 FormConditionUtil 中验证非法条件
private boolean verifyIllegalCondition(String input) {
    return Try.predicate(
        () -> input.matches(PATTERN_ILLEGAL_CONDITION_KEY),
        ErrorCode.builderModuleErrorMessage("表达式含非法字符"),
        input
    );
}
```

**规则**
- ✅ 所有断言验证必须使用 `Try.predicate()`
- ✅ 断言结果为 true 时会抛出异常
- ✅ 必须提供明确的错误码和错误信息

---

## 5. 错误码使用规范

### 5.1 使用预定义错误码

```java
// 使用基础错误码
Try.execute(
    () -> someOperation(),
    BasicErrorCode.SERVER_ERROR
);
```

### 5.2 构建自定义错误码

```java
// 创建自定义错误码
ErrorCode customError = ErrorCode.builder("CUSTOM_ERROR", "自定义错误信息", 400);

// 使用自定义错误码
String result = Try.get(
    () -> riskyOperation(),
    customError,
    "附加错误信息"
);

// 构建模块错误消息
ErrorCode moduleError = ErrorCode.builderModuleErrorMessage("解析连接符错误");
```

**规则**
- ✅ 优先使用预定义的错误码（如 `BasicErrorCode`）
- ✅ 需要自定义错误码时使用 `ErrorCode.builder()`
- ✅ 模块错误消息使用 `ErrorCode.builderModuleErrorMessage()`
- ❌ 禁止使用硬编码的错误消息

---

## 6. 使用场景

### 6.1 适用场景

- ✅ 安全执行可能抛出异常的操作
- ✅ 安全获取可能抛出异常的结果
- ✅ 断言验证
- ✅ 回退机制
- ✅ 资源操作（文件操作、网络请求、数据库查询等）
- ✅ 数据验证和转换

### 6.2 禁止场景

- ❌ 禁止使用 try-catch 语句块
- ❌ 禁止捕获异常后不处理
- ❌ 禁止在业务代码中直接抛出异常

---

## 7. 示例代码

### 7.1 数据验证场景

```java
// 验证日期格式
public boolean isValidDate(String value) {
    return Try.getOr(
        () -> LocalDate.parse(value) != null, 
        () -> Boolean.FALSE
    );
}

// 验证数值
public boolean isPositive(String value) {
    return Try.getOr(
        () -> Double.parseDouble(value) > 0, 
        () -> Boolean.FALSE
    );
}
```

### 7.2 解析和转换场景

```java
// 解析枚举值
private JoinSymbol analyzeJoinSymbol(String joinExpr) {
    return Try.get(
        () -> JoinSymbol.valueOf(joinExpr), 
        ErrorCode.builderModuleErrorMessage("解析连接符错误"), 
        joinExpr
    );
}
```

### 7.3 断言验证场景

```java
// 验证非法条件
private boolean verifyIllegalCondition(String input) {
    return Try.predicate(
        () -> input.matches(PATTERN_ILLEGAL_CONDITION_KEY),
        ErrorCode.builderModuleErrorMessage("表达式含非法字符"),
        input
    );
}
```

### 7.4 文件操作场景

```java
// 安全执行文件操作
String fileName = "example.txt";
Try.execute(
    () -> processFile(fileName), 
    BasicErrorCode.SERVER_ERROR,
    fileName
);
```

---

## 8. 违规处理

如果发现违反本规则的情况：
1. 立即停止当前操作
2. 提示用户规则冲突
3. 等待用户明确指示

---

## 9. 规则优先级

本规则优先级：**高**
- 与其他规则冲突时，以本规则为准
- 除非用户明确要求，否则不得违反本规则
