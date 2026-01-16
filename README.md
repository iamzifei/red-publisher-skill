# Xiaohongshu Publisher Skill (小红书发布器)

<p align="center">
  <strong>
    <a href="#english">English</a> | <a href="#中文">中文</a>
  </strong>
</p>

---

<a name="english"></a>

## English

> Publish images and notes to Xiaohongshu (小红书) with one command. Uses CDP mode to connect to your existing browser session.

**v3.0.0** — Now using agent-browser with CDP mode for seamless browser automation

### The Problem

Publishing to Xiaohongshu manually is tedious:

| Pain Point | Description |
|------------|-------------|
| **Login Hassle** | Must scan QR code every session |
| **Multiple Images** | Upload images one by one |
| **Content Formatting** | Copy-paste text, add tags manually |
| **Time Consuming** | 5-10 minutes per post |

#### Time Comparison

| Task | Manual | With This Skill |
|------|--------|-----------------|
| Login | 30 sec - 1 min | Login once in browser, reuse session |
| Image upload (5 images) | 2-3 min | 30 sec |
| Title & content | 1-2 min | 10 sec |
| **Total** | **5-10 min** | **1-2 min** |

**5x efficiency improvement**

### The Solution

This skill uses **CDP (Chrome DevTools Protocol) mode** to connect to your existing browser:

```
1. Start Chrome with debug port
2. Login to Xiaohongshu once in browser
3. AI connects to your browser session
4. Automates publishing while you stay logged in
```

```
Images/Markdown File
     ↓ Python parsing
Structured Data (title, content, images, tags)
     ↓ agent-browser --cdp 9222
Your Chrome Browser (already logged in)
     ↓
Draft/Published Note
```

### Key Features

- **🔌 CDP Mode**: Connect to your existing browser via `--cdp 9222` flag
- **🔐 Session Persistence**: Login once in Chrome, reuse session indefinitely
- **📤 Multi-Image Upload**: Upload up to 18 images at once
- **📝 Content Parsing**: Extract title, content, and tags from Markdown
- **✅ Safe by Default**: Saves as draft unless you specify publish
- **🛠️ agent-browser CLI**: Simple command-line interface, no MCP server needed

### What's New in v3.0.0

| Feature | v2.x | v3.0 |
|---------|------|------|
| Connection mode | New browser each time | CDP (connect to existing) |
| Login handling | QR code detection + auth files | Login once in your browser |
| Session management | Auth state files | Browser's own session |
| Command format | `npx agent-browser open ...` | `npx agent-browser --cdp 9222 open ...` |

### Requirements

