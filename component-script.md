# gonbay-script 模块说明文档

## 1. 文档目的

本文档用于说明 `gonbay-script` 模块的真实职责、适用场景、核心能力、接入方式和限制条件。

文档目标不是介绍 Aviator 的全部语法，而是让开发人员和 AI 在阅读后能够基于当前代码实现，正确判断：

- 什么时候应该使用 `gonbay-script`
- 应该如何初始化脚本执行器
- 如何扩展自定义函数
- 哪些能力当前实现支持，哪些能力当前实现并不支持
- 在生成脚本和接入代码时，应该遵守哪些规则

如果将本文档接入 AI 规则库，AI 应默认把本文档视为 `gonbay-script` 的使用契约，而不是把它当成“示例参考”。

## 2. 模块定位

`gonbay-script` 是一个基于 Aviator 的轻量脚本执行模块，定位是：

- 为业务系统提供表达式计算能力
- 支持按需创建 `AviatorEvaluatorInstance`
- 支持脚本语法校验
- 支持通过自定义函数把脚本执行与 Spring Bean 中的业务执行器打通

它不是一个完整的规则引擎，也不是通用脚本平台，更不是 DSL 编排引擎。

从当前实现看，这个模块更适合做以下事情：

- 动态公式计算
- 条件表达式判断
- 将部分简单业务动作下沉到脚本中编排
- 通过脚本调用预先注册好的业务执行器方法

不适合直接承担以下职责：

- 复杂长流程编排
- 多步骤事务控制
- 任意 HTTP 编排平台
- 不受控的用户脚本执行平台

## 3. 解决场景

### 3.1 适合的场景

#### 场景一：动态公式计算

例如：

- 运费计算
- 佣金计算
- 风险评分
- 动态价格规则

这类场景的特点是：

- 公式变化频繁
- 逻辑相对独立
- 适合通过脚本外置减少硬编码

#### 场景二：动态条件判断

例如：

- 根据用户属性决定是否命中规则
- 根据金额、数量、等级决定是否触发优惠
- 根据上下文参数执行不同分支

#### 场景三：脚本触发受控业务动作

例如：

- 脚本命中后调用某个执行器
- 把通知、生成对象、补充数据等动作封装到 `ScriptFunctionExecutor` 中
- 由脚本决定调用哪个执行器、哪个方法以及传递什么参数

这里的关键点是“受控”。
脚本本身不能直接任意调用系统 Bean，而是只能通过模块提供的函数入口，调用已经注册到上下文中的执行器。

## 4. 功能介绍

### 4.1 脚本执行

核心入口在 `AvaitorScriptClient`。

当前提供两类执行方式：

- 传入 `AviatorEvaluatorInstance` 执行
- 使用默认 `AviatorEvaluator.getInstance()` 执行

支持两种入参形式：

- `Object... variables`
- `Map<String, Object>`

对于 `Map<String, Object>`，模块会先把参数平铺为 `key1,value1,key2,value2...` 再交给 Aviator。

### 4.2 脚本校验

`AvaitorScriptClient.verify(String script)` 用于做脚本语法校验。

需要注意：

- 这里只能校验 Aviator 语法是否合法
- 不能保证脚本运行时一定成功
- 不能校验 `native$` 或 `http$` 对应的执行器是否存在
- 不能校验执行器方法签名是否匹配

所以它适合作为“脚本入库前的基础校验”，不适合作为“完整可执行性校验”。

### 4.3 脚本执行实例构建

`AvaitorScriptClient.instanceBuilder()` 提供一个 Builder，用于创建独立的 `AviatorEvaluatorInstance`。

当前可配置能力包括：

- 执行模式 `mode(EvalMode mode)`
- 执行超时 `evalTimeoutMs(long timeoutMs)`
- 最大循环次数 `maxLoopCount(int count)`
- 单个特性开关 `feature(Feature feature)`
- 沙箱模式 `sandboxMode()`
- 执行跟踪日志 `trace()` / `noTrace()`
- LRU 表达式缓存容量 `lruMaxCapcity(int capcity)`
- 自定义函数注册 `addFunction(AviatorFunction function)` / `addFunctions(List<AviatorFunction> functions)`

