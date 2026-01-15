# Xiaohongshu Publisher Skill (小红书发布器)

<p align="center">
  <strong>
    <a href="#english">English</a> | <a href="#中文">中文</a>
  </strong>
</p>

---

<a name="english"></a>

## English

> Publish images and notes to Xiaohongshu (小红书) with one command. Supports QR code login, multi-account management, and multi-image uploads.

**v2.0.0** — Now using agent-browser for reliable automation

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
| Login | 30 sec - 1 min | Auto-detected, prompted |
| Image upload (5 images) | 2-3 min | 30 sec |
| Title & content | 1-2 min | 10 sec |
| **Total** | **5-10 min** | **1-2 min** |

**5x efficiency improvement**

### The Solution

This skill automates the entire publishing workflow:

```
Images/Markdown File
     ↓ Python parsing
Structured Data (title, content, images, tags)
     ↓ agent-browser CLI
Xiaohongshu Creator Platform (browser automation)
     ↓
Draft/Published Note
```

### Key Features

- **QR Code Login Handling**: Detects login page, prompts you to scan QR code
- **👥 Multi-Account Support**: Manage multiple Xiaohongshu accounts with easy switching
- **🔐 Login State Persistence**: Save auth state after first login, skip QR scan next time
- **Multi-Image Upload**: Upload up to 18 images at once
- **Content Parsing**: Extract title, content, and tags from Markdown
- **Safe by Default**: Saves as draft unless you specify publish
- **agent-browser Powered**: Fast, reliable browser automation

### What's New in v2.0.0

| Feature | v1.x | v2.0 |
|---------|------|------|
| Platform | X (Twitter) | Xiaohongshu |
| Browser automation | Playwright MCP | agent-browser CLI |
| Login handling | Manual | QR code detection + prompt |
| Account management | Single | Multi-account support |
| Content type | Articles | Image notes |

### Requirements

