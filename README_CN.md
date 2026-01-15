# 小红书发布器 (Xiaohongshu Publisher Skill)

[English](README.md) | [中文](README_CN.md)

> 一键发布图片笔记到小红书。支持二维码登录，多图上传。

**v2.0.0** — 使用 agent-browser 实现可靠的浏览器自动化

---

## 痛点

手动发布小红书笔记太繁琐：

| 痛点 | 描述 |
|------|------|
| **登录麻烦** | 每次都要扫码登录 |
| **多图上传** | 一张一张上传图片 |
| **内容格式** | 复制粘贴文字，手动加标签 |
| **耗时长** | 每篇笔记 5-10 分钟 |

### 时间对比

| 任务 | 手动 | 使用本技能 |
|------|------|-----------|
| 登录 | 30秒 - 1分钟 | 自动检测，提示扫码 |
| 上传5张图片 | 2-3 分钟 | 30 秒 |
| 填写标题内容 | 1-2 分钟 | 10 秒 |
| **总计** | **5-10 分钟** | **1-2 分钟** |

**效率提升 5 倍**

---

## 解决方案

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

---

## v2.0.0 更新内容

| 功能 | v1.x | v2.0 |
|------|------|------|
| 平台 | X (Twitter) | 小红书 |
| 浏览器自动化 | Playwright MCP | agent-browser CLI |
| 登录处理 | 手动 | 二维码检测 + 提示 |
| 内容类型 | 长文 | 图片笔记 |

---

## 环境要求

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

---

## 安装

### 方法一：Git Clone（推荐）

```bash
git clone https://github.com/wshuyi/xiaohongshu-publisher-skill.git
cp -r xiaohongshu-publisher-skill/skills/xiaohongshu-publisher ~/.claude/skills/
```

### 方法二：插件市场

```
/plugin marketplace add wshuyi/xiaohongshu-publisher-skill
/plugin install xiaohongshu-publisher@wshuyi/xiaohongshu-publisher-skill
```

---

## 使用方法

### 自然语言

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

### 技能命令

```
/xiaohongshu-publisher /path/to/note.md
```

---

## 工作流程

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

---

## 二维码登录

当小红书需要登录时，技能会：

1. **首先尝试加载已保存的登录状态**（如已登录则跳过扫码）
2. 如果没有保存状态，检测二维码登录页面
3. **显示消息**："请使用小红书 App 扫描二维码登录"
4. 等待您完成登录
5. **保存登录状态**供下次使用（下次无需扫码！）
6. 继续发布流程

**提示**：手机上准备好小红书 App，方便快速扫码。

### 多账号支持

本技能支持多个小红书账号！每个账号单独保存：
```
~/.agent-browser/xiaohongshu-auth-default.json   # 默认账号
~/.agent-browser/xiaohongshu-auth-work.json      # 工作账号
~/.agent-browser/xiaohongshu-auth-personal.json  # 个人账号
```

**账号操作指令：**
- "用工作账号发布" → 使用工作账号
- "切换账号" → 列出并切换账号
- "添加新账号" → 添加新账号

**列出已保存的账号：**
```bash
ls ~/.agent-browser/xiaohongshu-auth-*.json
```

**删除账号：**
```bash
rm ~/.agent-browser/xiaohongshu-auth-<账号名>.json
```

---

## 内容格式

### 直接提供图片和文字

```
发布这些图片到小红书:
- /path/to/photo1.jpg
- /path/to/photo2.jpg
- /path/to/photo3.jpg

标题: 周末好去处
内容: 发现了一家超赞的咖啡店...
标签: 咖啡, 探店, 周末
```

### 使用 Markdown 文件

```markdown
# 周末好去处

![](./images/photo1.jpg)
![](./images/photo2.jpg)

发现了一家超赞的咖啡店，环境特别好！

推荐指数：⭐⭐⭐⭐⭐

#咖啡 #探店 #周末
```

---

## 项目结构

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
├── README.md                    # 英文说明
├── README_CN.md                 # 本文件
└── LICENSE
```

---

## 限制与最佳实践

| 项目 | 限制 |
|------|------|
| 每篇图片数 | 1-18 张 |
| 标题长度 | 建议 ~20 字符 |
| 内容长度 | 最多 ~1000 字符 |
| 标签数量 | 建议最多 5 个 |
| 图片格式 | JPG, PNG, GIF, WebP |

### 小贴士

1. **提前准备图片** - 运行前确保所有图片就绪
2. **手机准备好小红书** - 方便快速扫码
3. **使用草稿模式** - 发布前先检查
4. **压缩大图片** - 上传更快

---

## 常见问题

**Q: 为什么用 agent-browser 而不是 Playwright MCP？**
A: agent-browser 提供更简单的 CLI 接口，无需配置 MCP 服务器。

**Q: 二维码超时怎么办？**
A: 技能会等待最多 2 分钟。如果超时，重新运行即可。

**Q: 支持 Windows/Linux 吗？**
A: 目前仅支持 macOS。欢迎提交 PR 支持其他平台。

**Q: 图片上传失败？**
A: 检查：路径是否正确，格式是否支持（jpg/png/gif/webp），文件大小是否超限。

**Q: 可以直接发布而不是存草稿吗？**
A: 可以，在请求中说明 "发布" 或 "publish" 即可。

---

## 更新日志

### v2.0.0 (2025-01)
- **平台切换**：从 X (Twitter) 改为小红书
- **agent-browser**：用 agent-browser CLI 替代 Playwright MCP
- **二维码登录**：检测并提示用户登录
- **图片笔记**：专注于图片笔记而非长文

### v1.1.0 (2025-12)
- X Articles 的块索引定位
- 反向插入顺序
- 优化等待策略

### v1.0.0 (2025-12)
- 初始发布（X Articles 发布器）

---

## 许可证

MIT License - 见 [LICENSE](LICENSE)

## 作者

[wshuyi](https://github.com/wshuyi)

---

## 贡献

- **Issues**：报告问题或请求功能
- **PRs**：欢迎！特别是跨平台支持
