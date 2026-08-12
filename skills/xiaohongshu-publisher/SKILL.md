---
name: xiaohongshu-publisher
description: Publish images to Xiaohongshu (小红书) via CDP browser. Always saves as draft.
  新手引导：输入 `/小红书发布 新手`、「这个怎么用」「第一次用」「能干嘛」时，走 SKILL.md 的〈新手上路〉，不要直接开始干活。
---

# Xiaohongshu Publisher (小红书发布器)

Publish images and notes to Xiaohongshu (小红书) Creator Platform using agent-browser with CDP (Chrome DevTools Protocol) mode.

## Architecture: CDP Mode

This skill uses **CDP mode** - connecting to an existing browser instance where the user is already logged in. This approach:
- **Eliminates QR code scanning** - User logs in once in their browser
- **Leverages existing session** - Uses the browser's cookies and auth state
- **More stable** - No need to manage auth state files
- **agent-browser CLI** - Simple command-line interface with `--cdp` flag

## ⚠️ IMPORTANT: Draft Mode by Default

**This skill ALWAYS saves notes as DRAFT by default. It will NEVER auto-publish.**

Only click the "发布" (publish) button if the user EXPLICITLY requests immediate publishing with phrases like:
- "直接发布" / "立即发布" / "马上发布"
- "publish now" / "publish directly" / "publish immediately"
- "不要草稿，直接发" / "不存草稿"

If unsure, ALWAYS save as draft and let user review before publishing.

## Prerequisites

### 1. Launch Chrome with Remote Debugging

Before using this skill, the user must launch Chrome with remote debugging enabled:

**macOS:**
```bash
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222
```

**Or create an alias in ~/.zshrc or ~/.bashrc:**
```bash
alias chrome-debug='/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222'
```

Then simply run: `chrome-debug`

### 2. Login to Xiaohongshu

In the Chrome browser (with debug port), navigate to:
```
https://creator.xiaohongshu.com/publish/publish
```

Login using QR code scan with Xiaohongshu App. **This only needs to be done once** - the session persists in the browser.

**Note**: After login, you can **minimize the browser window** to the Dock. The browser must stay running (not closed) for CDP to work, but you don't need to see it.

### 3. Python Dependencies

```bash
pip install Pillow pyobjc-framework-Cocoa
```

## agent-browser CDP Commands Reference

All commands use the `--cdp 9222` flag to connect to the existing Chrome browser:

```bash
# Navigation
npx agent-browser --cdp 9222 open <url>          # Navigate to URL
npx agent-browser --cdp 9222 snapshot -i         # Get page snapshot with element refs

# Element Interaction
npx agent-browser --cdp 9222 click @e5           # Click element by ref
npx agent-browser --cdp 9222 fill @e2 "text"     # Fill input field
npx agent-browser --cdp 9222 type @e3 "text"     # Type text into element
npx agent-browser --cdp 9222 upload @e4 "/path/to/file.jpg"  # Upload file

# Keyboard & Wait
npx agent-browser --cdp 9222 press Enter         # Press key
npx agent-browser --cdp 9222 wait 2000           # Wait milliseconds
npx agent-browser --cdp 9222 wait --text "发布"  # Wait for text

# Screenshot
npx agent-browser --cdp 9222 screenshot          # Take screenshot
```

**Important**:
- Element refs (like @e5) come from `snapshot -i` output. Always take a snapshot before interacting.
- The `--cdp 9222` flag connects to the browser's debug port instead of launching a new browser.
- Do NOT use the `close` command as it would close the user's browser!

## Scripts

Located in `~/.claude/skills/xiaohongshu-publisher/scripts/`:

### parse_note.py
Parse Markdown and extract structured data for Xiaohongshu notes:
```bash
python parse_note.py <markdown_file> [--output json]
```
Returns JSON with: title, content, images (list of paths), tags

### copy_to_clipboard.py
Copy image to system clipboard for pasting:
```bash
python copy_to_clipboard.py image /path/to/image.jpg [--quality 80]
```

## Workflow

### Phase 0: Verify Browser Connection

**CRITICAL: First verify that the CDP browser is running and connected.**

1. **Check if browser is accessible**:
   ```bash
   npx agent-browser --cdp 9222 snapshot -i
   ```
   - If connection fails, tell user: "请先启动带调试端口的 Chrome 浏览器"

2. **Provide startup command if needed**:
   ```bash
   /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222
   ```

### Phase 1: Check Login Status

1. **Navigate to creator page** (if not already there):
   ```bash
   npx agent-browser --cdp 9222 open "https://creator.xiaohongshu.com/publish/publish"
   ```

2. **Take snapshot to check login status**:
   ```bash
   npx agent-browser --cdp 9222 snapshot -i
   ```
   - Look for "上传图片" button = logged in
   - Look for QR code or login form = need to login

