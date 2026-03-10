# Claude Code 配置同步

这是我的 Claude Code 配置仓库，用于多设备同步自定义命令和设置。

## 同步内容

- `commands/cc/` - 自定义命令（cc: 前缀的快捷命令）
- `.gitignore` - 忽略运行时生成的临时文件
- `settings.example.json` - 配置模板

## 使用方法

### 新设备首次同步

```bash
# 1. 克隆仓库（使用 dotfiles 方式，避免 ~/.claude 目录本身被 git 管理）
git clone --separate-git-dir=$HOME/.claude-git git@github.com:你的用户名/claude-dotfiles.git $HOME/.claude-tmp

# 2. 移动文件到 ~/.claude
rsync --recursive --verbose --exclude '.git' $HOME/.claude-tmp/ $HOME/.claude/

# 3. 清理临时目录
rm -rf $HOME/.claude-tmp

# 4. 配置 settings.json
cp ~/.claude/settings.example.json ~/.claude/settings.json
# 编辑 settings.json，填入你的 API token
```

### 或者更简单的方式：手动同步

```bash
# 1. 在其他位置克隆仓库
git clone git@github.com:你的用户名/claude-dotfiles.git ~/claude-config

# 2. 复制命令文件
cp -r ~/claude-config/commands/cc/* ~/.claude/commands/cc/

# 3. 复制配置模板
cp ~/claude-config/settings.example.json ~/.claude/settings.example.json
```

### 提交新配置

```bash
cd ~/.claude
git add commands/cc/
git commit -m "chore: 添加新命令"
~/.claude-git/git push  # 使用独立的 git 目录
```

## 自定义命令说明

| 命令 | 功能 |
|------|------|
| cc:commit | 生成 git commit 消息 |
| cc:review | 代码和业务审查 |
| cc:test | 测试所有接口 |
| cc:lint | 修复 lint 和 build 错误 |
| cc:workflow | 多子代理工作流执行 |
| cc:self-test | 自主测试 |
| cc:dr | 生成工作日报 |
| cc:ticket | 生成工单文本 |
| cc:fields | 修复前后端字段不一致 |
| cc:merge-develop | 合并当前分支到 develop |
