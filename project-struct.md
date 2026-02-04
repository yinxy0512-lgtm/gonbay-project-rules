# 初始化项目结构、分层、职责划分 规则文档

> 强制性约束 · 通用规范 · 全局适用
---

## 1. 规则目的
> 本规则定义在初期创建项目，和日常输出编码时 项目结构，包结构，代码结构，以及职责划分。 AI 编码助手在编写、修改任何 Java 代码时，必须强制严格遵守本规则。
---

## 2. 项目结构

### 2.1 项目模块结构

**规则**
- ✅ 使用maven分模块方式初始化项目结构(参考下方代码示例："项目结构")
- ✅ 模块之间依赖关系强制要求参考代码实例
- ✅ 禁止按照其他方式初始化项目结构

```
#项目结构
#✅必须按照这样的结构初始化先把古墓结构
gonbay-{project} 
    ├── pom.xml
    ├── README.md
    ├── gonbay-{project}-common
    │       ├── src
    │       │    ├── main
    │       │    │     ├── java
    │       ├── pom.xml
    ├── gonbay-{project}-api
    │       ├── src
    │       │    ├── main
    │       │    │     ├── java
    │       │    │     ├── resources
    │       ├── pom.xml
    ├── gonbay-{project}-service
    │       ├── src
    │       │    ├── main
    │       │    │     ├── java
    │       ├── pom.xml
    ├── gonbay-{project}-server
    │       ├── src
    │       │    ├── main
    │       │    │     ├── java
    │       │    │     ├── resources
    │       │    ├── test
    │       ├── pom.xml
```
**说明**
- 严格按照相同结构构建项目
- gonbay-{project} 项目根目录
- gonbay-{project}-common 该模块存放当前项目中公共的代码文件例如（domain、errcode、enums、constat 等）
- gonbay-{project}-api 该模块存放所有对外暴露的 api 接口、dto、vo
- gonbay-{project}-service 该模块存放所有业务逻辑实现和 orm 交互，是整个项目最核心的模块
- gonbay-{project}-server 该模块只负责写单元测试，只对 gonbay-{project}-service 依赖；可仅有 src/main/resources（如 application.yml）和 src/test，无 src/main/java 或可执行入口，禁止擅自添加 Spring Boot 启动类或打包插件

**当前项目参考（gonbay-task-engine）**
- 根：groupId=`net.gonbay.app.task.engine`，artifactId=`gonbay-task-engine`；parent=gonbay-core
- common：仅依赖 gonbay-basic-util；无 gonbay-common
- api：依赖 common、gonbay-basic-util、spring-cloud-openfeign-core、spring-boot-starter-validation
- service：依赖 api、gonbay-repository、gonbay-log、mysql-connector-java、gonbay-redisson、gonbay-cloud
- server：仅依赖 gonbay-task-engine-service；无 build/plugin、无 webflux/loadbalancer
- 生成或修改 pom 时必须以本仓库现有各模块 pom.xml 为准，不得套用其它项目模板。


### 2.2 模块中的包结构和职责划分
```java
# 模块结构中包的定义

# gonbay-{project} 是 pom 类型的跟模块，不需要包含任何包结构和代码文件

# gonbay-{project}-common 模块下的包明细：
    net.gonbay.app.{project}.constat 存放整个项目需要用到的常量类
    net.gonbay.app.{project}.domain  存放整个项目 orm 映射对应的实体类
    net.gonbay.app.{project}.enums 存放整个项目用到的枚举类
    net.gonbay.app.{project}.errcode 存放整个项目的自定义业务异常 code 类


# gonbay-{project}-api 模块下的包明细：
    net.gonbay.app.{project}.api 存放整个项目暴露的 api 接口类
    net.gonbay.app.{project}.dto 存放对应的dto 
    net.gonbay.app.{project}.vo 存放对应的 vo 

# gonbay-{project}-service 模块下的包明细：
    net.gonbay.app.{project}.mappers 存放orm 映射的 mapper 接口类
    net.gonbay.app.{project}.server 存放 api 定义的接口的实现类
    net.gonbay.app.{project}.service 存放业务接口
    net.gonbay.app.{project}.service.impl 存放业务实现类
    net.gonbay.app.{project}.struct 存放 MapStruct 转换器（一个 domain 对应一个 Convert）

# gonbay-{project}-server 不需要有任何包结构和代码（只是预留的模块）

```
**规则**
- ✅ 严格按照 "模块结构中包的定义"中的包结构 和职责划分定义包的结构
- ✅ 任何同职责的类都存放在指定职责的包结构中
- ✅ 禁止创建其他含义或结构的包
- ✅ 禁止将不属于一个职责的类放在其他职责的包中


