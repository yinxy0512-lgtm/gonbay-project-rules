# Lombok 组件使用规则

> 强制性约束 · 组件使用规范 · 全局适用

---

## 1. 规则目的

本规则定义了 Lombok 组件的使用规范。Lombok 是一个 Java 库，可以自动插入编辑器和构建工具中，用来消除 Java 中的冗余代码，尤其是 POJO 类中常见的 getter、setter、构造函数、equals、hashCode、toString 等方法。AI 编码助手在创建实体类、DTO、VO 等数据传输对象时，必须严格遵守本规则，使用 Lombok 注解减少样板代码。

---

## 2. 核心约束

### 2.1 必须遵守的规则

**必须使用**
- ✅ 所有 Entity、DTO、VO 类必须使用 Lombok 注解
- ✅ Entity 类必须使用 `@Data` 和 `@FieldNameConstants` 注解
- ✅ 工具类必须使用 `@UtilityClass` 注解
- ✅ 需要构建器模式的类必须使用 `@Builder` 或 `@SuperBuilder`
- ✅ 需要无参构造函数的类必须使用 `@NoArgsConstructor`
- ✅ 需要全参构造函数的类必须使用 `@AllArgsConstructor`

**禁止行为**
- ❌ 禁止手动编写 getter/setter 方法
- ❌ 禁止手动编写 toString、equals、hashCode 方法
- ❌ 禁止手动编写构造函数（除非有特殊需求）
- ❌ 禁止在 Entity、DTO、VO 类中不使用 Lombok 注解

---

## 3. Maven 依赖配置

### 3.1 依赖引入

项目根目录如果依赖 gonbay-core 父节点，则不需要考虑单独配置 lombok 依赖，会自动注入。

```xml
<!-- lombok -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>${lombok.version}</version>
    <scope>provided</scope>
</dependency>
```

其中 `${lombok.version}` 在 properties 中定义为 `1.18.38`。

**规则**
- ✅ 版本号使用项目统一版本管理
- ✅ scope 必须设置为 `provided`
- ❌ 禁止使用其他版本的 Lombok

---

## 4. 注解使用规范

### 4.1 @Data 注解

#### 4.1.1 功能说明

`@Data` 注解组合了 `@Getter`、`@Setter`、`@ToString`、`@EqualsAndHashCode` 和 `@RequiredArgsConstructor` 的功能，为类自动生成所有字段的 getter/setter 方法、toString 方法、equals 和 hashCode 方法。

#### 4.1.2 使用场景

- ✅ Entity 类（必须与 `@FieldNameConstants` 一起使用）
- ✅ DTO 类
- ✅ VO 类
- ✅ 配置类

#### 4.1.3 示例代码

```java
@Data
public class AuthUserDetail {
    /**
     * 用户编号
     */
    private String userCode;
    
    /**
     * 用户ID
     */
    private Long userId;
    
    /**
     * 登录标识(手机号/邮箱/用户名/OpenID等)
     */
    private String account;
}
```

**规则**
- ✅ Entity 类必须使用 `@Data` 注解
- ✅ 必须与 `@FieldNameConstants` 一起使用（用于字段名常量）

---

### 4.2 @FieldNameConstants 注解

#### 4.2.1 功能说明

`@FieldNameConstants` 注解为类生成字段名常量，用于在查询条件中引用字段名，避免硬编码字符串。

#### 4.2.2 使用场景

- ✅ Entity 类（必须使用）

#### 4.2.3 示例代码

```java
@Data
@FieldNameConstants
@TableName("pf_dict_type")
public class DictType extends BaseModel {
    private String code;
    private String levelType;
    private LocalDateTime updateTime;
}

// 使用方式
DictType.Fields.code  // 获取 "code" 字符串常量
```

**规则**
- ✅ 所有 Entity 类必须使用 `@FieldNameConstants` 注解
- ✅ 查询条件中必须使用 `Entity.Fields.fieldName` 引用字段名
- ❌ 禁止在查询条件中硬编码字段名字符串

