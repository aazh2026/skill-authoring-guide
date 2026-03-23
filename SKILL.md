---
name: skill-authoring-guide
description: Agent Skill Engineering Handbook - 从 Prompt 片段到工程化资产的完整方法论。涵盖 Skill 设计、Skill 系统、Skill 评估、Skill 治理四大体系。

input_schema:
  topic:
    type: string
    required: true
    enum: ["design", "trigger", "quality", "system", "governance", "template"]
    description: 要查询的主题
  skill_level:
    type: string
    required: false
    default: "intermediate"
    enum: ["beginner", "intermediate", "advanced"]
    description: 技能水平

output_schema:
  content:
    type: string
    description: 主题内容
  templates:
    type: array
    items:
      type: object
      properties:
        name: { type: string }
        template: { type: string }
  rubric:
    type: object
    description: 质量评估标准
  case_studies:
    type: array
    items: { type: string }

verification:
  - check: "output.content is not empty"
    severity: error
  - check: "topic in ['design', 'trigger', 'quality', 'system', 'governance', 'template']"
    severity: error

---

# Agent Skill Engineering Handbook

从 Prompt 片段到工程化资产的完整方法论。

## 核心理念

> **Skill ≠ Prompt，而是一个"可调用的结构化操作手册"**

Skill = 一组指令 + 资源 + 可按需加载的能力单元

## 四个体系

1. **Skill 设计** - 如何写好单个 Skill
2. **Skill 触发** - 如何让 Skill 被正确调用
3. **Skill 质量** - 如何评估 Skill 好坏
4. **Skill 系统** - 如何构建 Skill 生态
5. **Skill 治理** - 如何安全可控

---

# 1. Skill 设计体系

## 1.1 标准化结构（进化版）

```yaml
---
name: <skill-name>                    # 小写字母+下划线
description: |
  <场景化描述，说明何时调用>
  
  **触发场景：**
  - <场景1>
  - <场景2>
  
  **关键词：** <keyword1>, <keyword2>, <keyword3>

triggers:                             # NEW: 显式触发机制
  - type: keyword
    patterns: ["keyword1", "keyword2"]
  - type: intent
    patterns: ["intent1", "intent2"]
  - type: context
    condition: "file_type == 'javascript'"

input_schema:
  param1:
    type: string
    required: true
    description: <参数描述>
    validation: <验证规则>

output_schema:
  result:
    type: object
    description: <输出描述>

dependencies:                         # NEW: 依赖的skills
  - skill: <skill-name>
    version: ">=1.0.0"

verification:                         # NEW: 自动化验证
  - check: "<验证表达式>"
    severity: error|warning

---

# Skill: <Name>

## Intent
<这个skill解决什么问题>

## When to Use
<明确的触发条件>

## Capabilities
- [能力1]
- [能力2]
- [能力3]

## Constraints
- [边界1]
- [边界2]
- [边界3]

## Procedure
1. [步骤1]
2. [步骤2]
3. [步骤3]

## Examples
<完整的输入输出示例>

## Anti-patterns
<常见错误用法>
```

---

## 1.2 Trigger Engineering（核心章节）

### 触发来源

| 类型 | 机制 | 示例 |
|------|------|------|
| **Keyword** | 关键词匹配 | "test", "coverage" |
| **Intent** | 意图识别 | "写测试", "生成测试用例" |
| **Context** | 上下文条件 | 当前文件类型、项目结构 |
| **Semantic** | 语义相似度 | 向量匹配 |

### 触发设计原则

**1. 覆盖多表达**
```yaml
triggers:
  - type: keyword
    patterns:
      - "test"
      - "add test"
      - "coverage"
      - "write test"
      - "单元测试"
```

**2. 避免歧义**
```yaml
# ❌ 太泛，容易误触发
description: "处理代码"

# ✅ 精确
description: "为 JavaScript 函数生成 Jest 单元测试"
```

**3. 控制长度**
```yaml
# description <= 1024 tokens（上下文限制）
# 建议 200-500 字
```

### 触发优化示例

**❌ 差：**
```yaml
description: "generate test"
```

**✅ 好：**
```yaml
description: |
  Generate Jest unit tests for JavaScript functions.
  
  **Triggers:**
  - User says: "test", "add test", "coverage"
  - Current file: *.test.js, *.spec.js
  - Intent: testing, quality assurance
```

---

# 2. Skill 质量体系

## 2.1 Skill Quality Rubric

### 5 维度评估

| 维度 | 权重 | 评估标准 |
|------|------|----------|
| **Trigger Precision** | 0.25 | 触发精度：正确触发率 > 90%，误触发率 < 5% |
| **Actionability** | 0.25 | 可执行性：步骤明确，输出可直接使用 |
| **Reusability** | 0.20 | 复用性：脱离特定上下文仍可用 |
| **Composability** | 0.20 | 可组合性：可被其他 skill 调用 |
| **Determinism** | 0.10 | 确定性：相同输入输出稳定 |

