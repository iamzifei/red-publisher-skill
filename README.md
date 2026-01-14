# Xiaohongshu Publisher Skill (小红书发布器)

[English](README.md) | [中文](README_CN.md)

> Publish images and notes to Xiaohongshu (小红书) with one command. Supports QR code login and multi-image uploads.

**v2.0.0** — Now using agent-browser for reliable automation

---

## The Problem

Publishing to Xiaohongshu manually is tedious:

| Pain Point | Description |
|------------|-------------|
| **Login Hassle** | Must scan QR code every session |
| **Multiple Images** | Upload images one by one |
| **Content Formatting** | Copy-paste text, add tags manually |
| **Time Consuming** | 5-10 minutes per post |

### Time Comparison

| Task | Manual | With This Skill |
|------|--------|-----------------|
| Login | 30 sec - 1 min | Auto-detected, prompted |
| Image upload (5 images) | 2-3 min | 30 sec |
| Title & content | 1-2 min | 10 sec |
| **Total** | **5-10 min** | **1-2 min** |

**5x efficiency improvement**

---

## The Solution

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
- **Multi-Image Upload**: Upload up to 18 images at once
- **Content Parsing**: Extract title, content, and tags from Markdown
- **Safe by Default**: Saves as draft unless you specify publish
- **agent-browser Powered**: Fast, reliable browser automation

---

## What's New in v2.0.0

| Feature | v1.x | v2.0 |
|---------|------|------|
| Platform | X (Twitter) | Xiaohongshu |
| Browser automation | Playwright MCP | agent-browser CLI |
| Login handling | Manual | QR code detection + prompt |
| Content type | Articles | Image notes |

---

## Requirements

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

---

## Installation

### Method 1: Git Clone (Recommended)

```bash
git clone https://github.com/wshuyi/xiaohongshu-publisher-skill.git
cp -r xiaohongshu-publisher-skill/skills/xiaohongshu-publisher ~/.claude/skills/
```

### Method 2: Plugin Marketplace

```
/plugin marketplace add wshuyi/xiaohongshu-publisher-skill
/plugin install xiaohongshu-publisher@wshuyi/xiaohongshu-publisher-skill
```

---

## Usage

### Natural Language

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

### Skill Command

```
/xiaohongshu-publisher /path/to/note.md
```

---

## Workflow Steps

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

---

## QR Code Login

When Xiaohongshu requires login, the skill will:

1. Detect the QR code login page
2. **Display a message**: "Please scan the QR code with your Xiaohongshu app to log in."
3. Wait for you to complete the login
4. Continue with the publishing flow

**Tip**: Keep the Xiaohongshu app ready on your phone for quick scanning.

---

## Content Formats

### From Images + Text

```
发布这些图片到小红书:
- /path/to/photo1.jpg
- /path/to/photo2.jpg
- /path/to/photo3.jpg

标题: 周末好去处
内容: 发现了一家超赞的咖啡店...
标签: 咖啡, 探店, 周末
```

### From Markdown File

```markdown
# 周末好去处

![](./images/photo1.jpg)
![](./images/photo2.jpg)

发现了一家超赞的咖啡店，环境特别好！

推荐指数：⭐⭐⭐⭐⭐

#咖啡 #探店 #周末
```

---

## Project Structure

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
│   └── GUIDE.md                 # Detailed guide
├── README.md                    # This file
├── README_CN.md                 # Chinese version
└── LICENSE
```

---

## Limits & Best Practices

| Item | Limit |
|------|-------|
| Images per note | 1-18 |
| Title length | ~20 characters recommended |
| Content length | ~1000 characters max |
| Tags | Up to 5 recommended |
| Image formats | JPG, PNG, GIF, WebP |

### Tips

1. **Prepare images first** - Have all images ready before running
2. **Keep Xiaohongshu app handy** - For quick QR code scanning
3. **Use draft mode** - Review before publishing
4. **Compress large images** - Faster uploads

---

## FAQ

**Q: Why agent-browser instead of Playwright MCP?**
A: agent-browser provides a simpler CLI interface that's easier to use and doesn't require MCP server setup.

**Q: QR code timeout?**
A: The skill waits up to 2 minutes for login. If timeout occurs, restart the process.

**Q: Windows/Linux support?**
A: Currently macOS only. PRs welcome for cross-platform clipboard support.

**Q: Image upload failed?**
A: Check: valid path, supported format (jpg/png/gif/webp), file size within limits.

**Q: Can I publish directly instead of draft?**
A: Yes, specify "发布" or "publish" in your request instead of "存草稿" or "save draft".

---

## Changelog

### v2.0.0 (2025-01)
- **Platform switch**: Xiaohongshu instead of X (Twitter)
- **agent-browser**: Replace Playwright MCP with agent-browser CLI
- **QR code login**: Detect and prompt user for login
- **Image-centric**: Focus on image notes rather than articles

### v1.1.0 (2025-12)
- Block-index positioning for X Articles
- Reverse insertion order
- Optimized wait strategy

### v1.0.0 (2025-12)
- Initial release (X Articles publisher)

---

## License

MIT License - see [LICENSE](LICENSE)

## Author

[wshuyi](https://github.com/wshuyi)

---

## Contributing

- **Issues**: Report bugs or request features
- **PRs**: Welcome! Especially for Windows/Linux support
