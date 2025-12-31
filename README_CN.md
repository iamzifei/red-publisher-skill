# X Article Publisher Skill

[English](README.md) | [中文](README_CN.md)

> 一键将 Markdown 文章发布到 X (Twitter) Articles，告别繁琐的富文本编辑。

---

## 痛点分析

如果你习惯用 Markdown 写作，将内容发布到 X Articles 是一个**极其痛苦**的过程：

| 痛点 | 描述 |
|------|------|
| **格式丢失** | 从 Markdown 编辑器复制 → 粘贴到 X → 格式全部丢失 |
| **手动格式化** | 逐个设置 H2、粗体、链接 — 每篇文章 15-20 分钟 |
| **图片上传繁琐** | 每张图片 5 次点击：添加媒体 → 媒体 → 添加照片 → 选择 → 等待 |
| **位置容易出错** | 很难记住每张图片应该插入的位置 |

### 时间对比

| 操作 | 手动方式 | 使用本 Skill |
|------|----------|--------------|
| 格式转换 | 15-20 分钟 | 0（自动） |
| 封面图上传 | 1-2 分钟 | 10 秒 |
| 5 张内容图插入 | 5-10 分钟 | 1 分钟 |
| **总计** | **20-30 分钟** | **2-3 分钟** |

**效率提升 10 倍以上**

---

## 解决方案

本 Skill 自动化整个发布流程：

```
Markdown 文件
     ↓ Python 解析
结构化数据（标题、图片、HTML）
     ↓ Playwright MCP
X Articles 编辑器（浏览器自动化）
     ↓
保存草稿（绝不自动发布）
```

### 核心特性

- **富文本粘贴**：将 Markdown 转换为 HTML，通过剪贴板粘贴 — 所有格式完整保留
- **智能图片插入**：复制图片到剪贴板 → 点击段落 → 粘贴（2 步 vs 5 次点击）
- **精确定位**：提取上下文文本，精确定位插入位置
- **安全设计**：仅保存草稿，绝不自动发布

---

## 环境要求

| 要求 | 说明 |
|------|------|
| Claude Code | [claude.ai/code](https://claude.ai/code) |
| Playwright MCP | 浏览器自动化 |
| X Premium Plus | Articles 功能需要此订阅 |
| Python 3.9+ | 需安装以下依赖 |
| macOS | 目前仅支持 macOS |

```bash
pip install Pillow pyobjc-framework-Cocoa
```

---

## 安装方式

### 方式一：Git Clone（推荐）

```bash
git clone https://github.com/wshuyi/x-article-publisher-skill.git
cp -r x-article-publisher-skill/skills/x-article-publisher ~/.claude/skills/
```

### 方式二：插件市场

```
/plugin marketplace add wshuyi/x-article-publisher-skill
/plugin install x-article-publisher@wshuyi/x-article-publisher-skill
```

---

## 使用方法

### 自然语言

```
把 /path/to/article.md 发布到 X
```

```
帮我把这篇文章发到 X Articles：~/Documents/my-post.md
```

### Skill 命令

```
/x-article-publisher /path/to/article.md
```

---

## 工作流程

```
[1/7] 解析 Markdown...
      → 提取标题、封面图、内容图片
      → 转换为 HTML

[2/7] 打开 X Articles 编辑器...
      → 导航到 x.com/compose/articles

[3/7] 上传封面图...
      → 第一张图片作为封面

[4/7] 填写标题...

[5/7] 粘贴文章内容...
      → 通过剪贴板粘贴富文本
      → 所有格式完整保留

[6/7] 插入内容图片...
      → 根据上下文定位
      → 通过剪贴板粘贴

[7/7] 保存草稿...
      → ✅ 请手动预览并发布
```

---

## 支持的 Markdown 格式

| 语法 | 效果 |
|------|------|
| `# H1` | 文章标题（自动提取） |
| `## H2` | 二级标题 |
| `**粗体**` | **粗体文字** |
| `*斜体*` | *斜体文字* |
| `[文字](url)` | 超链接 |
| `> 引用` | 引用块 |
| `- 列表` | 无序列表 |
| `1. 列表` | 有序列表 |
| `![](img.jpg)` | 图片（第一张为封面） |

---

## 完整示例

### 输入：`article.md`

```markdown
# 2024 年最值得关注的 5 个 AI 工具

![封面](./images/cover.jpg)

人工智能工具在 2024 年迎来了爆发式增长。本文将介绍 5 个最值得关注的工具。

## 1. Claude：最强对话 AI

**Claude** 由 Anthropic 开发，在长文本理解方面表现出色。

> Claude 的上下文窗口高达 200K tokens。

![claude-demo](./images/claude-demo.png)

## 2. Midjourney：AI 绘画领导者

[Midjourney](https://midjourney.com) 是目前最受欢迎的 AI 绘画工具。

![midjourney](./images/midjourney.jpg)
```

### 执行命令

```
把 ~/article.md 发布到 X
```

### 执行结果

- 封面：`cover.jpg` 已上传
- 标题：「2024 年最值得关注的 5 个 AI 工具」
- 内容：富文本格式完整保留（H2、粗体、引用、链接）
- 图片：在正确位置插入
- 状态：**草稿已保存**（等待手动预览发布）

---

## 项目结构

```
x-article-publisher-skill/
├── .claude-plugin/
│   └── plugin.json              # 插件配置
├── skills/
│   └── x-article-publisher/
│       ├── SKILL.md             # Skill 核心指令
│       └── scripts/
│           ├── parse_markdown.py
│           └── copy_to_clipboard.py
├── docs/
│   └── GUIDE.md                 # 详细使用指南
├── README.md                    # 英文说明
├── README_CN.md                 # 中文说明（本文件）
└── LICENSE
```

---

## 常见问题

**Q: 为什么需要 Premium Plus？**
A: X Articles 是 Premium Plus 订阅专属功能，普通用户无法使用。

**Q: 支持 Windows/Linux 吗？**
A: 目前仅支持 macOS。欢迎贡献跨平台剪贴板支持的 PR。

**Q: 图片上传失败怎么办？**
A: 检查：路径是否正确、格式是否支持（jpg/png/gif/webp）、网络是否稳定。

**Q: 可以发布到多个账号吗？**
A: 不支持自动切换。请在浏览器中手动切换账号后再执行。

---

## 详细文档

- [完整使用指南](docs/GUIDE.md) — 包含详细说明和更多示例

---

## 许可证

MIT License - 见 [LICENSE](LICENSE)

## 作者

[wshuyi](https://github.com/wshuyi)

---

## 贡献

- **Issues**：报告 Bug 或提出功能建议
- **PR**：欢迎贡献代码，特别是 Windows/Linux 支持
