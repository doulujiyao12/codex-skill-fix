# Skill Fix

一个用于诊断和修复 Windows 上 Codex 自定义 Skill 无法显示、无法触发或加载失败问题的 Skill。

## 覆盖的问题

- `SKILL.md` 使用 UTF-8 BOM，导致 YAML Frontmatter 无法识别
- Frontmatter 首行、字段或 YAML 引号格式错误
- Skill 文件夹名称与 `name` 不一致
- 缺少 `agents/openai.yaml` UI 注册元数据
- Skill 安装到了错误的 `CODEX_HOME`
- Codex 客户端或现有任务没有重新加载 Skill
- 错误地删除 Plugin 缓存或其他 Codex 状态文件

## Windows 安装

```powershell
git clone https://github.com/doulujiyao12/codex-skill-fix.git "$env:USERPROFILE\.codex\skills\skill-fix"
```

安装后完全退出并重新启动 Codex，然后新建任务。

## 使用

在 Codex 中输入：

```text
使用 $skill-fix 检查这个 Skill 为什么没有显示。
```

## 关键验证

`SKILL.md` 必须以十六进制字节 `2D 2D 2D` 开始，而不是带 BOM 的 `EF BB BF`。
