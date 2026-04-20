# 🌿 Trellis + Claude Code + Codex 实用手册

> 作者：ZQYuan
> 日志：
>
> 
> - 2026-04-17创建此手册
>- 2026-04-18 修正 Claude Code 续工防串仓库：`memory` 仅作候选线索，必须先核对当前仓库的 `AGENTS.md`、`.trellis/tasks/`、`.trellis/workspace/`
> - 2026-04-18 补充 Codex 续工防串仓库：先核对 `AGENTS.md`、`.trellis/tasks/`、`.trellis/workspace/`，旧交接文档仅作可选增强材料
> - 2026-04-18 补充 `trellis init` 仅用于首次初始化，日常开工直接使用 `claude` / `codex`
> - 2026-04-18 扩充记忆与上下文管理
> - 2026-04-18 补充MEMORY.md 和 AGENTS.md防写爆规则
> - 2026-04-19修复第5章的开工（首次初始化后） / 进行中 / 收工 / 续工提示词

### 快速导航

- 想知道**当前仓库到底有什么**：看 **第 2 章**
- 想知道**Trellis 一般怎么工作**：看 **第 3 章**
- 想知道**第一次接入和日常开工的区别**：看 **第 4 章**
- 想直接照着操作：看 **第 5 章** 和 **第 7 章**
- 想避免续工串仓库：重点看 **1.1**、**5.4**、**6.2**
- 想区分哪些只是这台机器的经验：看 **附录 A**

---

## 1. 阅读说明：这份手册怎么读

这份手册统一使用 4 种事实标签：

- **[已验证]**：已经直接核对过当前仓库或当前机器上的事实。
- **[通用]**：Trellis / Claude Code / Codex 常见能力或常见工作方式，但不等于所有项目都一样。
- **[本机特例]**：只对这台 **Windows + Git Bash** 机器成立的规则。
- **[未验证]**：在一些 Trellis 项目里常见，但当前仓库没有直接证据，不能当成现实默认写死。

### 1.1 证据优先级

续工、排查、写文档时，统一按这个顺序判断：

1. **当前仓库实际存在的长期入口文件**（优先看 `AGENTS.md`）
2. **当前任务文件**（如 `.trellis/tasks/`）
3. **当前开发者 journal**（如 `.trellis/workspace/ZQYuan/journal-1.md`）
4. **当前仓库里实际存在的相关 spec / 工作流 / 交接文档**
5. **Claude memory / 旧会话 / 外部描述**

> 结论：**仓库事实优先，记忆只能辅助。** 别把“AI 记得什么”误当成“仓库现在真的有什么”。

---

## 2. 当前仓库现状总览

这一章只写**当前仓库已确认存在**的东西。

### 2.1 当前仓库已验证存在

[已验证] 项目根目录存在：

- `AGENTS.md`
- `.trellis/workflow.md`
- `.trellis/spec/backend/`
- `.trellis/spec/frontend/`
- `.trellis/spec/guides/`
- `.trellis/scripts/`
- `.trellis/workspace/ZQYuan/journal-1.md`
- `.codex/config.toml`
- `.codex/hooks/session-start.py`
- `.codex/agents/*.toml`
- `.codex/skills/parallel/SKILL.md`
- `.agents/skills/` 下多条技能目录（如 `start`、`record-session`、`finish-work`、`update-spec` 等）

[已验证] 当前仓库的 Trellis spec 至少包含这些规范文件：

- backend：`index.md`、`directory-structure.md`、`error-handling.md`、`logging-guidelines.md`、`database-guidelines.md`、`quality-guidelines.md`
- frontend：`index.md`、`directory-structure.md`、`component-guidelines.md`、`hook-guidelines.md`、`state-management.md`、`type-safety.md`、`quality-guidelines.md`
- guides：`index.md`、`code-reuse-thinking-guide.md`、`cross-layer-thinking-guide.md`

[已验证] 当前仓库存在开发日志：

- `.trellis/workspace/ZQYuan/journal-1.md`

[已验证] 当前仓库已出现归档任务示例：

- `.trellis/tasks/archive/2026-04/00-bootstrap-guidelines/task.json`

