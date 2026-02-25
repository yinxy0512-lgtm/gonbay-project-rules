# gonbay-basic-util 组件使用规则

> 强制性约束 · 组件使用规范 · 全局适用

---

## 1. 规则目的

本规则定义了 gonbay-basic-util 组件的使用规范。gonbay-basic-util 是一个基于 Java 21 开发的通用工具库，提供了丰富的实用工具类，涵盖 Bean 操作、字符串处理、时间管理、HTTP 请求、JSON 解析、反射操作、加密编解码、校验等功能。AI 编码助手在遇到对应场景时，必须严格遵守本规则，优先使用 gonbay-basic-util 提供的工具类，避免重复造轮子。

---

## 2. 核心约束

### 2.1 必须遵守的规则

**必须使用**
- ✅ 遇到对应场景必须使用 gonbay-basic-util 提供的工具类
- ✅ Bean 操作必须使用 `BeanUtil`
- ✅ 字符串操作必须使用 `StringUtil`
- ✅ 时间操作必须使用 `TimeUtil`
- ✅ JSON 操作必须使用 `JsonUtil`
- ✅ 空值检查必须使用 `EmptyCheckUtil`
- ✅ HTTP 请求必须使用 `HttpClientUtil`
- ✅ 数据验证必须使用 `VerifyUtil`
- ✅ 枚举操作必须使用 `EnumUtil`

**禁止行为**
- ❌ 禁止重复实现工具类功能
- ❌ 禁止使用其他工具库替代（如 Apache Commons、Guava）
- ❌ 禁止手动实现 Bean 拷贝、字符串处理等基础功能
- ❌ 禁止使用 Java 原生 API 处理复杂场景（应使用工具类）

---

## 3. 工具类使用规范

### 3.1 BeanUtil - Bean 操作工具类

#### 3.1.1 使用场景

- ✅ 实体类之间的属性拷贝
- ✅ DTO 与 Entity 之间的转换
- ✅ 集合中对象的批量拷贝
- ✅ 对象属性提取和映射
- ✅ 对象合并操作

#### 3.1.2 引用方式

```java
import net.gonbay.basic.utils.util.BeanUtil;
```

#### 3.1.3 常用方法

```java
// Bean属性拷贝（深copy）
UserDTO userDTO = BeanUtil.copy(userEntity, UserDTO.class);

// 集合拷贝
List<UserDTO> userDTOs = BeanUtil.copys(userEntities, UserDTO.class);

// Bean合并
BeanUtil.merge(source, target);

// Bean转Map
Map<String, Object> map = BeanUtil.beanToMap(bean);

// Map转Bean
T bean = BeanUtil.mapToBean(map, BeanClass.class);

// Bean克隆
T cloned = BeanUtil.cloneBean(bean);

// 一对一映射
Map<String, V> map = BeanUtil.oneToOneMapper(sources, "fieldName");

// 提取集合元素属性
List<Object> values = BeanUtil.extractByName(sources, "fieldName");

// 一对一装配
BeanUtil.joinOneToOne(source, target, "sourceField", "targetKey", "targetJoin");

// 一对多装配
BeanUtil.joinOneToMany(source, target, "sourceField", "targetKey", "targetJoin");

// 一对多映射
Map<String, List<V>> map = BeanUtil.oneToManyMapper(source, "fieldName");
```

#### 3.1.4 规则

- ✅ 所有 Bean 拷贝操作必须使用 `BeanUtil.copy()` 或 `BeanUtil.copys()`
- ✅ 禁止使用 Apache BeanUtils 或 Spring BeanUtils
- ❌ 禁止手动编写 getter/setter 进行属性拷贝

---

### 3.2 StringUtil - 字符串操作工具类

#### 3.2.1 使用场景

- ✅ 字符串格式化处理
- ✅ 字符串大小写转换
- ✅ 驼峰命名与下划线转换
- ✅ 字符串验证和提取
- ✅ 字符串填充和截取

