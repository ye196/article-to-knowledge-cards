# make-knowledge-cards

一个 Agent Skill：把一篇文章拆成 5–8 张可复习的知识卡片，让长文里的重点变成能被记住、被检索的短内容。

## 项目解决什么问题

读完一篇长文后，常见的问题是：划了很多线，却说不清真正学到了什么；过几天只记得零散句子；想复习又不想重读全文。

这个 Skill 把文章压缩成少量知识卡片：只保留中心论点、因果机制、关键数据、可执行步骤和常见误区，一张卡片只讲一个知识点。它不替用户补充外部知识，也不为了凑数量拆分内容，因此卡片可以直接作为复习材料使用。

## 主要功能

- 输入一篇粘贴的文章，或本地 `.md` / `.txt` 文件。
- 输出 5–8 张知识卡片；原文知识点不足 5 个时，按实际数量输出并说明原因。
- 每张卡片固定包含四部分：**标题 / 核心知识 / 简明解释 / 例子或自测问题**。
- 自动合并重复内容，保证一张卡片只有一个中心。
- 所有内容必须能在原文找到依据：不补充领域常识，不虚构数据、案例或因果。
- 原文是观点时写成「作者认为……」，不当作客观结论。
- 输出语言与原文一致。
- 用户明确要求表格、JSON 或写入文件时，按用户指定格式输出。

暂不支持：网页抓取、PDF 解析、Anki 导出、图形界面。遇到网页链接或 PDF，请粘贴正文，或先另存为 `.md` / `.txt` 再处理。

## 安装方法

### 环境要求

- 任意支持 Agent Skills 的客户端（Codex、Claude Code、Trae 等）。
- 无运行时依赖、无构建步骤：Skill 只由 `SKILL.md` 和 `agents/openai.yaml` 组成。

### 安装步骤

安装就是把整个 `skills/make-knowledge-cards` 目录放进客户端的 skills 目录。以 Codex 的用户级目录为例：

```bash
mkdir -p ~/.agents/skills
cp -r skills/make-knowledge-cards ~/.agents/skills/
```

Windows PowerShell：

```powershell
New-Item -ItemType Directory -Force "$HOME\.agents\skills" | Out-Null
Copy-Item -Recurse .\skills\make-knowledge-cards "$HOME\.agents\skills\"
```

也可以按客户端选择位置：

| 客户端 | 项目级目录 | 用户级目录 |
|---|---|---|
| Codex | `<仓库>/.agents/skills/` | `~/.agents/skills/` |
| Claude Code | `<项目>/.claude/skills/` | `~/.claude/skills/` |
| Trae | `<项目>/.trae/skills/` | `~/.trae-cn/skills/` |

安装后如果客户端没有立即识别，重启或重新加载即可。目录名必须保持 `make-knowledge-cards`，且与 `SKILL.md` 中的 `name` 一致。

## 使用方法

显式调用（Codex 中输入 `$` 可选择技能）：

```
用 $make-knowledge-cards 把下面这篇文章做成知识卡片：
<粘贴正文>
```

处理本地文件：

```
把 notes/sleep-and-memory.md 提炼成知识卡片
```

也可以直接描述任务，让客户端按 `description` 自动匹配：

```
帮我把这篇文章拆成知识卡片
整理成读书笔记卡片
提炼这篇文章的知识点
```

输出会以「共 N 张卡片」开头；若少于 5 张，会说明原文可提炼的知识点数量。除卡片外不添加寒暄、全文摘要或延伸阅读。

## 输入输出示例

输入（节选）：

```
白天学到的内容先暂存在海马体里，深睡时大脑会把这些信息反复"重播"，
并逐渐转移到大脑皮层，形成更稳定的长期记忆。换句话说，白天是"写入"，
夜里才是"存档"。
```

输出（节选，完整文章会生成 6 张）：

```markdown
共 6 张卡片

### 卡片 1：深睡把白天的记忆"存档"

**核心知识**：白天学到的内容暂存在海马体，深睡时被重播并转移到大脑皮层，形成更稳定的长期记忆。

**简明解释**：睡眠不是学习的暂停，非快速眼动睡眠的深睡阶段与记忆巩固关系最密切。白天负责写入，夜里负责存档。

**例子 / 自测**：为什么熬夜刷题反而可能记不牢？——因为压缩了深睡，白天的"写入"缺少夜里的"存档"。
```

信息量不足时不会凑数。例如一篇只有三个论点的短文，输出为：

```markdown
共 3 张卡片（原文可提炼的知识点只有 3 个）
```

## 目录结构

```
skills/make-knowledge-cards/
├── SKILL.md           # 技能说明、制作流程、事实边界与自检清单
└── agents/openai.yaml # UI 元数据（名称、简介、默认提示词）
```

## 开发与验证

本项目使用 OpenAI 官方 `skill-creator` 中的 `init_skill.py` 初始化：

```bash
python init_skill.py make-knowledge-cards --path skills \
  --interface display_name="Make Knowledge Cards" \
  --interface short_description="Turn an article into 5-8 knowledge cards" \
  --interface default_prompt="Use \$make-knowledge-cards to turn this article into 5-8 knowledge cards."
```

修改 `SKILL.md` 后运行官方校验脚本：

```bash
python quick_validate.py skills/make-knowledge-cards
# Skill is valid!
```

两个脚本来自 <https://github.com/openai/skills>（`skills/.system/skill-creator/scripts/`），不随本仓库分发。
