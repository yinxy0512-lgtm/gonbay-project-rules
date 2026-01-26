# MapStruct、领域模型转换与 ORM 查询条件组装组件使用规则

> 强制性约束 · 组件使用规范 · 全局适用

---

## 1. 规则目的

本规则定义了 MapStruct、DomainInfoParser 与 ORM 查询条件组装组件的使用规范。这些组件用于在不同层之间进行数据对象转换、动态查询条件构建等场景。AI 编码助手在进行领域模型转换和查询条件构建时，必须严格遵守本规则。

---

## 2. 组件介绍

### 2.1 MapStruct 组件

MapStruct 是一个代码生成器，用于在 Java Bean 类型之间进行映射。它在编译时生成映射实现类，相比手动编写映射代码或使用反射，具有更好的性能和类型安全性。

### 2.2 DomainInfoParser 组件

DomainInfoParser 是 gonbay-core 项目中自定义的领域模型解析工具类，用于解析带有特定注解的 DTO 类，提取字段的元数据信息，如查询条件、排序字段等，然后将其转换为可直接用于 ORM 查询的条件对象。

### 2.3 ORM 查询条件组装

该项目中使用了自定义的 SnakeFieldQueryWrapper 和 FormConditionUtil 工具类，结合 MyBatis-Plus 框架，实现了从 DTO 对象直接转换为数据库查询条件的功能，支持复杂的条件表达式和嵌套查询。

---

## 3. 核心约束

### 3.1 必须遵守的规则

**必须使用**
- ✅ 所有 DTO 与 Entity、Entity 与 VO 之间的转换必须使用 MapStruct
- ✅ 所有查询条件 DTO 必须使用 `@TransferFieldMarked` 注解
- ✅ 所有 VO 类必须使用 `@ViewFieldMarked` 注解
- ✅ 所有动态查询条件构建必须使用 `DomainInfoParser` 或 `BasicService.dtoInfosToWrapper()`
- ✅ 所有转换器接口必须放在 `struct` 包下，命名规则为 `{Entity}Convert`
- ✅ 一个 Domain 对应一个 Convert 转换器

**禁止行为**
- ❌ 禁止手动编写 getter/setter 进行对象转换
- ❌ 禁止使用反射进行对象转换（性能问题）
- ❌ 禁止使用 BeanUtils 进行对象拷贝（应使用 MapStruct）
- ❌ 禁止手动构建查询条件（应使用注解和工具类）
- ❌ 禁止在转换器中编写业务逻辑

---

## 4. MapStruct 使用规范

### 4.1 依赖配置

#### 4.1.1 Maven 依赖

项目根目录如果依赖 gonbay-core 父节点，则不需要考虑单独配置 lombok 依赖，会自动注入。


#### 4.1.2 编译器配置

在 `maven-compiler-plugin` 中配置注解处理器：


**规则**
- ✅ 必须配置注解处理器路径
- ✅ 必须包含 lombok-mapstruct-binding（Lombok 与 MapStruct 兼容）

### 4.2 转换器接口定义规范

#### 4.2.1 基本结构

```java
@Mapper(
    nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE,
    uses = {BasicStructMapper.class},
    imports = {TimeUtil.class},
    componentModel = "spring"
)
public interface BehaviorRuleConvert {
    
    /**
     * DTO转Domain
     */
    BehaviorRule toRule(BehaviorRuleUpdateDTO dto);
    
    /**
     * Domain转VO
     */
    BehaviorRuleVO toBehaviorRuleVO(BehaviorRule rule);
    
    /**
     * 更新Domain（使用@MappingTarget）
     */
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "name", ignore = true)
    @Mapping(target = "updateTime", expression = "java(TimeUtil.now())")
    void updateToRule(@MappingTarget BehaviorRule rule, BehaviorRuleUpdateDTO dto);
}
```

#### 4.2.2 @Mapper 注解参数

