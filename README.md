# git-delivery

给 AI agent 用的 Git 提交纪律技能：**到关键节点主动提醒提交 → 出计划等你授权 → 按项目规则原子提交 → 回报可反查的状态**。

**它管的是本地提交与版本备份。** 推送（`SKILL.md` §10）是可选能力：仅在仓库有远端、且你明确要求时才走；
无远端的仓库它只会告诉你「提交仅落本地」，不会自己加远端、也不会联网。

- 纯文本技能：不携带脚本，只调用环境里已有的 `git`，不下载、不装依赖。
- 无状态：不记录「上次执行」，每次从 git 现状重算，允许重复提醒。
- 零项目信息：提交风格、规则、验证命令全部从目标仓库现取。

## 最佳使用方案

### 方案一：日常项目（零配置，推荐起步）

1. 装进技能目录（见「安装」），重开会话。
2. 干活时说到「做完了 / 可以了 / 先这样」就够——**不必专门说「提交」**，这类说法本身就是节点信号。
3. 它会给出事实：改了几个文件 / 验证过没有 / 「要提交就说一声」，不问选择题。
4. 你说「提交」，它按目标项目的规则拆成原子提交，并回报可反查的 hash 与状态。

适合：个人项目、一次性任务、不想改项目文件。

### 方案二：长期项目 / 多 Agent 并行（做一次强绑定，推荐）

技能没有任何机制保证每轮都被加载。要让它不再依赖加载，**装完技能后多做一步**：
把「安装 → 装完之后」那段 5 行写进项目规则文件（`AGENTS.md` / `CLAUDE.md` / `CONTRIBUTING.md`）。
之后该项目**任何会话、任何 agent** 都会提醒。

适合：会持续几个月、多会话、多 Agent 的项目。

### 方案三：团队仓库

- 先让它读项目自己的规则（`SKILL.md` §3 规则源探测），**团队规范优先**，本技能只做兜底。
- 不要用 §14 覆盖团队已有规则，只补充缺失项——两处规则打架时没人裁决。

### 三条硬建议

1. **按闭环提交，不按文件数**：拆分依据是「能独立 revert」，不是「一天一次」。
2. **破坏性操作单独确认**：`reset --hard`、`clean -fd`、`push --force` 各要一次明确指令，不接受批量授权。
3. **别指望它自动盯着你**：未被加载时它不存在；长期项目请走方案二。

## 安装

### 方式零：npx（官方 CLI，最省事）

```bash
npx skills add 6jfhz5mzkm-sketch/git-delivery
```

先看它发现了什么（只读，不安装）：

```bash
npx skills add 6jfhz5mzkm-sketch/git-delivery --list
```

指定 agent、全局安装、非交互：

```bash
npx skills add 6jfhz5mzkm-sketch/git-delivery -a claude-code
npx skills add 6jfhz5mzkm-sketch/git-delivery -g -a claude-code -y
```

不安装、直接用一次：

```bash
npx skills use 6jfhz5mzkm-sketch/git-delivery@git-delivery --agent claude-code
```

说明与边界：

- 需要 **Node.js ≥ 22.20**（官方 CLI 的硬要求）。
- `skills` 是 Vercel Labs 的通用安装器（npm 包名 `skills`），支持 Claude Code、Codex、Cursor、OpenCode 等 80 多个 agent。
- 本仓库是**单技能仓库**（根目录一个 `SKILL.md`），该结构已由 CLI 源码 `discoverSkills` 确认为受支持；
  仓库名与技能名不同也没关系，安装后的技能名取自 `SKILL.md` 的 frontmatter（`git-delivery`）。
- ⚠️ **端到端未在本机实测**（本机 Node 22.19 低于要求）。若 `--list` 没有列出 `git-delivery`，
  说明 CLI 行为与源码判断有出入，请改用下面的 git clone 方式，并提 issue。

### 方式一：git clone（任何环境都能用）

**Linux / macOS**

```bash
# DSH
git clone https://github.com/6jfhz5mzkm-sketch/git-delivery.git ~/.dsh/skills/git-delivery

# Claude Code
git clone https://github.com/6jfhz5mzkm-sketch/git-delivery.git ~/.claude/skills/git-delivery
```

**Windows（PowerShell）**

```powershell
# DSH
git clone https://github.com/6jfhz5mzkm-sketch/git-delivery.git "$env:USERPROFILE\.dsh\skills\git-delivery"

# Claude Code
git clone https://github.com/6jfhz5mzkm-sketch/git-delivery.git "$env:USERPROFILE\.claude\skills\git-delivery"
```

上面的目标路径都是**完整的**，不需要再拼任何东西，复制即可执行。
其它 harness 用它自己的 skills 目录，唯一要求是 `SKILL.md` 位于 `git-delivery/` 根。

装完确认 `git-delivery/SKILL.md` 存在，然后重开会话（技能目录会被重新扫描）。
仓库为私有的话克隆需要凭据；公开仓库匿名克隆即可。

> 下文出现的 `<...>` 是占位符，**不要原样复制进命令**，按你的实际情况替换。

### 方式二：安装口令（整段复制给 agent，它自己拼路径）

**最短一版**

```
把 https://github.com/6jfhz5mzkm-sketch/git-delivery.git 克隆到你自己的技能目录下的 git-delivery 子目录，
装完确认 git-delivery/SKILL.md 在目录根，然后复述这个技能会在哪些情况下被触发。
```

**完整版（含把规则写进项目）**

