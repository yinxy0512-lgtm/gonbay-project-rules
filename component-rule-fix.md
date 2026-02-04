# 代码规范修复规则

> 强制性约束 · 代码规范 · 全局适用

---

## 1. 规则目的

本规则定义了代码编写过程中必须遵守的规范，包括类引用方式和查询条件构建规范。AI 编码助手在编写、修改任何 Java 代码时，必须强制严格遵守本规则。

---

## 2. 类引用规范

### 2.1 规则说明

所有类引用必须使用 `import` 语句导入，禁止在代码中使用全类名（FQCN）引用。

### 2.2 必须遵守的规则

**必须使用**
- ✅ 所有类引用必须通过 `import` 语句导入
- ✅ 在代码中使用简短的类名引用
- ✅ 保持 import 语句的整洁和有序

**禁止行为**
- ❌ 禁止在代码中使用全类名引用（如 `net.gonbay.app.task.engine.vo.TaskForecastVO`）
- ❌ 禁止在方法参数、返回类型、变量声明中使用全类名
- ❌ 禁止在类型转换、instanceof 判断中使用全类名
- ❌ 禁止在泛型参数中使用全类名

### 2.3 正例和反例

#### 2.3.1 方法参数和返回类型

**❌ 反例：使用全类名**

```java
package net.gonbay.app.task.engine.server;

import net.gonbay.app.task.engine.api.TaskInstanceApi;
import net.gonbay.app.task.engine.domain.TaskInstance;

@RestController
public class TaskInstanceServer implements TaskInstanceApi {

    @Override
    public void add(net.gonbay.app.task.engine.dto.TaskInstanceCreateDTO dto) {
        // 错误：方法参数使用全类名
    }

    @Override
    public List<net.gonbay.app.task.engine.vo.TaskVO> getTasksByInstanceId(Long instanceId) {
        // 错误：返回类型使用全类名
        return null;
    }
}
```

**✅ 正例：使用 import 导入**

```java
package net.gonbay.app.task.engine.server;

import net.gonbay.app.task.engine.api.TaskInstanceApi;
import net.gonbay.app.task.engine.domain.TaskInstance;
import net.gonbay.app.task.engine.dto.TaskInstanceCreateDTO;
import net.gonbay.app.task.engine.vo.TaskVO;

@RestController
public class TaskInstanceServer implements TaskInstanceApi {

    @Override
    public void add(TaskInstanceCreateDTO dto) {
        // 正确：使用导入的类名
    }

    @Override
    public List<TaskVO> getTasksByInstanceId(Long instanceId) {
        // 正确：使用导入的类名
        return null;
    }
}
```

#### 2.3.2 类型转换和 instanceof 判断

**❌ 反例：使用全类名**

```java
public void identifyCriticalPaths(List<TaskPathVO> paths, Map<Long, Object> timeCalculations) {
    for (TaskPathVO path : paths) {
        boolean isCritical = path.getPathNodes().stream()
            .allMatch(taskId -> {
                Object calculation = timeCalculations.get(taskId);
                if (calculation instanceof net.gonbay.app.task.engine.vo.TaskForecastVO) {
                    // 错误：instanceof 使用全类名
                    net.gonbay.app.task.engine.vo.TaskForecastVO calc = 
                        (net.gonbay.app.task.engine.vo.TaskForecastVO) calculation;
                    // 错误：类型转换使用全类名
                    return calc.getTotalFloat() != null && calc.getTotalFloat().compareTo(BigDecimal.ZERO) == 0;
                }
                return false;
            });
    }
}
```

**✅ 正例：使用 import 导入**

```java
import net.gonbay.app.task.engine.vo.TaskPathVO;
import net.gonbay.app.task.engine.vo.TaskForecastVO;

public void identifyCriticalPaths(List<TaskPathVO> paths, Map<Long, Object> timeCalculations) {
    for (TaskPathVO path : paths) {
        boolean isCritical = path.getPathNodes().stream()
            .allMatch(taskId -> {
                Object calculation = timeCalculations.get(taskId);
                if (calculation instanceof TaskTimeCalculationVO) {
                    // 正确：使用导入的类名
                    TaskTimeCalculationVO calc = (TaskTimeCalculationVO) calculation;
                    // 正确：使用导入的类名
                    return calc.getTotalFloat() != null && calc.getTotalFloat().compareTo(BigDecimal.ZERO) == 0;
                }
                return false;
            });
    }
}
```