### 4.4 自定义业务函数扩展

模块内置了两个可注册到 Aviator 的可变参数函数：

- `native$`
- `http$`

它们的执行流程本质上是：

1. 从脚本中读取执行器名称
2. 从 `ScriptFunctionExecutorContext` 中拿到对应执行器
3. 通过反射按方法名调用执行器上的公开方法
4. 将执行结果返回给 Aviator

也就是说，这两个函数当前都不是直接调用脚本里的匿名逻辑，而是做“脚本 -> 执行器 -> 方法”的桥接。

### 4.5 Spring 自动加载执行器

模块通过自动配置 `ScriptAutoLoadConfig` 注入 `InitScriptFunctionEvent`。

在 Spring 启动完成后，它会：

- 从 Spring 容器中扫描全部 `ScriptFunctionExecutor`
- 以 `executor.getExecutorName()` 为 key 注册到 `ScriptFunctionExecutorContext`

因此，想让脚本函数真正可用，执行器对象必须被 Spring 管理。

## 5. 核心设计说明

### 5.1 关键类职责

- `net.gonbay.script.core.AvaitorScriptClient`
  - 模块主入口
  - 负责脚本执行、校验、实例构建

- `net.gonbay.script.common.ScriptFunctionExecutor`
  - 自定义业务执行器接口
  - 用于定义脚本可调用的执行器名称

- `net.gonbay.script.core.ScriptFunctionExecutorContext`
  - 执行器注册表
  - 负责按名称保存和获取执行器

- `net.gonbay.script.core.InitScriptFunctionEvent`
  - Spring 启动后扫描并注册执行器

- `net.gonbay.script.core.function.NativeActionFunction`
  - 脚本函数 `native$`
  - 负责将脚本调用转成执行器反射调用

- `net.gonbay.script.core.function.HttpActionFunction`
  - 脚本函数 `http$`
  - 当前实现与 `native$` 的执行逻辑基本一致

### 5.2 当前设计的边界

这个模块当前只负责以下边界内的事情：

- 提供脚本执行能力
- 提供脚本与执行器的桥接能力
- 提供最基础的实例化配置能力

它不负责：

- 执行器生命周期管理以外的业务编排
- 参数模型治理
- 方法签名兼容适配
- 脚本安全白名单治理
- 脚本持久化、发布、灰度、版本控制

如果后续要往“规则平台”方向演进，这些能力都需要在模块外层补齐。

## 6. 使用方式

### 6.1 引入模块

在需要脚本能力的模块中引入：

```xml
<dependency>
    <groupId>net.gonbay</groupId>
    <artifactId>gonbay-script</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

### 6.2 基础脚本执行

适合纯表达式、纯计算场景。

```java
String script = """
       if(h>2){
         w * h * l * 2
       }else{
         w * h * l
       }
        """;

Object result = AvaitorScriptClient.execScript(
        script,
        "w", 2.3,
        "l", 3,
        "h", 2.5
);
```

适用建议：

- 规则简单时，优先用默认实例执行
- 不涉及自定义函数时，不必单独创建实例

### 6.3 使用独立实例执行

当你需要控制执行模式、超时、缓存和脚本特性时，使用 Builder 创建实例。

```java
AviatorEvaluatorInstance instance = AvaitorScriptClient.instanceBuilder()
        .trace()
        .mode(EvalMode.INTERPRETER)
        .maxLoopCount(2)
        .feature(Feature.Let)
        .feature(Feature.If)
        .evalTimeoutMs(1000)
        .lruMaxCapcity(128)
        .build();

Object result = AvaitorScriptClient.execScript(
        instance,
        script,
        "w", 2.3,
        "l", 3,
        "h", 2.5
);
```

适用建议：

- 多租户、不同业务域的脚本隔离时，使用独立实例
- 需要限制循环次数和超时时，必须自定义实例
- 需要缓存表达式时，建议设置 LRU 容量

### 6.4 脚本语法校验

```java
AvaitorScriptClient.verify(script);
```

推荐用法：

- 脚本保存前先校验语法
- 业务系统保存规则表达式前先做基础拦截

不建议误用为：

- 运行成功校验
- 执行器存在性校验
- 参数兼容性校验

### 6.5 扩展脚本执行器

如果需要让脚本调用业务能力，按下面步骤接入。

#### 第一步：实现执行器

```java
public class DemoScriptExecutor implements ScriptFunctionExecutor {

