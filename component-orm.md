# Repository 组件与业务 Service 使用规则

> 强制性约束 · 组件使用规范 · 全局适用

---

## 1. 规则目的

本规则定义了 Repository 组件与业务 Service 的使用规范。Repository 组件是基于 MyBatis-Plus 框架构建的数据访问层组件，提供了丰富的 CRUD 操作功能。AI 编码助手在进行数据访问操作时，必须严格遵守本规则，使用 BasicService 和 BasicServiceImpl 进行数据操作。

---

## 2. 组件介绍

### 2.1 核心组件

1. **BasicService**：基础服务接口，扩展了 MyBatis-Plus 的 IService 接口，提供更丰富的查询和操作方法。
2. **BasicServiceImpl**：基础服务实现类，继承了 MyBatis-Plus 的 ServiceImpl，实现了 BasicService 接口。
3. **SnakeFieldQueryWrapper**：扩展的查询包装器，支持驼峰命名到下划线命名的自动转换，防止 SQL 注入，并提供字段选择功能。
4. **FormConditionUtil**：表单条件解析工具，能够将前端传递的条件对象解析为 SQL 查询条件。

### 2.2 主要特性

- ✅ 字段名自动转换：自动将驼峰命名的字段名转换为数据库的下划线命名
- ✅ 动态字段选择：支持包含或排除特定字段的查询
- ✅ 复杂条件构建：支持复杂的查询条件构建，包括嵌套条件
- ✅ 安全防护：内置 SQL 注入检测机制
- ✅ 分页支持：集成 MyBatis-Plus 分页插件

---

## 3. 核心约束

### 3.1 必须遵守的规则

**必须使用**
- ✅ 所有 Service 实现类必须继承 `BasicServiceImpl<Mapper, Entity>`
- ✅ 所有 Service 接口必须继承 `BasicService<Entity>`
- ✅ 所有查询条件构建必须使用 `SnakeFieldQueryWrapper` 或 `BasicService` 提供的方法
- ✅ 所有动态查询必须使用 `@TransferFieldMarked` 注解和 `BasicService.dtoInfosToWrapper()`
- ✅ 所有分页查询必须使用 `BasicService` 提供的分页方法
- ✅ 所有字段引用必须使用 `Entity.Fields.fieldName` 常量

**禁止行为**
- ❌ 禁止直接使用 MyBatis-Plus 的 `ServiceImpl` 或 `IService`
- ❌ 禁止手动构建 `LambdaQueryWrapper`（应使用 `SnakeFieldQueryWrapper`）
- ❌ 禁止在查询条件中硬编码字段名字符串
- ❌ 禁止在 Service 层直接操作 Mapper
- ❌ 禁止绕过 BasicService 直接使用 MyBatis-Plus API

---

## 4. Service 定义规范

### 4.1 Service 接口定义

```java
package net.gonbay.app.dictionary.service;
import net.gonbay.app.dictionary.domain.DictItem;
import net.gonbay.repository.core.BasicService;

/**
 * 字典项Service
 * @author Floatin
 */
public interface DictItemService extends BasicService<DictItem> {
    // 自定义业务方法
}
```

**规则**
- ✅ Service 接口必须继承 `BasicService<Entity>`
- ✅ Service 接口必须放在 `service` 包下
- ✅ 所有 Server 层调用的 service 方法都在对应的 service 中声明

### 4.2 Service 实现类定义

```java
package net.gonbay.app.dictionary.service.impl;

import net.gonbay.app.dictionary.domain.DictItem;
import net.gonbay.app.dictionary.mappers.DictItemMapper;
import net.gonbay.app.dictionary.service.DictItemService;
import net.gonbay.repository.core.BasicServiceImpl;
import org.springframework.stereotype.Service;

/**
 * 字典项ServiceImpl
 * @author Floatin
 */
@Service
public class DictItemServiceImpl extends BasicServiceImpl<DictItemMapper, DictItem> 
        implements DictItemService {
    // 实现具体的业务方法
}
```

**规则**
- ✅ Service 实现类必须继承 `BasicServiceImpl<Mapper, Entity>`
- ✅ Service 实现类必须实现对应的 Service 接口
- ✅ Service 实现类必须放在 `service.impl` 包下
- ✅ Service 实现类必须使用 `@Service` 注解
- ✅ 所有 Service 声明的接口全部都在 ServiceImpl 实现

---

## 5. Mapper 定义规范

### 5.1 Mapper 接口定义

```java
package net.gonbay.app.dictionary.mappers;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import net.gonbay.app.dictionary.domain.DictItem;

/**
 * 字典项Mapper
 * @author Floatin
 */
public interface DictItemMapper extends BaseMapper<DictItem> {
    // 自定义查询方法可以在这里定义
}
```

**规则**
- ✅ Mapper 接口必须继承 `BaseMapper<Entity>`
- ✅ Mapper 接口必须放在 `mappers` 包下
- ✅ 所有 SQL 查询都需要在 Mapper（ORM）定义
- ✅ 复杂查询使用 `@Select` 注解定义 SQL

---