#### 2.3.3 泛型参数

**❌ 反例：使用全类名**

```java
public void example() {
    List<net.gonbay.app.task.engine.vo.TaskPathVO> paths = pathCalculationService.calculateAllPaths(instanceId, graph);
    // 错误：泛型参数使用全类名
    
    Map<Long, net.gonbay.app.task.engine.vo.TaskForecastVO> timeCalculations = 
        timeCalculationService.calculateAllTaskTimes(instanceId, graph);
    // 错误：泛型参数使用全类名
}
```

**✅ 正例：使用 import 导入**

```java
import net.gonbay.app.task.engine.vo.TaskPathVO;
import net.gonbay.app.task.engine.vo.TaskForecastVO;

public void example() {
    List<TaskPathVO> paths = pathCalculationService.calculateAllPaths(instanceId, graph);
    // 正确：使用导入的类名
    
    Map<Long, TaskTimeCalculationVO> timeCalculations = 
        timeCalculationService.calculateAllTaskTimes(instanceId, graph);
    // 正确：使用导入的类名
}
```

#### 2.3.4 Wrappers 泛型参数

**❌ 反例：使用全类名**

```java
private List<Long> getPredecessors(Long taskId, Long instanceId) {
    return taskRelService.list(
        Wrappers.<net.gonbay.app.task.engine.domain.TaskRel>lambdaQuery()
            .eq(net.gonbay.app.task.engine.domain.TaskRel::getInstanceId, instanceId)
            .eq(net.gonbay.app.task.engine.domain.TaskRel::getCurTaskId, taskId)
    ).stream()
        .map(net.gonbay.app.task.engine.domain.TaskRel::getPreTaskId)
        .collect(Collectors.toList());
}
```

**✅ 正例：使用 import 导入**

```java
import net.gonbay.app.task.engine.domain.TaskRel;

private List<Long> getPredecessors(Long taskId, Long instanceId) {
    return taskRelService.list(
        Wrappers.<TaskRel>lambdaQuery()
            .eq(TaskRel::getInstanceId, instanceId)
            .eq(TaskRel::getCurTaskId, taskId)
    ).stream()
        .map(TaskRel::getPreTaskId)
        .collect(Collectors.toList());
}
```

---

## 3. 逻辑删除查询规范

### 3.1 规则说明

项目中的 `BasicService` 和 `BasicServiceImpl` 已经实现了逻辑删除的自动过滤，所有继承自 `BasicService` 的查询方法都会自动过滤已删除的记录。因此，在构建查询条件时，不需要显式添加 `.eq(Entity::getDeleted, 0)` 条件。

### 3.2 必须遵守的规则

**必须遵守**
- ✅ 所有查询条件构建时，不需要显式添加逻辑删除判断
- ✅ 依赖 `BasicService` 提供的自动逻辑删除过滤功能
- ✅ 保持查询条件的简洁性

**禁止行为**
- ❌ 禁止在查询条件中显式添加 `.eq(Entity::getDeleted, 0)`
- ❌ 禁止在查询条件中显式添加 `.eq(Entity::getDeleted, false)`
- ❌ 禁止手动判断逻辑删除状态

### 3.3 正例和反例

#### 3.3.1 基本查询条件

**❌ 反例：显式添加逻辑删除判断**

```java
@Override
public Map<Long, List<Long>> buildDAG(Long instanceId) {
    // 错误：显式添加逻辑删除判断
    List<Task> tasks = taskService.list(
        Wrappers.<Task>lambdaQuery()
            .eq(Task::getInstanceId, instanceId)
            .and(w -> w.isNull(Task::getPid).or().eq(Task::getPid, instanceId))
            .eq(Task::getDeleted, 0)  // 错误：不需要显式判断
    );
    
    // 错误：显式添加逻辑删除判断
    List<TaskRel> dependencies = taskRelService.list(
        Wrappers.<TaskRel>lambdaQuery()
            .eq(TaskRel::getInstanceId, instanceId)
            .in(TaskRel::getPreTaskId, taskIds)
            .in(TaskRel::getCurTaskId, taskIds)
            .eq(TaskRel::getDeleted, 0)  // 错误：不需要显式判断
    );
    
    return buildGraph(tasks, dependencies);
}
```