#### 3.2.2 引用方式

```java
import net.gonbay.basic.utils.util.StringUtil;
```

#### 3.2.3 常用方法

```java
// 字符串包装
String wrapped = StringUtil.wrap("text", "\"");

// 驼峰转下划线
String snakeCase = StringUtil.camelToSnake("userName");

// 下划线转驼峰
String camelCase = StringUtil.snakeToCamel("user_name");

// 驼峰转连字符
String kebabCase = StringUtil.camelToKebab("userName");

// 字符串拼接
String joined = StringUtil.join("-", "a", "b", "c");

// 首字母大写/小写
String capitalized = StringUtil.capitalize("hello");
String uncapitalized = StringUtil.uncapitalize("Hello");

// 字符串截取
String left = StringUtil.left("hello", 3);
String right = StringUtil.right("hello", 3);
String mid = StringUtil.mid("hello", 1, 3);

// 移除前缀/后缀
String withoutPrefix = StringUtil.removePrefix("prefix_text", "prefix_");
String withoutSuffix = StringUtil.removeSuffix("text_suffix", "_suffix");

// 字符串填充
String padded = StringUtil.padLeft("123", 5, "0");
String paddedRight = StringUtil.padRight("123", 5, "0");

// 提取数字
String digits = StringUtil.extractDigits("abc123def456");

// 判断是否为数字
boolean isNumeric = StringUtil.isNumeric("123");

// 转义通配符
String escaped = StringUtil.escapeLikeWildcards("test%_value");
```

#### 3.2.4 规则

- ✅ 所有字符串操作必须使用 `StringUtil`
- ✅ 命名转换必须使用 `StringUtil.camelToSnake()` 等方法
- ❌ 禁止使用 `StringUtils`（Apache Commons 或其他库）

---

### 3.3 TimeUtil - 时间操作工具类

#### 3.3.1 使用场景

- ✅ 时间格式转换
- ✅ 时间计算和比较
- ✅ 时间范围生成
- ✅ 日期时间验证

#### 3.3.2 引用方式

```java
import net.gonbay.basic.utils.util.TimeUtil;
```

#### 3.3.3 常用方法

```java
// 时间解析
LocalDateTime ldt = TimeUtil.parse("2023-12-01 10:30:00");
LocalDate date = TimeUtil.parseDate("2023-12-01");
LocalDateTime custom = TimeUtil.parse("2023-12-01", "yyyy-MM-dd");

// 时间格式化
String formatted = TimeUtil.format(ldt);
String customFormatted = TimeUtil.format(ldt, "yyyy-MM-dd HH:mm:ss");

// 获取当前时间
LocalDateTime now = TimeUtil.now();
LocalDateTime today = TimeUtil.today();

// 日期边界
LocalDateTime start = TimeUtil.startOfDay(ldt);
LocalDateTime end = TimeUtil.endOfDay(ldt);
LocalDateTime monthStart = TimeUtil.startOfMonth(ldt);
LocalDateTime monthEnd = TimeUtil.endOfMonth(ldt);

// 时间判断
boolean isWeekend = TimeUtil.isWeekend(ldt);
boolean isLeapYear = TimeUtil.isLeapYear(ldt);
boolean isBetween = TimeUtil.isBetween(ldt, start, end);
boolean isSameDay = TimeUtil.isSameDay(ldt1, ldt2);

// 时间计算
long days = TimeUtil.daysBetween(start, end);
long minutes = TimeUtil.minutesBetween(start, end);
int compare = TimeUtil.compare(ldt1, ldt2);

// 时间范围生成
List<LocalDateTime> days = TimeUtil.rangeByDays(start, end);
List<LocalDateTime> hours = TimeUtil.rangeByHours(start, end);
List<LocalDateTime> months = TimeUtil.rangeByMonths(start, end);

// 时间分割
List<LocalDateTime[]> segments = TimeUtil.splitByDays(start, end, 2);
List<LocalDateTime[]> hourSegments = TimeUtil.splitByHours(start, end, 2);
```

