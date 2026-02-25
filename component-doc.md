# API 注释规范与 Smart-Doc 使用规则

> 强制性约束 · 文档生成规范 · 全局适用

## 1. 规则目的

本规则定义了 API 层、DTO、VO、DOMAIN 等领域模型的注释规范，以及 Smart-Doc 第三方文档生成组件的使用规范。确保生成的 API 文档清晰、完整、符合项目标准。

> 注释与文档规则以本文件为唯一权威来源，其他规则文件仅作引用说明。

---

## 2. 注释生成规则

### 2.1 注释语言与格式

**必须遵守**
- ✅ 所有注释使用**简体中文**
- ✅ 注释内容简洁明了，不说废话
- ✅ 类级别注释使用 `@author Floatin`（统一作者标识）
- ✅ 方法级别注释使用 `@apiNote` 补充业务说明

**禁止行为**
- ❌ 禁止使用英文注释（除非技术术语无法翻译）
- ❌ 禁止冗余、重复的注释内容
- ❌ 禁止注释与代码实现不一致

---

### 2.2 API 接口注释规范

#### 2.2.1 接口类注释

```java
/**
 * 字典项管理-Api
 *
 * @author Floatin
 * @apiNote 字典项管理相关接口说明
 * @restApi
 */
@FeignClient(name = "gonbay-platform-dictionary", contextId = "dictItemApi")
public interface DictItemApi {
    // ...
}
```

**规范要求**
- ✅ 类注释必须包含功能描述
- ✅ 必须包含 `@author Floatin`
- ✅ 必须包含 `@apiNote` 说明接口用途
- ✅ 必须包含 `@restApi` 标记（用于 Smart-Doc 识别）

---

#### 2.2.2 接口方法注释

**基础格式**
```java
/**
 * 方法功能描述
 * @extension function {方法名称}
 * @extension in {入参类型}（如有）
 * @extension out {出参类型}（如有）
 * @param {参数名} {参数说明}
 * @return {返回值说明}
 * @apiNote {业务说明}
 */
```

**示例**
```java
/**
 * 快速批量插入字典项
 * @extension function add
 * @extension in BatchDictItemDTO
 * @param dto 字典项信息
 * @apiNote 快速批量插入字典项，不处理字典项扩展数据，只处理同一级字典项
 */
@PostMapping(name = "新增字典项", value = "/dict/items")
void add(@Validated(FormVerifyGroup.Insert.class) @RequestBody BatchDictItemDTO dto);
```

---

### 2.3 DTO/VO/DOMAIN 注释规范

#### 2.3.1 类级别注释

```java
/**
 * 字典项DTO
 * @author Floatin
 */
@Data
public class DictItemDTO {
    // ...
}
```

**规范要求**
- ✅ 类注释必须说明类的用途（DTO/VO/DOMAIN）
- ✅ 必须包含 `@author Floatin`

---

#### 2.3.2 属性级别注释

**基础属性注释**
```java
/**
 * 唯一标识
 */
@NotNull(groups = {FormVerifyGroup.Update.class}, message = "ID不能为空")
private Long id;
```

**复合类型属性注释（必须使用 @extension name）**
```java
/**
 * 字典项扩展数据
 * @extension name DictItemExtensionDTO
 * @apiNote 一般情况下只要单条字典项更新时才维护，批量更新和新增操作属于快捷操作，不维护扩展树形
 */
@Null(groups = {FormVerifyGroup.Insert.class})
private DictItemExtensionDTO extension;
```

---

## 3. @extension 使用规范

### 3.1 @extension function - 方法名称标记

**规则**
- ✅ **每一个接口的请求方法都必须具备** `@extension function {当前方法名称}`
- ✅ 方法名称使用驼峰命名，与 Java 方法名保持一致
- ✅ 必须放在方法注释的第一行或紧随功能描述之后

**示例**
```java
/**
 * 快速批量插入字典项
 * @extension function add
 * @extension in BatchDictItemDTO
 */
@PostMapping(name = "新增字典项", value = "/dict/items")
void add(@Validated(FormVerifyGroup.Insert.class) @RequestBody BatchDictItemDTO dto);
```

**错误示例**
```java
// ❌ 缺少 @extension function
/**
 * 快速批量插入字典项
 * @extension in BatchDictItemDTO
 */
void add(@RequestBody BatchDictItemDTO dto);
```

---