| Requirement | Details |
|-------------|---------|
| Claude Code | [claude.ai/code](https://claude.ai/code) |
| Chrome | Launch with `--remote-debugging-port=9222` |
| Python 3.9+ | With dependencies below |
| macOS | Currently macOS only |

```bash
# Install Python dependencies
pip install Pillow pyobjc-framework-Cocoa
```

### Pre-requisite: Launch Chrome with Debug Port

Before using this skill, start Chrome with remote debugging:

```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222
```

**Tip**: Add an alias to your shell config (`~/.zshrc` or `~/.bashrc`):
```bash
alias chrome-debug='/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222'
```

Then login to Xiaohongshu once: https://creator.xiaohongshu.com/publish/publish

**Tip**: After login, you can minimize the browser to the Dock. It must stay running, but you don't need to see it.

### Installation

#### Method 1: Git Clone (Recommended)

```bash
git clone https://github.com/wshuyi/xiaohongshu-publisher-skill.git
cp -r xiaohongshu-publisher-skill/skills/xiaohongshu-publisher ~/.claude/skills/
```

#### Method 2: Plugin Marketplace

```
/plugin marketplace add wshuyi/xiaohongshu-publisher-skill
/plugin install xiaohongshu-publisher@wshuyi/xiaohongshu-publisher-skill
```

### Usage

#### Natural Language

```
发布这些图片到小红书: /path/to/photo1.jpg, /path/to/photo2.jpg
标题是"周末探店"
```

```
Publish /path/to/note.md to Xiaohongshu
```

```
帮我把这篇笔记发到小红书，存草稿就行
```

#### Skill Command

```
/xiaohongshu-publisher /path/to/note.md
```

### Workflow Steps

```
[1/5] Verify browser connection...
      → Connect to Chrome via CDP (port 9222)
      → Check if already logged in

[2/5] Parse content...
      → Extract title, content, images, tags

[3/5] Upload images...
      → Upload all images (1-18 supported)

[4/5] Fill title and content...
      → Add title, description, tags

[5/5] Save draft...
      → ✅ Review and publish manually
      → (Or publish directly if requested)
```

### Multi-Account Support

With CDP mode, account switching is handled by Chrome profiles:

1. **Use Chrome profiles** - Create different Chrome profiles for different accounts
2. **Start Chrome with profile** - Launch Chrome with the desired profile before connecting
3. **Or switch accounts in browser** - Login/logout in your browser, then use the skill

### Content Formats

#### From Images + Text

```
发布这些图片到小红书:
- /path/to/photo1.jpg
- /path/to/photo2.jpg
- /path/to/photo3.jpg

标题: 周末好去处
内容: 发现了一家超赞的咖啡店...
标签: 咖啡, 探店, 周末
```

#### From Markdown File

```markdown
# 周末好去处

![](./images/photo1.jpg)
![](./images/photo2.jpg)

发现了一家超赞的咖啡店，环境特别好！

推荐指数：⭐⭐⭐⭐⭐

#咖啡 #探店 #周末
```

### Limits & Best Practices

| Item | Limit |
|------|-------|
| Images per note | 1-18 |
| Title length | ~20 characters recommended |
| Content length | ~1000 characters max |
| Tags | Up to 5 recommended |
| Image formats | JPG, PNG, GIF, WebP |

#### Tips

1. **Prepare images first** - Have all images ready before running
2. **Keep Xiaohongshu app handy** - For quick QR code scanning
3. **Use draft mode** - Review before publishing
4. **Compress large images** - Faster uploads

### FAQ

**Q: Why CDP mode?**
A: CDP mode connects to your existing browser session. Benefits:
- No separate browser window
- Reuse your existing login
- Session persists as long as browser is open
- Same agent-browser commands, just add `--cdp 9222`

**Q: Browser not connecting?**
A: Make sure Chrome is running with debug port:
```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222
```

**Q: Session expired?**
A: Just re-login in your Chrome browser. The AI will use your active session.

**Q: Windows/Linux support?**
A: Currently macOS only. PRs welcome for cross-platform support.

**Q: Image upload failed?**
A: Check: valid path, supported format (jpg/png/gif/webp), file size within limits.

**Q: Can I publish directly instead of draft?**
A: Yes, specify "直接发布" or "publish now" in your request.

### Project Structure

```
xiaohongshu-publisher-skill/
├── .claude-plugin/
│   └── plugin.json              # Plugin config
├── skills/
│   └── xiaohongshu-publisher/
│       ├── SKILL.md             # Skill instructions
│       └── scripts/
│           ├── parse_note.py    # Content parser
│           └── copy_to_clipboard.py
├── docs/
│   └── GUIDE.md                 # Detailed guide (中文)
├── README.md                    # This file (bilingual)
└── LICENSE
```

---

<a name="中文"></a>

## 中文

> 一键发布图片笔记到小红书。使用 CDP 模式连接到您已有的浏览器会话。

**v3.0.0** — 使用 agent-browser + CDP 模式实现无缝浏览器自动化

### 痛点

手动发布小红书笔记太繁琐：

| 痛点 | 描述 |
|------|------|
| **登录麻烦** | 每次都要扫码登录 |
| **多图上传** | 一张一张上传图片 |
| **内容格式** | 复制粘贴文字，手动加标签 |
| **耗时长** | 每篇笔记 5-10 分钟 |

#### 时间对比

| 任务 | 手动 | 使用本技能 |
|------|------|-----------|
| 登录 | 30秒 - 1分钟 | 浏览器登录一次，复用会话 |
| 上传5张图片 | 2-3 分钟 | 30 秒 |
| 填写标题内容 | 1-2 分钟 | 10 秒 |
| **总计** | **5-10 分钟** | **1-2 分钟** |

**效率提升 5 倍**

### 解决方案

本技能使用 **CDP (Chrome DevTools Protocol) 模式** 连接到您已有的浏览器：

```
1. 启动带调试端口的 Chrome
2. 在浏览器中登录小红书（只需一次）
3. AI 连接到您的浏览器会话
4. 自动化发布，保持登录状态
```

```
图片/Markdown 文件
     ↓ Python 解析
结构化数据 (标题, 内容, 图片, 标签)
     ↓ agent-browser --cdp 9222
您的 Chrome 浏览器 (已登录)
     ↓
草稿/已发布笔记
```

### 核心功能

- **🔌 CDP 模式**：通过 `--cdp 9222` 参数连接到您已有的浏览器
- **🔐 会话持久化**：浏览器登录一次，会话无限复用
- **📤 多图上传**：一次上传最多 18 张图片
- **📝 内容解析**：从 Markdown 提取标题、内容、标签
- **✅ 默认存草稿**：不会自动发布，除非明确指定
- **🛠️ agent-browser CLI**：简单的命令行界面，无需 MCP 服务器

### v3.0.0 更新内容

| 功能 | v2.x | v3.0 |
|------|------|------|
| 连接模式 | 每次新开浏览器 | CDP (连接已有浏览器) |
| 登录处理 | 二维码检测 + auth 文件 | 浏览器登录一次即可 |
| 会话管理 | Auth 状态文件 | 浏览器自带会话 |
| 命令格式 | `npx agent-browser open ...` | `npx agent-browser --cdp 9222 open ...` |

### 环境要求

| 需求 | 详情 |
|------|------|
| Claude Code | [claude.ai/code](https://claude.ai/code) |
| Chrome | 启动时加 `--remote-debugging-port=9222` |
| Python 3.9+ | 需要下列依赖 |
| macOS | 目前仅支持 macOS |

```bash
# 安装 Python 依赖
pip install Pillow pyobjc-framework-Cocoa
```

### 前置条件：启动带调试端口的 Chrome

使用本技能前，先启动带远程调试的 Chrome：

```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222
```

**小贴士**：在 shell 配置文件 (`~/.zshrc` 或 `~/.bashrc`) 中添加别名：
```bash
alias chrome-debug='/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222'
```

然后在浏览器中登录小红书：https://creator.xiaohongshu.com/publish/publish

**小贴士**：登录后可以把浏览器最小化到 Dock。浏览器需要保持运行，但不需要看到它。

### 安装

#### 方法一：Git Clone（推荐）

```bash
git clone https://github.com/wshuyi/xiaohongshu-publisher-skill.git
cp -r xiaohongshu-publisher-skill/skills/xiaohongshu-publisher ~/.claude/skills/
```

#### 方法二：插件市场

```
/plugin marketplace add wshuyi/xiaohongshu-publisher-skill
/plugin install xiaohongshu-publisher@wshuyi/xiaohongshu-publisher-skill
```

### 使用方法

#### 自然语言

```
发布这些图片到小红书: /path/to/photo1.jpg, /path/to/photo2.jpg
标题是"周末探店"
```

```
帮我把这篇笔记发到小红书，存草稿就行
```

```
把 /path/to/note.md 发布到小红书
```

#### 技能命令

```
/xiaohongshu-publisher /path/to/note.md
```

### 工作流程

```
[1/5] 验证浏览器连接...
      → 通过 CDP 连接到 Chrome（端口 9222）
      → 检查是否已登录

[2/5] 解析内容...
      → 提取标题、内容、图片、标签

[3/5] 上传图片...
      → 上传所有图片（支持 1-18 张）

[4/5] 填写标题和内容...
      → 添加标题、描述、标签

[5/5] 保存草稿...
      → ✅ 请手动检查后发布
      → （或直接发布，如果用户要求）
```

### 多账号支持

使用 CDP 模式时，账号切换通过 Chrome 用户配置文件实现：

1. **使用 Chrome 配置文件** - 为不同账号创建不同的 Chrome 配置文件
2. **指定配置文件启动** - 使用目标配置文件启动 Chrome
3. **或在浏览器中切换** - 在浏览器中登录/登出，然后使用本技能

### 内容格式

#### 直接提供图片和文字

```
发布这些图片到小红书:
- /path/to/photo1.jpg
- /path/to/photo2.jpg
- /path/to/photo3.jpg

标题: 周末好去处
内容: 发现了一家超赞的咖啡店...
标签: 咖啡, 探店, 周末
```

#### 使用 Markdown 文件

```markdown
# 周末好去处

![](./images/photo1.jpg)
![](./images/photo2.jpg)

发现了一家超赞的咖啡店，环境特别好！

推荐指数：⭐⭐⭐⭐⭐

#咖啡 #探店 #周末
```

### 限制与最佳实践

| 项目 | 限制 |
|------|------|
| 每篇图片数 | 1-18 张 |
| 标题长度 | 建议 ~20 字符 |
| 内容长度 | 最多 ~1000 字符 |
| 标签数量 | 建议最多 5 个 |
| 图片格式 | JPG, PNG, GIF, WebP |

#### 小贴士

1. **提前准备图片** - 运行前确保所有图片就绪
2. **手机准备好小红书** - 方便快速扫码
3. **使用草稿模式** - 发布前先检查
4. **压缩大图片** - 上传更快

### 常见问题

**Q: 为什么使用 CDP 模式？**
A: CDP 模式连接到您已有的浏览器会话。好处：
- 无需单独的浏览器窗口
- 复用已有的登录状态
- 只要浏览器开着，会话就一直有效
- 同样的 agent-browser 命令，只需加 `--cdp 9222`

**Q: 浏览器连接不上？**
A: 确保 Chrome 以调试端口启动：
```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222
```

**Q: 会话过期了？**
A: 直接在 Chrome 浏览器中重新登录即可。AI 会使用您当前的会话。

**Q: 支持 Windows/Linux 吗？**
A: 目前仅支持 macOS。欢迎提交 PR 支持其他平台。

**Q: 图片上传失败？**
A: 检查：路径是否正确，格式是否支持（jpg/png/gif/webp），文件大小是否超限。

**Q: 可以直接发布而不是存草稿吗？**
A: 可以，在请求中说明 "直接发布" 即可。

### 项目结构

```
xiaohongshu-publisher-skill/
├── .claude-plugin/
│   └── plugin.json              # 插件配置
├── skills/
│   └── xiaohongshu-publisher/
│       ├── SKILL.md             # 技能说明
│       └── scripts/
│           ├── parse_note.py    # 内容解析
│           └── copy_to_clipboard.py
├── docs/
│   └── GUIDE.md                 # 详细指南
├── README.md                    # 本文件（双语）
└── LICENSE
```

---

## Changelog / 更新日志

### v3.0.0 (2025-01)
- **CDP Mode / CDP 模式**: Connect to existing browser via `--cdp 9222` flag / 通过 `--cdp 9222` 参数连接到已有浏览器
- **Session Persistence / 会话持久化**: Login once in browser, reuse indefinitely / 浏览器登录一次，无限复用
- **Simpler Setup / 更简单的配置**: Just start Chrome with debug port / 只需启动带调试端口的 Chrome
- **Same CLI / 相同的命令行**: Same agent-browser commands, just add `--cdp 9222` / 相同的命令，只需加 `--cdp 9222`

### v2.0.0 (2025-01)
- **Platform switch / 平台切换**: Xiaohongshu instead of X (Twitter) / 从 X 改为小红书
- **agent-browser**: Replace Playwright MCP with agent-browser CLI / 用 agent-browser CLI 替代 Playwright MCP
- **QR code login / 二维码登录**: Detect and prompt user for login / 检测并提示用户登录
- **Multi-account / 多账号**: Support multiple accounts with easy switching / 支持多账号管理和切换
- **Image-centric / 图片笔记**: Focus on image notes rather than articles / 专注于图片笔记而非长文

### v1.1.0 (2024-12)
- Block-index positioning for X Articles
- Reverse insertion order
- Optimized wait strategy

### v1.0.0 (2024-12)
- Initial release (X Articles publisher)

---

## License / 许可证

MIT License - see [LICENSE](LICENSE)

## Author / 作者

[wshuyi](https://github.com/wshuyi)

## Contributing / 贡献

- **Issues**: Report bugs or request features / 报告问题或请求功能
- **PRs**: Welcome! Especially for Windows/Linux support / 欢迎！特别是跨平台支持
