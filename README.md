# 龙虾成长教练（Claw Growth Coach）

`龙虾成长教练（Claw Growth Coach）` 是一个面向 Codex / AI 助手环境的成长教练 skill。  
它的目标不是提供泛泛的 motivational content，而是把用户的模糊成长诉求压缩为一个可运行的成长系统：有初始化、有记忆、有行动闭环、有定时 check-in，也有复盘和持续校准。

这个仓库是一个 **skill 包**，不是 Rust 项目，也不涉及 Final Fantasy XIV、游戏日志解析、SQLite 仪表盘或 REST API。

---

## 核心定位

这个 skill 主要帮助用户完成以下事情：

- 澄清长期目标与当前重点
- 把抽象成长诉求转为可执行闭环
- 诊断卡点发生在输入、理解、行动、反馈、复盘还是身份整合
- 在拖延、分散、低能量、低专注时快速恢复行动
- 改善学习迁移、决策质量与心流进入条件
- 通过长期记忆建立持续性，而不是每次从零开始

它适合这类请求：

- “帮我搭一个成长系统”
- “我知道很多，但总做不出来”
- “我想让 AI 变成我的长期成长教练”
- “我需要带记忆、会追踪我状态的教练”
- “我想定时被提醒、被拉回到主线目标”

---

## 主要能力

### 1. 初始化建档

第一次使用时，skill 会优先进行初始化，而不是直接进入泛化建议模式。

初始化会建立用户成长基线，收集例如：

- 长期目标 `north_star`
- 当前重点 `current_focus`
- 核心瓶颈 `core_bottleneck`
- 优势 `strengths`
- 反复失败模式 `failure_patterns`
- 重点学习领域 `learning_domains`
- 时间预算 `time_budget`
- 精力节律 `energy_rhythm`
- 偏好语言 `preferred_language`
- 教练风格偏好 `support_style`
- check-in 偏好 `check_in_style`
- 不可突破约束 `non_negotiables`
- 30 天成功定义 `definition_of_success`

初始化后会先总结给用户确认，确认后才写入记忆。

### 2. 长期记忆

skill 使用全局记忆目录中的专用记忆文件维持连续性。

默认读取：

- `PROFILE.md`
- `ACTIVE.md`
- `CLAW_GROWTH_COACH.md`

其中 `CLAW_GROWTH_COACH.md` 记录该 skill 的长期上下文，例如：

- 长期目标
- 当前重点
- 当前项目
- 重复性失败模式
- 时间与精力约束
- 偏好语言
- 定时 check-in 配置

### 3. 语言自适应

默认输出语言取决于用户和 skill 的第一次有意义对话语言。

规则如下：

- 如果记忆里已有 `preferred_language`，优先使用
- 如果还没有记忆，就根据用户第一次有意义回复自动推断
- 如果 skill 必须先开口而用户还没回复过，默认使用英语
- 如果用户后续明确要求切换语言，skill 会更新记忆

### 4. 成长闭环诊断

skill 会把用户成长问题放进一个标准闭环里分析：

`输入 -> 结构化理解 -> 小实践 -> 真实尝试 -> 反馈 -> 反思 -> 二次尝试 -> 身份强化`

它会帮助识别当前坏掉的是哪一环，例如：

- 输入过载
- 理解不成结构
- 没有行动设计
- 回避真实尝试
- 反馈过慢
- 没有复盘
- 没有二次迭代
- 没有身份升级

### 5. 行动压缩

它会把大目标压缩为低阻力下一步，例如：

- 今天最小动作
- 当前最重要动作
- 下一小时动作
- 适合低能量状态的动作

### 6. 心流恢复

当用户无法专注时，skill 会判断问题更接近哪一类：

- 任务太难
- 任务太简单
- 目标不清楚
- 反馈太慢
- 精力太低
- 干扰太多

然后重新设计任务颗粒度与执行方式，让用户更容易回到投入状态。