### 3.2 @extension in - 入参类型标记

**规则**
- ✅ **每个接口的请求方法如果有入参（复合类型：领域对象、对象集合等）必须标记** `@extension in {入参类型}`
- ✅ 如果新增和修改使用同一个 DTO，需要加前缀区分：
  - 新增场景：`@extension in Create{DTO类型}`，例如 `@extension in CreateDictItemDTO`
  - 修改场景：`@extension in Update{DTO类型}`，例如 `@extension in UpdateDictItemDTO`
- ✅ 基础类型（String、Long、Integer、List<String> 等）不需要标记
- ✅ 复合类型（DTO、VO、DOMAIN 及其集合）必须标记

**示例 - 单一 DTO**
```java
/**
 * 修改字典项
 * @extension function update
 * @extension in DictItemDTO
 * @param dto 字典项信息
 */
@PutMapping(name = "修改字典项", value = "/dict/item")
void update(@Validated(FormVerifyGroup.Update.class) @RequestBody DictItemDTO dto);
```

**示例 - 新增和修改使用同一 DTO（需要加前缀）**
```java
// 假设 DictItemDTO 同时用于新增和修改
/**
 * 新增字典项
 * @extension function add
 * @extension in CreateDictItemDTO
 * @param dto 字典项信息
 */
@PostMapping(name = "新增字典项", value = "/dict/item")
void add(@Validated(FormVerifyGroup.Insert.class) @RequestBody DictItemDTO dto);

/**
 * 修改字典项
 * @extension function update
 * @extension in UpdateDictItemDTO
 * @param dto 字典项信息
 */
@PutMapping(name = "修改字典项", value = "/dict/item")
void update(@Validated(FormVerifyGroup.Update.class) @RequestBody DictItemDTO dto);
```

**示例 - 集合类型**
```java
/**
 * 批量删除字典项
 * @extension function batchDelete
 * @param codes 字典项code列表
 */
@DeleteMapping(name = "批量删除字典项", value = "/dict/item/batch")
void batchDelete(@NotNull @RequestBody List<String> codes);
// 注意：List<String> 是基础类型集合，不需要 @extension in
```

---

### 3.3 @extension out - 出参类型标记

**规则**
- ✅ **每个接口的请求方法如果有出参（复合类型：领域对象、领域对象集合）必须标记** `@extension out {返回值类型}`
- ✅ 基础类型（String、Long、Integer、void 等）不需要标记
- ✅ 复合类型（DTO、VO、DOMAIN 及其集合）必须标记

**示例 - 单一 VO**
```java
/**
 * 字典项详情：code
 * @extension function getByCode
 * @extension out DictItemVO
 * @param code 字典项编码
 * @return 字典项信息
 */
@GetMapping(name = "根据code获取字典项", value = "/dict/item-by-code")
DictItemVO get(@RequestParam("code") @NotNull String code);
```

**示例 - 集合类型**
```java
/**
 * 获取字典项树型列表
 * @extension function getByDictTypeCode
 * @extension out TidyDictItemVO
 * @param dictTypeCode 字典分类编码
 * @return 字典项列表
 */
@GetMapping(name = "根据字典分类编码获取字典项列表", value = "/dict/item/type")
List<TidyDictItemVO> getByDictTypeCode(@RequestParam("dictTypeCode") @NotNull String dictTypeCode);
```

**示例 - void 返回**
```java
/**
 * 快速批量插入字典项
 * @extension function add
 * @extension in BatchDictItemDTO
 * @param dto 字典项信息
 */
@PostMapping(name = "新增字典项", value = "/dict/items")
void add(@Validated(FormVerifyGroup.Insert.class) @RequestBody BatchDictItemDTO dto);
// 注意：void 返回不需要 @extension out
```

---

### 3.4 @extension query - 查询方法标记

**规则**
- ✅ **所有标记 `@PostMapping` 的请求如果是查询请求方法，则需要在注释上追加** `@extension query` **注释用来标记这是查询方法**
- ✅ 仅适用于使用 `@PostMapping` 但实际是查询操作的接口
- ✅ 标记位置：在 `@extension function` 之后，其他 `@extension` 标记之前

**使用场景**
- 使用 POST 方法进行复杂查询（通常因为查询参数复杂，需要使用 RequestBody）
- 使用 POST 方法进行分页查询
- 使用 POST 方法进行条件查询