### 2.2 当前仓库项目身份

[已验证] 根据 `AGENTS.md`，当前仓库不是单纯的 Web 项目，而是一个以**嵌入式学习资料、GD32 固件工程、课程示例**为主的单仓库。

[已验证] 当前仓库已确认的项目级决策包括：

- `GD_Firmware_Template/`：保留上游原版镜像，用于学习和对照。
- `GD_Firmware_Template_with_dependence/GD_Firmware_Template/`：作为自包含增强版，用于直接开发和跨机器迁移。

### 2.3 不要误解成“所有 Trellis 项目都这样”

[通用] 你在别的 Trellis 项目里，可能看见类似目录。

[已验证] 但**本手册后续凡是提到当前仓库结构，默认只以上面这些已核对到的路径为准**。

---

## 3. Trellis 的通用工作模型

这一章说的是 **Trellis 一般怎么组织 AI 开发流程**，不是说每个项目都会一模一样。

### 3.1 Trellis 大体在做什么

[通用] Trellis 主要解决 4 类问题：

| 问题 | 通用做法 |
|------|----------|
| AI 容易忘上下文 | 用 `AGENTS.md`、`.trellis/workspace/`、任务目录维持项目记忆 |
| AI 容易不按约定写代码 | 用 `.trellis/spec/` 放规范与思考指南 |
| 多轮开发容易失焦 | 用 `.trellis/tasks/` 跟踪任务与阶段产物 |
| 多工具协作容易串上下文 | 用项目级文件做共享入口，而不是只依赖某个工具的私有记忆 |

### 3.2 当前仓库已经具备哪些通用部件

[已验证] 当前仓库里，下面这些“通用部件”已经真实存在：

- 项目共享入口：`AGENTS.md`
- 工作流说明：`.trellis/workflow.md`
- 规范目录：`.trellis/spec/`
- 工作日志目录：`.trellis/workspace/`
- 脚本目录：`.trellis/scripts/`
- Codex 项目配置：`.codex/`
- 共享技能目录：`.agents/skills/`

### 3.3 哪些内容不能写成绝对事实

下面这些说法，容易把“通用能力”误写成“当前现实”：

- “Trellis 一定会自动注入所有规范”
- “所有平台都支持同一种斜杠命令”
- “所有项目都默认已经配好 Codex hooks”
- “只要启动 AI 就一定知道最近做了什么”

更稳妥的写法应该是：

- [通用] Trellis **通常会**通过项目文件、脚本、hooks 或技能目录帮助 AI 建立上下文。
- [已验证] 当前仓库确实存在 `.codex/hooks/session-start.py` 和 `.agents/skills/`。
- [本机特例] 某些全局配置是否已打开，取决于这台机器，而不是项目本身。

---

## 4. 首次接入 vs 日常开工

这一章最重要的结论只有一个：

> **`trellis init` 是首次接入命令，不是日常开工命令。**

### 4.1 首次接入时做什么

[通用] 新项目第一次接入 Trellis，通常是：

```bash
git init
npm install -g @mindfoldhq/trellis@latest
trellis init -u ZQYuan --claude --codex --skip-existing
```

适用场景：

- 第一次给项目接入 Trellis
- 旧项目第一次补装 Claude / Codex 支持
- 明确要重建初始化产物

### 4.2 日常开工时做什么

[通用] 如果项目已经接入过 Trellis，日常开工通常只需要进入项目再启动工具：

```bash
cd "C:/Users/21906/Desktop/你的项目"
claude
```

或：

```bash
cd "C:/Users/21906/Desktop/你的项目"
codex
```

### 4.3 为什么不能把 `trellis init` 当作“启动按钮”

[通用] 因为它的职责是**初始化配置**，不是**恢复当天工作现场**。

重复乱跑的风险包括：

- 重新处理初始化文件
- 让你误以为“续工 = 再 init 一次”
- 增加项目级配置混乱概率

### 4.4 本机环境特例

[本机特例] 这台机器是 **Windows + Git Bash**，有 3 条要单独记住：

1. 手动跑 Python 脚本时，优先用 `python`，不要默认写 `python3`
2. 做任何脚本操作前，先确认真实工作目录
3. 路径尽量写成带引号的 Git Bash / 通用格式

