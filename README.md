# conversation-namer

一个轻量的 Codex Skill：根据当前对话的主要任务目标，生成规范中文标题，并直接更新当前会话名称。

在对话中输入 `$conversation-namer` 即可调用。调用后无需再次确认；已有标题（包括人工标题）可以被替换。

## 命名效果

默认格式：

```text
MM-DD｜类型｜主题
```

示例中的日期假设为会话真实创建日期：

| 主要任务 | 标题示例 |
|---|---|
| 排查并修复 Tab 滑动回弹 | `10-10｜修复｜Tab 滑动回弹` |
| 优化弹窗关闭动画 | `10-10｜优化｜弹窗关闭动画` |
| 开发在线客服 | `10-10｜功能｜在线客服` |
| Review 分类模块代码 | `10-10｜评审｜分类模块` |
| 研究 Taro 小程序架构方案 | `10-10｜设计｜Taro 小程序架构` |

## 使用环境

本 Skill 面向能够使用以下 Codex 桌面端工具的会话：

- `read_thread`：读取线程元数据，包括真实创建时间和原标题。
- `set_thread_title`：更新当前会话标题。

实际工具名通常为 `mcp__codex_app__read_thread` 和 `mcp__codex_app__set_thread_title`。查询元数据时，需要可信的当前线程身份；Skill 优先使用上下文中的线程元数据，必要时读取 `CODEX_THREAD_ID`。

无需配置独立模型 API 或安装脚本依赖。Codex CLI 或其他环境如果没有这些桌面端工具，就无法完成直接改名。

## 安装

将本仓库的整个文件夹放到用户级 Skills 目录：

```text
~/.agents/skills/conversation-namer/
├── SKILL.md
├── README.md
└── LICENSE
```

GitHub 仓库发布后，也可以使用 Git 安装。将 `YOUR_USERNAME` 替换为仓库所属的 GitHub 用户名或组织名；目标目录应尚不存在：

```bash
git clone https://github.com/YOUR_USERNAME/conversation-namer.git \
  ~/.agents/skills/conversation-namer
```

安装后在 Codex 中查找 `conversation-namer`。如果没有出现，重启 Codex 后再试。

## 调用

在需要命名的当前会话中输入：

```text
$conversation-namer
```

成功后会收到类似回复：

```text
已改名为：10-10｜修复｜Tab 滑动回弹
```

也可以在调用时附加明确的命名要求，例如：

```text
$conversation-namer 使用 YY-MM-DD 日期格式，时区为 Asia/Shanghai
```

要求只作用于这次调用。修改默认规则时，直接编辑 `SKILL.md`；本 Skill 没有独立配置文件或自定义子命令解析器。

## 命名规则

- 日期取会话实际创建时间，默认时区为 `Asia/Shanghai`。不会用调用当天日期或最后更新时间替代。
- 类型为功能、修复、优化、重构、排查、设计、评审、测试、发布之一，以用户已确认的主要目标为准。
- 主题使用简洁中文，保留业务名词和技术名词，长度为 4–20 个 Unicode 字符，推荐 6–14 个。
- 默认不包含项目名；用户明确要求时，可从可信项目元数据获取。
- 避免泛化主题、重复信息、换行、控制字符和敏感信息。
- 主要目标发生明确变化后，再次调用可更新标题；运行测试等临时操作不会单独改变任务类型。

完整规则见 [SKILL.md](SKILL.md)。

## 处理范围与失败行为

每次调用只处理当前会话。没有后台自动命名、生命周期 Hooks、历史批量整理或回滚功能；调用本身表示允许更新当前标题。

- 标题已经与候选一致：保留原标题，不重复写入。
- 任务目标不够清楚：保留原标题并说明内容不足。
- 无法取得真实创建时间：给出不含日期的建议并说明原因，此次不改名。
- 改名工具不可用或调用失败：说明实际情况，不把建议标题当作已更新结果。

安装或编辑 Skill 本身不会触发改名。

## 许可证

本项目采用 [MIT License](LICENSE)。
