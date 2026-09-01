# project-mind

> *A project-memory skill for AI coding agents: auto-builds a knowledge + progress memory from a project's docs, with manual and automatic updates.*

`project-mind` 是一个面向 AI 编码 agent（如 Reasonix/Claude 风格的 agent 环境）的技能（skill）。打开一个项目后，它会**根据项目内文档自动创建记忆**（知识库 + 进度），并在后续对话中**手动或自动更新**记忆，让 agent 快速理解项目、跨会话续作。

## 核心概念

记忆存放在项目内 `.reasonix/memory/` 下，分三个文件：

| 文件 | 内容 | 更新频率 |
|---|---|---|
| `knowledge.md` | 静态知识：术语表、架构、技术选型、约定、文档摘要 | 低 |
| `progress.md` | 动态进度：已完成、进行中、下一步、更新记录 | 高（每次任务结束） |
| `index.md` | 索引：模块 → 关键词 → 章节位置 | 随 knowledge 更新 |

## 三条核心流程

1. **初始化**：会话启动检测到无记忆时，扫描项目文档（README、`docs/**`、主源文件、`*.bib` 等），提取术语表/架构/约定/摘要，生成三文件。
2. **手动更新**：用户说"更新记忆" → 重扫项目文档，增量合并（`<!-- 人工确认 -->` 条目优先保留，疑似过时先询问再删）。
3. **自动更新**：任务收尾把完成事项与决策写回 `progress.md`；文档实质变化时触发知识更新。

## 效果：能省多少上下文 / token

以下用真实项目（`paper_e`，一篇 LaTeX 论文：`main.tex` ≈ 61 KB + `ref.bib` ≈ 12 KB + 插件设计文档）实测：

| 场景 | 读入内容 | 大小 | 估算 token |
|---|---|---|---|
| 无插件（为理解项目读核心文档） | `main.tex` + `ref.bib` + 设计文档 | ~79 KB | **~19.7k** |
| 有插件 —— 会话启动 | `index.md` + `progress.md` | ~1.7 KB | **~0.4k** |
| 有插件 —— 知识查询 | `knowledge.md` 对应小节（按需） | 1 节 | ~0.2k |

**每会话约省 98% 上下文**（≈19.7k → ≈0.4k tokens），且知识按需查询、不必每次重读整个项目。

> token 估算按 ~4 字符/token；项目文档以英文 LaTeX 为主。实际数值随项目规模与语言变化。

### 为什么能省这么多

没有插件时，agent 每次会话都要重新读入项目文档来"理解项目"；有插件后，它只需：

1. **启动注入** `index.md`（知道"有什么"）+ `progress.md`（知道"做到哪"）——都很短；
2. **按需查询** `knowledge.md` 的对应小节——只读需要的那一段。

其余内容不进上下文，省下的 token 留给真正的任务。

## 安装

把 `SKILL.md` 放入 agent 的技能目录（本插件安装位置）：

```
~/.agents/skills/project-memory/SKILL.md
```

或直接使用打包产物 `project-mind.skill`。

## 记忆文件格式示例

`knowledge.md`：
```markdown
# 项目知识
## 术语表
| 术语 | 解释 |
|---|--|
## 架构与模块
## 技术选型与约定
## 文档摘要
```

`progress.md`：
```markdown
# 项目进度
## 已完成
## 进行中
## 下一步
## 更新记录（倒序）
- YYYY-MM-DD：...
```

## License

MIT — 详见 [LICENSE](LICENSE)。