3. **If login required**:
   - Tell user: "请在浏览器中扫码登录小红书，登录后告诉我"
   - Wait for user confirmation
   - Take new snapshot to verify login success

### Phase 2: Parse Content

If user provides a markdown file, parse it:

```bash
python ~/.claude/skills/xiaohongshu-publisher/scripts/parse_note.py /path/to/note.md
```

Output JSON:
```json
{
  "title": "Note Title",
  "content": "Note description/content text...",
  "images": ["/path/to/img1.jpg", "/path/to/img2.jpg"],
  "tags": ["tag1", "tag2"]
}
```

### Phase 3: Upload Images

1. **Take snapshot** to get current element refs:
   ```bash
   npx agent-browser --cdp 9222 snapshot -i
   ```

2. **Find the upload input element** (look for file input or "上传图片" area)

3. **Upload images**:
   ```bash
   # Single image
   npx agent-browser --cdp 9222 upload @e<ref> "/path/to/image1.jpg"

   # Multiple images (comma-separated)
   npx agent-browser --cdp 9222 upload @e<ref> "/path/to/img1.jpg,/path/to/img2.jpg,/path/to/img3.jpg"
   ```

4. **Wait for uploads to complete**:
   ```bash
   npx agent-browser --cdp 9222 wait 3000
   ```

5. **Take new snapshot** after upload completes

### Phase 4: Fill Title and Content

1. **Find title input field** from snapshot (placeholder usually contains "标题")

2. **Fill title**:
   ```bash
   npx agent-browser --cdp 9222 fill @e<title_ref> "Your Note Title"
   ```
   - Title limit: ~20 characters recommended

3. **Find content/description field** (placeholder usually contains "描述" or "正文")

4. **Fill content**:
   ```bash
   npx agent-browser --cdp 9222 fill @e<content_ref> "Your note content here..."
   ```
   - Content limit: ~1000 characters max

### Phase 5: Add Tags (Optional)

If tags are provided:

1. **Find tag input area** from snapshot

2. **Add each tag**:
   ```bash
   npx agent-browser --cdp 9222 click @e<add_tag_ref>
   npx agent-browser --cdp 9222 fill @e<tag_input_ref> "tag1"
   npx agent-browser --cdp 9222 press Enter
   ```
   - Repeat for each tag (max 5 recommended)

### Phase 6: Save as Draft (Default) or Publish

#### Default Action: Save as Draft

1. **Find "存草稿" button** from snapshot

2. **Click draft button**:
   ```bash
   npx agent-browser --cdp 9222 click @e<draft_button_ref>
   ```

3. **Verify success** - take snapshot or wait for confirmation

#### Only If User Explicitly Requests Publishing

**ONLY if user said "直接发布", "立即发布", or "publish now":**

1. **Find "发布" button** from snapshot

2. **Click publish button**:
   ```bash
   npx agent-browser --cdp 9222 click @e<publish_button_ref>
   ```

**When in doubt, ALWAYS save as draft.**

### Phase 7: Verify and Report

1. **Take final snapshot** to verify success:
   ```bash
   npx agent-browser --cdp 9222 snapshot -i
   ```

2. **Report to user**:
   - Draft saved: "草稿已保存！请在小红书 App 中预览和发布。"
   - Published: "笔记已发布成功！"

## Example Flows

### Example 1: Basic Image Publish

User: "发布这些图片到小红书: /path/to/photo1.jpg, /path/to/photo2.jpg, 标题是'周末好去处'"

```bash
# 1. Verify connection and check login
npx agent-browser --cdp 9222 snapshot -i

# 2. Navigate to publish page (if needed)
npx agent-browser --cdp 9222 open "https://creator.xiaohongshu.com/publish/publish"

# 3. Take snapshot, verify "上传图片" visible
npx agent-browser --cdp 9222 snapshot -i

# 4. Upload images
npx agent-browser --cdp 9222 upload @e<ref> "/path/to/photo1.jpg,/path/to/photo2.jpg"
npx agent-browser --cdp 9222 wait 3000

# 5. Take new snapshot, fill title
npx agent-browser --cdp 9222 snapshot -i
npx agent-browser --cdp 9222 fill @e<title_ref> "周末好去处"

# 6. Save as draft
npx agent-browser --cdp 9222 click @e<draft_ref>
```

### Example 2: Markdown File Publish

User: "把这个 markdown 发到小红书: /path/to/note.md"