#### 3.3.4 规则

- ✅ 所有时间操作必须使用 `TimeUtil`
- ✅ 日期类型统一使用 `LocalDateTime` 或 `LocalDate`
- ✅ 禁止使用 `Date` 或 `Calendar`
- ❌ 禁止使用其他时间工具库（如 Joda-Time）

---

### 3.4 JsonUtil - JSON 操作工具类

#### 3.4.1 使用场景

- ✅ JSON 序列化和反序列化
- ✅ JSON 字符串解析
- ✅ JSON 路径查询
- ✅ YAML 与 JSON 互转

#### 3.4.2 引用方式

```java
import net.gonbay.basic.utils.util.JsonUtil;
```

#### 3.4.3 常用方法

```java
// 对象转JSON
String json = JsonUtil.to(object);
String prettyJson = JsonUtil.to(object, true);

// JSON转对象
T object = JsonUtil.from(jsonString, TargetClass.class);

// 泛型反序列化
TypeReference<List<User>> typeRef = new TypeReference<List<User>>() {};
List<User> users = JsonUtil.from(json, typeRef);

// JSON路径查询
JsonNode node = JsonUtil.getByExpr(json, "field.subField");
String text = JsonUtil.getTextByExpr(json, "field.subField");
Boolean bool = JsonUtil.getBooleanByExpr(json, "field.flag");
Integer intValue = JsonUtil.getIntByExpr(json, "field.number");
Long longValue = JsonUtil.getLongByExpr(json, "field.number");

// 验证JSON合法性
boolean isValid = JsonUtil.isValidJson(json);

// YAML与JSON互转
String json = JsonUtil.yamlToJson(yamlStr);
String yaml = JsonUtil.jsonToYaml(jsonStr);
```

#### 3.4.4 规则

- ✅ 所有 JSON 操作必须使用 `JsonUtil`
- ✅ 禁止直接使用 Jackson 或 Gson
- ❌ 禁止手动拼接 JSON 字符串

---

### 3.5 EmptyCheckUtil - 空值检查工具类

#### 3.5.1 使用场景

- ✅ 参数非空验证
- ✅ 集合空值检查
- ✅ 字符串空值检查
- ✅ 对象空值判断

#### 3.5.2 引用方式

```java
import net.gonbay.basic.utils.util.EmptyCheckUtil;
```

#### 3.5.3 常用方法

```java
// 对象空值检查
boolean isEmpty = EmptyCheckUtil.isEmpty(obj);
boolean isNotEmpty = EmptyCheckUtil.isNotEmpty(obj);
boolean isNull = EmptyCheckUtil.isNull(obj);
boolean isNotNull = EmptyCheckUtil.isNotNull(obj);

// 字符串空值检查
boolean isEmpty = EmptyCheckUtil.isEmpty(str);
boolean isNotEmpty = EmptyCheckUtil.isNotEmpty(str);

// 集合空值检查
boolean isEmpty = EmptyCheckUtil.isEmpty(collection);
boolean isNotEmpty = EmptyCheckUtil.isNotEmpty(collection);

// Map空值检查
boolean isEmpty = EmptyCheckUtil.isEmpty(map);
boolean isNotEmpty = EmptyCheckUtil.isNotEmpty(map);

// 数组空值检查
boolean isEmpty = EmptyCheckUtil.isEmptyByArray(array);
boolean isNotEmpty = EmptyCheckUtil.isNotEmptyByArray(array);
```

#### 3.5.4 规则

- ✅ 所有空值检查必须使用 `EmptyCheckUtil`
- ✅ 禁止使用 `StringUtils.isEmpty()` 或其他库的空值检查方法
- ❌ 禁止手动编写空值判断逻辑

---

### 3.6 HttpClientUtil - HTTP 请求工具类

#### 3.6.1 使用场景

- ✅ REST API 调用
- ✅ HTTP 服务集成
- ✅ 微服务间通信
- ✅ 第三方服务调用