## 2.3 POM 的模板生成规则

### 2.3.0 命名与引用约定（强制）

**groupId**
- 格式：`net.gonbay.app.{业务域}`，业务域为多级时用点号，例如 `net.gonbay.app.task.engine`。
- 所有子模块的 groupId 与根 POM 的 groupId 必须一致。
- ❌ 禁止使用 `net.gonbay.app.dictionary` 等其它项目占位符；禁止 groupId 与根 POM 不一致。

**artifactId 与 parent 引用**
- 根模块：`gonbay-{project}`（如 gonbay-task-engine）。根目录名、根 pom 的 `<artifactId>`、子模块 `<parent><artifactId>` 必须一致。
- common：`gonbay-{project}-common`。
- api：`gonbay-{project}-api`。
- service：`gonbay-{project}-service`。
- server：`gonbay-{project}-server`。
- ❌ 禁止使用 `gonbay-platform-{project}`、`gonbay-platform-{project}-common`、`gonbay-platform-{project}-api` 等任何带 `platform` 的 artifactId 或 parent 引用。

**版本**
- 子模块版本与根一致，使用 `1.0-SNAPSHOT` 或由父管理；子模块间依赖统一使用 `${project.parent.version}`，禁止写死其它项目的版本占位。

---

**根 pom：gonbay-{project}/pom.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>net.gonbay</groupId>
        <artifactId>gonbay-core</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath></relativePath>
    </parent>

    <packaging>pom</packaging>

    <modules>
        <module>gonbay-{project}-common</module>
        <module>gonbay-{project}-api</module>
        <module>gonbay-{project}-service</module>
        <module>gonbay-{project}-server</module>
    </modules>

    <groupId>net.gonbay.app.{业务域}</groupId>
    <artifactId>gonbay-{project}</artifactId>
    <version>1.0-SNAPSHOT</version>
</project>
```

**common：gonbay-{project}-common/pom.xml**
- parent 必须为：`groupId` = 根 groupId，`artifactId` = `gonbay-{project}`，`relativePath` = `../pom.xml`。
- artifactId 必须为 `gonbay-{project}-common`，name 为 `gonbay-{project}-common`。
- 依赖：仅允许项目已使用的公共组件（如 `gonbay-basic-util` 或 `gonbay-common` 之一，按项目现有约定）。禁止擅自改为未在项目中使用的 artifact。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>net.gonbay.app.{业务域}</groupId>
        <artifactId>gonbay-{project}</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <groupId>net.gonbay.app.{业务域}</groupId>
    <artifactId>gonbay-{project}-common</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>gonbay-{project}-common</name>
    <description>项目公共模块</description>

    <dependencies>
        <dependency>
            <groupId>net.gonbay</groupId>
            <artifactId>gonbay-basic-util</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
</project>
```