```
帮我安装一个技能：
1. 先告诉我你的技能目录在哪（DSH 是 ~/.dsh/skills/，Claude Code 是 ~/.claude/skills/）
2. 把 https://github.com/6jfhz5mzkm-sketch/git-delivery.git 克隆到你刚才报的那个目录下的 git-delivery 子目录
3. 确认 git-delivery/SKILL.md 在目录根
4. 复述这个技能会在哪些情况下被触发
5. 检查当前项目规则文件（AGENTS.md / CLAUDE.md / CONTRIBUTING.md）里有没有 Git 提交规则；
   没有就把 SKILL.md §14 那 5 行加进去（合并进已有 Git 章节，不新建平行章节），动手前先把改动给我看
6. 一句话告诉我装好了
```

两版都**不含尖括号占位符**，可以直接整段发给 agent。

第 4 步是验收：如果复述里没有「提交」「推上去」「该不该重来」「reset 之后还能救吗」这类场景，
说明它没真正读到 `SKILL.md`，别信「已装好」。

### 方式三：离线安装（无网络 / 内网）

下载仓库 zip → 解压，保证 `SKILL.md` 位于 `git-delivery/` 目录根，再放进技能目录。

### 装完之后（推荐）：让项目每次都读到它

技能是「按需加载」的文本，**没有任何机制保证它每一轮都出现**。装完技能后多做一步，把它绑定到项目上：

把下面这段加进项目规则文件（`AGENTS.md` / `CLAUDE.md` / `CONTRIBUTING.md`）：

```markdown
## Git 提交节点
- 一个功能/修复/文档阶段完成，或验证命令刚通过时，先跑一次 git 节点自查并提醒，不自动提交。
- 提交前输出四段汇报（做了什么 / 改了哪些文件 / 有无漏做多做 / 怎么验证+结果），等明确授权。
- 禁止 `git add .`；逐条显式 add；提交后反查 `git log -1` 才算完成。
- 破坏性操作（reset --hard / clean -fd / push --force / branch -D）单独确认。
- 结束时回报 `STATUS:` + 证据行。
```

怎么放：

| 项目情况 | 做法 |
|---|---|
| 已有 Git 章节（如「## Git 规则」） | **合并进那一节**，不要新增平行章节——两处规则打架时没人裁决 |
| 没有任何规则文件 | 可以新建，但要意识到这是改变项目的协作约定；不建也行，只影响提醒稳定性 |
| 团队仓库 | 团队规范优先，只补缺失项，不要覆盖已有规则 |

之后这个项目**任何会话、任何 agent** 读规则文件时都会看到它 → 提醒不再依赖本技能是否被加载。
这一步由你决定做不做；技能不会自己往项目里写文件。

> **注意**：只有用「方式二：安装口令」装，装完那一刻才会被提示做这一步。
> 手动 `git clone` **不会自动提示**——需要你之后说一句「检查一下这个项目有没有 git 提交规则」，
> 或者自己把上面那段贴进去。

### 验证装对了

对 agent 说「现在该提交吗」：

- 正常：只做只读检查，给出「提交节点」提醒或「当前无改动」，**不会提交任何东西**。
- 异常：直接提交了东西，或报找不到技能。

### 卸载

删掉 `git-delivery/` 目录即可。它不写全局配置、不装依赖、不留状态文件。

## 怎么触发

说出下列任一意图即可，不需要记命令：

| 你想干什么 | 这样说 |
|---|---|
| 提交 | 「提交」「存一下」「备份当前版本」 |
| 推送（可选） | 「推上去」「上传到仓库」——**仓库没有远端时不会执行** |
| 节点 | 「做完了」「可以了」「先这样」（比「提交」更常说的节点信号） |
| 拿不准 | 「现在该提交吗」「改动这么大是不是该重来」「从哪个版本改最省事」 |
| 出事故 | 「提交错了」「误删了」「reset 之后还能救吗」 |
| 新项目 | 「新项目开工」「要建 .gitignore」 |

技能**不会自己提交**——必须你明确说「提交」。它会先给你看计划和验证结果。

## 典型对话

```
你：这个功能做完了，你看下
它：（节点自查）改动了 3 个文件（+120 / −8），验证未跑；要提交就说一声
你：提交
它：（四段汇报 + 拆分计划）拟拆 2 笔：① feat(core): … ② docs: …
你：可以
它：COMMITTED（hash / git log 反查一致 / 工作区状态 / 推送情况）
```

## 边界（重要）

- **无法保证被加载**：宿主决定是否加载，技能自身无法注册、定时、订阅事件或改宿主配置。
  未被加载的那一轮，它不存在——不要把它当成「会一直提醒」的守卫。
- **没有强制力**：它只是文本。真正的拦截靠 harness 审批、服务端保护分支、本地 hook。
- **无状态**：不写任何「上次执行」记录，每次从 git 现状重算；允许重复提醒。
- **默认离线**：`status / diff / log / add / commit / tag` 全程不联网；只有 `fetch / pull / push` 是网络操作，且需你明确要求。
- **不含任何项目信息**：提交风格、规则、验证命令全部从目标项目现取。

## 目录

```
git-delivery/
├── SKILL.md                     给 agent 看的完整流程（§0 闸门 → §15 独立性约束）
├── README.md                    给人看的这份说明
└── references/
    └── public-export.md         发布到公共托管时的通用脱敏步骤（按需读）
```

## 快速自检

装好后随便找个仓库，对 agent 说「现在该提交吗」。
正常表现：只做只读检查，输出「提交节点」提醒或「当前无改动」，**不会**直接提交任何东西。
