<div align="center">

![Outsource Logo](docs/outsource-logo.png)

# 外包.skill

### 把机械体力活外包给低配 session，让主线专注判断和写代码

[![Outsource](https://img.shields.io/badge/Outsource-skill-6B7280.svg?labelColor=111827)](SKILL.md)
[![Agent](https://img.shields.io/badge/Agent-skill-6B7280.svg?labelColor=0099FF)](#快速开始)
[![Templates](https://img.shields.io/badge/Templates-6%20scenarios-6B7280.svg?labelColor=7C3AED)](references/scenarios)
[![License](https://img.shields.io/badge/License-MIT-6B7280.svg?labelColor=16A34A)](LICENSE)

**Delegate the noise. Keep the judgment.**

[English](README_en.md) · 简体中文

[快速开始](#快速开始) · [核心理念](#核心理念) · [工作流程](#工作流程) · [场景模板](#场景模板) · [文件协议](#文件协议) · [产物示例](#产物示例)

</div>

---

## 简介

Outsource 是一个面向 Agent 的工作流 skill。

它不是一个构建工具，也不是一个自动化脚本集合，而是一套「机械任务委派协议」：当主线 session 遇到编译构建、跑测试、查版本、解析 FQN、grep 日志、探测环境、采集数据等容易刷屏的体力活时，不在主线上直接执行，而是生成一份自包含的委派提示词，交给低配模型 session 或 subagent 去跑。

主线只消费低配 session 回传的结构化报告，把上下文留给真正需要判断的事情：架构、决策、代码修改和结果取舍。

适合这些场景：

- 构建日志很长，不想污染主线上下文
- 要查依赖版本、类名、方法签名，但不想让主线消耗大量 token
- 要 grep 日志、跑单测、探服务可达性，只需要事实和证据
- 想让便宜模型处理重复查询，让强模型保留全局判断能力

---

## 核心理念

### 噪音不要进主线

一次失败的构建、一个大日志 grep、一个 jar 包符号扫描，可能产生几百到几千行输出。Outsource 的目标是让这些原始噪音停留在低配 session 里，主线只接收几十行结构化报告。

这节省的不只是 token，更是主线的上下文窗口。

### 强模型做判断，弱模型跑体力活

写代码、定架构、判断风险，需要强模型和完整上下文。跑命令、查版本、贴错误原文，只需要低配模型听话、只读、如实回报。

Outsource 把这两类工作拆开：主线定义任务，低配 session 执行任务，主线再基于证据继续推进。

### 提示词即协议

每次委派都必须是一份五段式提示词：

| 段落 | 作用 |
|---|---|
| 角色 | 定位目标 session 是哪个领域的专家，并交代本次任务 |
| 背景 | 给出路径、技术栈、版本、意图，避免目标 session 盲跑 |
| 步骤 | 写清可执行命令、目的和必要替代项 |
| 铁律 | 约束只读、逐字原文、查不到说查不到、不确定要标注 |
| 产出 | 内嵌报告格式，让用户一次粘贴即可 |

---

## 工作流程

| 阶段 | 主线做什么 | 目标 session 做什么 | 产出 |
|---|---|---|---|
| 1. 判断任务 | 判断是不是机械、验证、探索类任务 | - | 决定是否外包 |
| 2. 生成提示词 | 按五段式写自包含委派提示词 | - | 可直接粘贴的提示词 |
| 3. 执行委派 | 让用户选择 subagent 或新 session | 跑命令、查数据、保留证据 | 结构化报告 |
| 4. 消费报告 | 基于证据继续写代码或决策 | - | PASS / FAIL / 下一轮委派 |

Outsource 按任务规模分三档：

| 档位 | 适用 | 处理方式 |
|---|---|---|
| 档 0 | 极小且干净，例如读一个短文件、取一个值 | 主线直接处理 |
| 档 1 | 小到中等、机械、可能刷屏或要几步 | 委派，通常建议 subagent |
| 档 2 | 大量重复、输出很长、需要人工把关或独立计费 | 委派，通常建议新 session |

---

## 快速开始

把这个仓库作为 Agent skill 使用时，核心入口是：

```text
SKILL.md
```

在支持 skills 的 Agent 环境中，当你提出类似请求时会触发 Outsource：

```text
帮我外包跑一下构建验证
```

```text
生成一份查依赖版本和 FQN 的委派提示词
```

```text
把日志排查委派给低配 session，要求按报告格式返回
```

主线会产出一份完整提示词。你可以把它交给：

| 跑法 | 适合情况 |
|---|---|
| subagent | 当前环境支持 agent 工具，任务中等，想自动收回报告 |
| 新 session | 想走独立便宜额度，或任务较大、需要人工把关 |

注意：Outsource 本身不提供 CLI，也不会替你安装依赖或修改目标项目。它的产物是可执行的委派提示词和报告协议。

---

## 场景模板

当前内置 6 个场景模板：

| 场景 | 文件 | 一句话 |
|---|---|---|
| 构建 / 编译验证 | [`build-compile.md`](references/scenarios/build-compile.md) | 跑构建，报告 PASS/FAIL 和错误原文 |
| 依赖 / 版本 / 符号 FQN 解析 | [`dependency-symbol.md`](references/scenarios/dependency-symbol.md) | 查版本、制品存在性、类全限定名和签名 |
| 数据查询 / 采集 | [`data-query.md`](references/scenarios/data-query.md) | 只读查询 DB、接口、CLI 或文件，原值返回 |
| 日志 / 报错排查 | [`log-debug.md`](references/scenarios/log-debug.md) | 按 pattern grep 日志，只报命中证据 |
| 环境 / 连通性探测 | [`env-probe.md`](references/scenarios/env-probe.md) | 探端口、服务、接口和健康检查端点 |
| 深度学习 / 数据集核查 | [`dataset-inspect.md`](references/scenarios/dataset-inspect.md) | 核查规模、schema、标签分布、坏样本和泄漏 |

通用骨架位于：

```text
references/templates.md
```

新增场景的作者流程位于：

```text
references/authoring.md
```

---

## 文件协议

```text
outsource/
├── SKILL.md                         # skill 入口：触发条件、工作流、执行模式、反模式
├── LICENSE                          # MIT License
└── references/
    ├── templates.md                 # 五段式委派提示词骨架
    ├── authoring.md                 # 新增自定义场景的作者流程
    └── scenarios/
        ├── index.md                 # 场景索引
        ├── build-compile.md         # 构建 / 编译验证
        ├── dependency-symbol.md     # 依赖 / 版本 / FQN 解析
        ├── data-query.md            # 数据查询 / 采集
        ├── log-debug.md             # 日志 / 报错排查
        ├── env-probe.md             # 环境 / 连通性探测
        └── dataset-inspect.md       # 数据集核查
```

---

## 委派铁律

所有委派提示词都应守住这些规则：

- 只读，不改文件，不提交代码，除非任务明确允许写
- 版本号、FQN、报错、命中行、签名等关键信息逐字原文返回
- 查不到就说查不到，绝不编造看似合理的答案
- 不确定或有歧义时必须标注
- 只报事实与证据，不替主线下主观结论
- 卡点如网络、认证、缺工具、超时要如实报告，不反复硬试

---

## 设计边界

Outsource 当前只负责：

- 定义委派方法论
- 提供五段式提示词骨架
- 提供常见研发场景模板
- 约束低配 session 的报告格式和证据要求

它不负责：

- 自动执行构建、测试或查询命令
- 安装依赖或管理运行环境
- 替用户选择 subagent 还是新 session
- 在没有证据时替目标 session 推断根因
- 处理不可逆副作用操作

---

## 产物示例

下面是一份构建验证类任务的委派提示词形态。

```text
# 角色
你是构建与编译领域专家。本次任务：验证某工程当前能否编译通过、依赖能否拉取，并如实回报结果。

# 背景（帮你理解在找什么）
- 对象：/path/to/project，技术栈 maven
- 为什么做：刚调整依赖版本，需要确认当前代码能否编译

# 步骤（按序，每步留真实输出作证据）
1. cd /path/to/project
2. 跑构建：mvn -q -DskipTests compile　目的：看能否编译通过
3. 若失败，完整保留错误输出，尤其首个 error 和依赖解析错误

# 铁律
- 只读，不改任何文件、不动 pom/配置。
- 错误信息逐字贴原文，别概括成"有个编译错误"。
- 分清是代码编译错还是依赖拉取错，分别如实报。
- 只报事实，不下主观结论。
- 卡点如实写，别反复重试。

# 产出（你按下面格式回传）
## 构建验证报告
1. 编译结论：PASS / FAIL
2. 关键错误原文（FAIL 时）：
   ```
   <粘贴>
   ```
3. 依赖拉取问题：<有/无；哪个依赖、什么错>
4. 卡点：<无则写"无">
```

目标 session 回传报告后，主线再基于事实继续修代码、改依赖或发起下一轮更聚焦的委派。

---

## License

MIT License. See [`LICENSE`](LICENSE).