| Requirement | Details |
|-------------|---------|
| Claude Code | [claude.ai/code](https://claude.ai/code) |
| agent-browser | `npm install -g agent-browser` or use npx |
| Python 3.9+ | With dependencies below |
| macOS | Currently macOS only |

```bash
# Install Python dependencies
pip install Pillow pyobjc-framework-Cocoa

# Install agent-browser (optional, can use npx)
npm install -g agent-browser
```

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
[1/6] Parse content...
      → Extract title, content, images, tags

[2/6] Open Xiaohongshu creator page...
      → Navigate to creator.xiaohongshu.com/publish/publish

[3/6] Handle login (if needed)...
      → If QR code detected: PROMPT USER TO SCAN
      → Wait for login completion

[4/6] Upload images...
      → Upload all images (1-18 supported)

[5/6] Fill title and content...
      → Add title, description, tags

[6/6] Save draft...
      → ✅ Review and publish manually
      → (Or publish directly if requested)
```

### Multi-Account Support

This skill supports multiple Xiaohongshu accounts! Each account is saved separately:

```
~/.agent-browser/xiaohongshu-auth-default.json   # Default account
~/.agent-browser/xiaohongshu-auth-work.json      # Work account
~/.agent-browser/xiaohongshu-auth-personal.json  # Personal account
```

#### Account Commands

| Command | Action |
|---------|--------|
| "用工作账号发布" | Use work account |
| "切换账号" / "switch account" | List and switch accounts |
| "添加新账号" / "add account" | Add a new account |
| "删除账号" / "delete account" | Remove an account |

#### Manage Accounts via CLI

```bash
# List saved accounts
ls ~/.agent-browser/xiaohongshu-auth-*.json

# Delete an account
rm ~/.agent-browser/xiaohongshu-auth-<account_name>.json
```

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

**Q: Why agent-browser instead of Playwright MCP?**
A: agent-browser provides a simpler CLI interface that's easier to use and doesn't require MCP server setup.

**Q: QR code timeout?**
A: The skill waits up to 2 minutes for login. If timeout occurs, restart the process.

**Q: Windows/Linux support?**
A: Currently macOS only. PRs welcome for cross-platform clipboard support.

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

> 一键发布图片笔记到小红书。支持二维码登录，多账号管理，多图上传。

**v2.0.0** — 使用 agent-browser 实现可靠的浏览器自动化

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
| 登录 | 30秒 - 1分钟 | 自动检测，提示扫码 |
| 上传5张图片 | 2-3 分钟 | 30 秒 |
| 填写标题内容 | 1-2 分钟 | 10 秒 |
| **总计** | **5-10 分钟** | **1-2 分钟** |

**效率提升 5 倍**

### 解决方案

本技能自动化整个发布流程：

```
图片/Markdown 文件
     ↓ Python 解析
结构化数据 (标题, 内容, 图片, 标签)
     ↓ agent-browser CLI
小红书创作平台 (浏览器自动化)
     ↓
草稿/已发布笔记
```

### 核心功能

- **二维码登录处理**：检测登录页面，提示用户扫码
- **👥 多账号支持**：管理多个小红书账号，轻松切换
- **🔐 登录状态持久化**：首次登录后保存状态，下次无需扫码
- **多图上传**：一次上传最多 18 张图片
- **内容解析**：从 Markdown 提取标题、内容、标签
- **默认存草稿**：不会自动发布，除非明确指定
- **agent-browser 驱动**：快速、可靠的浏览器自动化

### v2.0.0 更新内容

| 功能 | v1.x | v2.0 |
|------|------|------|
| 平台 | X (Twitter) | 小红书 |
| 浏览器自动化 | Playwright MCP | agent-browser CLI |
| 登录处理 | 手动 | 二维码检测 + 提示 |
| 账号管理 | 单账号 | 多账号支持 |
| 内容类型 | 长文 | 图片笔记 |

### 环境要求

| 需求 | 详情 |
|------|------|
| Claude Code | [claude.ai/code](https://claude.ai/code) |
| agent-browser | `npm install -g agent-browser` 或使用 npx |
| Python 3.9+ | 需要下列依赖 |
| macOS | 目前仅支持 macOS |

```bash
# 安装 Python 依赖
pip install Pillow pyobjc-framework-Cocoa

# 安装 agent-browser (可选，可以用 npx)
npm install -g agent-browser
```

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
[1/6] 解析内容...
      → 提取标题、内容、图片、标签

[2/6] 打开小红书创作平台...
      → 导航到 creator.xiaohongshu.com/publish/publish

[3/6] 处理登录（如需要）...
      → 检测到二维码：提示用户扫码
      → 等待登录完成

[4/6] 上传图片...
      → 上传所有图片（支持 1-18 张）

[5/6] 填写标题和内容...
      → 添加标题、描述、标签

[6/6] 保存草稿...
      → ✅ 请手动检查后发布
      → （或直接发布，如果用户要求）
```

### 多账号支持

本技能支持多个小红书账号！每个账号单独保存：

```
~/.agent-browser/xiaohongshu-auth-default.json   # 默认账号
~/.agent-browser/xiaohongshu-auth-work.json      # 工作账号
~/.agent-browser/xiaohongshu-auth-personal.json  # 个人账号
```

#### 账号操作指令

| 指令 | 操作 |
|------|------|
| "用工作账号发布" | 使用工作账号 |
| "切换账号" | 列出并切换账号 |
| "添加新账号" | 添加新账号 |
| "删除账号" | 删除指定账号 |

#### 命令行管理账号

```bash
# 列出已保存的账号
ls ~/.agent-browser/xiaohongshu-auth-*.json

# 删除账号
rm ~/.agent-browser/xiaohongshu-auth-<账号名>.json
```

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

**Q: 为什么用 agent-browser 而不是 Playwright MCP？**
A: agent-browser 提供更简单的 CLI 接口，无需配置 MCP 服务器。

**Q: 二维码超时怎么办？**
A: 技能会等待最多 2 分钟。如果超时，重新运行即可。

**Q: 支持 Windows/Linux 吗？**
A: 目前仅支持 macOS。欢迎提交 PR 支持其他平台。

**Q: 图片上传失败？**
A: 检查：路径是否正确，格式是否支持（jpg/png/gif/webp），文件大小是否超限。

**Q: 可以直接发布而不是存草稿吗？**
A: 可以，在请求中说明 "直接发布" 或 "publish now" 即可。

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

### v2.0.0 (2025-01)
- **Platform switch / 平台切换**: Xiaohongshu instead of X (Twitter) / 从 X 改为小红书
- **agent-browser**: Replace Playwright MCP with agent-browser CLI / 用 agent-browser CLI 替代 Playwright MCP
- **QR code login / 二维码登录**: Detect and prompt user for login / 检测并提示用户登录
- **Multi-account / 多账号**: Support multiple accounts with easy switching / 支持多账号管理和切换
- **Image-centric / 图片笔记**: Focus on image notes rather than articles / 专注于图片笔记而非长文

### v1.1.0 (2025-12)
- Block-index positioning for X Articles
- Reverse insertion order
- Optimized wait strategy

### v1.0.0 (2025-12)
- Initial release (X Articles publisher)

---

## License / 许可证

MIT License - see [LICENSE](LICENSE)

## Author / 作者

[wshuyi](https://github.com/wshuyi)

## Contributing / 贡献

- **Issues**: Report bugs or request features / 报告问题或请求功能
- **PRs**: Welcome! Especially for Windows/Linux support / 欢迎！特别是跨平台支持