- `nullValuePropertyMappingStrategy`：空值属性映射策略，推荐使用 `IGNORE`
- `imports`：导入类，可在映射表达式中使用
- `componentModel`：组件模型，必须设置为 `"spring"`（用于 Spring 依赖注入）

#### 4.2.3 @Mapping 注解

- `target`：目标属性名
- `source`：源属性名
- `ignore`：忽略该属性的映射
- `expression`：自定义表达式（如 `"java(TimeUtil.now())"`）

#### 4.2.4 @MappingTarget 注解

用于更新现有对象而非创建新对象。

**规则**
- ✅ 转换器接口必须使用 `@Mapper` 注解
- ✅ `componentModel` 必须设置为 `"spring"`
- ✅ 必须使用 uses = {BasicStructMapper.class},
- ✅ 转换器必须放在 `struct` 包下
- ✅ 转换器命名规则：`{Entity}Convert`
- ❌ 禁止在转换器中编写业务逻辑

---

## 5. DomainInfoParser 使用规范

### 5.1 视图字段信息解析

#### 5.1.1 获取视图字段信息

```java
import net.gonbay.basic.utils.util.DomainInfoParser;

// 获取所有视图字段信息
List<ViewFieldInfo> viewFields = DomainInfoParser.getViewFieldInfos(VoClass.class);

// 根据可见性筛选
List<ViewFieldInfo> visibleFields = DomainInfoParser.getViewFieldInfosByVisibled(VoClass.class, true);
```

#### 5.1.2 VO 类定义规范

```java
@Data
public class UserVO {
    @ViewFieldMarked(value = "用户名", visibled = true, isDbMapped = true)
    private String username;

    @ViewFieldMarked(value = "邮箱", visibled = true, isDbMapped = true)
    private String email;

    @ViewFieldMarked(value = "创建时间", visibled = false, isDbMapped = true)
    private Date createTime;
}
```

**规则**
- ✅ 所有 VO 类字段必须使用 `@ViewFieldMarked` 注解
- ✅ `value` 属性必须设置字段的中文名称
- ✅ `visibled` 属性控制字段是否可见
- ✅ `isDbMapped` 属性标识字段是否映射到数据库

### 5.2 传输字段信息解析

#### 5.2.1 获取传输字段信息

```java
// 获取 DTO 中的查询条件信息
List<TransferFieldInfo> transferFields = DomainInfoParser.getTransferFieldInfos(dto);
```

#### 5.2.2 DTO 类定义规范

```java
public class PendingTaskDTO {
    
    @TransferFieldMarked()
    private String instanceCode;

    @TransferFieldMarked()
    private String instanceName;

    @TransferFieldMarked(value = "executorId", condition = SqlCondition.in)
    private List<String> executorIds;

    @TransferFieldMarked(value = "deptId", condition = SqlCondition.in)
    private List<Long> deptIds;

    @TransferFieldMarked(value = "instanceCreateTime", condition = SqlCondition.ge, isOrderField = true)
    private String startInstanceCreateTime;

    @TransferFieldMarked(value = "instanceCreateTime", condition = SqlCondition.le)
    private String endInstanceCreateTime;
}
```

#### 5.2.3 @TransferFieldMarked 注解属性

- `value()`：查询字段名称（默认：空字符串）
- `isQueryField()`：是否是查询条件属性（默认：true）
- `isOrderField()`：是否是排序属性（默认：false）
- `isAscend()`：是否是正序，默认降序（默认：false）
- `isAndOr()`：是否是 And 查询，默认是 true（默认：true）
- `condition()`：查询条件，默认 EQ 查询（默认：SqlCondition.eq）

**规则**
- ✅ 所有查询条件 DTO 字段必须使用 `@TransferFieldMarked` 注解
- ✅ `value` 属性必须指定数据库字段名（驼峰会自动转换为下划线）
- ✅ `condition` 属性必须指定查询操作符
- ❌ 禁止在 DTO 中手动构建查询条件

---