示例：

```bash
pwd
python --version
git status --short
```

如果你是在本机手动跑 Trellis 脚本，建议写成：

```bash
python "./.trellis/scripts/task.py" list
```

> [本机特例] 这一条来自当前机器的实际使用经验，**不是** Trellis 的通用文档规则。

---

## 5. 日常操作 SOP：开工（首次初始化后） / 进行中 / 收工 / 续工

这一章只保留一套唯一流程，避免前后重复。

### 5.1 开工（首次初始化后）

#### Claude Code

```bash
cd "C:/Users/21906/Desktop/你的项目"
claude
```

进入 Claude Code 后，推荐直接输入：

```text
/trellis:start
```

#### Codex

```bash
cd "C:/Users/21906/Desktop/你的项目"
codex
```

进入 Codex 后，推荐直接输入：

```text
继续当前项目，先核对 AGENTS.md、当前任务和最新 journal，再开始做功能。
```

### 5.2 进行中

#### 需求还没理清

Claude Code：

```text
/trellis:brainstorm
```

Codex：

```text
帮我梳理一下需求，再给我一个最小可行方案
```

#### 开始正式开发

无论用哪个工具，建议都把提示词说清楚：

```text
先读当前仓库事实，再做修改；如果要参考旧上下文，只采用能被当前仓库文件印证的信息
```

### 5.3 收工

#### Claude Code 常用收工顺序

先输入：

```text
/trellis:finish-work
```

**然后由人类自己在工程根目录打开终端输入以下git指令：**

> ⚠️ 重要提醒：Git 只能提交**当前仓库里的文件**。
> 如果你修改的是仓库外文件（例如桌面上的独立 `Trellis-使用手册.md`），那么：
>
> - `git status` 可能仍显示 `working tree clean`
> - `git add "文件路径"` 可能报 `pathspec did not match any files`
> - 这种情况下不能在当前仓库执行提交，也不应该继续 `/trellis:record-session`
>
> 换句话说：**只有文件实际位于当前仓库目录内时，下面这些 Git 命令才适用。**

如果你只会最常用的 Git 提交流程，直接照抄下面这组：

```bash
git status
git add "文件路径"
git commit -m "这里写这次改了什么"
```

例如你这次改的是本手册，可以直接写成：

```bash
git status
git add "Trellis-使用手册.md"
git commit -m "更新 Trellis 使用手册中的开工收工续工指令"
```

如果你已经提前 `git add` 过文件，也可以只执行：

```bash
git commit -m "更新 Trellis 使用手册中的开工收工续工指令"
```

再在**人工已测试并且代码已经提交后**输入：

```text
/trellis:record-session
```


> ⚠️ 关键校正：`/trellis:record-session` 的详细命令定义明确要求它只能在 **human has tested and committed the code** 之后使用；
> 因此正确顺序是：
>
> `写代码 / 测试 → /trellis:finish-work → 人类 git status + git add + git commit -m "这里写这次改了什么" → /trellis:record-session`

必要时再输入：

```text
把这次沉淀成规范，并更新项目共享上下文
```

#### Codex 常用收工顺序

先输入：

```text
帮我做完工检查。按 finish-work 的顺序检查：代码质量、测试、spec 是否需要更新、有没有跨层影响。
```

**然后由人类自己在工程根目录打开终端输入以下git指令：**

> ⚠️ 重要提醒：Git 只能提交**当前仓库里的文件**。
> 如果你修改的是仓库外文件（例如桌面上的独立 `Trellis-使用手册.md`），那么：
>
> - `git status` 可能仍显示 `working tree clean`
> - `git add "文件路径"` 可能报 `pathspec did not match any files`
> - 这种情况下不能在当前仓库执行提交，也不应该继续记录 journal / 归档任务
>
> 换句话说：**只有文件实际位于当前仓库目录内时，下面这些 Git 命令才适用。**

如果你只会最常用的 Git 提交流程，直接照抄下面这组：

```bash
git status
git add "文件路径"
git commit -m "这里写这次改了什么"
```

例如你这次改的是本手册，可以直接写成：