### 评分标准

```
Score = 0.25*Trigger + 0.25*Action + 0.20*Reuse + 0.20*Compose + 0.10*Determinism
```

| 分数 | 等级 | 说明 |
|------|------|------|
| 90-100 | S | 生产级，可规模化使用 |
| 80-89 | A | 可用，小优化后上线 |
| 70-79 | B | 需要针对性改进 |
| <70 | C | 不建议使用 |

### 评估检查清单

**Trigger Precision**
- [ ] 关键词覆盖主要表达方式
- [ ] 意图描述清晰无歧义
- [ ] 有明确的反例说明

**Actionability**
- [ ] 步骤编号明确
- [ ] 每个步骤有具体动作
- [ ] 输出格式标准化

**Reusability**
- [ ] 不依赖特定项目结构
- [ ] 参数可配置
- [ ] 有默认值

**Composability**
- [ ] 输入输出标准化
- [ ] 有清晰的接口定义
- [ ] 可被其他 skill 调用

**Determinism**
- [ ] 相同输入输出一致
- [ ] 无随机行为
- [ ] 错误处理稳定

---

## 2.2 Skill 评估方法

### 自动化测试

```yaml
# test_cases.yaml
test_cases:
  - id: TC001
    input:
      query: "帮我写测试"
    expected_trigger: true
    expected_score: ">=0.9"
    
  - id: TC002
    input:
      query: "今天天气如何"
    expected_trigger: false
```

### 人工评估

```yaml
# evaluation.yaml
evaluators:
  - role: "资深开发者"
    criteria:
      - "步骤是否可执行"
      - "输出是否实用"
  - role: "产品经理"
    criteria:
      - "是否解决实际问题"
      - "是否易于理解"
```

---

# 3. Skill 系统体系

## 3.1 Skill Graph（技能图谱）

### 从单 Skill 到 Skill Pipeline

```
需求分析 → 设计 → 编码 → 测试 → 发布
    ↓        ↓      ↓      ↓      ↓
  prd    design   code   test  deploy
  skill  skill   skill  skill  skill
```

### Skill 类型定义

| 类型 | 说明 | 示例 |
|------|------|------|
| **Atomic Skill** | 原子技能，不可再分 | 写单元测试 |
| **Composite Skill** | 组合技能，调用其他 skills | 完整开发流程 |
| **Meta Skill** | 元技能，选择和编排 skills | 根据需求选择技术栈 |

### Skill Pipeline 示例

```yaml
# pipeline.yaml
name: full_development_workflow
description: 完整开发流程

stages:
  - name: requirements
    skill: prd_analyzer
    condition: "input.type == 'feature'"
    
  - name: design
    skill: architecture_designer
    depends_on: [requirements]
    
  - name: implementation
    skill: code_generator
    depends_on: [design]
    
  - name: testing
    skill: test_generator
    depends_on: [implementation]
    
  - name: review
    skill: code_reviewer
    depends_on: [testing]

error_handling:
  - stage: testing
    on_failure: retry_with_fix
    max_retries: 3
```

### Skill 依赖管理

```yaml
dependencies:
  - skill: code_linter
    version: ">=1.0.0"
    required: true
    
  - skill: test_runner
    version: ">=2.0.0"
    required: false  # 可选依赖
    condition: "config.include_tests == true"
```

---

## 3.2 Skill 生态系统

### Skill 分层架构

```
┌─────────────────────────────────────┐
│  Layer 3: Business Skills           │
│  (PRD生成、用户故事、竞品分析)         │
├─────────────────────────────────────┤
│  Layer 2: Domain Skills             │
│  (API设计、数据库设计、UI组件)         │
├─────────────────────────────────────┤
│  Layer 1: Technical Skills          │
│  (代码生成、测试生成、重构)            │
├─────────────────────────────────────┤
│  Layer 0: Foundation Skills         │
│  (文件操作、Git操作、Shell命令)        │
└─────────────────────────────────────┘
```

---

# 4. Skill 治理体系

## 4.1 Skill 安全

### 风险评估

| 风险 | 说明 | 防护措施 |
|------|------|----------|
| **代码注入** | Skill生成恶意代码 | 沙箱执行、人工Review |
| **信息泄露** | Skill读取敏感信息 | 权限控制、数据脱敏 |
| **外部调用** | Skill调用危险API | 白名单机制、审计日志 |
| **提示注入** | 用户输入污染Skill | 输入验证、参数化 |

### 安全策略

```yaml
security:
  sandbox:
    enabled: true
    network: false  # 禁止网络访问
    filesystem: readonly  # 只读文件系统
    
  permissions:
    - resource: filesystem
      access: ["read", "write"]
      paths: ["/workspace"]
    - resource: network
      access: []  # 禁止
      
  audit:
    log_level: "info"
    retention: "30d"
```

## 4.2 Skill 版本管理

