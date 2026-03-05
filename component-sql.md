# SQL 建表通用规则（Task Engine）

> 强制性约束 · SQL DDL 规范 · 可复用组件

---

## 1. 规则目的

本规定定义了 DDL sql 生成时候的通用规范，必须严格遵守本规则
- 统一表命名规则、字段命名与公共字段
- 统一主键、索引、字符集、注释规范
- 避免同类语义出现多种命名导致的维护成本

---

## 2. 通用模式归纳

### 2.1 表命名模式
- 以下划线“_”为分隔符来表示多单词语义
- 表名都需要以前缀开始  例：tm_*  tm 就是表的前缀

### 3.2 列命名模式

- 一律 `snake_case`
- 主键列固定 `id`
- 关联列统一 `*_id`（如 `task_id`、`instance_id`、`module_id`）
- 时间列统一 `*_time`
- 状态/类型语义列统一字符串化：`status`、`status_tag`、`type`、`biz_type`
- JSON 扩展列使用语义名：`biz_entity`、`rule_config`、`task_graph`、`risk_factors`
### 3.3 通用列模式

所有业务表应统一包含以下“基础四列”：

- `id bigint NOT NULL COMMENT '唯一标识'`
- `deleted tinyint DEFAULT '0' COMMENT '删除标识'`
- `company_id bigint DEFAULT NULL COMMENT '公司ID'`
- `create_time datetime DEFAULT NULL COMMENT '创建时间'`

高频可选通用列（按场景启用）：

- 操作审计：`operator_id`
- 文本描述：`remark`
- 组织归属：`dept_id`、`post_id`

### 3.4 约束与索引模式

- 主键模式：`PRIMARY KEY (id)`（13/13）
- 当前脚本存在 `UNIQUE KEY id (id)` 与主键重复（13/13）
- 字符集统一：`utf8mb4` + `utf8mb4_0900_ai_ci`
- 存储引擎统一：`InnoDB`
- 外键约束：样本中未使用 `FOREIGN KEY`，由应用层维护关联完整性
- 二级索引：仅在高频扫描/匹配场景增加（如时间检查点表）

---

## 4. 可复用约束规则

### 4.1 必须遵守

- ✅ 新表命名必须使用 `tm_` 或 `tm_hi_` 前缀
- ✅ 列命名必须使用 `snake_case`
- ✅ 每张表必须包含基础四列：`id/deleted/company_id/create_time`
- ✅ 所有主键必须使用 `bigint`，由应用层生成 ID（禁止自增）
- ✅ 每个字段必须写中文注释，表必须写中文注释
- ✅ 默认字符集/排序规则必须与现有保持一致：`utf8mb4`、`utf8mb4_0900_ai_ci`
- ✅ 逻辑删除字段统一为 `deleted`，默认值 `0`

### 4.2 建议遵守

- 建议新增表补充通用查询索引：`(company_id, deleted)`、`(create_time)`
- 建议状态/类型统一 `varchar(32)`，单位类统一 `varchar(16)`
- 建议跨表同义字段保持同名（例如统一使用 `next_task_id`，避免 `cur_task_id/next_task_id` 并存）
- 建议 JSON 字段用于扩展结构化数据，避免过度拆分导致演进成本高

### 4.3 禁止行为

- ❌ 禁止出现驼峰列名
- ❌ 禁止使用自增主键
- ❌ 禁止缺失基础四列
- ❌ 禁止同一语义使用多种拼写（如 `assign`/`assgine`）
- ❌ 禁止无注释或注释与语义不一致

---

## 5. 标准建表模板（可直接复用）

```sql
CREATE TABLE `tm_xxx` (
  `id` bigint NOT NULL COMMENT '唯一标识',
  `deleted` tinyint NOT NULL DEFAULT '0' COMMENT '删除标识',
  `company_id` bigint DEFAULT NULL COMMENT '公司ID',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `operator_id` bigint DEFAULT NULL COMMENT '操作人ID',
  `name` varchar(64) DEFAULT NULL COMMENT '名称',
  `biz_type` varchar(32) DEFAULT NULL COMMENT '业务类型',
  `status` varchar(32) DEFAULT NULL COMMENT '状态',
  `remark` text COMMENT '描述',
  PRIMARY KEY (`id`),
  KEY `idx_xxx_company_deleted` (`company_id`, `deleted`),
  KEY `idx_xxx_create_time` (`create_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='xxx业务表';
```

历史表模板：

```sql
CREATE TABLE `tm_hi_xxx` (
  `id` bigint NOT NULL COMMENT '唯一标识',
  `deleted` tinyint NOT NULL DEFAULT '0' COMMENT '删除标识',
  `company_id` bigint DEFAULT NULL COMMENT '公司ID',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `operator_id` bigint DEFAULT NULL COMMENT '操作人ID',
  `rel_id` bigint DEFAULT NULL COMMENT '关联运行态记录ID',
  `remark` varchar(255) DEFAULT NULL COMMENT '描述',
  PRIMARY KEY (`id`),
  KEY `idx_hi_xxx_rel_id` (`rel_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='xxx历史表';
```

---

## 6. 风险与注意事项

- 当前样本中 `PRIMARY KEY(id)` 与 `UNIQUE KEY id(id)` 重复，属于冗余约束；新表建议仅保留主键。
- 当前样本未使用外键，数据一致性依赖应用层事务与校验；跨表写入必须显式做一致性控制。
- 样本存在命名不一致：
  - `cur_task_id` 与 `next_task_id` 语义重叠
  - `assgine_id` 疑似拼写问题（建议统一为 `assignee_id`）
- 若继续沿用上述不一致命名，会提升 API/Domain/Mapper 的转换与维护成本。

---

## 7. 落地检查清单

- [ ] 表名前缀是否符合 `tm_` / `tm_hi_`
- [ ] 是否包含基础四列
- [ ] 是否全部为 `snake_case`
- [ ] 主键是否为 `bigint` 且非自增
- [ ] 字符集/排序规则/引擎是否统一
- [ ] 字段与表注释是否完整且语义一致
- [ ] 是否存在同义字段多命名或拼写不一致
