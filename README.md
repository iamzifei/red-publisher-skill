# X Article Publisher Skill

[English](README.md) | [中文](README_CN.md)

> Publish Markdown articles to X (Twitter) Articles with one command. Say goodbye to tedious rich text editing.

---

## The Problem

If you're used to writing in Markdown, publishing to X Articles is a **painful process**:

| Pain Point | Description |
|------------|-------------|
| **Format Loss** | Copy from Markdown editor → Paste to X → All formatting gone |
| **Manual Formatting** | Set each H2, bold, link manually — 15-20 min per article |
| **Tedious Image Upload** | 5 clicks per image: Add media → Media → Add photo → Select → Wait |
| **Position Errors** | Hard to remember where each image should go |

### Time Comparison

| Task | Manual | With This Skill |
|------|--------|-----------------|
| Format conversion | 15-20 min | 0 (automatic) |
| Cover image | 1-2 min | 10 sec |
| 5 content images | 5-10 min | 1 min |
| **Total** | **20-30 min** | **2-3 min** |

**10x efficiency improvement**

---

## The Solution

This skill automates the entire publishing workflow:

```
Markdown File
     ↓ Python parsing
Structured Data (title, images, HTML)
     ↓ Playwright MCP
X Articles Editor (browser automation)
     ↓
Draft Saved (never auto-publishes)
```

### Key Features

- **Rich Text Paste**: Convert Markdown to HTML, paste via clipboard — all formatting preserved
- **Smart Image Insertion**: Copy image to clipboard → Click paragraph → Paste (2 steps vs 5 clicks)
- **Precise Positioning**: Extract context text to locate exact insertion points
- **Safe by Design**: Only saves as draft, never publishes automatically

---

## Requirements

| Requirement | Details |
|-------------|---------|
| Claude Code | [claude.ai/code](https://claude.ai/code) |
| Playwright MCP | Browser automation |
| X Premium Plus | Required for Articles feature |
| Python 3.9+ | With dependencies below |
| macOS | Currently macOS only |

```bash
pip install Pillow pyobjc-framework-Cocoa
```

---

## Installation

### Method 1: Git Clone (Recommended)

```bash
git clone https://github.com/wshuyi/x-article-publisher-skill.git
cp -r x-article-publisher-skill/skills/x-article-publisher ~/.claude/skills/
```

### Method 2: Plugin Marketplace

```
/plugin marketplace add wshuyi/x-article-publisher-skill
/plugin install x-article-publisher@wshuyi/x-article-publisher-skill
```

---

## Usage

### Natural Language

```
Publish /path/to/article.md to X
```

```
Help me post this article to X Articles: ~/Documents/my-post.md
```

### Skill Command

```
/x-article-publisher /path/to/article.md
```

---

## Workflow Steps

```
[1/7] Parse Markdown...
      → Extract title, cover image, content images
      → Convert to HTML

[2/7] Open X Articles editor...
      → Navigate to x.com/compose/articles

[3/7] Upload cover image...
      → First image becomes cover

[4/7] Fill title...

[5/7] Paste article content...
      → Rich text via clipboard
      → All formatting preserved

[6/7] Insert content images...
      → Locate by context text
      → Paste via clipboard

[7/7] Save draft...
      → ✅ Review and publish manually
```

---

## Supported Markdown

| Syntax | Result |
|--------|--------|
| `# H1` | Article title (extracted) |
| `## H2` | Section headers |
| `**bold**` | **Bold text** |
| `*italic*` | *Italic text* |
| `[text](url)` | Hyperlinks |
| `> quote` | Blockquotes |
| `- item` | Unordered lists |
| `1. item` | Ordered lists |
| `![](img.jpg)` | Images (first = cover) |

---

## Example

### Input: `article.md`

```markdown
# 5 AI Tools Worth Watching in 2024

![cover](./images/cover.jpg)

AI tools exploded in 2024. Here are 5 worth your attention.

## 1. Claude: Best Conversational AI

**Claude** by Anthropic excels at long-context understanding.

> Claude's context window reaches 200K tokens.

![claude-demo](./images/claude-demo.png)

## 2. Midjourney: AI Art Leader

[Midjourney](https://midjourney.com) is the most popular AI art tool.

![midjourney](./images/midjourney.jpg)
```

### Command

```
Publish ~/article.md to X
```

### Result

- Cover: `cover.jpg` uploaded
- Title: "5 AI Tools Worth Watching in 2024"
- Content: Rich text with H2, bold, quotes, links
- Images: Inserted at correct positions
- Status: **Draft saved** (ready for manual review)

---

## Project Structure

```
x-article-publisher-skill/
├── .claude-plugin/
│   └── plugin.json              # Plugin config
├── skills/
│   └── x-article-publisher/
│       ├── SKILL.md             # Skill instructions
│       └── scripts/
│           ├── parse_markdown.py
│           └── copy_to_clipboard.py
├── docs/
│   └── GUIDE.md                 # Detailed guide
├── README.md                    # This file
├── README_CN.md                 # Chinese version
└── LICENSE
```

---

## FAQ

**Q: Why Premium Plus?**
A: X Articles is exclusive to Premium Plus subscribers.

**Q: Windows/Linux support?**
A: Currently macOS only. PRs welcome for cross-platform clipboard support.

**Q: Image upload failed?**
A: Check: valid path, supported format (jpg/png/gif/webp), stable network.

**Q: Can I publish to multiple accounts?**
A: Not automatically. Switch accounts in browser manually before running.

---

## Documentation

- [Detailed Usage Guide](docs/GUIDE.md) — Complete documentation with examples

---

## License

MIT License - see [LICENSE](LICENSE)

## Author

[wshuyi](https://github.com/wshuyi)

---

## Contributing

- **Issues**: Report bugs or request features
- **PRs**: Welcome! Especially for Windows/Linux support