```yaml
versioning:
  strategy: "semver"
  
  compatibility:
    - version: "1.x"
      api: "stable"
    - version: "2.x"
      api: "breaking"
      migration_guide: "/docs/migration/v2.md"
```

## 4.3 Skill 注册与发现

```yaml
registry:
  name: "organization-skill-registry"
  
  metadata:
    - name
    - description
    - author
    - version
    - tags
    - quality_score
    - usage_count
    
  search:
    - full_text: true
    - semantic: true
    - filters: [tag, author, quality]
```

---

# 5. 实战案例

## 5.1 Case Study: PRD 生成流水线

### 场景
从需求到技术方案的完整流程。

### Skill Pipeline

```
需求描述 → PRD生成 → 技术方案 → 任务拆解
    ↓          ↓          ↓          ↓
  input    prd_writer  tech_design  task_breakdown
```

### Skill 定义

**Skill 1: prd_writer**
```yaml
name: prd_writer
description: 根据需求描述生成产品需求文档

triggers:
  - type: intent
    patterns: ["写PRD", "产品需求", "需求文档"]

input_schema:
  requirement: { type: string, required: true }
  product_type: { type: string, enum: ["web", "mobile", "api"] }

output_schema:
  prd:
    title: string
    background: string
    user_stories: array
    acceptance_criteria: array
    non_functional: array
```

**Skill 2: tech_design**
```yaml
name: tech_design
description: 根据PRD生成技术方案
dependencies:
  - skill: prd_writer
    required: true

input_schema:
  prd: { type: object, required: true }
  tech_stack: { type: string, enum: ["react", "vue", "angular"] }

output_schema:
  design:
    architecture: string
    database_schema: object
    api_endpoints: array
    components: array
```

**Skill 3: task_breakdown**
```yaml
name: task_breakdown
description: 将技术方案拆解为开发任务
dependencies:
  - skill: tech_design
    required: true

output_schema:
  tasks:
    - id: string
      title: string
      description: string
      estimate: string
      dependencies: array
```

### 使用示例

```
User: 帮我做一个电商平台的购物车功能

Agent:
  1. 调用 prd_writer → 生成 PRD
  2. 调用 tech_design → 生成技术方案
  3. 调用 task_breakdown → 生成任务列表

Output:
  - PRD: 包含用户故事、验收标准
  - 技术方案: 架构设计、API设计
  - 任务列表: 10个任务，带估时和依赖
```

---

## 5.2 Case Study: 代码重构流水线

### Skill Pipeline

```
代码分析 → 重构建议 → 代码修改 → 测试验证
    ↓           ↓           ↓           ↓
  analyzer   suggestor   modifier   validator
```

---

# 6. 工具与资源

## 6.1 Skill 开发工具

```bash
# CLI 工具
skill-cli create <name>        # 创建新 skill
skill-cli validate <file>      # 验证 skill 格式
skill-cli test <file>          # 运行测试
skill-cli publish <file>       # 发布到 registry
skill-cli evaluate <file>      # 质量评估
```

## 6.2 Skill 模板库

| 模板 | 用途 | 链接 |
|------|------|------|
| atomic-skill | 原子技能 | /templates/atomic.md |
| composite-skill | 组合技能 | /templates/composite.md |
| meta-skill | 元技能 | /templates/meta.md |
| pipeline | 流水线 | /templates/pipeline.md |

## 6.3 参考资源

- [GitHub Copilot Skills](https://github.com/github/copilot)
- [Claude Code Best Practices](https://docs.anthropic.com)
- [LangChain Tools](https://python.langchain.com)

---

# 7. 总结

## 核心理念回顾

1. **Skill 是工程资产** - 不是一次性 Prompt
2. **Trigger 是关键** - 写得再好，触发不了就没用
3. **质量可评估** - 5维度评估体系
4. **系统是目标** - 从单 Skill 到 Skill Graph
5. **治理不可少** - 安全、版本、注册

## 演进路径

```
Level 1: 写好单个 Skill
    ↓
Level 2: 设计触发机制
    ↓
Level 3: 建立质量评估
    ↓
Level 4: 构建 Skill 系统
    ↓
Level 5: 完善治理体系
```

---

# 附录：快速参考

## 检查清单（发布前必查）

- [ ] 名称：小写字母+下划线
- [ ] 描述：场景化，有触发条件和关键词
- [ ] Triggers：关键词/意图/上下文至少一种
- [ ] Capabilities：3-5个具体能力点
- [ ] Constraints：至少2条边界约束
- [ ] Input Schema：类型、必填、验证完整
- [ ] Output Schema：格式明确，字段完整
- [ ] Dependencies：如有依赖，版本明确
- [ ] Verification：可自动验证的规则
- [ ] Procedure：步骤编号，逻辑清晰
- [ ] Examples：2-3个，覆盖不同场景
- [ ] Anti-patterns：常见错误用法
- [ ] Quality Score：自评 >= 80分

---

> **最终目标：让 Skill Engineering 成为 AI 时代的软件工程方法论**