    @Override
    public String getExecutorName() {
        return "demo";
    }

    public BaseModel show(Map<String, Object> param) {
        System.out.println(param);
        return new BaseModel();
    }
}
```

实际接入时，应将该执行器声明为 Spring Bean，例如 `@Component`，否则不会被自动注册。

#### 第二步：注册脚本函数

```java
AviatorEvaluatorInstance instance = AvaitorScriptClient.instanceBuilder()
        .mode(EvalMode.ASM)
        .addFunction(NativeActionFunction.INSTANCE)
        .build();
```

如果脚本中使用 `http$`，则还需要显式注册 `HttpActionFunction`。

#### 第三步：编写脚本

```java
String funScript = """
        native$("demo", v1, seq.map("key1", v2, "key2", "val2"))
        """;

Map<String, Object> param = Map.of(
        "v1", "show",
        "v2", "name"
);

Object result = AvaitorScriptClient.execScript(instance, funScript, param);
```

这里脚本含义是：

- `"demo"`：执行器名称
- `v1`：方法名，这里实际解析为 `"show"`
- `seq.map(...)`：业务方法参数

最终会调用：

```java
demoScriptExecutor.show(Map<String, Object> param)
```

## 7. 使用案例

### 7.1 案例一：动态体积公式

#### 目标

根据高度决定体积计算规则。

#### 脚本

```java
String script = """
       if(h>2){
         w * h * l * 2
       }else{
         w * h * l
       }
        """;
```

#### 参数

```java
Map<String, Object> param = Map.of(
        "w", 2.3,
        "l", 3,
        "h", 2.5
);
```

#### 适用说明

适用于公式经常变化，但参数模型简单、无复杂副作用的场景。

### 7.2 案例二：脚本触发业务执行器

#### 目标

让脚本根据规则命中情况调用某个受控业务动作。

#### 执行器

```java
public class DemoScriptExecutor implements ScriptFunctionExecutor {

    @Override
    public String getExecutorName() {
        return "demo";
    }

    public BaseModel show(Map<String, Object> param) {
        return new BaseModel();
    }
}
```

#### 脚本

```java
String script = """
        native$("demo", "show", seq.map("name", userName, "level", userLevel))
        """;