```bash
git status
git add "Trellis-使用手册.md"
git commit -m "更新 Trellis 使用手册中的开工收工续工指令"
```

如果你已经提前 `git add` 过文件，也可以只执行：

```bash
git commit -m "更新 Trellis 使用手册中的开工收工续工指令"
```

提交完成后，再输入：

```text
我已经人工测试并提交代码了。把这次工作记录到 journal 日志里；如果任务已完成就归档。
```

必要时再输入：

```text
把需要长期保留的项目决策更新到 AGENTS.md，并把这次经验写进 spec 规范
```

### 5.4 续工

这是最容易串仓库的环节，所以只保留一套标准提示词。

#### 通用续工提示词

```text
先确认当前仓库身份：读取 AGENTS.md、.trellis/tasks 当前任务、.trellis/workspace 最新 journal。
如果当前仓库里确实存在相关 spec、workflow 或交接文档，再补读它们。
最后只采用那些能被当前仓库文件印证的背景信息。
我准备开始工作了，我有自己的工作计划,但是我的工作计划视实际情况而定，没有固定，所以不能提前写入，所以可以直接开工吗？
```

#### 为什么这样最稳

- 先锁定当前仓库身份
- 再看任务与日志
- 再补读当前仓库里实际存在的辅助材料
- 最后才允许参考外部材料或旧记忆

> 结论：**续工不是“回忆昨天”，而是“重新建立今天这个仓库的证据链”。**

---

## 6. 上下文、目录职责与证据优先级

这一章负责回答：**到底谁管什么，先看谁，什么不能乱写。**

### 6.1 各类记录分别做什么

| 位置 | 作用 | 谁能看到 |
|------|------|----------|
| `AGENTS.md` | 项目共享入口，放长期有效的项目摘要、决策、关键路径 | Claude / Codex 等项目工具 |
| `.trellis/tasks/` | 任务与需求产物 | 项目内共享 |
| `.trellis/workspace/` | 开发日志、session 记录 | 项目内共享 |
| `.trellis/spec/` | 规范、约定、思考指南 | 项目内共享 |
| `~/.claude/projects/.../memory/` | Claude 专属长期记忆 | 仅 Claude |
| 交接文档 | 某阶段的完整背景材料 | 任何能读文件的工具 |

### 6.2 谁是第一信源

统一规则如下：

1. [已验证] `AGENTS.md` 等当前仓库长期入口文件
2. [已验证] `.trellis/tasks/` 当前任务或归档任务
3. [已验证] `.trellis/workspace/` 最新 journal
4. [已验证] 当前仓库里实际存在的 `.trellis/spec/`、`.trellis/workflow.md`、交接文档等辅助材料
5. [通用] Claude memory / 旧会话摘要 / 外部口头描述

### 6.3 `AGENTS.md` 应该写什么

适合写进 `AGENTS.md` 的内容：

- 项目是什么
- 当前最重要的项目级决策
- 关键目录应该怎么用
- 哪些路径容易混淆
- 当前环境的长期注意事项

不适合写进 `AGENTS.md` 的内容：

- 每次改了哪些文件
- 一次性的排查过程
- 大段流水账
- 普通提交历史

### 6.4 journal 应该写什么

适合写进 `.trellis/workspace/` journal 的内容：

- 这次 session 做了什么
- 产出了哪些关键文件
- 有哪些后续动作
- 某个阶段为什么算完成

[已验证] 当前仓库的 `.trellis/workspace/ZQYuan/journal-1.md` 就在记录这类内容，比如 BSP 学习指南编写、GD32 原版模板整理等会话记录。

### 6.5 Claude memory 应该怎么用

[通用] Claude memory 适合保存：

- 用户偏好
- 跨会话仍有价值的项目决策
- 外部信息入口
- 当前仓库里看不出来、但未来仍影响判断的背景

[关键规则] **Claude memory 只能作候选线索，不能跳过仓库事实校验。**

### 6.6 “防写爆”规则

一句话版：

- `AGENTS.md` 是地图
- journal 是施工日志
- spec 是规范手册
- memory 是 Claude 的长期提醒