**✅ 正例：依赖自动逻辑删除过滤**

```java
@Override
public Map<Long, List<Long>> buildDAG(Long instanceId) {
    // 正确：不需要显式判断逻辑删除，BasicService 会自动过滤
    List<Task> tasks = taskService.list(
        Wrappers.<Task>lambdaQuery()
            .eq(Task::getInstanceId, instanceId)
            .and(w -> w.isNull(Task::getPid).or().eq(Task::getPid, instanceId))
    );
    
    // 正确：不需要显式判断逻辑删除
    List<TaskRel> dependencies = taskRelService.list(
        Wrappers.<TaskRel>lambdaQuery()
            .eq(TaskRel::getInstanceId, instanceId)
            .in(TaskRel::getPreTaskId, taskIds)
            .in(TaskRel::getCurTaskId, taskIds)
    );
    
    return buildGraph(tasks, dependencies);
}
```

#### 3.3.2 统计查询

**❌ 反例：显式添加逻辑删除判断**

```java
private void updateParentTaskStatistics(Long parentTaskId) {
    // 错误：显式添加逻辑删除判断
    long completedCount = taskService.count(
        Wrappers.<Task>lambdaQuery()
            .eq(Task::getPid, parentTaskId)
            .eq(Task::getDeleted, 0)  // 错误：不需要显式判断
            .in(Task::getStatus,
                TaskStatusEnum.COMPLETED.name(),
                TaskStatusEnum.CANCELLED.name()
            )
    );
    
    Task parentTask = taskService.getById(parentTaskId);
    parentTask.setCompletedSubTaskCount((int) completedCount);
    taskService.updateById(parentTask);
}
```

**✅ 正例：依赖自动逻辑删除过滤**

```java
private void updateParentTaskStatistics(Long parentTaskId) {
    // 正确：不需要显式判断逻辑删除
    long completedCount = taskService.count(
        Wrappers.<Task>lambdaQuery()
            .eq(Task::getPid, parentTaskId)
            .in(Task::getStatus,
                TaskStatusEnum.COMPLETED.name(),
                TaskStatusEnum.CANCELLED.name()
            )
    );
    
    Task parentTask = taskService.getById(parentTaskId);
    parentTask.setCompletedSubTaskCount((int) completedCount);
    taskService.updateById(parentTask);
}
```

#### 3.3.3 关联查询

**❌ 反例：显式添加逻辑删除判断**

```java
private List<Long> findDownstreamTasks(Long taskId, Long instanceId) {
    // 错误：显式添加逻辑删除判断
    List<TaskRel> dependencies = taskRelService.list(
        Wrappers.<TaskRel>lambdaQuery()
            .eq(TaskRel::getInstanceId, instanceId)
            .eq(TaskRel::getPreTaskId, taskId)
            .eq(TaskRel::getDeleted, 0)  // 错误：不需要显式判断
    );
    
    return dependencies.stream()
        .map(TaskRel::getCurTaskId)
        .collect(Collectors.toList());
}
```

**✅ 正例：依赖自动逻辑删除过滤**

```java
private List<Long> findDownstreamTasks(Long taskId, Long instanceId) {
    // 正确：不需要显式判断逻辑删除
    List<TaskRel> dependencies = taskRelService.list(
        Wrappers.<TaskRel>lambdaQuery()
            .eq(TaskRel::getInstanceId, instanceId)
            .eq(TaskRel::getPreTaskId, taskId)
    );
    
    return dependencies.stream()
        .map(TaskRel::getCurTaskId)
        .collect(Collectors.toList());
}
```

#### 3.3.4 Server 层查询

**❌ 反例：显式添加逻辑删除判断**

```java
@Override
public List<TaskVO> getTasksByInstanceId(Long instanceId) {
    // 错误：显式添加逻辑删除判断
    List<Task> tasks = taskService.list(
        Wrappers.<Task>lambdaQuery()
            .eq(Task::getInstanceId, instanceId)
            .eq(Task::getDeleted, 0)  // 错误：不需要显式判断
    );
    return taskConverter.domainToVo(tasks);
}

@Override
public List<TaskRiskLevelVO> getByTaskId(Long taskId) {
    // 错误：显式添加逻辑删除判断
    List<TaskRiskLevel> riskLevels = taskRiskLevelService.list(
        Wrappers.<TaskRiskLevel>lambdaQuery()
            .eq(TaskRiskLevel::getTaskId, taskId)
            .eq(TaskRiskLevel::getDeleted, 0)  // 错误：不需要显式判断
            .orderByDesc(TaskRiskLevel::getCalculatedAt)
    );
    return taskRiskLevelConverter.domainToVo(riskLevels);
}
```