**api：gonbay-{project}-api/pom.xml**
- parent 同上；artifactId = `gonbay-{project}-api`，name = `gonbay-{project}-api`。
- 依赖：本项目的 common、gonbay-basic-util（版本用 `${project.parent.version}`）、Feign 声明（使用 `spring-cloud-openfeign-core`，或项目已采用的 Feign 依赖）、`spring-boot-starter-validation`。禁止使用项目未采用的 Feign 依赖（如擅自改为 spring-cloud-starter-openfeign 等）。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>net.gonbay.app.{业务域}</groupId>
        <artifactId>gonbay-{project}</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <groupId>net.gonbay.app.{业务域}</groupId>
    <artifactId>gonbay-{project}-api</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>gonbay-{project}-api</name>
    <description>对外 API 模块</description>

    <dependencies>
        <dependency>
            <groupId>net.gonbay.app.{业务域}</groupId>
            <artifactId>gonbay-{project}-common</artifactId>
            <version>${project.parent.version}</version>
        </dependency>
        <dependency>
            <groupId>net.gonbay</groupId>
            <artifactId>gonbay-basic-util</artifactId>
            <version>${project.parent.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-openfeign-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
    </dependencies>
</project>
```

**service：gonbay-{project}-service/pom.xml**
- parent 同上；artifactId = `gonbay-{project}-service`。
- 依赖：本项目的 api（`${project.parent.version}`）、gonbay-repository、gonbay-log、mysql-connector-java(8.0.33)、gonbay-redisson、gonbay-cloud。禁止擅自增删上述已约定的依赖。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>net.gonbay.app.{业务域}</groupId>
        <artifactId>gonbay-{project}</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <groupId>net.gonbay.app.{业务域}</groupId>
    <artifactId>gonbay-{project}-service</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>gonbay-{project}-service</name>
    <description>核心业务模块</description>

    <dependencies>
        <dependency>
            <groupId>net.gonbay.app.{业务域}</groupId>
            <artifactId>gonbay-{project}-api</artifactId>
            <version>${project.parent.version}</version>
        </dependency>
        <dependency>
            <groupId>net.gonbay</groupId>
            <artifactId>gonbay-repository</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>net.gonbay</groupId>
            <artifactId>gonbay-log</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.33</version>
        </dependency>
        <dependency>
            <groupId>net.gonbay</groupId>
            <artifactId>gonbay-redisson</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>net.gonbay</groupId>
            <artifactId>gonbay-cloud</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
</project>
```

**server：gonbay-{project}-server/pom.xml**
- 职责：仅依赖 service、编写并运行对 service/api 的单元测试；可不包含可运行 Spring Boot 应用的 main 或打包配置。
- parent 同上；artifactId = `gonbay-{project}-server`。
- 依赖：仅允许 `gonbay-{project}-service`，版本使用 `${project.parent.version}`。
- ❌ 禁止在未明确要求的前提下添加：spring-boot-maven-plugin、mainClass、spring-boot-starter-webflux、spring-cloud-starter-loadbalancer、finalName 等。若项目确需可运行 Server 应用，须按项目已有约定添加 build/依赖，不得套用其它项目的模板。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>net.gonbay.app.{业务域}</groupId>
        <artifactId>gonbay-{project}</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <groupId>net.gonbay.app.{业务域}</groupId>
    <artifactId>gonbay-{project}-server</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>gonbay-{project}-server</name>
    <description>服务端/测试模块</description>

    <dependencies>
        <dependency>
            <groupId>net.gonbay.app.{业务域}</groupId>
            <artifactId>gonbay-{project}-service</artifactId>
            <version>${project.parent.version}</version>
        </dependency>
    </dependencies>
</project>
```

### 2.3.1 常见错误与禁止行为（防止乱生成）

- ❌ 禁止使用 `gonbay-platform-*` 作为 artifactId 或 parent 的 artifactId。
- ❌ 禁止子模块 parent 的 groupId/artifactId 与根 pom 不一致。
- ❌ 禁止 common 的 artifactId 写成 `gonbay-platform-{project}-common` 或任何非 `gonbay-{project}-common` 的形式。
- ❌ 禁止 api 的 name 写成 `gonbay-platform-{project}-api`。
- ❌ 禁止在 server 中擅自添加 spring-boot-maven-plugin、mainClass、webflux、loadbalancer 等；仅保留对 service 的依赖，除非项目已约定可运行 Server。
- ❌ 禁止在未查阅本项目已有 pom 的前提下，套用其它项目（如 dictionary）的 groupId、artifactId、依赖或 build 配置。
- ✅ 生成或修改 pom 前，必须先对照本规则与当前项目根目录及各子模块的 pom.xml，保证 parent、groupId、artifactId、modules、依赖与现有约定一致。

**规则**
- ✅ 严格按照 2.3.0 命名与引用约定及各模块 pom 模板初始化或修改 pom。
- ✅ 严格按照模块依赖顺序：common → api → service → server；子模块间版本统一使用 `${project.parent.version}`。
- ✅ 禁止擅自扩展或替换各模块已约定的依赖；新增依赖须与项目既有用法一致。


## 2.4 职责代码的模板规范

> Constant 常量定义代码模板规则示例

```java
package net.gonbay.app.dictionary.constant;

/**
 * 字典常量
 * @author Floatin
 */
public interface DictConstant {
    /**
     * 字典项缓存前缀
     */
    String DICT_ITEM_CACHE_PREFIX = "dict:item:"
    /**
     * 字典项按分类缓存前缀
     */
    String DICT_ITEM_BY_TYPE_CACHE_PREFIX = "dict:item:type:";
    /**
     * 字典项标签最大长度
     */
    int DICT_ITEM_LABEL_MAX_LENGTH = 255;

    /**
     * 扩展值最大长度
     */
    int EXTENSION_VALUE_MAX_LENGTH = 1000;
}
```
**规则**
- ✅必须遵守常量定义模板生成常量
- ✅必须在constant包下创建
- ✅一个项目只生成一个 Constant 常量类，并且命名根绝 {project}命名


> Domain 代码模板规则示例
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
- ✅必须遵守Domain定义模板生成Domain类
- ✅必须在domain包下创建
- ✅一张表结构对应一个 Domain类，并且命名规则是用 "_" 转换横驼峰
- ✅ 默认依赖了 lombok 每个 Domain 都需要 引用 @Data @FieldNameConstants 注解，不需要用 getter setter 方法
- ✅ 指定 @TableName 对应数据库表名
- ✅ 必须继承 BaseModel，BaseModel 定义了公共属性如： id、createTime、deleted、companyId 等属性
- ✅ Domain 定义属性，日期统一使用 LocalDateTime、数据库 json 类型对应 JsonNode、所有的 status type 等语义的字段全部都用 String 代替

> enum 代码模板规则示例
```java
/**
 * 字典等级类型枚举
 * @author Floatin
 */
public enum DictLevelTypeEnum implements MetaEnum<String> {

    /**
     * 系统级
     */
    SYSTEM("系统级"),
    /**
     * 业务级
     */
    BUSINESS("业务级"),
    /**
     * 用户级
     */
    USER("用户级");
    
    private  String label;
    
    DictLevelTypeEnum(String label) {
        this.label = label;
    }
    @Override
    public String getCode() {
        return this.name();
    }

    @Override
    public String getMetaValue() {
        return this.label;
    }
}
```
**规则**
- ✅必须遵enum模板代码生成Enum
- ✅必须在enums包下创建
- ✅必须实现 MetaEnum<String> 实现 getCode 和 getetaValue 方法

> errcode 代码模板规则示例
```java
/**
 * 字典错误码 1000-1100
 * @author Floatin
 */
public enum DictErrorCode implements ErrorCode {
    /**
     * 字典分类不存在
     */
    DICT_TYPE_NOT_FOUND("字典分类不存在", 1000),
    /**
     * 字典分类编码已存在
     */
    DICT_TYPE_CODE_EXISTS("字典分类编码已存在", 1001)

    private int status;
    private String message;

    DictErrorCode(String message, int status) {
        this.status = status;
        this.message = message;
    }

    @Override
    public String getCode() {
        return this.name();
    }

    @Override
    public String getMessage() {
        return message;
    }

    @Override
    public int getStatus() {
        return this.status;
    }
}
```
**规则**
- ✅必须遵守 errcode模板代码生成ErrorCode
- ✅必须在errorcode包下创建
- ✅必须实现 ErrorCode接口，并实现 getCode getMessage getStatus 方法


> api 接口类代码模板规则示例

```java
package net.gonbay.app.dictionary.api;

import jakarta.validation.constraints.NotNull;
import net.gonbay.app.dictionary.dto.BatchDictItemDTO;
import net.gonbay.app.dictionary.dto.DictItemDTO;
import net.gonbay.app.dictionary.vo.DictItemVO;
import net.gonbay.app.dictionary.vo.RouterDictItemVo;
import net.gonbay.app.dictionary.vo.TidyDictItemVO;
import net.gonbay.common.constant.FormVerifyGroup;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * 字典项管理-Api
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
     * 修改字典项
     * @extension function update
     * @extension in DictItemDTO
     * @param dto 字典项信息
     * @apiNote 修改字典项 包含对字典项扩展数据进行维护，所有扩展数据都是非必填的。目的是增加字典的功能性
     */
    @PutMapping(name = "修改字典项", value = "/dict/item")
    void update(@Validated(FormVerifyGroup.Update.class) @RequestBody DictItemDTO dto);


    /**
     * 字典项详情：ID
     * @extension function getById
     * @param id 字典项ID
     * @return 字典项信息
     * @apiNote 字典项详情通过 ID 包含对应的字典扩展数据
     */
    @GetMapping(name = "根据ID获取字典项", value = "/dict/item-by-id")
    DictItemVO get(@RequestParam("id") @NotNull Long id);

   
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
     * 批量删除字典项
     * @extension function batchDelete
     * @param codes 字典项code列表
     * @apiNote 批量删除字典项
     */
    @DeleteMapping(name = "批量删除字典项", value = "/dict/item/batch")
    void batchDelete(@NotNull @RequestBody List<String> codes);



    /**
     * 获取字典项路由信息
     * @extension function getDictItemRouterInfo
     * @extension out RouterDictItemVo
     * @return 字典项路由信息
     * @apiNote 获取字典项路由信息，目前设计思路是帮助前端做字典组件回显渲染时做数据支撑
     */
    @PostMapping(name = "获取字典项路由信息", value = "/dict/item/router/info")
    List<RouterDictItemVo> getDictItemRouterInfo(@NotNull @RequestBody List<String> codes);


    /**
     * 调度任务记录分页查询
     * @param requester
     * @return
     */
    @PostMapping(name = "调度任务记录分页查询-表单条件",value = ScheduleTaskConstat.BASE_URI+"/task/record/pager/form-condition")
    PageResult<ScheduleTaskRecordVO> pagerByFormCondition(@RequestBody PageRequestForm<FormCondition> requester);

}
```
**规则**
- ✅必须遵守 api模板代码生成Api接口类文件
- ✅必须在api包下创建
- ✅必须实现 加入@FeignClient 注解
- ✅注释文档必须足够说明内容，详细 采用 @apiNode 写更详细的注
- ✅严格按照 rest 风格定义接口地址和请求方式
- ✅不允许有事务注解，所有需要一个事务完成的业务逻辑都在 service 中实现
- ✅只做验证，类型转换，或查询相关业务逻辑，所有更复杂的业务逻辑都需要在 service 中实现

> Server 代码模板规则示例

```java
package net.gonbay.app.dictionary.server;