#### 3.6.2 引用方式

```java
import net.gonbay.basic.utils.util.HttpClientUtil;
```

#### 3.6.3 常用方法

```java
// 同步GET请求
Result result = HttpClientUtil.get("http://api.example.com/data", Result.class);
Result withHeaders = HttpClientUtil.get("http://api.example.com/data", headers, Result.class);

// 同步POST请求
Result result = HttpClientUtil.post("http://api.example.com/data", jsonBody, Result.class);
Result withHeaders = HttpClientUtil.post("http://api.example.com/data", headers, jsonBody, Result.class);

// 异步请求
CompletableFuture<Result> future = HttpClientUtil.getAsync("http://api.example.com/data", Result.class);
CompletableFuture<Result> postFuture = HttpClientUtil.postAsync("http://api.example.com/data", jsonBody, Result.class);

// PUT/DELETE请求
Result putResult = HttpClientUtil.put("http://api.example.com/data", jsonBody, Result.class);
HttpClientUtil.delete("http://api.example.com/data", Result.class);

// 添加全局请求头
HttpClientUtil.addGlobalHeader("Authorization", "Bearer token");
```

#### 3.6.4 规则

- ✅ 所有 HTTP 请求必须使用 `HttpClientUtil`
- ✅ 禁止使用 RestTemplate、OkHttp 或其他 HTTP 客户端
- ❌ 禁止手动创建 HTTP 连接

---

### 3.7 VerifyUtil - 数据验证工具类

#### 3.7.1 使用场景

- ✅ 数据校验
- ✅ 表单验证
- ✅ 参数验证
- ✅ 格式检查

#### 3.7.2 引用方式

```java
import net.gonbay.basic.utils.util.VerifyUtil;
```

#### 3.7.3 常用方法

```java
// 必填验证
boolean required = VerifyUtil.isRequired(value);

// 日期验证
boolean isDate = VerifyUtil.isValidDate(dateStr);
boolean isDatetime = VerifyUtil.isValidDatetime(datetimeStr);

// 数字验证
boolean isInteger = VerifyUtil.isInteger(value);
boolean isDecimal = VerifyUtil.isDecimal(value);
boolean isPositive = VerifyUtil.isPositive(value);

// 长度验证
boolean minLength = VerifyUtil.isLenthMin(value, 5);
boolean maxLength = VerifyUtil.isLenthMax(value, 50);

// 范围验证
boolean min = VerifyUtil.isMin(value, 0);
boolean max = VerifyUtil.isMax(value, 100);

// 正则匹配
boolean matches = VerifyUtil.matches(value, regex);

// 特殊格式验证
boolean isAlphabet = VerifyUtil.isAlphabet(value);
boolean isAlphanumeric = VerifyUtil.isAlphanumeric(value);
boolean isChinese = VerifyUtil.isChinese(value);
boolean isIP = VerifyUtil.isIPAddress(value);
boolean isEnum = VerifyUtil.isEnumValue(value, EnumClass.class);
```

#### 3.7.4 规则

- ✅ 所有数据验证必须使用 `VerifyUtil`
- ✅ 禁止使用 Apache Commons Validator 或其他验证库
- ❌ 禁止手动编写正则表达式验证逻辑

---

### 3.8 EnumUtil - 枚举操作工具类

#### 3.8.1 使用场景

- ✅ 枚举信息转换为Map
- ✅ 根据枚举名称获取元数据
- ✅ 枚举列表构建

#### 3.8.2 引用方式

```java
import net.gonbay.basic.utils.util.EnumUtil;
```

#### 3.8.3 常用方法

```java
// 将枚举转换为Map（适用于实现了MetaEnum接口的枚举）
Map<String, T> enumMap = EnumUtil.mataEnumInfo(EnumClass.class);

// 根据code获取枚举实例
T metaData = EnumUtil.getMetaData(EnumClass.class, "code");
```