## 6. ORM 查询条件组装规范

### 6.1 使用 BasicService.dtoInfosToWrapper()

#### 6.1.1 基本用法

```java
@Service
public class SomeBusinessService extends BasicServiceImpl<SomeMapper, SomeEntity> {
    
    public PageResult<SomeVO> queryByDto(PageRequestForm<PendingTaskDTO> request) {
        return this.pageByPagerDto(
            request,
            // 结果转换函数
            entityList -> entityList.stream()
                .map(this::convertToVO)
                .collect(Collectors.toList()),
            // 是否包含/排除指定字段，可为 null
            null,
            // 指定字段数组，可为空
        );
    }
}
```

#### 6.1.2 自定义查询条件构建

```java
@Repository
public class CustomQueryService {
    
    @Autowired
    private BasicService<SomeEntity> basicService;
    
    public List<SomeEntity> customQuery(PendingTaskDTO condition) {
        // 使用 DTO 信息直接构建查询条件
        LambdaQueryWrapper<SomeEntity> wrapper = basicService.dtoInfosToWrapper(
            condition,           // 查询条件 DTO
            true,                // 包含指定字段
            "field1", "field2"   // 指定字段
        );
        
        return basicService.list(wrapper);
    }
}
```

**规则**
- ✅ 所有动态查询必须使用 `BasicService.dtoInfosToWrapper()` 或 `pageByPagerDto()`
- ✅ 禁止手动构建 `LambdaQueryWrapper`
- ❌ 禁止在 Service 层直接使用 MyBatis-Plus 的 Wrapper

### 6.2 使用 DomainInfoParser 手动构建

#### 6.2.1 手动构建查询条件

```java
@Service
public class SomeService {
    
    public void example() {
        PendingTaskDTO dto = new PendingTaskDTO();
        // 设置查询条件...
        
        // 使用 DomainInfoParser 解析 DTO 中的查询条件信息
        List<TransferFieldInfo> conditionList = DomainInfoParser.getTransferFieldInfos(dto);
        
        // 使用解析后的信息构建查询条件
        SnakeFieldQueryWrapper<SomeEntity> wrapper = new SnakeFieldQueryWrapper<>();
        if (EmptyCheckUtil.isNotEmpty(conditionList)) {
            conditionList.forEach(dtoInfo -> {
                if (dtoInfo.getValue() != null) {
                    OperatorSymbol symbol = OperatorSymbol.valueOf(dtoInfo.getCondition());
                    JoinSymbol join = dtoInfo.isAnd() ? JoinSymbol.and : JoinSymbol.or;
                    symbol.apply(wrapper, dtoInfo.getName(), dtoInfo.getValue(), join);
                }
            });
        }
    }
}
```

**规则**
- ✅ 仅在 `BasicService.dtoInfosToWrapper()` 不满足需求时使用手动构建
- ✅ 必须使用 `DomainInfoParser.getTransferFieldInfos()` 解析 DTO
- ❌ 禁止直接操作 Wrapper，必须通过工具类

### 6.3 FormCondition 动态查询

#### 6.3.1 使用 FormCondition

```java
// 使用 FormCondition 进行动态查询
public PageResult<Entity> queryWithFormCondition(PageRequestForm<FormCondition> param) {
    // FormConditionUtil 会解析条件表达式并构建查询条件
    // 例如：{"name_eq": "value", "age_ge": 18, "createTime_lt": "2023-01-01"}
    return this.pageByFormCondition(param);
}
```

**规则**
- ✅ FormCondition 适用于前端动态传递查询条件的场景
- ✅ 条件表达式格式：`{field}_{operator}: value`
- ❌ 禁止在 FormCondition 中传递 SQL 片段

---

## 7. 使用场景

### 7.1 适用场景

- ✅ DTO 与 Entity 之间的转换
- ✅ Entity 与 VO 之间的转换
- ✅ 动态查询条件构建
- ✅ 元数据解析（视图字段、查询字段）
- ✅ 快速查询封装