## 6. Entity 类定义规范

### 6.1 Entity 类标准结构

```java
// 实体类定义
@Data
@FieldNameConstants
@TableName("pf_work_flow_model")
public class WorkFlowModel extends BaseModel {
    private String name;
    private String processId;
    private String fragment;
    
    // 字段名常量（由 @FieldNameConstants 自动生成）
    public static class Fields {
        public static final String name = "name";
        public static final String processId = "processId";
        public static final String fragment = "fragment";
    }
}
```

**规则**
- ✅ Entity 类必须使用 `@FieldNameConstants` 注解生成字段名常量
- ✅ 查询条件中必须使用 `Entity.Fields.fieldName` 引用字段名
- ❌ 禁止在查询条件中硬编码字段名字符串

---

## 7. 查询方法使用规范

### 7.1 分页查询方法

#### 7.1.1 pageByFormCondition - 基于表单条件分页查询

```java
// 基于表单条件的分页查询，返回转换后的 VO 对象
PageResult<SomeVO> pageByFormCondition(
    PageRequestForm<FormCondition> param, 
    ResultsFunction<T, R> convert
);

// 基于表单条件的分页查询，返回原始 Domain 对象
PageResult<SomeEntity> pageByFormCondition(PageRequestForm<FormCondition> param);

// 基于表单条件的分页查询，支持字段过滤
PageResult<SomeVO> pageByFormCondition(
    PageRequestForm<FormCondition> param, 
    ResultsFunction<T, R> convert, 
    Boolean isExclude, 
    String... fields
);
```

#### 7.1.2 pageByPagerDto - 基于 DTO 分页查询

```java
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
```

#### 7.1.3 pageByRequestForm - 自定义条件分页查询

```java
public PageResult<WorkFlowInstanceVO> pageByCondition(PageRequestForm<WorkFlowInstanceQueryDTO> param) {
    return this.pageByRequestForm(
        param,
        (wrapper, condition) -> {
            // 构建查询条件
            if (StringUtils.isNotBlank(condition.getInstanceId())) {
                wrapper.lambda().eq(WorkFlowInstance::getInstanceId, condition.getInstanceId());
            }
            if (StringUtils.isNotBlank(condition.getStatus())) {
                wrapper.lambda().eq(WorkFlowInstance::getStatus, condition.getStatus());
            }
        },
        // 结果转换函数
        workFlowInstanceConvert::toVo,
        null // 不过滤字段
    );
}
```

**规则**
- ✅ 所有分页查询必须使用 `BasicService` 提供的分页方法
- ✅ 必须提供结果转换函数（Entity 转 VO）
- ✅ 禁止手动构建分页对象

### 7.2 列表查询方法

#### 7.2.1 list - 基于 Wrapper 列表查询

```java
// 基于 Wrapper 的列表查询，支持结果转换
List<SomeVO> list(Wrapper<T> wrapper, ResultsFunction<T, R> convert);
```

#### 7.2.2 listByDto - 基于 DTO 列表查询

```java
// 基于 DTO 的列表查询，支持字段过滤和结果转换
List<SomeVO> listByDto(
    DTO dto, 
    ResultsFunction<T, R> convert, 
    Boolean isIncludeOrExclude, 
    String... fields
);
```

#### 7.2.3 listByEq - 基于相等条件列表查询

```java
// 根据指定字段的相等条件进行查询
List<WorkFlowTaskVO> listByInstanceId(String instanceId) {
    return this.listByEq(WorkFlowTask::getInstanceId, instanceId, task -> {
        // 转换为VO
        WorkFlowTaskVO vo = new WorkFlowTaskVO();
        // 转换逻辑
        return vo;
    });
}
```

#### 7.2.4 listByFks - 基于外键列表查询

```java
// 根据指定字段的多个值进行查询
List<WorkFlowTaskVO> listByUserIds(List<String> userIds) {
    return this.listByFks(WorkFlowTask::getAssignee, userIds, task -> {
        WorkFlowTaskVO vo = new WorkFlowTaskVO();
        // 转换逻辑
        return vo;
    });
}
```

**规则**
- ✅ 所有列表查询必须使用 `BasicService` 提供的方法
- ✅ 必须提供结果转换函数
- ❌ 禁止直接使用 Mapper 的 list 方法

### 7.3 单条记录查询方法

#### 7.3.1 get - 查询单条记录

```java
// 查询单条记录并进行结果转换
SomeVO get(Wrapper<T> wrapper, ResultFunction<T, R> convert);
```

**规则**
- ✅ 单条记录查询必须使用 `BasicService.get()` 方法
- ✅ 必须提供结果转换函数

### 7.4 字段过滤查询

#### 7.4.1 使用 SnakeFieldQueryWrapper

```java
// 只查询指定字段
SnakeFieldQueryWrapper<WorkFlowModel> wrapper = new SnakeFieldQueryWrapper<>(WorkFlowModel.class);
wrapper.selectIncluded(
    WorkFlowModel.Fields.id,
    WorkFlowModel.Fields.name,
    WorkFlowModel.Fields.processId
);
wrapper.lambda().eq(WorkFlowModel::getId, modelId);
return this.getOne(wrapper);

// 排除指定字段
wrapper.selectExcluded(
    WorkFlowModel.Fields.extensionContainer,
    WorkFlowModel.Fields.remark,
    WorkFlowModel.Fields.fragment
);
```

