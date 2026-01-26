# Gonbay Core API与Server开发规范

## 1. API接口规范

### 1.1 接口定义规范

- **包命名**：API接口统一放置在 `api` 包下，如 `net.gonbay.app.[module].api`
- **命名规范**：接口名以 `Api` 结尾，如 `ShopUserApi`、`BehaviorRuleApi`
- **注解要求**：所有API接口必须使用 `@FeignClient` 注解标记
- **文档注释**：每个接口必须有完整的JavaDoc注释，包含作者信息和功能描述

### 1.2 FeignClient配置规范

```java
@FeignClient(name = "[服务名称]", contextId = "[上下文ID]")
public interface XxxApi {
    // 接口方法
}
```

- **name属性**：服务名称，通常为微服务的应用名称
- **contextId属性**：上下文ID，格式为 `[模块名]ApiContext`，避免Bean冲突

### 1.3 方法设计规范

- **路径规范**：
  - GET方法使用 `@GetMapping`
  - POST方法使用 `@PostMapping` 
  - PUT方法使用 `@PutMapping`
  - DELETE方法使用 `@DeleteMapping`
- **参数验证**：对于需要验证的参数，使用 `@Validated` 注解配合验证组
- **路径参数**：使用 `@PathVariable`、`@RequestParam`、`@RequestBody` 等注解
- **命名注释**：在注解中使用 `name` 属性对API功能进行简要描述

### 1.4 DTO和VO规范

- **入参DTO**：以 `DTO` 结尾，如 `EditScheduleTaskDTO`
- **出参VO**：以 `VO` 结尾，如 `ScheduleTaskVO`
- **分页参数**：统一使用 `PageRequestForm<T>` 进行分页封装
- **返回结果**：统一使用 `Result` 类型包装返回值

## 2. Server类规范

### 2.1 类定义规范

- **包命名**：Server类统一放置在 `server` 包下，如 `net.gonbay.app.[module].server`
- **命名规范**：类名以 `Server` 结尾，如 `ShopUserServer`、`BehaviorRuleServer`
- **继承关系**：Server类必须实现对应的API接口
- **注解要求**：使用 `@RestController` 注解标记为REST控制器

### 2.2 依赖注入规范

- **自动装配**：使用 `@Autowired` 注解注入Service层依赖
- **懒加载**：当存在循环依赖时，使用 `@Lazy` 注解进行懒加载
- **字段注入**：推荐使用字段注入而非构造函数注入

### 2.3 代码结构规范

```java
@RestController
public class XxxServer implements XxxApi {

    @Autowired
    private XxxService xxxService;

    @Autowired
    private XxxConverter xxxConverter;
    
    // 实现接口方法...
}
```

- **类注释**：包含完整的JavaDoc注释和作者信息
- **字段注入**：按照Service、Converter、Repository等顺序组织
- **方法实现**：严格按照API接口定义实现

## 3. 常量定义规范

### 3.1 接口常量

- **服务名称**：在 `constat` 包下的常量接口中定义，如 `ScheduleTaskConstat`
- **基础路径**：定义统一的基础URI，如 `BASE_URI = "schedule"`
- **客户端名称**：定义Feign客户端名称，如 `FEIGN_CLIENT_NAME`

### 3.2 命名空间规范

- **服务模块**：`net.gonbay.app.[业务域].[模块名]`
- **API包**：`net.gonbay.app.[业务域].[模块名].api`
- **Server包**：`net.gonbay.app.[业务域].[模块名].server`
- **Common包**：`net.gonbay.app.[业务域].[模块名].common`
- **DTO包**：`net.gonbay.app.[业务域].[模块名].dto`
- **VO包**：`net.gonbay.app.[业务域].[模块名].vo`

## 4. 验证规范

- **验证分组**：使用 `FormVerifyGroup.Insert` 和 `FormVerifyGroup.Update` 等验证组
- **参数验证**：在API层进行参数验证，使用 `@Validated` 注解
- **业务校验**：在Service层进行业务逻辑校验

## 5. 注解使用规范

### 5.1 控制器注解

- `@RestController`：标记REST控制器
- `@GetMapping/@PostMapping/@PutMapping/@DeleteMapping`：HTTP方法映射
- `@PathVariable/@RequestParam/@RequestBody`：参数绑定

### 5.2 业务注解

- `@OperationLog`：操作日志记录
- `@PerformanceLog`：性能日志记录  
- `@NoResubmit`：防止重复提交
- `@Cacheable`：缓存注解

## 6. 异常处理规范

- **错误码**：定义专门的错误码类，如 `ScheduleErrorCode`、`BehaviorErrorCode`
- **异常处理**：使用统一的异常处理机制
- **Try块**：使用 `Try` 类进行异常捕获和处理

## 7. 分页规范

- **分页请求**：使用 `PageRequestForm<T>` 统一封装分页参数
- **分页结果**：使用 `PageResult<T>` 统一封装分页结果
- **条件查询**：使用 `FormCondition` 进行条件封装

## 8. 最佳实践

### 8.1 API设计最佳实践

1. 接口职责单一，功能明确
2. 参数和返回值清晰，类型安全
3. 错误处理完善，异常信息友好
4. 文档完整，易于理解和使用

### 8.2 Server实现最佳实践

1. 严格遵循API契约，不改变接口定义
2. 业务逻辑下沉到Service层，Server层仅做适配
3. 合理使用注解增强功能
4. 注意性能优化和资源管理

### 8.3 安全考虑

1. 输入参数验证，防止恶意输入
2. 权限控制，确保接口访问安全
3. 敏感信息脱敏，保护数据安全
4. 防止重复提交，使用相应注解

## 9. 示例代码

### 9.1 API接口示例

```java
@FeignClient(name = "gonbay-behavior", contextId = "behaviorActionApi")
public interface BehaviorActionApi {
    
    @PostMapping("/behavior/action/creater")
    void add(@Validated({FormVerifyGroup.Insert.class}) @RequestBody BehaviorActionDTO dto);
    
    @PutMapping("/behavior/action/updater") 
    void update(@Validated({FormVerifyGroup.Update.class}) @RequestBody BehaviorActionDTO dto);
}
```

### 9.2 Server实现示例

```java
@RestController
public class BehaviorActionServer implements BehaviorActionApi {

    @Autowired
    private BehaviorActionService behaviorActionService;
    
    @Autowired
    private BehaviorRuleService behaviorRuleService;

    @Override
    public void add(BehaviorActionDTO dto) {
        // 实现业务逻辑
    }
    
    @Override  
    public void update(BehaviorActionDTO dto) {
        // 实现业务逻辑
    }
}
```