### 7.2 禁止场景

- ❌ 禁止在转换器中编写业务逻辑
- ❌ 禁止手动编写对象转换代码
- ❌ 禁止绕过注解直接构建查询条件

---

## 8. 示例代码

### 8.1 MapStruct 转换器完整示例

```java
@Mapper(
    nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE,
    imports = {TimeUtil.class},
    componentModel = "spring"
)
public interface BehaviorRuleConvert {

    /**
     * DTO转Domain
     */
    default BehaviorRule toRule(BehaviorRuleUpdateDTO dto, BehaviorStatus status) {
        BehaviorRule rule = new BehaviorRule();
        rule.setId(dto.getId());
        rule.setType(dto.getType());
        rule.setStatus(status.name());
        rule.setName(dto.getName());
        rule.setRemark(dto.getRemark());
        rule.setBeforeExpression(dto.getBeforeExpression());
        rule.setAfterExpression(dto.getAfterExpression());
        rule.setViewExpression(dto.getViewExpression());
        rule.setJsonSchema(dto.getJsonSchema());
        LocalDateTime now = TimeUtil.now();
        rule.setCreateTime(now);
        rule.setUpdateTime(now);
        return rule;
    }

    /**
     * 更新行为规则 dto to wrapper
     */
    default Wrapper<BehaviorRule> toUpdateWrapperByDto(BehaviorRuleUpdateDTO dto) {
        LambdaUpdateWrapper<BehaviorRule> wrapper = Wrappers.<BehaviorRule>lambdaUpdate().eq(BehaviorRule::getId, dto.getId());
        wrapper.set(dto.getRemark() != null, BehaviorRule::getRemark, dto.getRemark());
        wrapper.set(dto.getAfterExpression() != null, BehaviorRule::getAfterExpression, dto.getAfterExpression());
        wrapper.set(dto.getBeforeExpression() != null, BehaviorRule::getBeforeExpression, dto.getBeforeExpression());
        wrapper.set(BehaviorRule::getJsonSchema, JsonUtil.to(dto.getJsonSchema()));
        wrapper.set(BehaviorRule::getType, dto.getType());
        wrapper.set(BehaviorRule::getUpdateTime, TimeUtil.now());
        return wrapper;
    }

    /**
     * 更新行为规则 dto to domain
     */
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "name", ignore = true)
    @Mapping(target = "updateTime", expression = "java(TimeUtil.now())")
    void updateToRule(@MappingTarget BehaviorRule newRule, BehaviorRuleUpdateDTO dto);

    /**
     * domain 转 vo
     */
    BehaviorRuleVO toBehaviorRuleVO(BehaviorRule rule);
}
```

### 8.2 查询条件 DTO 完整示例

```java
public class PendingTaskDTO {
    
    @TransferFieldMarked()
    private String instanceCode;

    @TransferFieldMarked()
    private String instanceName;

    @TransferFieldMarked(value = "executorId", condition = SqlCondition.in)
    private List<String> executorIds;

    @TransferFieldMarked(value = "deptId", condition = SqlCondition.in)
    private List<Long> deptIds;

    @TransferFieldMarked(value = "postId", condition = SqlCondition.in)
    private List<Long> postIds;

    @TransferFieldMarked(value = "instanceCreateTime", condition = SqlCondition.ge, isOrderField = true)
    private String startInstanceCreateTime;

    @TransferFieldMarked(value = "instanceCreateTime", condition = SqlCondition.le)
    private String endInstanceCreateTime;

    @TransferFieldMarked()
    private String assignee;
}
```

---

## 9. 违规处理

如果发现违反本规则的情况：
1. 立即停止当前操作
2. 提示用户规则冲突
3. 等待用户明确指示

---

## 10. 规则优先级

本规则优先级：**高**
- 与其他规则冲突时，以本规则为准
- 除非用户明确要求，否则不得违反本规则