**✅ 正例：依赖自动逻辑删除过滤**

```java
@Override
public List<TaskVO> getTasksByInstanceId(Long instanceId) {
    // 正确：不需要显式判断逻辑删除
    List<Task> tasks = taskService.list(
        Wrappers.<Task>lambdaQuery()
            .eq(Task::getInstanceId, instanceId)
    );
    return taskConverter.domainToVo(tasks);
}

@Override
public List<TaskRiskLevelVO> getByTaskId(Long taskId) {
    // 正确：不需要显式判断逻辑删除
    List<TaskRiskLevel> riskLevels = taskRiskLevelService.list(
        Wrappers.<TaskRiskLevel>lambdaQuery()
            .eq(TaskRiskLevel::getTaskId, taskId)
            .orderByDesc(TaskRiskLevel::getCalculatedAt)
    );
    return taskRiskLevelConverter.domainToVo(riskLevels);
}
```

---

## 4. 属性注释规范

### 4.1 规则说明

所有 DTO、VO、领域模型（Domain）类中的属性必须有清晰的 JavaDoc 注释，注释应该简洁明了，准确描述属性的含义和用途。

### 4.2 必须遵守的规则

**必须遵守**
- ✅ 所有 DTO 类属性必须有 JavaDoc 注释
- ✅ 所有 VO 类属性必须有 JavaDoc 注释
- ✅ 所有 Domain 类属性必须有 JavaDoc 注释
- ✅ 注释必须清晰简洁，准确描述属性含义
- ✅ 注释格式使用标准的 JavaDoc 格式（`/** */`）

**禁止行为**
- ❌ 禁止属性没有注释
- ❌ 禁止使用无意义的注释（如 `// 属性`、`// 字段`）
- ❌ 禁止注释过于冗长或包含无关信息
- ❌ 禁止注释与实际属性含义不符

### 4.3 正例和反例

#### 4.3.1 DTO 类属性注释

**❌ 反例：属性没有注释**

```java
/**
 * 新增任务实例 DTO
 *
 * @author Floatin
 */
@Data
public class TaskInstanceCreateDTO {

    private Long operatorId;
    private Long moduleId;
    private String name;
    private String bizType;
    private String status;
    private LocalDateTime expectedCompleteTime;
    private JsonNode bizEntity;
    private String remark;
}
```

**✅ 正例：属性有清晰注释**

```java
/**
 * 新增任务实例 DTO
 *
 * @author Floatin
 */
@Data
public class TaskInstanceCreateDTO {

    /**
     * 操作人ID
     */
    private Long operatorId;

    /**
     * 任务模型ID
     */
    private Long moduleId;

    /**
     * 任务实例名称
     */
    private String name;

    /**
     * 业务类型
     */
    private String bizType;

    /**
     * 状态
     */
    private String status;

    /**
     * 预期完成时间
     */
    private LocalDateTime expectedCompleteTime;

    /**
     * 业务实体（JSON格式）
     */
    private JsonNode bizEntity;

    /**
     * 描述
     */
    private String remark;
}
```

#### 4.3.2 VO 类属性注释

**❌ 反例：属性没有注释（仅依赖注解）**

```java
/**
 * 任务实例 VO
 *
 * @author Floatin
 */
@Data
public class TaskInstanceVO {

    @ViewFieldMarked(value = "任务实例ID", visibled = true, isDbMapped = true)
    private Long id;

    @ViewFieldMarked(value = "公司ID", visibled = true, isDbMapped = true)
    private Long companyId;

    @ViewFieldMarked(value = "操作人ID", visibled = true, isDbMapped = true)
    private Long operatorId;
}
```

**✅ 正例：属性有清晰注释（同时保留注解）**

