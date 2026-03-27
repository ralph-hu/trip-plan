# Trip Plan 项目迁移说明书

## 前置条件

1. 新电脑已安装 [Claude Code](https://claude.ai/code)（`npm install -g @anthropic-ai/claude-code`）
2. Claude Code 已登录（运行 `claude` 后按提示登录）
3. 已安装 Git

## 迁移步骤

### Step 1：克隆项目

```bash
cd ~/Desktop
git clone https://github.com/ralph-hu/trip-plan.git trip_plan
cd trip_plan
```

### Step 2：恢复 Claude Code Memory

Memory 让 Claude 记住你的偏好和项目状态。路径格式为 `~/.claude/projects/<项目路径编码>/memory/`。

```bash
# 先启动一次 claude 让它自动创建项目目录（在 trip_plan 目录下）
cd ~/Desktop/trip_plan
claude --print "hello" 2>/dev/null || true

# 找到 Claude 为这个项目创建的目录（路径中 / 被替换成 -）
# 例如 macOS: ~/.claude/projects/-Users-<用户名>-Desktop-trip_plan/
# 需要根据实际路径调整，核心规则是把项目绝对路径的 / 换成 -

PROJECT_MEMORY_DIR=$(find ~/.claude/projects -type d -name "memory" 2>/dev/null | head -1)

# 如果上面没找到，手动创建（替换 <用户名>）：
# mkdir -p ~/.claude/projects/-Users-<用户名>-Desktop-trip_plan/memory

# 复制 memory 文件
cp .claude-backup/memory/* "$PROJECT_MEMORY_DIR/"
```

**验证：**
```bash
ls "$PROJECT_MEMORY_DIR/"
# 应该看到: MEMORY.md  feedback_sop_approach.md  project_trip_status.md  user_profile.md
```

### Step 3：恢复 Skills

```bash
# 复制 skills
cp -r .claude-backup/skills/* ~/.claude/skills/

# 复制 agents
mkdir -p ~/.claude/agents
cp .claude-backup/agents/* ~/.claude/agents/
```

**验证：**
```bash
ls ~/.claude/skills/rms-gpu-query/SKILL.md
ls ~/.claude/agents/skills-generator.md
```

### Step 4：恢复插件配置（可选）

如果需要恢复之前启用的插件（Context7、Skill Creator、Telegram）：

```bash
cp .claude-backup/settings.json ~/.claude/settings.json
```

> 注意：Telegram 插件需要重新配置 bot token 和 pairing。
> 其他 marketplace 插件会在首次使用时自动下载。

### Step 5：验证

```bash
cd ~/Desktop/trip_plan
claude
```

进入 Claude Code 后测试：
- 输入 "你还记得我是谁吗？" → 应该能读到 memory 里的用户信息
- 输入 "项目当前状态是什么？" → 应该能读到预订状态
- 打开 `index.html` 检查旅行攻略页面正常显示

## 目录结构说明

```
.claude-backup/
├── SETUP.md              ← 本说明书
├── settings.json         ← Claude Code 全局设置（启用的插件）
├── memory/               ← 项目记忆（用户画像、偏好、项目状态）
│   ├── MEMORY.md         ← 记忆索引
│   ├── user_profile.md   ← 用户信息
│   ├── feedback_sop_approach.md  ← 工作偏好
│   └── project_trip_status.md    ← 预订进度
├── skills/               ← 自定义技能
│   └── rms-gpu-query/    ← GPU 查询技能
└── agents/               ← 自定义 agent
    └── skills-generator.md
```

## 注意事项

- `CLAUDE.md` 在项目根目录，随 git 一起走，不需要额外处理
- `.claude/settings.local.json`（项目级权限白名单）也在 git 里
- Memory 中的路径引用是相对的，换电脑后仍然有效
- 如果新电脑用户名不同，memory 目录的父级路径会变，按 Step 2 操作即可