#### 3.8.4 规则

- ✅ 枚举必须实现 `MetaEnum` 接口
- ✅ 枚举操作必须使用 `EnumUtil`
- ❌ 禁止手动遍历枚举值

---

### 3.9 其他工具类

#### 3.9.1 CodecUtil - 编码解码工具类

```java
import net.gonbay.basic.utils.util.CodecUtil;

// 链式编码转换
String base64 = CodecUtil.fromString("Hello").toBase64().asBase64();
byte[] bytes = CodecUtil.fromHex("48656c6c6f").toBytes().asBytes();
String uuid = CodecUtil.randomUUID().toUUIDString();
```

#### 3.9.2 PlaceholderUtil - 占位符替换工具类

```java
import net.gonbay.basic.utils.util.PlaceholderUtil;

// 占位符替换
String result = PlaceholderUtil.replace(template, dataMap);
String strict = PlaceholderUtil.replaceStrict(template, dataMap);
boolean isValid = PlaceholderUtil.validate(template, dataMap);
```

#### 3.9.3 ReflectUtil - 反射工具类

```java
import net.gonbay.basic.utils.util.ReflectUtil;

// 字段操作
Object value = ReflectUtil.getFieldValue(obj, field);
ReflectUtil.setFieldValue(obj, field, value);

// 方法调用
Object result = ReflectUtil.invokeMethod(obj, method, args);

// 创建实例
T instance = ReflectUtil.newInstance(Class.class);
```

#### 3.9.4 TreeBuildUtil - 树形结构构建工具类

```java
import net.gonbay.basic.utils.util.TreeBuildUtil;

// 构建树形结构（适用于实现了TreeNode接口的类）
List<TreeNode> tree = TreeBuildUtil.tree(nodeList);
```

#### 3.9.5 DomainInfoParser - 领域模型解析工具类

```java
import net.gonbay.basic.utils.util.DomainInfoParser;

// 获取视图字段信息
List<ViewFieldInfo> viewFields = DomainInfoParser.getViewFieldInfos(VoClass.class);
List<ViewFieldInfo> visibleFields = DomainInfoParser.getViewFieldInfosByVisibled(VoClass.class, true);

// 获取传输字段信息
List<TransferFieldInfo> transferFields = DomainInfoParser.getTransferFieldInfos(dto);
```

#### 3.9.6 DecisionExecutor - 决策执行器

```java
import net.gonbay.basic.utils.decision.DecisionExecutor;

// 创建决策执行器
DecisionExecutor<Context, State> executor = new DecisionExecutor<>(rootDefinition);
executor.addNode(node1, node2, node3);
executor.execute(context);
```

---

## 4. 使用场景

### 4.1 适用场景

- ✅ Bean 属性拷贝和转换
- ✅ 字符串处理和格式化
- ✅ 时间日期操作
- ✅ JSON 序列化和反序列化
- ✅ HTTP 请求调用
- ✅ 数据验证和校验
- ✅ 空值检查
- ✅ 枚举操作
- ✅ 编码解码转换
- ✅ 反射操作
- ✅ 树形结构构建

### 4.2 禁止场景

- ❌ 禁止在工具类中实现业务逻辑
- ❌ 禁止修改工具类源码
- ❌ 禁止绕过工具类直接使用底层 API

---

## 5. Codex 触发条件

- ✅ 出现字符串、时间、JSON、校验等通用操作时必须使用本组件工具类
- ✅ 需要 Bean 属性拷贝或集合映射时必须使用 `BeanUtil`
- ✅ 需要 HTTP 请求或通用校验时必须使用 `HttpClientUtil`、`VerifyUtil`

---

## 6. 违规处理

如果发现违反本规则的情况：
1. 立即停止当前操作
2. 提示用户规则冲突
3. 等待用户明确指示

---

## 7. 规则优先级

本规则优先级：**高**
- 与其他规则冲突时，以本规则为准
- 除非用户明确要求，否则不得违反本规则