```bash
# 1. Parse markdown
python ~/.claude/skills/xiaohongshu-publisher/scripts/parse_note.py /path/to/note.md

# 2. Extract: title, content, images, tags from JSON output

# 3. Verify connection
npx agent-browser --cdp 9222 snapshot -i

# 4. Upload all images from parsed result
npx agent-browser --cdp 9222 upload @e<ref> "/path/to/img1.jpg,/path/to/img2.jpg"

# 5. Fill title and content
npx agent-browser --cdp 9222 fill @e<title_ref> "parsed title"
npx agent-browser --cdp 9222 fill @e<content_ref> "parsed content"

# 6. Add tags if present
npx agent-browser --cdp 9222 fill @e<tag_ref> "tag1"
npx agent-browser --cdp 9222 press Enter

# 7. Save as draft
npx agent-browser --cdp 9222 click @e<draft_ref>
```

### Example 3: Direct Publish

User: "直接发布这些图到小红书，不用草稿"

```bash
# [Same as above until Step 6]

# Find and click "发布" button (NOT 存草稿)
npx agent-browser --cdp 9222 click @e<publish_ref>
```

## Critical Rules

1. **🚨 NEVER AUTO-PUBLISH** - ALWAYS save as draft by default
2. **🔌 ALWAYS USE --cdp 9222** - Every command must include this flag
3. **❌ NEVER USE close COMMAND** - Would close user's browser!
4. **📸 TAKE SNAPSHOTS FREQUENTLY** - Page state changes, always get fresh refs
5. **⏳ WAIT AFTER UPLOADS** - Give time for images to process
6. **🔄 HANDLE LOGIN GRACEFULLY** - Guide user to login in browser if needed
7. **📝 RESPECT CONTENT LIMITS** - Title ~20 chars, Content ~1000 chars
8. **🖼️ IMAGE LIMITS** - 1-18 images per note

## Troubleshooting

### CDP Connection Failed

If `snapshot` fails to connect:

1. **Verify Chrome is running with debug port**:
   ```bash
   lsof -i :9222
   ```

2. **If not running, start Chrome**:
   ```bash
   /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222
   ```

3. **Check for port conflicts** - another process might be using 9222

### Session Expired

If previously logged in but now seeing login page:

1. **Tell user to re-login in browser**: "您的登录状态已过期，请在浏览器中重新扫码登录"

2. **Wait for user confirmation**

3. **Take new snapshot to verify**

### Upload Failed

- Check image file exists and is valid format (jpg, png, gif, webp)
- Check file size (Xiaohongshu has limits)
- Try uploading one at a time
- Take screenshot to see actual error:
  ```bash
  npx agent-browser --cdp 9222 screenshot error.png
  ```

### Element Not Found

- Page structure may have changed
- Always take a fresh snapshot before interacting
- Look for similar elements with different refs

## Why CDP Mode with agent-browser?

### Advantages:

1. **No QR code scanning per session** - Login persists in browser
2. **User's own browser** - Existing cookies and preferences
3. **Simple CLI** - Same agent-browser commands, just add `--cdp 9222`
4. **No MCP server** - Direct CLI invocation
5. **Reliable auth** - Browser handles session management

### Trade-offs:

1. **Requires browser running** - User must start Chrome with debug port
2. **Single browser instance** - One session at a time
3. **Manual first login** - User does initial QR scan in browser

## Supported Content

| Type | Details |
|------|---------|
| Images | JPG, PNG, GIF, WebP (1-18 images) |
| Title | Up to ~20 characters recommended |
| Content | Up to ~1000 characters |
| Tags | Up to 5 tags recommended |


## 新手上路（用户不知道该输入什么时，走这里）

**触发**：`/小红书发布 新手`、「这个怎么用」「第一次用」「能干嘛」「带我走一遍」，
以及用户输入了技能名却没有给任何任务的时候。

这个模式的铁律：**不假设、不索取**。用户可能什么都没准备，
不要一上来就问他要文件、要 API key、要具体需求。按下面四步走：

**一、先说清楚这是什么（三句话以内）**

一句话：**把图文直接发到小红书草稿箱**。
它连的是你已经登录的浏览器，不需要你交出账号密码，
而且**永远只存草稿**，发不发由你在小红书里点。

**二、给编号选项，让他按回车就能继续**

不要问开放式问题（「你想做什么？」对新手是负担）。给 3 个选项加一个默认：

```
想先看哪个？（直接回车 = 1）
  1. 先看看整个流程长什么样（示例）
  2. 把这组图发成草稿
  3. 先讲讲要准备什么
```

**三、直接演示一遍，边做边解释**

选完立刻做给他看，用**示例数据**，不需要他提供任何东西。
每做完一步，加一行「💡 刚才发生了什么」，一句话说明这步的意义。

**四、毕业**

演示完只问一个是非题：「要不要用你自己的笔记真跑一遍？」
答是就进正常流程；答否就告诉他随时回来输 `/小红书发布 新手`。

**关于前置条件**：这个技能需要 一个开着调试端口的 Chrome，并且已经登录小红书创作服务平台。
**新手模式下不要提前索取**——先用示例数据演示完，到第四步真跑的时候再引导他配置，
并说清楚在哪配、怎么拿。新手最容易在这一步流失。