**示例 - POST 查询方法**
```java
/**
 * 获取字典项路由信息
 * @extension function getDictItemRouterInfo
 * @extension query
 * @extension out RouterDictItemVo
 * @param codes 字典项code列表
 * @return 字典项路由信息
 * @apiNote 获取字典项路由信息，目前设计思路是帮助前端做字典组件回显渲染时做数据支撑
 */
@PostMapping(name = "获取字典项路由信息", value = "/dict/item/router/info")
List<RouterDictItemVo> getDictItemRouterInfo(@NotNull @RequestBody List<String> codes);
```

**示例 - POST 分页查询方法**
```java
/**
 * 分页查询字典项列表
 * @extension function pageQuery
 * @extension query
 * @extension in DictItemQueryDTO
 * @extension out PageResult<DictItemVO>
 * @param queryDTO 查询条件
 * @return 分页结果
 * @apiNote 分页查询字典项列表，支持多条件组合查询
 */
@PostMapping(name = "分页查询字典项", value = "/dict/item/page")
PageResult<DictItemVO> pageQuery(@RequestBody DictItemQueryDTO queryDTO);
```

**注意**
- ❌ 非查询操作的 POST 方法（如新增、修改）不需要标记 `@extension query`
- ❌ GET 请求方法不需要标记 `@extension query`（GET 方法本身就是查询）

---

### 3.5 @extension name - 嵌套类型标记

**规则**
- ✅ **如果入参实体和出参实体都是领域模型，且其中还有其他领域模型，则必须在注释上标记** `@extension name {具体类型名称}`
- ✅ **如果某个领域对象实体中有属性是 JsonNode 或者 Map 结构，也需要指定** `@extension name {类型名称}`
- ✅ 如果当前属性类型中还有嵌套的领域模型，则依次复用这个规则（递归标记）
- ✅ 标记位置：在属性注释中，紧跟在属性说明之后

**使用场景**
1. DTO 中包含其他 DTO/DOMAIN 类型属性
2. VO 中包含其他 VO/DOMAIN 类型属性
3. DOMAIN 中包含其他 DOMAIN 类型属性
4. 集合类型中的元素是领域模型
5. **属性类型是 JsonNode（如 `com.fasterxml.jackson.databind.JsonNode`）**
6. **属性类型是 Map（如 `Map<String, String>`、`Map<String, Object>` 等）**

**示例 - DTO 中包含 DTO**
```java
/**
 * 字典项DTO
 * @author Floatin
 */
@Data
public class DictItemDTO {
    /**
     * 字典项扩展数据
     * @extension name DictItemExtensionDTO
     * @apiNote 一般情况下只要单条字典项更新时才维护，批量更新和新增操作属于快捷操作，不维护扩展树形
     */
    @Null(groups = {FormVerifyGroup.Insert.class})
    private DictItemExtensionDTO extension;
}
```

**示例 - DTO 中包含集合类型（集合元素是领域模型）**
```java
/**
 * 批量字典项操作
 * @author Floatin
 */
@Data
public class BatchDictItemDTO {
    /**
     * 批量字典项
     * 批量添加维护字典项值
     * @extension name DictItemDTO
     */
    @NotEmpty(groups = {FormVerifyGroup.Insert.class, FormVerifyGroup.Update.class})
    private List<DictItemDTO> items;
}
```

**示例 - VO 中包含 DOMAIN**
```java
/**
 * 字典项VO
 * @author Floatin
 */
@Data
public class DictItemVO {
    /**
     * 字典项扩展数据
     * @extension name DictItemExtension
     */
    private DictItemExtension extension;
}
```

**示例 - VO 中包含集合类型（集合元素是领域模型）**
```java
/**
 * 字典分类VO
 * @author Floatin
 */
@Data
public class DictTypeVO {
    /**
     * 字典项的值，在列表接口中不做获取
     * @extension name DictItem
     */
    private List<DictItem> items;
}
```

**示例 - 属性类型是 JsonNode**
```java
/**
 * 字典项扩展数据
 * @author Floatin
 */
@Data
public class DictItemExtensionDTO {
    /**
     * 扩展变量，json 类型，可以传任意结构的 json 数据
     * 根据业务来自定义扩展属性
     * @extension name JsonNode
     */
    private JsonNode variables;
}
```