---

### 4.3 @Builder 注解

#### 4.3.1 功能说明

`@Builder` 注解为类生成构建器模式的实现，提供流畅的链式调用方式来创建对象实例。

#### 4.3.2 使用场景

- ✅ 需要构建器模式的类
- ✅ 复杂对象的创建
- ✅ 测试数据准备

#### 4.3.3 示例代码

```java
@Getter
@Builder
public class ViewFieldInfo {
    /**
     * 属性名称
     */
    private String name;
    
    /**
     * 属性标签
     */
    private String label;
    
    /**
     * 属性类型
     */
    private String type;
    
    /**
     * 是否可见
     */
    private boolean visibled;
    
    /**
     * 是否是数据库映射
     */
    private boolean isDbMapped;
}

// 使用方式
ViewFieldInfo field = ViewFieldInfo.builder()
    .name("username")
    .label("用户名")
    .type("string")
    .visibled(true)
    .isDbMapped(true)
    .build();
```

**规则**
- ✅ 需要构建器模式的类必须使用 `@Builder` 注解
- ✅ 通常与 `@Getter` 一起使用（不需要 setter）
- ❌ 禁止手动实现构建器模式

---

### 4.4 @SuperBuilder 注解

#### 4.4.1 功能说明

`@SuperBuilder` 注解类似于 `@Builder`，但支持继承层次结构，可以在具有继承关系的类中使用构建器模式。

#### 4.4.2 使用场景

- ✅ 具有继承关系的类需要构建器模式
- ✅ 父类和子类都需要构建器

#### 4.4.3 示例代码

```java
@Data
@SuperBuilder
public class CreateTableInfo {
    private String table;
    private String comment;
    private List<ColumnInfo> columns;
    
    public String getColumnsInfo() {
        // 业务逻辑
        return "";
    }
}
```

**规则**
- ✅ 继承类需要构建器时必须使用 `@SuperBuilder`
- ✅ 父类和子类都必须使用 `@SuperBuilder`

---

### 4.5 @NoArgsConstructor 注解

#### 4.5.1 功能说明

`@NoArgsConstructor` 注解生成无参构造函数，通常用于框架需要无参构造函数的场景，如 ORM 映射。

#### 4.5.2 使用场景

- ✅ Entity 类（ORM 框架需要）
- ✅ 需要无参构造函数的类

#### 4.5.3 示例代码

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class TableNamed {
    private String name;
    private String tableName;
}
```

**规则**
- ✅ Entity 类必须使用 `@NoArgsConstructor`（ORM 框架需要）
- ✅ 通常与 `@AllArgsConstructor` 一起使用

---

### 4.6 @AllArgsConstructor 注解

#### 4.6.1 功能说明

`@AllArgsConstructor` 注解生成包含所有字段的构造函数，提供一次性初始化所有字段的构造方式。

#### 4.6.2 使用场景

- ✅ 需要全参构造函数的类
- ✅ 与 `@NoArgsConstructor` 一起使用

#### 4.6.3 示例代码

```java
@Getter
@AllArgsConstructor
public class RwMethod {
    private Method readMethod;
    private Method writeMethod;
}
```

**规则**
- ✅ 需要全参构造函数时使用 `@AllArgsConstructor`
- ✅ 通常与 `@NoArgsConstructor` 一起使用

---

### 4.7 @UtilityClass 注解

#### 4.7.1 功能说明

`@UtilityClass` 注解标记此类为工具类，不能被实例化，确保类中所有方法都是静态的，防止意外实例化。

#### 4.7.2 使用场景

- ✅ 工具类（所有方法都是静态方法）

#### 4.7.3 示例代码

```java
import lombok.experimental.UtilityClass;