```java
/**
 * 任务实例 VO
 *
 * @author Floatin
 */
@Data
public class TaskInstanceVO {

    /**
     * 任务实例ID
     */
    @ViewFieldMarked(value = "任务实例ID", visibled = true, isDbMapped = true)
    private Long id;

    /**
     * 公司ID
     */
    @ViewFieldMarked(value = "公司ID", visibled = true, isDbMapped = true)
    private Long companyId;

    /**
     * 操作人ID
     */
    @ViewFieldMarked(value = "操作人ID", visibled = true, isDbMapped = true)
    private Long operatorId;
}
```

#### 4.3.3 Domain 类属性注释

**❌ 反例：属性没有注释**

```java
/**
 * 任务实例
 *
 * @author Floatin
 */
@Data
@FieldNameConstants
@TableName("tm_task_instance")
public class TaskInstance extends BaseModel {

    private Long operatorId;
    private Long moduleId;
    private String name;
    private String bizType;
    private String status;
}
```

**✅ 正例：属性有清晰注释**

```java
/**
 * 任务实例
 *
 * @author Floatin
 */
@Data
@FieldNameConstants
@TableName("tm_task_instance")
public class TaskInstance extends BaseModel {

    /**
     * 操作人ID
     */
    private Long operatorId;

    /**
     * 任务模型ID
     */
    private Long moduleId;

    /**
     * 任务实例名称
     */
    private String name;

    /**
     * 业务类型
     */
    private String bizType;

    /**
     * 状态
     */
    private String status;
}
```

### 4.4 注释编写规范

**注释格式**
- ✅ 使用标准的 JavaDoc 格式：`/** */`
- ✅ 注释内容简洁明了，通常一行即可
- ✅ 如需补充说明，可在括号内添加（如：`预期完成时间（业务期望）`）

**注释内容**
- ✅ 准确描述属性的业务含义
- ✅ 对于枚举类型字段，可说明可选值
- ✅ 对于时间字段，可说明时间来源或计算方式
- ✅ 对于关联字段，可说明关联关系

**禁止内容**
- ❌ 禁止复制属性名作为注释（如：`operatorId` 的注释不能是 `操作人ID` 如果属性名已经很清楚）
- ❌ 禁止包含实现细节
- ❌ 禁止包含过时的信息
- ❌ 禁止使用无意义的词汇（如：`字段`、`属性`、`变量`）

---

## 5. 违规处理

如果发现违反本规则的情况：
1. 立即停止当前操作
2. 修复所有违规代码
3. 确保所有类引用都通过 import 导入
4. 移除所有显式的逻辑删除判断条件
5. 为所有 DTO、VO、Domain 类属性添加清晰的 JavaDoc 注释

---

## 6. 规则优先级

本规则优先级：**高**
- 与其他规则冲突时，以本规则为准
- 除非用户明确要求，否则不得违反本规则
- 所有代码生成和修改都必须遵守本规则

---

## 7. 检查清单

在提交代码前，请确保：
- ✅ 所有类引用都通过 import 语句导入
- ✅ 代码中没有任何全类名引用
- ✅ 所有查询条件都没有显式的逻辑删除判断
- ✅ 所有 Service 层查询都依赖 BasicService 的自动逻辑删除过滤
- ✅ 所有 DTO 类属性都有清晰的 JavaDoc 注释
- ✅ 所有 VO 类属性都有清晰的 JavaDoc 注释
- ✅ 所有 Domain 类属性都有清晰的 JavaDoc 注释
- ✅ Service 之间无循环依赖（见第 8 节）

---

## 8. 循环依赖规避规范

### 8.1 规则说明

Spring 容器中，若 A 依赖 B、B 又依赖 A（或 A→B→C→A），会形成循环依赖，导致启动失败或行为异常。状态类、编排类 Service 之间尤其容易出现互相引用。本规则要求：**禁止 Service 之间形成循环依赖**；**获取实体（instance、task 等）的职责放在调用方，通过参数传入，状态/编排服务内部不再按 ID 反查**。

### 8.2 必须遵守的规则

**必须遵守**
- ✅ Service 实现类之间不得形成环状依赖（A 注入 B 且 B 注入 A，或更长环路）。
- ✅ 状态/编排类服务（如任务状态、实例状态）若需要「当前实例、当前任务或列表」，应由**调用方**先查询，再通过方法参数传入；服务内部仅做校验、更新与发事件，不再为同一业务动作重复 getById/list。
- ✅ 若存在「A 需调 B、B 又需调 A」的业务逻辑，应拆分为：调用方先调 A、再调 B（或通过事件/监听器在另一侧执行），或将一方改为仅接收参数、由调用方负责取数后传入。
- ✅ 未使用的 Service 注入应移除，避免无意义的依赖链。

