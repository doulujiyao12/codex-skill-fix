---
name: skill-fix
description: "诊断并修复 Windows 上 Codex 自定义 Skill 已复制但不显示、无法触发或加载失败的问题。用于检查 UTF-8 BOM、YAML Frontmatter、agents/openai.yaml 注册元数据、安装路径、目录名、缓存和客户端重载。"
---

# Skill Fix

## 目标

定位并修复 Windows Codex 自定义 Skill 的发现与加载故障。不要把“目录存在”直接判断为“安装成功”；必须完成字节、格式、路径、注册和重载验证。

## 排查流程

### 1. 确认实际 Codex Home

检查 `CODEX_HOME`；若未设置，默认使用 `%USERPROFILE%\.codex`。目标文件必须位于：

```text
%CODEX_HOME%\skills\<skill-name>\SKILL.md
```

### 2. 检查 UTF-8 BOM

读取 `SKILL.md` 的前三个字节：

- `EF BB BF`：含 UTF-8 BOM，必须去除。
- `2D 2D 2D`：直接以 `---` 开始，符合要求。

PowerShell 检查：

```powershell
$bytes = [IO.File]::ReadAllBytes($skillPath)
($bytes[0..7] | ForEach-Object { $_.ToString('X2') }) -join ' '
```

用 UTF-8 without BOM 重写：

```powershell
$text = [IO.File]::ReadAllText($skillPath, [Text.Encoding]::UTF8)
[IO.File]::WriteAllText($skillPath, $text, [Text.UTF8Encoding]::new($false))
```

写入前保留备份或确认文件受 Git 管理，修改后再次检查首字节。

### 3. 校验 Frontmatter

第一行必须严格为 `---`，之前不能有 BOM、空格或空行：

```yaml
---
name: example-skill
description: "说明这个 Skill 做什么，以及何时应该使用。"
---
```

- 文件夹名与 `name` 必须一致。
- `name` 只使用小写字母、数字和连字符，长度不超过 64。
- 优先只保留 `name` 和 `description`。
- `description` 使用双引号，尤其是包含冒号或其他 YAML 特殊字符时。

### 4. 检查 UI 注册元数据

确认 `<skill-name>\agents\openai.yaml` 存在：

```yaml
interface:
  display_name: "Example Skill"
  short_description: "简短说明"
  default_prompt: "使用 $example-skill 完成任务。"
```

检查该文件为有效 UTF-8，且 `$skill-name` 与 `SKILL.md` 中的 `name` 一致。

### 5. 检查主文件规模

Codex 建议 `SKILL.md` 控制在 500 行以内。过长时保留核心流程和资源导航，将细节移动到 `references/`。不要未经说明删减关键流程。

### 6. 重载并验证

1. 完全退出 Codex，并在任务管理器确认相关客户端进程结束。
2. 重新启动并新建任务；旧任务通常保留创建时的 Skill 快照。
3. 输入 `$` 检查列表，或输入 `使用 $<skill-name> ...`。
4. 不要假定存在 `codex skill list`；先运行 `codex --help` 确认当前版本支持的命令。

### 7. 最后处理缓存

只有编码、Frontmatter、路径、注册元数据和完全重启均正确后，才考虑缓存：

- 先定位当前版本实际使用的缓存目录。
- 不要把 Plugin 缓存与 Skill 发现缓存混为一谈。
- 删除缓存前说明影响并请求批准。
- 不要删除登录凭据、会话数据库或未知状态文件。

## 验收清单

- 路径与实际 `CODEX_HOME` 一致；
- `SKILL.md` 首字节不是 `EF BB BF`；
- 第一行严格为 `---`；
- Frontmatter 可解析，名称与目录一致；
- `agents/openai.yaml` 存在且正确；
- 客户端完全重启并在新任务测试；
- Skill 能通过 `$skill-name` 显式触发。

报告时分别说明“文件已安装”“格式验证通过”和“客户端已发现”，不要合并为一个结论。