**示例 - 属性类型是 Map**
```java
/**
 * 字典项路由信息VO
 * @author Floatin
 */
@Data
public class RouterDictItemVo {
    /**
     * 路由
     * @extension name Map
     */
    private Map<String, String> routes;
}
```

**示例 - VO 中包含 Map 类型（字典元数据）**
```java
/**
 * 字典元数据VO
 * @author Floatin
 */
@Getter
public class DictMetaVO {
    /**
     * 字典级别枚举，字典分类的levelType取值范围
     * @extension name Map
     */
    private Map<String, String> dictLevelTypes = EnumUtil.mataEnumInfo(DictLevelTypeEnum.class);

    /**
     * 可见类型枚举，字典项中 enableFlag 的取值范围
     * @extension name Map
     */
    private Map<String, String> visiableTypes = EnumUtil.mataEnumInfo(VisibleType.class);
}
```

**示例 - 嵌套多层领域模型（递归标记）**
```java
// 假设 DictItemExtensionDTO 中包含 DictItemDTO
/**
 * 字典项扩展数据
 * @author Floatin
 */
@Data
public class DictItemExtensionDTO {
    /**
     * 扩展变量，json 类型，可以传任意结构的 json 数据
     * 根据业务来自定义扩展属性
     * @extension name JsonNode
     */
    private JsonNode variables;
    
    /**
     * 关联的字典项（假设存在）
     * @extension name DictItemDTO
     */
    private DictItemDTO relatedItem;
}
```

---

## 4. Smart-Doc 配置说明

### 4.1 配置文件位置

配置文件位于：`gonbay-platform-dictionary-api/src/main/resources/smart-doc.json`

### 4.2 配置项说明

```json
{
  "openApiTagNameType": "CLASS_NAME",        // API 标签命名方式：使用类名
  "outPath": "./doc/",                       // 文档输出路径
  "packageFilters": "net.gonbay.*",         // 包过滤规则
  "serverUrl": "http://localhost:7711",     // 服务器地址
  "isStrict": false,                        // 是否严格模式
  "projectName": "系统字典管理-Api",         // 项目名称
  "appToken": "b84e887708e44b9088dd872d2daa02f8",  // Torna 应用令牌
  "openUrl": "http://torna.gonbay.cn:7700/api",    // Torna 接口地址
  "debugEnvName": "本地测试环境",            // 调试环境名称
  "debugEnvUrl": "http://localhost:7079",   // 调试环境地址
  "tornaDebug": true,                       // 是否启用 Torna 调试
  "errorCodeDictionaries": [...],           // 错误码字典配置
  "dataDictionaries": [...]                 // 数据字典配置
}
```

### 4.3 错误码字典配置

```json
"errorCodeDictionaries": [
    {
        "title": "字典异常码",
        "enumClassName": "net.gonbay.app.dictionary.errcode.DictErrorCode",
        "codeField": "status",
        "descField": "message"
    }
]
```

### 4.4 数据字典配置

```json
"dataDictionaries": [
    {
        "title": "可见状态类型",
        "enumClassName": "net.gonbay.common.enums.VisibleType",
        "codeField": "code",
        "descField": "metaValue"
    },
    {
        "title": "应用类型-持续迭代",
        "enumClassName": "net.gonbay.common.enums.ApplicationType",
        "codeField": "code",
        "descField": "metaValue"
    },
    {
        "title": "字典等级类型",
        "enumClassName": "net.gonbay.app.dictionary.enums.DictLevelTypeEnum",
        "codeField": "code",
        "descField": "metaValue"
    }
]
```

---

## 5. 完整示例

### 5.1 API 接口完整示例