谁都不是流水账垃圾桶。写爆了只会害以后的人继续踩坑，真是不优雅。

---

## 7. 命令与提示词速查

这一章只给速查，不重复解释原理。

### 7.1 系统终端命令

| 命令 | 用途 | 备注 |
|------|------|------|
| `trellis --version` | 查看 Trellis 版本 | 新装或升级后确认 |
| `trellis init -u ZQYuan --claude --codex --skip-existing` | 首次初始化项目 | 不是日常启动命令 |
| `trellis update` | 更新项目里的 Trellis 模板文件 | 进入项目后再执行 |
| `claude` | 启动 Claude Code | 日常开工 |
| `codex` | 启动 Codex | 日常开工 |

### 7.2 Claude Code 对话内常见命令

| 命令 | 用途 |
|------|------|
| `/trellis:start` | 开工时可直接原样输入 |
| `/trellis:brainstorm` | 梳理需求 |
| `/trellis:finish-work` | 完工检查；**时机：测试后、提交前** |
| `/trellis:record-session` | 记录本次工作；**时机：人类测试并提交后** |
| `/trellis:break-loop` | 复盘问题根因 |
| `/trellis:update-spec` | 把经验沉淀进规范 |
| `/trellis:check` | 规则与质量检查 |

### 7.3 Codex 常用自然语言替代表达

| 目的 | 建议说法 |
|------|----------|
| 开工 | `继续当前项目，先核对 AGENTS.md、当前任务和最新 journal，再开始做功能。` |
| 梳理需求 | `先帮我把需求拆清楚，再给我最小可行方案。` |
| 完工检查 | `帮我做完工检查。按 finish-work 的顺序检查：代码质量、测试、spec 是否需要更新、有没有跨层影响。` |
| 记录日志 | `我已经人工测试并提交代码了。把这次工作记录到 journal 日志里；如果任务已完成就归档。` |
| 沉淀规范 | `把这次经验写进 spec 规范，并更新 AGENTS.md 中需要长期保留的项目决策。` |
| 防串仓库续工 | `只采用能被当前仓库文件印证的信息。` |

### 7.4 Git 最小速查卡

下面这些是最常用、最适合直接照抄的 Git 命令：

#### 1）查看当前改动

```bash
git status
```

作用：查看哪些文件改了、哪些文件已暂存、当前分支是否干净。

#### 2）查看具体改动内容

```bash
git diff
```

作用：查看尚未提交的具体改动。

#### 3）把某个文件加入提交列表

```bash
git add "文件路径"
```

例如：

```bash
git add "Trellis-使用手册.md"
```

作用：把指定文件加入本次提交。

#### 4）正式提交本次改动

```bash
git commit -m "这里写这次改了什么"
```

例如：

```bash
git commit -m "更新 Trellis 使用手册中的开工收工续工指令"
```

作用：生成一次提交记录。

#### 5）查看最近提交历史

```bash
git log --oneline -10
```

作用：查看最近 10 次提交，适合参考仓库现有提交风格。

#### 6）拉取远程最新代码

```bash
git pull
```

作用：同步远程仓库最新内容。建议先运行 `git status`，确认没有未处理的本地改动。

#### 7）把本地提交推送到远程

```bash
git push
```

作用：把本地已经 commit 的内容推送到远程仓库。

#### 常用组合模板

最常用的一组通常是：

```bash
git status
git add "文件路径"
git commit -m "这里写这次改了什么"
```

例如：

```bash
git status
git add "Trellis-使用手册.md"
git commit -m "更新 Trellis 使用手册中的开工收工续工指令"
```

### 7.5 本机脚本提醒

[本机特例] 在这台机器上，手动调用 `.trellis/scripts/*.py` 时优先用：

```bash
python "./.trellis/scripts/get_context.py" --mode record
```

而不是默认：

```bash
python3 "./.trellis/scripts/get_context.py" --mode record
```

---

## 8. FAQ 与边界说明

### Q1：`trellis init` 能不能当作每天启动命令？

不能。

- [通用] 它是初始化命令
- [通用] 日常开工应直接启动 `claude` 或 `codex`

### Q2：Claude 和 Codex 是不是一定共享所有记忆？

