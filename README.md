# neo-write-skill

> 把碎片化的想法，变成**你自己声音**的平台内容——不是千篇一律的 AI 腔。
>
> A Claude Code / Agent **Skill** that turns fragmented ideas into platform-ready content written in *your own voice*. It interviews you step by step, drafts from a voice profile it learns over time, and gets more like you every time you use it.

一个写作类 Agent Skill。你把脑子里的碎片丢进来，它一步步问你（发哪个平台、想达到什么、什么结构、什么角度），然后**按你的写法**起草。你每改一次稿，它就更懂你怎么说话。

## 它解决什么

- 有想法但**写不出来 / 不会表达**。
- AI 写出来的东西**千篇一律、没有个人特点**。
- 同一个想法，发朋友圈、推特、小红书、公众号各有各的形状，懒得每次重想。

## 核心特点

1. **一步步面试**：不替你拍脑袋，问清平台/目标/结构/角度，但每个问题都先给推荐项，点一下就过。
2. **越用越像你**：一个 `voice-profile.md` 声音档案，记录你的句子节奏、口头禅、忌用词、标点习惯。你每改一次稿，它把规律写回档案。
3. **从头就不像 AI**：内置「AI 腔黑名单」，交稿前自检。理念是**不事后去 AI 味，而是从一开始就用你的写法写**。
4. **零外部依赖**：标题公式、AI 腔识别、状态持久化全部内置，独立可跑，不调用任何其它 skill。

## 架构

```
                      你: /输出  +  碎片想法
                              │
                              ▼
        ┌─────────────────────────────────────────┐
        │  SKILL.md   ← 编排器：6 步流程，按需加载    │
        └─────────────────────────────────────────┘
           │读         │读          │读       │读       │读+写(状态)
           ▼           ▼            ▼         ▼         ▼
      interview.md  platforms.md  titles.md  voice-engine  memory/
      (怎么问)      (平台规范)    (标题公式)  (引擎+黑名单) ├─ voice-profile.md ★真源(私有)
                                                          └─ samples/         你的语料(私有)
```

SKILL.md 只装流程不装知识，知识拆进分文件，用到哪个读哪个（progressive disclosure）。

## 安装

1. 克隆到本地：
   ```bash
   git clone https://github.com/ImNeo-hub/neo-write-skill.git
   ```
2. 软链到 skills 目录（Claude Code 和 Codex 都支持）：
   ```bash
   ln -s /path/to/neo-write-skill ~/.claude/skills/neo-write-skill
   ln -s /path/to/neo-write-skill ~/.codex/skills/neo-write-skill
   ```
3. 创建声音档案（必做，否则写出来还是 AI 腔）：
   ```bash
   cp neo-write-skill/memory/voice-profile.template.md neo-write-skill/memory/voice-profile.md
   ```

然后在 Claude Code 里喊 `/nws`（或说「帮我写」「用我的声音写」），它会带你跑冷启动校准——你贴几篇自己满意的旧文，它抽出你的声音特征填进档案。

## 用法

- `/输出`、`/表达`、`/写`，或直接说「帮我把这个想法写出来」「这条发小红书怎么写」。
- 把碎片丢进去 → 回答几个快速问题 → 拿到初稿 → 改 → 它学会 → 下次更像你。

## 它怎么「越用越像你」

```
你改稿/吐槽 → diff「你把什么换成了什么」→ 抽成规则 → 写回 voice-profile.md「学习日志」
                                                              │
                                            攒到约 15 条 → 合并进正式维度、去重
```

学习不靠「记住对话」（对话会丢），靠把规律**沉淀到磁盘上的档案**。用得越多，档案越厚，AI 味越少。

## 文件结构

| 文件 | 作用 | 入库？ |
|---|---|---|
| `SKILL.md` | 编排器，6 步主流程 | ✅ |
| `interview.md` | 面试问法（批量问、给默认） | ✅ |
| `platforms.md` | 朋友圈/推特/小红书/公众号 规范 | ✅ |
| `titles.md` | 内置标题公式 | ✅ |
| `voice-engine.md` | 校准 + 学习回路 + AI 腔黑名单 | ✅ |
| `memory/voice-profile.template.md` | 空白档案模板 | ✅ |
| `memory/voice-profile.md` | **你的**声音档案 | ❌ 私有 |
| `memory/samples/` | **你的**真实语料 | ❌ 私有 |

## 🔒 隐私（重要）

`memory/voice-profile.md` 和 `memory/samples/` 里是**你的私人写作甚至日记**，已被 `.gitignore` 屏蔽，**不会进仓库**。公开仓库只包含引擎和空白模板。

> ⚠️ 推送前自己确认一次：`git status` 不应出现 `voice-profile.md` 或 samples 里的真实语料。

## 内化的思路（学习，不调用）

本 skill 内化了这些常见做法，但**不在运行时调用**任何外部 skill，因此可独立运行：

- **AI 写作特征识别** → 内置「AI 腔黑名单」自检（理念：识别 AI 味是为了找回自己的写法，不是伪装成人类）。
- **标题公式库** → 内置 `titles.md` 精简版。
- **跨会话状态持久化** → 内置 `voice-profile.md` 读写机制。

## License

[MIT](LICENSE) © 2026 Neo