```java
/**
 * 字典项管理-Api
 *
 * @author Floatin
 * @apiNote 字典项管理相关接口说明
 * @restApi
 */
@FeignClient(name = "gonbay-platform-dictionary", contextId = "dictItemApi")
public interface DictItemApi {

    /**
     * 快速批量插入字典项
     * @extension function add
     * @extension in BatchDictItemDTO
     * @param dto 字典项信息
     * @apiNote 快速批量插入字典项，不处理字典项扩展数据，只处理同一级字典项
     */
    @PostMapping(name = "新增字典项", value = "/dict/items")
    void add(@Validated(FormVerifyGroup.Insert.class) @RequestBody BatchDictItemDTO dto);

    /**
     * 字典项详情：code
     * @extension function getByCode
     * @extension out DictItemVO
     * @param code 字典项编码
     * @return 字典项信息
     * @apiNote 字典项详情，包含对应的字典扩展数据
     */
    @GetMapping(name = "根据code获取字典项", value = "/dict/item-by-code")
    DictItemVO get(@RequestParam("code") @NotNull String code);

    /**
     * 获取字典项树型列表
     * @extension function getByDictTypeCode
     * @extension out TidyDictItemVO
     * @param dictTypeCode 字典分类编码
     * @return 字典项列表
     * @apiNote 根据字典分类获取字典项树型列表
     */
    @GetMapping(name = "根据字典分类编码获取字典项列表", value = "/dict/item/type")
    List<TidyDictItemVO> getByDictTypeCode(@RequestParam("dictTypeCode") @NotNull String dictTypeCode);

    /**
     * 获取字典项路由信息
     * @extension function getDictItemRouterInfo
     * @extension query
     * @extension out RouterDictItemVo
     * @param codes 字典项code列表
     * @return 字典项路由信息
     * @apiNote 获取字典项路由信息，目前设计思路是帮助前端做字典组件回显渲染时做数据支撑
     */
    @PostMapping(name = "获取字典项路由信息", value = "/dict/item/router/info")
    List<RouterDictItemVo> getDictItemRouterInfo(@NotNull @RequestBody List<String> codes);
}
```

### 5.2 DTO 完整示例

```java
/**
 * 字典项DTO
 * @author Floatin
 */
@Data
public class DictItemDTO {
    
    /**
     * 唯一标识
     */
    @NotNull(groups = {FormVerifyGroup.Update.class}, message = "ID不能为空")
    @Null(groups = {FormVerifyGroup.Insert.class})
    private Long id;

    /**
     * 字典分类code
     */
    @NotBlank(groups = {FormVerifyGroup.Insert.class, FormVerifyGroup.Update.class}, message = "字典分类code不能为空")
    @Size(max = 32, message = "字典分类code长度不能超过32个字符")
    private String dictTypeCode;

    /**
     * 字典项扩展数据
     * @extension name DictItemExtensionDTO
     * @apiNote 一般情况下只要单条字典项更新时才维护，批量更新和新增操作属于快捷操作，不维护扩展树形
     */
    @Null(groups = {FormVerifyGroup.Insert.class})
    private DictItemExtensionDTO extension;
}
```

### 5.3 VO 完整示例

```java
/**
 * 字典项VO
 * @author Floatin
 */
@Data
public class DictItemVO {
    
    /**
     * 唯一标识
     */
    private Long id;
    
    /**
     * 字典分类code
     */
    private String dictTypeCode;
    
    /**
     * 字典项扩展数据
     * @extension name DictItemExtension
     */
    private DictItemExtension extension;
}
```

---

## 6. 检查清单

在编写或修改 API 接口、DTO、VO、DOMAIN 时，请确保：

### 6.1 API 接口检查清单

- [ ] 接口类包含 `@author Floatin`、`@apiNote`、`@restApi`
- [ ] 每个接口方法都包含 `@extension function {方法名}`
- [ ] 有复合类型入参的方法都包含 `@extension in {类型}`
- [ ] 有复合类型出参的方法都包含 `@extension out {类型}`
- [ ] 使用 `@PostMapping` 的查询方法都包含 `@extension query`
- [ ] 新增和修改使用同一 DTO 时，已加前缀区分（Create/Update）
- [ ] 方法注释包含 `@apiNote` 业务说明
- [ ] 注释内容简洁明了，不说废话

### 6.2 DTO/VO/DOMAIN 检查清单

- [ ] 类注释包含 `@author Floatin`
- [ ] 类注释说明类的用途（DTO/VO/DOMAIN）
- [ ] 复合类型属性都包含 `@extension name {类型}`
- [ ] JsonNode 类型属性都包含 `@extension name JsonNode`
- [ ] Map 类型属性都包含 `@extension name Map`
- [ ] 嵌套的领域模型已递归标记 `@extension name`
- [ ] 属性注释简洁明了，说明属性用途

---

## 7. 违规示例与修正

### 7.1 缺少 @extension function

**错误示例**
```java
/**
 * 快速批量插入字典项
 * @extension in BatchDictItemDTO
 * @param dto 字典项信息
 */
@PostMapping(name = "新增字典项", value = "/dict/items")
void add(@RequestBody BatchDictItemDTO dto);
```