不是。

- [已验证] 当前仓库共享的是 `AGENTS.md`、`.trellis/`、项目内 `.codex/`、`.agents/skills/`
- [通用] Claude 私有 memory 不会自动变成 Codex 可见内容

### Q3：如果我清空了 Claude 对话怎么办？

[通用] 可以重新使用 `/trellis:start`。

但更稳的做法仍是：

1. 先核对 `AGENTS.md`
2. 再看 `.trellis/tasks/`
3. 再看 `.trellis/workspace/`
4. 最后按需补读当前仓库里实际存在的 spec / workflow / 交接文档

### Q4：项目里已经有 `AGENTS.md`，还要不要写交接文档？

要看需求。

- [通用] `AGENTS.md` 适合放首页摘要
- [通用] 交接文档适合放某一阶段的大背景、时间线和材料清单
- [原则] 交接文档不是必需品，但一旦使用，仍要先确认它真实存在于当前项目

### Q5：当前仓库有没有后端 / 前端规范？

[已验证] 有，且已经在 `.trellis/spec/backend/` 与 `.trellis/spec/frontend/` 下落地。

### Q6：当前仓库有没有 Codex 项目配置？

[已验证] 有，项目根目录下存在 `.codex/`。

### Q7：当前仓库有没有共享技能目录？

[已验证] 有，项目根目录下存在 `.agents/skills/`。

### Q8：能不能只靠 memory 续工？

不能。

因为 memory 不是仓库事实源。正确顺序永远是：

1. 当前仓库长期入口文件
2. 当前任务
3. 当前 journal
4. 当前仓库里实际存在的辅助材料
5. 最后才是 memory

### Q9：`/trellis:record-session` 能不能在 `git commit` 之前执行？

不能。

- [已验证] 当前仓库的 `record-session` 命令定义明确写着：**only be used AFTER the human has tested and committed the code**
- [结论] 正确顺序是：`写代码 / 测试 → /trellis:finish-work → 人类 git commit → /trellis:record-session`
- [提醒] 第 5.3 节里的“收工顺序”是摘要；一旦涉及前置条件，应该以命令原文为准

### Q10：`git commit -m "你的提交说明"` 里的“提交说明”应该写什么？

写你这次改动的一句话总结，不要原样写“你的提交说明”。

例如：

```bash
git commit -m "更新 Trellis 使用手册中的开工收工续工指令"
```

如果不会写英文提交说明，直接写中文也完全可以。

---

## 附录 A：本机环境特例

这些内容只说明**这台机器**，别拿去硬套别的环境。

### A.1 当前机器的已知前提

[本机特例] 当前使用环境为：

- Windows
- Git Bash
- Python 手动调用优先使用 `python`

### A.2 建议的环境探测命令

新会话刚开始，先跑：

```bash
pwd
python --version
git status --short
```

用途：

- `pwd`：确认真实工作目录
- `python --version`：确认解释器可用性
- `git status --short`：确认仓库状态

### A.3 为什么这条要单列

因为它反映的是**这台机器的踩坑经验**，不是 Trellis 官方语义。

---

## 附录 B：Trellis 常见但不应默认脑补的内容

下面这些在很多 Trellis 项目里都可能出现，但**如果当前仓库没证据，就别写死**。

### B.1 常见但需谨慎表述的说法

- “启动后一定自动注入全部上下文”
- “Spec 每次都会自动完整加载”
- “Codex 已全局开启 hooks”
- “所有项目都一定有交接文档”
- “所有 Trellis 项目都会自动并行开发、自动建 PR”

### B.2 正确写法模板

错误写法：

- `当前项目已配好 Codex 全局 hooks`
- `Trellis 会自动注入所有规范`
- `AGENTS.md 是所有项目第一信源`

更好的写法：

- [已验证] 当前仓库存在项目级 `.codex/` 配置文件。
- [通用] Codex 是否启用某些全局能力，还取决于本机用户目录配置。
- [通用] `AGENTS.md` 常被用作项目共享入口；当前仓库里它也确实存在。

### B.3 本手册的底线

**只把看见的写成事实，只把常识写成通用，只把本机经验写成本机特例。**