import jakarta.annotation.Resource;
import net.gonbay.app.dictionary.api.DictTypeApi;
import net.gonbay.app.dictionary.domain.DictItem;
import net.gonbay.app.dictionary.domain.DictType;
import net.gonbay.app.dictionary.service.DictItemService;
import net.gonbay.app.dictionary.struct.DictItemConverter;
import net.gonbay.app.dictionary.struct.DictTypeConverter;
import net.gonbay.app.dictionary.dto.DictTypeDTO;
import net.gonbay.app.dictionary.errcode.DictErrorCode;
import net.gonbay.app.dictionary.service.DictTypeService;
import net.gonbay.app.dictionary.vo.DictMetaVO;
import net.gonbay.app.dictionary.vo.DictTypeGroupVO;
import net.gonbay.app.dictionary.vo.DictTypeVO;
import net.gonbay.basic.utils.util.EmptyCheckUtil;
import net.gonbay.common.exception.Try;
import net.gonbay.redisson.annotation.NoResubmit;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RestController;

import java.time.LocalDateTime;
import java.util.*;


/**
 * 字典分类Server
 * @author Floatin
 */
@RestController
public class DictTypeServer implements DictTypeApi {

    @Autowired
    private DictItemService dictItemService;
    @Autowired
    private DictTypeService dictTypeService;
    @Autowired
    private DictTypeConverter dictTypeConverter;
    @Autowired
    private DictItemConverter dictItemConverter;