**正确示例**
```java
/**
 * 快速批量插入字典项
 * @extension function add
 * @extension in BatchDictItemDTO
 * @param dto 字典项信息
 */
@PostMapping(name = "新增字典项", value = "/dict/items")
void add(@RequestBody BatchDictItemDTO dto);
```

### 7.2 缺少 @extension in

**错误示例**
```java
/**
 * 修改字典项
 * @extension function update
 * @param dto 字典项信息
 */
@PutMapping(name = "修改字典项", value = "/dict/item")
void update(@RequestBody DictItemDTO dto);
```

**正确示例**
```java
/**
 * 修改字典项
 * @extension function update
 * @extension in DictItemDTO
 * @param dto 字典项信息
 */
@PutMapping(name = "修改字典项", value = "/dict/item")
void update(@RequestBody DictItemDTO dto);
```

### 7.3 缺少 @extension out

**错误示例**
```java
/**
 * 字典项详情：code
 * @extension function getByCode
 * @param code 字典项编码
 * @return 字典项信息
 */
@GetMapping(name = "根据code获取字典项", value = "/dict/item-by-code")
DictItemVO get(@RequestParam("code") @NotNull String code);
```

**正确示例**
```java
/**
 * 字典项详情：code
 * @extension function getByCode
 * @extension out DictItemVO
 * @param code 字典项编码
 * @return 字典项信息
 */
@GetMapping(name = "根据code获取字典项", value = "/dict/item-by-code")
DictItemVO get(@RequestParam("code") @NotNull String code);
```

### 7.4 缺少 @extension name

**错误示例**
```java
/**
 * 字典项DTO
 * @author Floatin
 */
@Data
public class DictItemDTO {
    /**
     * 字典项扩展数据
     */
    private DictItemExtensionDTO extension;
}
```

**正确示例**
```java
/**
 * 字典项DTO
 * @author Floatin
 */
@Data
public class DictItemDTO {
    /**
     * 字典项扩展数据
     * @extension name DictItemExtensionDTO
     */
    private DictItemExtensionDTO extension;
}
```

### 7.5 缺少 @extension query

**错误示例**
```java
/**
 * 获取字典项路由信息
 * @extension function getDictItemRouterInfo
 * @extension out RouterDictItemVo
 * @param codes 字典项code列表
 * @return 字典项路由信息
 */
@PostMapping(name = "获取字典项路由信息", value = "/dict/item/router/info")
List<RouterDictItemVo> getDictItemRouterInfo(@NotNull @RequestBody List<String> codes);
```

**正确示例**
```java
/**
 * 获取字典项路由信息
 * @extension function getDictItemRouterInfo
 * @extension query
 * @extension out RouterDictItemVo
 * @param codes 字典项code列表
 * @return 字典项路由信息
 */
@PostMapping(name = "获取字典项路由信息", value = "/dict/item/router/info")
List<RouterDictItemVo> getDictItemRouterInfo(@NotNull @RequestBody List<String> codes);
```

---

## 8. Smart-Doc 阻断检查清单

出现以下情况必须停止输出并说明原因：

- 缺少类级注释或 `@restApi` 标记
- 接口方法缺少 `@extension function`
- 复合类型入参/出参缺少 `@extension in` 或 `@extension out`
- DTO/VO/DOMAIN 缺少类级注释或字段注释

---

## 9. 执行优先级

1. **最高优先级**：`@extension function` 必须存在
2. **高优先级**：`@extension in`、`@extension out`、`@extension query` 必须正确标记
3. **中优先级**：`@extension name` 必须标记嵌套类型
4. **低优先级**：注释内容清晰度、格式规范

---

## 10. 合规声明

AI 编码助手在执行任何 API 接口、DTO、VO、DOMAIN 相关代码任务时，隐式声明：

> "本次操作完全符合《API 注释规范与 Smart-Doc 使用规则》中的所有约束项，包括但不限于注释生成规则、@extension 使用规范、Smart-Doc 配置等各方面的约束。"

---

## 11. 规则修改与扩展策略

AI 编码助手：
- 未经用户明确指示不得修改本规则文件
- 用户明确指示时允许更新，但必须记录变更原因与范围
- 不允许主动建议规则变更
- 不允许推断未来规则方向

> 规则的制定与演进仅由人类负责。

---

**文档版本**：v1.0  
**最后更新**：2026-01-06  
**维护者**：Floatin
