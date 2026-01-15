# 小红书发布器使用指南

> 一键发布图片笔记到小红书。支持二维码登录，多图上传。

**v2.0.0** — 使用 agent-browser 实现可靠的浏览器自动化

---

## v2.0.0 更新亮点

| 功能 | v1.x | v2.0 |
|------|------|------|
| 平台 | X (Twitter) | 小红书 |
| 浏览器自动化 | Playwright MCP | agent-browser CLI |
| 登录处理 | 手动 | 二维码检测 + 提示 |
| 内容类型 | 长文 | 图片笔记 |

---

## 目录

1. [解决的痛点](#1-解决的痛点)
2. [解决方案](#2-解决方案)
3. [执行方式](#3-执行方式)
4. [完整示例](#4-完整示例)
5. [常见问题](#5-常见问题)

---

## 1. 解决的痛点

### 1.1 小红书是什么？

小红书（Xiaohongshu/RedNote）是中国最大的生活方式分享平台之一，用户可以发布图片笔记分享生活经验、购物心得、旅行攻略等。

访问入口：https://creator.xiaohongshu.com/publish/publish

### 1.2 手动发布的痛点

#### 痛点一：登录麻烦

每次发布都需要扫码登录，即使刚登录过也可能需要重新扫码。

#### 痛点二：图片上传繁琐

```
选择图片 → 等待上传 → 调整顺序 → 重复以上步骤
```

每张图片都需要单独操作，5张图片就需要重复5次。

#### 痛点三：内容格式化

需要手动输入标题、描述、添加标签，还要注意字数限制。

#### 痛点四：重复劳动

如果你经常发布内容，这些操作需要**每次重复**。

### 1.3 时间成本对比

| 操作 | 手动方式 | 使用本 Skill |
|------|----------|--------------|
| 登录 | 30秒-1分钟 | 自动检测，提示扫码 |
| 上传5张图片 | 2-3 分钟 | 30 秒 |
| 填写内容 | 1-2 分钟 | 10 秒 |
| 总计 | **5-10 分钟** | **1-2 分钟** |

**效率提升：5 倍以上**

---

## 2. 解决方案

### 2.1 技术架构

本 Skill 通过以下技术栈实现自动化：

```
┌─────────────────┐
│   图片/Markdown  │
└────────┬────────┘
         │ Python 解析
         ▼
┌─────────────────┐
│  结构化数据 (JSON) │
│  - title        │
│  - content      │
│  - images       │
│  - tags         │
└────────┬────────┘
         │ agent-browser CLI
         ▼
┌─────────────────┐
│  小红书创作平台    │
│  (浏览器自动化)   │
└─────────────────┘
```

#### 核心组件

| 组件 | 作用 |
|------|------|
| `parse_note.py` | 解析 Markdown/文本，提取标题、内容、图片、标签 |
| `copy_to_clipboard.py` | 将图片复制到系统剪贴板 |
| agent-browser | 控制浏览器，模拟用户操作 |
| Claude Code | 协调整个流程 |

### 2.2 工作流程

```
Step 1: 解析内容
        ↓
Step 2: 打开小红书创作平台
        ↓
Step 3: 处理登录（QR码扫码）
        ↓
Step 4: 上传图片（1-18张）
        ↓
Step 5: 填写标题和内容
        ↓
Step 6: 添加标签（可选）
        ↓
Step 7: 保存草稿/发布
```

### 2.3 关键技术：agent-browser

agent-browser 是一个轻量级的浏览器自动化 CLI 工具：

```bash
# 导航
npx agent-browser open https://creator.xiaohongshu.com/publish/publish

# 获取页面快照（包含元素引用）
npx agent-browser snapshot -i

# 点击元素（使用快照中的 ref）
npx agent-browser click @e5

# 填写表单
npx agent-browser fill @e3 "笔记标题"

# 上传文件
npx agent-browser upload @e2 "/path/to/image.jpg"

# 等待
npx agent-browser wait --text "发布成功"
```

**优点**：
- 无需 MCP 服务器配置
- 命令行直接使用
- 快照提供精确的元素引用

### 2.4 QR码登录处理（支持多账号）

小红书经常需要扫码登录，本 Skill 支持多账号管理：

1. **确定使用哪个账号**（默认/工作/个人等）
2. **尝试加载该账号的登录状态**
3. 如果已登录，跳过扫码直接继续
4. 如果需要登录，检测登录页面（查找 QR 码元素）
5. 通知用户："请使用小红书 App 扫描二维码登录"
6. 等待登录完成（最多 2 分钟）
7. **询问账号名称**并保存状态
8. 检测到登录成功后继续流程

```
┌─────────────────────────────────────┐
│  多账号登录流程                         │
│                                     │
│  1. 扫描二维码登录                      │
│  2. 为此账号起个名字（如：work）          │
│  3. 登录状态已保存到:                    │
│     ~/.agent-browser/xiaohongshu-auth-work.json │
│  4. 下次使用「工作账号」无需扫码！         │
└─────────────────────────────────────┘
```

#### 账号存储位置

每个账号单独保存：
```
~/.agent-browser/xiaohongshu-auth-default.json   # 默认账号
~/.agent-browser/xiaohongshu-auth-work.json      # 工作账号
~/.agent-browser/xiaohongshu-auth-personal.json  # 个人账号
```

#### 账号操作

```bash
# 列出所有账号
ls ~/.agent-browser/xiaohongshu-auth-*.json

# 删除特定账号
rm ~/.agent-browser/xiaohongshu-auth-work.json

# 删除所有账号
rm ~/.agent-browser/xiaohongshu-auth-*.json
```

#### 使用指定账号

- "用工作账号发布" → 使用 work 账号
- "用个人账号发布" → 使用 personal 账号
- "切换账号" → 列出并选择账号

### 2.5 安全设计

**默认保存草稿**：本 Skill 默认将内容保存为草稿，最终发布需要用户明确指定。

```
✅ 默认行为：保存草稿
✅ 用户可以：指定直接发布
❌ 不会发生：未经同意自动发布
```

---

## 3. 执行方式

### 3.1 前置条件

#### 条件一：小红书账号

确保你有小红书账号，并且手机上安装了小红书 App（用于扫码登录）。

#### 条件二：安装 agent-browser

```bash
# 全局安装（推荐）
npm install -g agent-browser

# 或使用 npx（无需安装）
npx agent-browser --version
```

#### 条件三：安装 Python 依赖

```bash
pip install Pillow pyobjc-framework-Cocoa
```

验证安装：
```bash
python -c "from AppKit import NSPasteboard; print('OK')"
```

#### 条件四：安装本 Skill

**方式 A：Git Clone（推荐）**

```bash
git clone https://github.com/wshuyi/xiaohongshu-publisher-skill.git
cp -r xiaohongshu-publisher-skill/skills/xiaohongshu-publisher ~/.claude/skills/
```

**方式 B：插件市场**

```
/plugin marketplace add wshuyi/xiaohongshu-publisher-skill
/plugin install xiaohongshu-publisher@wshuyi/xiaohongshu-publisher-skill
```

### 3.2 触发指令

安装完成后，在 Claude Code 中使用以下方式触发：

#### 方式一：自然语言

```
发布这些图片到小红书: /path/to/photo1.jpg, /path/to/photo2.jpg
标题是"周末探店"
```

```
帮我把 ~/Documents/note.md 发到小红书，存草稿就行
```

```
把这几张照片发到小红书，标题"美食分享"，内容"今天发现一家超好吃的店！"
```

#### 方式二：Skill 命令

```
/xiaohongshu-publisher /path/to/note.md
```

### 3.3 操作流程

触发后，Claude 会自动执行以下步骤：

```
[1/6] 解析内容...
      → 标题：「周末探店」
      → 发现 3 张图片
      → 内容：「发现了一家超赞的咖啡店...」
      → 标签：咖啡, 探店, 周末

[2/6] 打开小红书创作平台...
      → 导航到 creator.xiaohongshu.com/publish/publish

[3/6] 处理登录...
      → 检测到 QR 码
      → 请使用小红书 App 扫描二维码登录
      → 等待登录完成...
      → 登录成功！

[4/6] 上传图片...
      → 上传 photo1.jpg
      → 上传 photo2.jpg
      → 上传 photo3.jpg
      → 全部上传完成

[5/6] 填写内容...
      → 标题：「周末探店」
      → 描述：「发现了一家超赞的咖啡店...」
      → 标签：#咖啡 #探店 #周末

[6/6] 保存草稿...
      → ✅ 草稿已保存
      → 请在小红书 App 中预览并发布
```

### 3.4 注意事项

1. **准备好手机**：扫码登录需要手机上的小红书 App
2. **保持浏览器可见**：agent-browser 需要控制浏览器窗口
3. **图片路径**：确保图片路径有效且文件存在
4. **网络稳定**：图片上传需要稳定的网络连接
5. **图片限制**：每篇笔记最多 18 张图片

---

## 4. 完整示例

### 4.1 示例一：直接提供图片和文字

```
发布这些图片到小红书:
- ~/Photos/coffee1.jpg
- ~/Photos/coffee2.jpg
- ~/Photos/coffee3.jpg

标题: 周末咖啡探店
内容: 发现了一家隐藏在小巷子里的咖啡店，环境超级棒，咖啡也很好喝！推荐他们家的拿铁～
标签: 咖啡, 探店, 周末好去处
```

### 4.2 示例二：使用 Markdown 文件

文件路径：`~/Documents/coffee-note.md`

```markdown
# 周末咖啡探店

![](./photos/coffee1.jpg)
![](./photos/coffee2.jpg)
![](./photos/coffee3.jpg)

发现了一家隐藏在小巷子里的咖啡店！

环境超级棒，适合拍照📷
咖啡也很好喝，推荐他们家的拿铁～

地址：XX路XX号
人均：40元

#咖啡 #探店 #周末好去处
```

执行命令：
```
把 ~/Documents/coffee-note.md 发布到小红书
```

### 4.3 执行结果

Claude 会：

1. **解析文件**，输出：
   ```json
   {
     "title": "周末咖啡探店",
     "content": "发现了一家隐藏在小巷子里的咖啡店！\n\n环境超级棒，适合拍照📷\n咖啡也很好喝，推荐他们家的拿铁～\n\n地址：XX路XX号\n人均：40元",
     "images": [
       "~/Documents/photos/coffee1.jpg",
       "~/Documents/photos/coffee2.jpg",
       "~/Documents/photos/coffee3.jpg"
     ],
     "tags": ["咖啡", "探店", "周末好去处"]
   }
   ```

2. **自动操作浏览器**：
   - 打开创作平台
   - 处理 QR 码登录
   - 上传 3 张图片
   - 填写标题和内容
   - 添加标签

3. **完成提示**：
   ```
   ✅ 草稿已保存！

   请在小红书 App 中预览笔记效果，确认无误后发布。
   ```

---

## 5. 常见问题

### Q1: 为什么用 agent-browser 而不是 Playwright MCP？

A: agent-browser 提供更简单的 CLI 接口：
- 无需配置 MCP 服务器
- 命令行直接使用
- 轻量级，快速启动

### Q2: QR 码扫码超时怎么办？

A: 技能会等待最多 2 分钟。如果超时：
1. 重新运行命令
2. 提前准备好手机，快速扫码

### Q3: 支持 Windows 吗？

A: 目前仅支持 macOS，因为剪贴板操作使用了 `pyobjc-framework-Cocoa`。Windows 支持需要替换为 `pywin32`，欢迎贡献 PR。

### Q4: 图片上传失败怎么办？

A: 检查以下几点：
- 图片路径是否正确
- 图片格式是否支持（jpg, png, gif, webp）
- 文件大小是否超过限制
- 网络连接是否稳定

### Q5: 可以直接发布而不是存草稿吗？

A: 可以，在请求中明确说明：
```
发布这些图片到小红书，直接发布：...
```

### Q5.5: 每次都要扫码登录吗？

A: 不需要！技能支持**多账号登录状态持久化**：
- 首次登录后，状态会保存到 `~/.agent-browser/xiaohongshu-auth-<账号名>.json`
- 下次使用时自动加载，无需重新扫码
- 支持多个账号，每个账号单独保存
- 如果登录过期，删除对应文件重新扫码即可：
  ```bash
  rm ~/.agent-browser/xiaohongshu-auth-<账号名>.json
  ```

### Q5.6: 如何管理多个小红书账号？

A: 技能支持多账号管理：

**添加账号**：说 "添加新账号" 或 "登录另一个账号"

**切换账号**：说 "切换账号" 或 "用XX账号发布"

**列出账号**：
```bash
ls ~/.agent-browser/xiaohongshu-auth-*.json
```

**删除账号**：
```bash
rm ~/.agent-browser/xiaohongshu-auth-<账号名>.json
```

### Q6: 图片数量有限制吗？

A: 小红书每篇笔记最多支持 18 张图片，至少需要 1 张。

### Q7: 标题和内容有字数限制吗？

A:
- 标题：建议 20 字符以内
- 内容：最多约 1000 字符
- 标签：建议最多 5 个

---

## 附录：项目结构

```
xiaohongshu-publisher-skill/
├── .claude-plugin/
│   └── plugin.json           # 插件配置
├── skills/
│   └── xiaohongshu-publisher/
│       ├── SKILL.md          # Skill 核心指令
│       └── scripts/
│           ├── parse_note.py      # 内容解析
│           └── copy_to_clipboard.py # 剪贴板操作
├── docs/
│   └── GUIDE.md              # 本文档
├── README.md
├── README_CN.md
└── LICENSE
```

---

## 反馈与贡献

- **GitHub**: https://github.com/wshuyi/xiaohongshu-publisher-skill
- **Issues**: 遇到问题请提交 Issue
- **PR**: 欢迎贡献代码，特别是 Windows/Linux 支持

---

*本文档由 Claude Code 生成，最后更新：2025-01*
