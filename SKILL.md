---
name: skill-authoring-guide
description: 编写高质量 Skill 的完整指南。提供标准化结构、场景化描述、边界约束、SOP 流程和示例模板，帮助开发者创建 LLM 友好的 Agent Skill。

input_schema:
  skill_type:
    type: string
    required: true
    enum: ["extraction", "transformation", "analysis", "decision", "tool", "composite"]
    description: Skill 类型
  complexity:
    type: string
    required: false
    default: "medium"
    enum: ["simple", "medium", "complex"]
    description: 复杂程度
  target_model:
    type: string
    required: false
    default: "claude"
    enum: ["claude", "gpt", "gemini", "general"]
    description: 目标 LLM 模型

output_schema:
  template:
    type: string
    description: 完整的 skill.md 模板
  checklist:
    type: array
    description: 编写检查清单
    items: { type: string }
  examples:
    type: array
    description: 参考示例
    items:
      type: object
      properties:
        scenario: { type: string }
        snippet: { type: string }
  common_pitfalls:
    type: array
    description: 常见陷阱
    items: { type: string }

verification:
  - check: "output.template contains '## Description'"
    severity: error
  - check: "output.template contains '## Input/Output Schema' or '## Input Schema'"
    severity: error
  - check: "len(output.checklist) >= 8"
    severity: error
  - check: "output.template contains trigger keywords or 'when to use'"
    severity: warning
  - check: "output.template contains Constraints or Guardrails"
    severity: warning

---

# Skill 编写指南

编写高质量 skill.md 的完整指南：**消除歧义，明确边界**。

## 1. 标准化结构布局

一个 LLM 友好的 skill.md 应该包含以下模块：

```markdown
# Skill: [Name]

## Description
[场景化描述，说明何时调用]

## Capabilities
- [功能点1]
- [功能点2]

## Constraints/Rules
- [不能做什么1]
- [不能做什么2]

## Input/Output Schema
[参数定义]

## Workflow/Steps
[执行逻辑]

## Examples
[2-3个示例]
```

---

## 2. 编写"场景化"描述 (Trigger-focused)

**❌ 错误示例：**
> 查询天气

**✅ 正确示例：**
> 当用户询问特定城市的实时气温、湿度或未来 7 天的天气预报时，调用此技能。
> 关键词：天气、温度、降雨、 forecast、明天、下周

**要点：**
- 写触发场景，不是功能列表
- 埋入用户可能说的关键词
- 明确"什么时候用我"

---

## 3. 输入输出的"极简主义"与"强类型"

**参数命名：**
- ✅ `start_date` 
- ❌ `d1`

**强制格式要求：**
```markdown
## Input Schema

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| query_date | string | 是 | 查询日期 |

**格式要求：**
- 必须转换为 YYYY-MM-DD 格式
- 如果用户说"明天"，根据当前时间计算具体日期
```

**输出预期：**
```markdown
## Output Schema

- 格式：JSON
- 字段：temperature (number), humidity (number), forecast (array)
- 单位：摄氏度、百分比
```

---

## 4. 设定严苛的边界约束 (Guardrails)

防止 Agent "拿锤子看什么都是钉子"。

**否定约束：**
> 如果用户询问政治敏感话题，严禁调用此搜索技能。

**范围约束：**
> 仅支持查询 2000 年以后的论文数据。

**权限约束：**
> 在没有获得用户明确的订单号之前，不得执行取消订单操作。

**格式：**
```markdown
## Constraints

- 严禁执行 DELETE, DROP, UPDATE 等写操作
- 单次查询返回行数不得超过 100 行
- 如果用户询问薪资数据，回复"无权访问"
```

---

## 5. Workflow：给 Agent 的"SOP"

复杂技能需要明确的执行步骤。

```markdown
## Workflow

1. **解析输入**
   - 提取关键词
   - 验证参数格式

2. **预处理**
   - 转换日期格式
   - 补全默认参数

3. **执行核心逻辑**
   - 调用 API/工具
   - 处理异常情况

4. **后处理**
   - 格式化输出
   - 添加元数据

5. **验证结果**
   - 检查输出完整性
   - 确保符合 schema
```

**要点：**
- 步骤编号明确
- 每个步骤有具体动作
- 包含异常处理分支

---

## 6. 示例 (Few-shot) 的质量

示例是提升鲁棒性最快的方法。

**推荐格式：**
```markdown
## Examples

### 示例1：基本查询

**User:** 帮我看看北京明天冷不冷？

**Agent Thought:** 用户询问天气趋势，需要调用 weather 技能。

**Tool Call:**
```json
{
  "tool": "get_weather",
  "parameters": {
    "city": "Beijing",
    "date": "2024-05-21"
  }
}
```

**Result:**
```json
{
  "temperature": "15°C-22°C",
  "condition": "多云",
  "recommendation": "适合穿薄外套"
}
```

**Agent Response:** 北京明天温度 15-22°C，多云，建议穿薄外套。
```

**要点：**
- 展示完整链路：User → Thought → Tool → Result → Response
- 包含 2-3 个不同场景
- 覆盖正常和边界情况

