<p align="center">
  <h1 align="center">xshang-skill</h1>
</p>

<p align="center">
  <em>"先跑起来再说。够用就上，迭代优化。"</em>
</p>

<p align="center">
  <a href="https://claude.ai/code"><img src="https://img.shields.io/badge/Claude%20Code-Skill-blueviolet" alt="Claude Code"></a>
  <a href="https://cursor.com"><img src="https://img.shields.io/badge/Cursor-Skill-blue" alt="Cursor"></a>
  <a href="https://agentskills.io"><img src="https://img.shields.io/badge/AgentSkills-Standard-green" alt="AgentSkills"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License: MIT"></a>
</p>

<br/>

<p align="center">
你让 AI 做功能，它上来就写代码，写完跟你想的不一样？<br/>
你让 AI 修 bug，它猜来猜去改了一堆无关代码？<br/>
你让 AI 给技术方案，它给你一个"各有优劣，取决于需求"？<br/>
你让 AI 帮你规划项目，它给你一个大而全但落不了地的方案？<br/>
</p>

<p align="center">
<strong>装上这个 Skill，AI 就按你的规矩干活。<br/>先问清楚再动手，分模块执行，不懂就问，做完简要汇报。</strong>
</p>

---

## 这个 Skill 有什么不一样？

不是"教 AI 写代码"，是把一套**实战验证的人机协作操作系统**装进去了：

| 场景 | 普通 AI | 装了 xshang-skill 的 AI |
|------|---------|------------------------|
| "我想做 X" | 直接开始写代码 | 先给 3 个技术方案，等你拍板再动手 |
| 修 bug | 猜测性地改一堆代码 | 先看报错，分析根因，改一处测一处 |
| 技术选型 | "各有优劣，取决于需求" | "推荐方案 A，因为...。B 的优势在你这里用不上" |
| 修了 3 轮还有 bug | 继续在同一个方案上打补丁 | "这个方案不稳定，建议换成 XXX" |
| 做完功能 | 交代码就完事 | 提醒更新四类文档，规范 commit message |
| 汇报 | 写一大段总结 | 三句话：改了什么、影响什么、注意什么 |

---

## 蒸馏了什么进去？

### 5 个心智模型

| 模型 | 一句话解释 |
|------|-----------|
| 人定方向，AI执行，人验收 | 你是产品经理+验收者，AI 是架构师+程序员 |
| 够用就上，迭代优化 | v1 能跑就发，不追求一步到位 |
| 做一个拉一个 | 改代码时把上下游一起考虑 |
| 不懂就问，不要猜 | 不确定的地方问用户，猜错了返工更慢 |
| 修不好就换方案 | 同一个 bug 修 3 轮没好，换思路 |

### 8 阶段工作流

```
构思 → 技术选型 → 计划制定 → 分模块执行 → 整体测试 → 查漏补缺 → 文档编写 → 提交
```

### 11 套代码模板

Pinia Store / Axios 封装 / API 模块 / Composable / 路由守卫 / 数据获取 / 表格分页 / 表单提交 / 删除确认 / ECharts 集成 / Vite 配置

### 踩坑手册

从 194 次真实对话中提炼的 15+ 个高频错误及解法，覆盖：Mermaid 中文兼容、PDF 导出、混合内容、Electron 打包、Nginx 部署等。

---

## 安装

### Claude Code

```bash
# 安装到当前项目
mkdir -p .claude/skills
git clone https://github.com/yourname/xshang-skill.git .claude/skills/xshang-skill

# 或全局安装
git clone https://github.com/yourname/xshang-skill.git ~/.claude/skills/xshang-skill
```

### Cursor

```bash
mkdir -p .cursor/skills
git clone https://github.com/yourname/xshang-skill.git .cursor/skills/xshang-skill
```

> 安装后 AI 会自动识别并按 xshang 工作流协作。也可以直接说"用 xshang 流程"手动触发。

---

## Skill 结构

```
xshang/
├── SKILL.md                          # 入口：触发条件、工作流、响应模式
└── references/
    ├── workflow.md                    # 8 阶段工作流、心智模型、决策规则、文档体系
    ├── voice.md                      # 指令分类、语气特征、情绪响应、提问规范
    ├── codebase.md                   # 技术栈、11 套代码模板、UI 范式
    └── troubleshooting.md            # 排查原则、15+ 高频错误速查、踩坑手册
```

遵循 [AgentSkills](https://agentskills.io) 开放标准，渐进式加载：AI 先读 SKILL.md 判断是否触发 → 触发后按需加载 references。

---

## 快速上手（不安装 Skill 也能用）

复制下面这段到你的 `CLAUDE.md` 或 `.cursorrules`：

```markdown
## AI 协作规则 (xshang)

1. 我说"我想做 X"时，先给技术方案（3个选项），不要直接写代码
2. 方案确认后，出分模块计划，等我审批
3. 执行时不确定的地方必须问我，不要猜
4. 改代码时考虑上下游影响
5. 不要加计划外的功能、注释、"优化"
6. 做完后提醒我更新四类文档：项目总结、技术文档、模块框架、部署文档
7. 回复简洁直接，中文为主，技术词用英文
8. 提问时给 3 个选项
9. commit 格式：vX.Y.Z type(scope): 中文描述
10. 同一个 bug 修 3 轮还没好就换方案
11. 我说"继续完善"就接着干，不要重新问我要做什么
12. 粘贴错误日志时先分析根因，不要盲改
```

---

## 蒸馏过程

1. **对话历史收集** — 提取 194 条真实 AI 协作对话（Trae IDE）
2. **Git 历史分析** — 分析 27 次提交的模式、版本策略、迭代节奏
3. **代码风格提取** — 扫描 34 个 Vue 组件、15 个 API 模块、6 个工具模块
4. **指令分类统计** — 对 194 条指令按类型分布归类（命令型/定位型/需求型/错误型/API型）
5. **踩坑模式识别** — 从反复修复的对话中提炼错误模式和解法
6. **用户自述校准** — 用户描述自己的工作流，修正推断偏差

---

## 持续更新

| 素材类型 | 更新哪个文件 |
|----------|-------------|
| 新的代码范式 | `references/codebase.md` |
| 新的踩坑经验 | `references/troubleshooting.md` |
| 工作流调整 | `references/workflow.md` |
| 沟通风格变化 | `references/voice.md` |

---

<p align="center">
  <em>"你把控方向，AI 负责干活，做之前先对齐，做的时候不懂就问，做完了一起检查，修不好就换方案。"</em>
</p>

<p align="center">
  MIT License
</p>