    @Override
    @NoResubmit
    public void add(DictTypeDTO dto) {
        // 检查编码是否已存在
        dictTypeService.verifyRepeatCode(dto.getCode(), null);
        DictType domain = dictTypeConverter.dtoToDomain(dto);
        domain.setCreateTime(LocalDateTime.now());
        dictTypeService.saveType(domain);
    }

    @Override
    @NoResubmit
    public void update(DictTypeDTO dto) {
        // 检查记录是否存在
        DictType dictType = dictTypeService.getDictTypeByCode(dto.getCode());
        Try.predicate(() -> dictType == null, DictErrorCode.DICT_TYPE_NOT_FOUND, dto.getCode());
        dictTypeConverter.toUpdate(dictType, dto);
        dictTypeService.updateType(dictType);
    }

    @Override
    public DictMetaVO getEnumDictTypes() {
        return DictMetaVO.INSTANCE;
    }

    @Override
    public PageResult<ScheduleTaskRecordVO> pagerByFormCondition(PageRequestForm<FormCondition> requester) {
        return scheduleTaskRecordService.pageByFormCondition(requester,list -> {
            return scheduleTaskRecordConverter.domainToVo(list);
        });
    }

    @Override
    public void verifyRepeat(String code, boolean isTypeCode) {
        if (isTypeCode) {
            dictTypeService.verifyRepeatCode(code, null);
        } else {
            dictItemService.verifyRepatCode(Collections.singleton(code), 1);
        }
    }
}
```

**规则**
- ✅必须遵守Server模板代码生成Server实现类文件
- ✅必须在server包下创建
- ✅必须实现Api接口 及其其中方法
- ✅Server 实现的方法逻辑只有 验证参数、查询、组装、调用Service 业务方法等. 不允许在 Server层写其他更多更重要的业务逻辑代码


> Mapper 代码模板规则示例
```java
package net.gonbay.app.dictionary.mappers;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import net.gonbay.app.dictionary.domain.DictItem;