@UtilityClass
public class BeanUtil {
    // 所有方法必须是静态的
    public <T> T copyProperties(Object source, Class<T> targetClass) {
        // 实现代码
        return null;
    }
}
```

**规则**
- ✅ 所有工具类必须使用 `@UtilityClass` 注解
- ✅ 工具类中所有方法必须是静态的
- ❌ 禁止在工具类中定义实例方法

---

### 4.8 @Getter 和 @Setter 注解

#### 4.8.1 功能说明

`@Getter` 和 `@Setter` 注解分别自动生成 getter 和 setter 方法，可以选择性地为特定字段生成访问器方法。

#### 4.8.2 使用场景

- ✅ 只需要 getter 或 setter 的类
- ✅ 与 `@Builder` 一起使用（只需要 getter）

**规则**
- ✅ 仅在需要选择性生成访问器时使用
- ✅ 通常与 `@Builder` 一起使用

---

### 4.9 @ToString 注解

#### 4.9.1 功能说明

`@ToString` 注解自动生成 toString 方法，提供对象的字符串表示形式，便于调试和日志记录。

**规则**
- ✅ `@Data` 已包含 `@ToString`，通常不需要单独使用
- ✅ 仅在需要自定义 toString 行为时单独使用

---

### 4.10 @EqualsAndHashCode 注解

#### 4.10.1 功能说明

`@EqualsAndHashCode` 注解自动生成 equals 和 hashCode 方法，确保对象比较和哈希计算的一致性。

**规则**
- ✅ `@Data` 已包含 `@EqualsAndHashCode`，通常不需要单独使用
- ✅ 仅在需要自定义 equals/hashCode 行为时单独使用

---

### 4.11 @Value 注解

#### 4.11.1 功能说明

`@Value` 注解创建不可变类，相当于 `@Getter + @FieldDefaults(makeFinal=true, level=AccessLevel.PRIVATE) + @AllArgsConstructor + @ToString + @EqualsAndHashCode`。

#### 4.11.2 使用场景

- ✅ 创建不可变的数据对象
- ✅ 值对象（Value Object）

**规则**
- ✅ 不可变对象使用 `@Value` 注解
- ❌ 禁止在可变对象上使用 `@Value`

---

## 5. Entity 类标准模板

### 5.1 完整示例

```java
package net.gonbay.app.dictionary.domain;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;
import lombok.experimental.FieldNameConstants;
import net.gonbay.common.domain.BaseModel;
import java.time.LocalDateTime;

/**
 * 字典分类
 * @author Floatin
 */
@Data
@FieldNameConstants
@TableName("pf_dict_type")
public class DictType extends BaseModel {
    
    /**
     * 字典分类码
     */
    private String code;

    /**
     * 等级类型
     */
    private String levelType;

    /**
     * 更新时间
     */
    private LocalDateTime updateTime;

    /**
     * 业务类型
     */
    private String bizType;

    /**
     * 扩展属性
     */
    private JsonNode extension;
}
```

**规则**
- ✅ Entity 类必须使用 `@Data` 和 `@FieldNameConstants` 注解
- ✅ Entity 类必须继承 `BaseModel`
- ✅ Entity 类必须使用 `@TableName` 指定表名
- ✅ 日期类型统一使用 `LocalDateTime`
- ✅ JSON 类型对应 `JsonNode`
- ✅ status、type 等语义字段使用 `String` 类型

---

## 6. 使用场景

### 6.1 适用场景

- ✅ 数据实体类（Entity）
- ✅ 数据传输对象（DTO）
- ✅ 值对象（VO）
- ✅ 配置类
- ✅ 工具类
- ✅ 测试类
- ✅ 需要构建器模式的复杂对象

### 6.2 禁止场景

- ❌ 禁止在工具类中使用 `@Data`（应使用 `@UtilityClass`）
- ❌ 禁止在不可变对象上使用 `@Data`（应使用 `@Value`）

---

## 7. Codex 触发条件

- ✅ 新增 Entity/DTO/VO 时必须使用 Lombok 注解
- ✅ 工具类必须使用 `@UtilityClass`
- ✅ 需要构建器模式时必须使用 `@Builder` 或 `@SuperBuilder`

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