### 7. 决策辅助

在用户面临关键选择时，skill 会帮助：

- 拓宽选项
- 做现实检验
- 分离短期情绪与长期价值
- 设计低风险实验

### 8. 定时自唤起

这个 skill 既支持 Codex 的 thread heartbeat，也支持本地 Python 定时器。

默认节律是：

- 每小时一次
- 每天 `08:00-22:00`

定时唤起时，它会优先做轻量 check-in，而不是完整深聊。

---

## 仓库结构

当前仓库的关键文件如下：

```text
claw-growth-coach/
├── README.md
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    └── memory-schema.md
```

各文件职责：

- `SKILL.md`
  skill 的主定义文件，包含触发描述、初始化逻辑、记忆规则、定时唤起规则、主要功能与守则。
- `agents/openai.yaml`
  UI 显示名、简述与默认 prompt。
- `references/memory-schema.md`
  记忆文件的建议结构与更新规则。
- `README.md`
  当前这个说明文件，用于帮助人类理解这个 skill 做什么、不做什么。

---

## 定时唤起方式

### 1. App 原生 heartbeat

如果你在 Codex 桌面环境中使用，可以通过 thread heartbeat 定期唤起当前对话。

这适合：

- 同一线程内持续 coaching
- 不想自己维护本地守护进程
- 希望使用平台原生自动化

### 2. 本地 Python 调度器

如果你希望在 heartbeat 之外再有一层本地调度，可以使用外部 Python 定时器。

当前已实现的本地文件包括：

- `claw_growth_coach_scheduler.py`
- `start-claw-growth-coach-scheduler.bat`

这个调度器会：

- 从本地自动化配置中读取 `target_thread_id`
- 在每天 `08:00-22:00` 之间每小时发送一次 prompt
- 使用 `codex exec resume <thread_id> <prompt>` 把 check-in 发回当前线程

如果没有自动化文件，也可以通过环境变量手动指定线程：

- `CLAW_GROWTH_COACH_THREAD_ID`

---

## 记忆文件

skill 依赖的核心记忆文件是：

- `${CODEX_HOME:-~/.codex}/memories/CLAW_GROWTH_COACH.md`

这个文件是 skill 的长期记忆核心，通常包含：

- Initialization Status
- North Star
- Current Focus
- Current Project
- Main Bottlenecks
- Strengths
- Failure Patterns
- Learning Domains
- Time Budget
- Energy Rhythm
- Preferred Language
- Preferred Coaching Style
- Check-In Preferences
- Recurring Cadence
- Constraints
- Success Definition
- Active Experiments
- Reusable Notes

---

## 当前项目真实状态

为了避免再次出现 README 与实际实现不一致，这里明确写清楚当前项目状态：

- 这是一个 **skill 项目**
- 主逻辑定义在 `SKILL.md`
- 有初始化问答
- 有长期记忆
- 有语言记忆
- 有定时自唤起规则
- 有 heartbeat 版本调度
- 有本地 Python 版本调度

当前项目 **不包含**：

- Rust 代码
- Cargo.toml
- FFXIV 相关功能
- 游戏日志解析
- SQLite 分析数据库
- Warp Web 服务器
- HTML 仪表盘
- REST API

---

## 后续建议

如果你继续完善这个 skill，下一步最值得做的是：

1. 给 `SKILL.md` 增加更强的中文默认输出规则
2. 增加标准化的初始化提问批次模板
3. 增加“每日启动 / 中途纠偏 / 晚间复盘”三套 check-in prompt
4. 给 Python 调度器增加日志输出与 Windows 计划任务安装脚本
5. 把记忆更新规则再做得更谨慎，避免写入短期噪音

---

## 许可证

如果你要把这个 skill 单独开源或长期维护，建议后续补一个明确的 LICENSE 文件。  
当前仓库里如果没有 LICENSE，这个 README 不再假定使用 MIT 或其他许可证。