/**
 * 字典项Mapper
 * @author Floatin
 */
public interface DictItemMapper extends BaseMapper<DictItem> {

     /**
     * 获取待办任务分页数据
     * @param ipage
     * @param wrapper
     * @return
     */
    @Select("""
       select * from (
          SELECT
              r.PARENT_TASK_ID_ as parent_task_id,
              r.NAME_ as task_name,
              r.TASK_DEF_KEY_ as activity_id,
              r.ASSIGNEE_ as assignee,
              r.PRIORITY_ as priority,
              r.CREATE_TIME_ as task_create_time,
              r.SUSPENSION_STATE_ as suspension_state,
              r.FORM_KEY_ as form_key,
              r.CATEGORY_ as category,
              r.EXECUTION_ID_ as execution_id,
              r.ID_ as task_id,
              r.PROC_INST_ID_ as instance_id,
              r.PROC_DEF_ID_ as  definition_id,
              i.biz_data_type,
              i.biz_data_code,
              i.instance_code,
              i.operator_id,
              i.deploy_id,
              i.post_id,
              i.executor_id,
              i.name as instance_name,
              i.create_time as instance_create_time
          from ACT_RU_TASK r
                   inner join work_flow_instance i
                              on r.PROC_INST_ID_=i.instance_id
               where r.SUSPENSION_STATE_=1
         ) a ${ew.customSqlSegment}
    """)
    Page<PendingTaskVO> pageByPendingTask(IPage<PendingTaskVO> ipage, @Param(Constants.WRAPPER) Wrapper<PendingTaskVO> wrapper);

}
```
**规则**
- ✅必须遵守 mapper模板代码生成Mapper接口类文件
- ✅必须在mappers包下创建
- ✅必须继承BaseMapper<T>
- ✅后续 SQL都需要在Mapper（ORM）定义

> Service 代码模板规则示例
```java
package net.gonbay.app.dictionary.service;
import net.gonbay.app.dictionary.domain.DictItem;
import net.gonbay.app.dictionary.dto.DictItemDTO;
import net.gonbay.app.dictionary.vo.DictItemVO;
import net.gonbay.app.dictionary.vo.TidyDictItemVO;
import net.gonbay.common.domain.PageRequestForm;
import net.gonbay.common.enums.VisibleType;
import net.gonbay.repository.core.BasicService;
import java.util.List;
import java.util.Set;
/**
 * 字典项Service
 * @author Floatin
 */