**禁止行为**
- ❌ 禁止两个及以上 Service 实现类互相注入（如 TaskStatusServiceImpl 注入 TaskInstanceStatusService，TaskInstanceStatusServiceImpl 又注入 TaskStatusService）。
- ❌ 禁止在状态/编排服务内部仅为了校验或判断而再次按 ID 查询调用方已持有的实体（导致重复查询且易形成循环依赖）。
- ❌ 禁止保留未被调用的 Service 依赖（死注入）。

### 8.3 推荐做法

- **参数化**：状态服务接口设计为接收「实体或实体列表」入参，例如 `startInstance(TaskInstance instance)`、`checkAndUpdateToPendingComplete(TaskInstance instance, List<Task> tasks)`，由 Controller/Server 或监听器先查 instance/tasks 再调用。
- **职责拆分**：例如「取消实例」时，由调用方先对实例下所有任务调用任务状态服务更新为取消，再调用实例状态服务仅更新实例并发事件；实例状态服务不再依赖任务状态服务。
- **事件解耦**：若「任务完成」后需触发「检查实例是否待完成」，可由任务状态服务只发布事件，由监听器查询 instance 与 rootTasks 后调用实例状态服务，避免任务状态服务直接依赖实例状态服务。

### 8.4 正例与反例

**❌ 反例：Service 互相注入且内部重复查库**

```java
// TaskInstanceStatusServiceImpl 内
@Autowired
private TaskStatusService taskStatusService;

public void cancelInstance(Long instanceId) {
    TaskInstance taskInstance = taskInstanceService.getById(instanceId);  // 内部查 instance
    List<Task> tasks = taskService.list(...);                             // 内部查 tasks
    for (Task task : tasks) {
        taskStatusService.updateTaskStatus(task.getId(), ...);            // 依赖 TaskStatusService，易形成循环
    }
    // ...
}

// TaskStatusServiceImpl 内
@Autowired
private TaskInstanceStatusService taskInstanceStatusService;

public void completeTask(Long taskId, ...) {
    Task task = taskService.getById(taskId);
    // ...
    taskInstanceStatusService.checkAndUpdateToPendingComplete(task.getInstanceId());  // 又依赖 TaskInstanceStatusService → 循环
}
```

**✅ 正例：调用方取数传参，状态服务不互相依赖**

```java
// 接口：以实体入参，由调用方取数
void cancelInstance(TaskInstance instance);
void checkAndUpdateToPendingComplete(TaskInstance instance, List<Task> tasks);

// TaskInstanceStatusServiceImpl：不再注入 TaskStatusService，不再 getById(instanceId)/list(tasks)
public void cancelInstance(TaskInstance instance) {
    // 仅更新 instance、发事件；任务的取消由调用方先调 TaskStatusService
}

// TaskInstanceServer（调用方）：先查 instance 和 tasks，再分别调任务状态与实例状态
public void cancelInstance(Long instanceId) {
    TaskInstance instance = taskInstanceService.getById(instanceId);
    List<Task> tasks = taskService.list(taskConverter.toQueryWrapperByInstanceId(instanceId));
    for (Task task : tasks) {
        taskStatusService.updateTaskStatus(task.getId(), CALCELED.name(), CANCELLED.name());
    }
    taskInstanceStatusService.cancelInstance(instance);
}

// TaskStatusServiceImpl：不再注入 TaskInstanceStatusService；实例待完成由监听器查 instance+rootTasks 后调用
// 监听器内：
TaskInstance instance = taskInstanceService.getById(instanceId);
List<Task> rootTasks = taskService.list(taskConverter.toQueryWrapperRootTasksByInstanceId(instanceId));
taskInstanceStatusService.checkAndUpdateToPendingComplete(instance, rootTasks);
```

### 8.5 违规处理

若发现 Service 间存在循环依赖或违反「调用方取数传参」的用法：
1. 将「获取 instance/task」的职责上移到调用方（Controller/Server 或事件监听器）。
2. 状态/编排服务接口改为接收实体或实体列表入参，实现类去掉对另一状态服务的依赖。
3. 移除未使用的 Service 注入。
4. 必要时用事件+监听器拆开双向调用，保证依赖为单向。