---

## 7. 实用模板参考

### 模板 A：数据查询类

```markdown
---
name: sql_data_analyst
description: 从公司内部销售数据库提取数据并生成分析报告。当用户询问销售额、利润率或客户分布时使用。

input_schema:
  query_type:
    type: string
    required: true
    enum: ["sales", "profit", "customer_distribution"]
  time_range:
    type: string
    required: true
    description: "如：2024-Q1, last_month, ytd"
  filters:
    type: object
    required: false
    properties:
      region: { type: string }
      product_line: { type: string }

output_schema:
  data:
    type: array
    items: { type: object }
  summary:
    type: string
  chart_recommendation:
    type: string

verification:
  - check: "output.data.length <= 100"
    severity: error

---

# Skill: SQL_Data_Analyst

## Description
用于从公司内部销售数据库中提取数据并生成分析报告。

**触发场景：**
- 用户询问销售额、订单量、客单价
- 需要查看同比/环比趋势
- 分析客户地域分布

**关键词：** 销售额、利润、客户、趋势、对比、排名

## Capabilities
- 自动编写符合 PostgreSQL 语法的 SQL 语句
- 对查询结果进行趋势分析和同比/环比计算
- 推荐合适的图表类型

## Constraints
- 严禁执行 DELETE, DROP, UPDATE 等写操作
- 单次查询返回行数不得超过 100 行
- 如果用户询问薪资相关数据，回复"无权访问"
- 不处理实时数据，仅 T+1 数据

## Input/Output Schema
[见上文]

## Workflow
1. 理解用户意图并提取时间维度
2. 构建 SQL 并通过 db_executor 工具执行
3. 对返回的数据进行多维解读
4. 生成自然语言摘要

## Examples
[2-3个完整示例]
```

### 模板 B：内容生成类

```markdown
---
name: blog_post_generator
description: 根据主题生成技术博客文章。当用户需要撰写技术文章、教程或文档时使用。

input_schema:
  topic:
    type: string
    required: true
    description: 文章主题
  audience:
    type: string
    required: false
    default: "intermediate"
    enum: ["beginner", "intermediate", "advanced"]
  word_count:
    type: integer
    required: false
    default: 1500
    minimum: 500
    maximum: 5000

output_schema:
  title:
    type: string
  content:
    type: string
  outline:
    type: array
  code_examples:
    type: array

verification:
  - check: "len(output.content) >= input.word_count * 0.9"
    severity: warning

---

# Skill: Blog_Post_Generator

## Description
根据主题生成技术博客文章。

**触发场景：**
- 用户说"帮我写一篇关于XXX的文章"
- 需要技术教程或入门指南
- 为产品撰写说明文档

**关键词：** 写文章、教程、指南、文档、博客

## Capabilities
- 生成结构化的技术文章
- 自动包含代码示例
- 根据受众调整技术深度

## Constraints
- 不包含敏感或版权争议内容
- 代码示例必须可运行
- 技术概念需要解释清楚
- 不生成过长的文章（<5000字）

## Workflow
1. 分析 topic，确定核心技术概念
2. 生成文章大纲
3. 撰写内容，插入代码示例
4. 根据 audience 调整技术深度
5. 输出完整文章

## Examples
[示例...]
```

---

## 8. 检查清单 (Checklist)

发布前检查：

- [ ] **名称**：简洁、动词开头、无歧义
- [ ] **描述**：场景化，包含触发条件和关键词
- [ ] **Capabilities**：3-5 个具体功能点
- [ ] **Constraints**：至少 2 条硬性约束
- [ ] **Input Schema**：所有参数有类型、必填、描述
- [ ] **Output Schema**：格式明确，字段完整
- [ ] **Workflow**：步骤清晰，逻辑完整
- [ ] **Examples**：2-3 个，覆盖不同场景
- [ ] **Verification**：可自动验证的规则

---

## 9. 常见陷阱

**❌ 描述太抽象**
> "处理数据"

**✅ 描述具体**
> "从 CSV 文件提取销售数据并计算月度汇总"

---

**❌ 没有边界**
> 技能可以做任何事

**✅ 明确边界**
> "仅处理销售数据，不处理用户信息"

---

**❌ 示例太少**
> 只有 1 个简单示例

**✅ 示例覆盖全面**
> 正常输入、边界输入、错误处理

---

**❌ 参数不严谨**
> `date: 日期`

**✅ 参数精确**
> `date: 格式 YYYY-MM-DD，如 2024-05-21`

---

## 10. 进阶技巧：自迭代

**当 Agent 在某任务上反复出错时，不要直接改代码，先改 skill.md。**

在 Constraints 里加一条针对该错误的"补丁"：

```markdown
## Constraints

- ...原有约束...
- ⚠️ **重要**：用户说"便宜"时，指的是价格低，不是质量差
```

通常能解决 80% 的意图识别问题。

---

## 总结

> **高质量的 skill.md = 清晰的边界 + 具体的示例 + 可验证的输出**

写好 skill.md，Agent 才能发挥最大价值。