public interface DictItemService extends BasicService<DictItem> {

}
```
**规则**
- ✅必须遵守 Service模板代码生成Service接口类文件
- ✅必须在service包下创建
- ✅必须继承 BasicService<T> 接口
- ✅ 所有 Server层所调用的 service 都在对应的 service 中声明

> ServiceImpl 代码模板规则示例

```java
package net.gonbay.app.dictionary.service.impl;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.toolkit.Wrappers;
import net.gonbay.app.dictionary.domain.DictItem;
import net.gonbay.app.dictionary.domain.DictItemExtension;
import net.gonbay.app.dictionary.dto.DictItemDTO;
import net.gonbay.app.dictionary.errcode.DictErrorCode;
import net.gonbay.app.dictionary.mappers.DictItemMapper;
import net.gonbay.app.dictionary.service.DictItemExtensionService;
import net.gonbay.app.dictionary.service.DictItemService;
import net.gonbay.app.dictionary.struct.DictItemConverter;
import net.gonbay.app.dictionary.vo.DictItemVO;
import net.gonbay.app.dictionary.vo.TidyDictItemVO;
import net.gonbay.basic.utils.util.EmptyCheckUtil;
import net.gonbay.common.domain.PageRequestForm;
import net.gonbay.common.enums.VisibleType;
import net.gonbay.common.exception.Try;
import net.gonbay.repository.core.BasicServiceImpl;
import net.gonbay.repository.core.Condition;
import net.gonbay.repository.core.SnakeFieldQueryWrapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.util.StringUtils;
import java.util.List;
import java.util.Optional;
import java.util.Set;
import static net.gonbay.repository.common.OperatorSymbol.*;

/**
 * 字典项ServiceImpl
 * @author Floatin
 */
@Service
public class DictItemServiceImpl extends BasicServiceImpl<DictItemMapper, DictItem> implements DictItemService {

}
```
**规则**
- ✅必须遵守 ServiceImpl模板代码生成ServiceImpl实现类文件
- ✅必须在service.impl包下创建
- ✅必须实现对应的 Service的所有接口，并继承 BasicServiceImpl<Mapper,Domain>
- ✅ 所有Service声明的接口全部都在 ServiceImpl实现