```

#### 适用说明

适用于“规则判断在脚本里，真正业务动作在 Java 服务里”的场景。
这种方式比把完整业务逻辑直接写入脚本更可控、更易维护。

## 8. AI 使用规则库标准说明

本节是面向 AI 的直接使用规则。AI 在使用 `gonbay-script` 时，应默认遵守以下要求。

### 8.1 什么时候使用

满足以下条件时，优先考虑 `gonbay-script`：

- 需求是动态公式、表达式、规则条件判断
- 逻辑可通过 Aviator 表达式完成
- 需要通过脚本调用受控业务动作
- 业务方希望把简单规则外置而不是写死在 Java 代码中

出现以下情况时，不应优先使用 `gonbay-script`：

- 需求是复杂编排、事务流程、审批流
- 需要真正的 HTTP 调度平台
- 需要复杂权限隔离、多版本治理、发布中心
- 需要执行任意用户上传代码

### 8.2 AI 接入步骤

AI 生成接入方案时，应按如下顺序输出：

1. 先判断需求是纯表达式执行，还是脚本调用业务执行器
2. 纯表达式场景直接使用 `AvaitorScriptClient.execScript`
3. 若需要执行控制，使用 `instanceBuilder()` 创建独立实例
4. 若需要脚本调用业务方法，必须先实现 `ScriptFunctionExecutor`
5. 执行器必须交给 Spring 管理，确保启动后会被自动注册
6. 必须把 `NativeActionFunction` 或 `HttpActionFunction` 注册到实例中
7. 再编写脚本并传入执行参数

### 8.3 AI 必须知道的当前实现限制

#### 限制一：`http$` 当前不是 HTTP 客户端函数

从当前源码看，`HttpActionFunction` 虽然命名为 `http$`，但实际执行逻辑与 `native$` 一样，都是：

- 查找 `ScriptFunctionExecutor`
- 反射调用执行器方法

因此，AI 不应把 `http$` 解释为“脚本直接发 HTTP 请求”的能力。

#### 限制二：自定义函数至少需要 3 个脚本参数

当前 `native$` 和 `http$` 的参数校验条件是：

```java
objects.length <= 2
```

满足该条件会直接抛错。
这意味着当前实现下，脚本至少要传：

- 执行器名
- 方法名
- 至少一个业务参数

所以 AI 不应生成下面这种零业务参数调用：

```java
native$("demo", "show")
```

当前实现下，这种写法会失败。

#### 限制三：不要依赖 `features(Set<Feature>)`

Builder 构造器中已经初始化了 `features = new HashSet<>()`。
但 `features(Set<Feature> features)` 方法内部判断条件是：

```java
if (this.features == null && features != null)
```

这会导致正常情况下传入的 `Set<Feature>` 不会生效。

因此，AI 应优先生成：

```java
.feature(Feature.Let)
.feature(Feature.If)
```

而不是：

```java
.features(Set.of(Feature.Let, Feature.If))
```

#### 限制四：执行器方法不要重载

模块通过 `ReflectUtil.getMethodMapper(clazz)` 按“方法名 -> Method”建映射。
如果执行器中存在同名重载方法，会带来方法映射冲突风险。

因此，AI 在生成 `ScriptFunctionExecutor` 实现时，应避免方法重载，保持一个方法名只对应一个公开方法。

#### 限制五：执行器必须是 Spring Bean

执行器注册依赖 `SpringKit.getBeansOfType(ScriptFunctionExecutor.class)`。
如果只是普通对象，没有交给 Spring 管理，脚本运行时就拿不到执行器。

#### 限制六：`verify` 只做语法校验

AI 不应把 `verify()` 作为最终运行保障。
完整可执行性仍然依赖：

- 函数是否注册
- 执行器是否注册
- 方法是否存在
- 参数类型是否匹配

### 8.4 AI 推荐输出模板

当 AI 判断需求适合使用 `gonbay-script` 时，推荐按以下结构组织方案：

1. 说明是否属于“纯表达式”还是“脚本调用执行器”
2. 指出选择 `AvaitorScriptClient` 的原因
3. 给出实例初始化方式
4. 给出脚本样例
5. 给出 Java 调用样例
6. 补充当前实现限制和注意事项

## 9. 注意事项与风险

### 9.1 安全风险

虽然 Builder 提供了 `sandboxMode()`，但当前模块并没有形成一套完整的脚本安全治理体系。
如果脚本来源不可信，仍然需要在业务层增加：

- 脚本来源控制
- 操作白名单
- 执行超时控制
- 审计日志

### 9.2 可维护性风险

如果把过多业务逻辑直接写进脚本，会带来：

- 调试困难
- 版本追踪困难
- 测试困难
- 参数语义不清晰

推荐把脚本控制在“表达式和轻编排”层，把核心业务动作留在 Java 执行器中。

### 9.3 演进风险

当前模块更像脚本能力底座，不是完整规则平台。
如果未来要支撑更复杂业务，建议在模块外层补齐：

- 脚本存储模型
- 规则版本管理
- 执行上下文模型
- 权限与发布治理
- 运行审计与回放能力

## 10. 总结

`gonbay-script` 当前是一个“轻量脚本执行与业务函数桥接模块”。

它最适合：

- 动态公式
- 条件表达式
- 轻量规则动作触发

它当前最关键的使用原则是：

- 简单计算用 `AvaitorScriptClient`
- 受控业务调用用 `ScriptFunctionExecutor + native$`
- 执行器必须由 Spring 托管
- 不要把 `http$` 当成真正的 HTTP 能力
- 不要依赖 `features(Set<Feature>)`
- 不要生成无业务参数的 `native$`/`http$` 调用

如果 AI 严格按照本文档输出接入方案，基本可以正确使用当前版本的 `gonbay-script` 模块。