#### 7.4.2 使用 pageByPagerDto 字段过滤

```java
// 排除敏感字段查询
public PageResult<UserVO> pageWithoutSensitiveFields(PageRequestForm<UserQueryDTO> param) {
    return this.pageByPagerDto(
        param,
        this::convertToVO,
        false, // false 表示排除字段
        User.Fields.password, User.Fields.phone // 要排除的字段
    );
}
```

**规则**
- ✅ 字段过滤必须使用 `SnakeFieldQueryWrapper.selectIncluded()` 或 `selectExcluded()`
- ✅ 字段名必须使用 `Entity.Fields.fieldName` 常量
- ❌ 禁止硬编码字段名字符串

---

## 8. 复杂查询条件构建

### 8.1 使用 @TransferFieldMarked 注解

```java
public class PendingTaskDTO {
    @TransferFieldMarked(value = "executorId", condition = SqlCondition.in)
    private List<String> executorIds;

    @TransferFieldMarked(value = "instanceCreateTime", condition = SqlCondition.ge, isOrderField = true)
    private String startInstanceCreateTime;

    @TransferFieldMarked(value = "instanceCreateTime", condition = SqlCondition.le)
    private String endInstanceCreateTime;
}
```

### 8.2 使用 BasicService.dtoInfosToWrapper()

```java
LambdaQueryWrapper<SomeEntity> wrapper = basicService.dtoInfosToWrapper(
    condition,           // 查询条件 DTO
    true,                // 包含指定字段
    "field1", "field2"   // 指定字段
);
return basicService.list(wrapper);
```

**规则**
- ✅ 所有动态查询条件必须使用 `@TransferFieldMarked` 注解
- ✅ 查询条件构建必须使用 `BasicService.dtoInfosToWrapper()` 或 `pageByPagerDto()`
- ❌ 禁止手动构建查询条件

---

## 9. FormCondition 动态查询

### 9.1 使用 FormCondition

```java
// 前端传递的条件格式示例
/*
{
  "current": 1,
  "size": 10,
  "condition": {
    "name_like": "test",
    "status_eq": "ACTIVE",
    "createTime_ge": "2023-01-01",
    "and_createTime_le": "2023-12-31"
  }
}
*/

@Service
public class SomeEntityServiceImpl extends BasicServiceImpl<SomeEntityMapper, SomeEntity> 
        implements SomeEntityService {

    @Override
    public PageResult<SomeEntityVO> pageByFormCondition(PageRequestForm<FormCondition> param) {
        return this.pageByFormCondition(
            param,
            entity -> {
                // 将实体转换为 VO
                SomeEntityVO vo = new SomeEntityVO();
                // 转换逻辑
                return vo;
            }
        );
    }
}
```

**规则**
- ✅ FormCondition 适用于前端动态传递查询条件的场景
- ✅ 条件表达式格式：`{field}_{operator}: value`
- ✅ 支持的操作符：`eq`, `ne`, `gt`, `ge`, `lt`, `le`, `like`, `in` 等
- ❌ 禁止在 FormCondition 中传递 SQL 片段

---

## 10. 物理删除方法

### 10.1 forceDeleteById - 强制物理删除

```java
// 根据 ID 强制物理删除（绕过逻辑删除）
basicService.forceDeleteById(id);

// 根据 ID 集合强制物理删除
basicService.forceDeleteByIds(ids);
```

**规则**
- ✅ 物理删除必须使用 `forceDeleteById()` 或 `forceDeleteByIds()`
- ✅ 仅在确实需要物理删除时使用
- ❌ 禁止直接使用 Mapper 的 delete 方法进行物理删除

---

## 11. ID 生成方法

### 11.1 nextId - 获取下一个 ID

```java
// 获取下一个 ID（使用雪花算法）
Long nextId = basicService.nextId();
```

**规则**
- ✅ 所有 ID 生成必须使用 `BasicService.nextId()`
- ✅ 禁止使用数据库自增 ID
- ❌ 禁止手动生成 ID

---

## 12. 使用场景

### 12.1 适用场景

- ✅ 基础 CRUD 操作
- ✅ 动态条件查询
- ✅ 分页查询
- ✅ 复杂业务查询
- ✅ 字段过滤查询
- ✅ 外键查询

### 12.2 禁止场景

- ❌ 禁止在 Service 层直接操作 Mapper
- ❌ 禁止绕过 BasicService 直接使用 MyBatis-Plus API
- ❌ 禁止手动构建查询条件

---

## 13. 违规处理

如果发现违反本规则的情况：
1. 立即停止当前操作
2. 提示用户规则冲突
3. 等待用户明确指示

---

## 14. 规则优先级

本规则优先级：**高**
- 与其他规则冲突时，以本规则为准
- 除非用户明确要求，否则不得违反本规则
