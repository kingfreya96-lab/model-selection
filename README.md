# model-selection

一个面向 Codex / Claude Code 等 Agent 的**基模选型 Skill**。核心思想：不要问"哪个模型最强"，先问"在我的场景里，什么情况算失败"。通过 **失败模式硬筛 → 加权评分卡 → 最小业务评测集 → 路由降级** 四步漏斗，输出可复核的选型决策档案。

## 它解决什么

- 用公开榜单拍脑袋选模型
- 只看每百万 token 单价，忽略修复轮次带来的真实成本
- Agent 场景只看 SWE-bench，不看自家工具 schema 下的函数调用合法率
- 选完一个模型就结束，没有降级与路由

## 安装

### Codex

```bash
# 全局（所有项目可用）
cp -r model-selection ~/.agents/skills/

# 或项目级
cp -r model-selection <your-repo>/.agents/skills/
```

### Claude Code

```bash
cp -r model-selection ~/.claude/skills/
```

安装后重启会话。

## 触发方式

隐式：直接描述任务，例如"帮我们给客服 Agent 选一个基模"。

显式：`$model-selection`（Codex）或 `/model-selection`（Claude Code）。

## 目录结构

```
model-selection/
├── SKILL.md
└── references/
    ├── failure-modes.md
    ├── scorecard-template.csv
    ├── scorecard-method.md
    ├── benchmarks.md
    ├── routing-decision-tree.md
    └── output-template.md
```

## 使用流程

1. 让 Agent 加载本 skill，描述任务与"什么算失败"
2. 按硬排除淘汰候选
3. 填评分卡（权重从失败模式反推）
4. 在 50–200 条真实样本上做前两名配对比较
5. 设计路由与降级链
6. 输出决策简报（`references/output-template.md`）

## License

MIT