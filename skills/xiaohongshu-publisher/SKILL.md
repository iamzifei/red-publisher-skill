---
name: xiaohongshu-publisher
description: Publish images and notes to Xiaohongshu (小红书) using agent-browser automation. Use when user wants to publish images to Xiaohongshu, or mentions "publish to 小红书", "post to Xiaohongshu", "发布到小红书", or wants help with Xiaohongshu note publishing. Handles QR code login and uploads images with text content. ALWAYS saves as draft by default.
---

# Xiaohongshu Publisher (小红书发布器)

Publish images and notes to Xiaohongshu (小红书) Creator Platform using agent-browser automation.

## ⚠️ IMPORTANT: Draft Mode by Default

**This skill ALWAYS saves notes as DRAFT by default. It will NEVER auto-publish.**

Only click the "发布" (publish) button if the user EXPLICITLY requests immediate publishing with phrases like:
- "直接发布" / "立即发布" / "马上发布"
- "publish now" / "publish directly" / "publish immediately"
- "不要草稿，直接发" / "不存草稿"

If unsure, ALWAYS save as draft and let user review before publishing.

## Prerequisites

- **agent-browser CLI**: Install via `npm install -g agent-browser` or use npx
- User must have a Xiaohongshu account
- Python 3.9+ with dependencies: `pip install Pillow pyobjc-framework-Cocoa`

## Scripts

Located in `~/.claude/skills/xiaohongshu-publisher/scripts/`:

### parse_note.py
Parse Markdown and extract structured data for Xiaohongshu notes:
```bash
python parse_note.py <markdown_file> [--output json]
```
Returns JSON with: title, content, images (list of paths)

### copy_to_clipboard.py
Copy image to system clipboard for pasting:
```bash
python copy_to_clipboard.py image /path/to/image.jpg [--quality 80]
```

## agent-browser Commands Reference

agent-browser is a CLI tool for browser automation. Key commands:

```bash
# Navigation
npx agent-browser open <url>          # Navigate to URL
npx agent-browser snapshot -i         # Get page snapshot with element refs

# Element Interaction
npx agent-browser click @e5           # Click element by ref
npx agent-browser fill @e2 "text"     # Fill input field
npx agent-browser type @e3 "text"     # Type text into element
npx agent-browser upload @e4 "/path/to/file.jpg"  # Upload file

# Keyboard & Wait
npx agent-browser press Enter         # Press key
npx agent-browser wait 2000           # Wait milliseconds
npx agent-browser wait --text "发布"  # Wait for text

# Screenshot & Close
npx agent-browser screenshot          # Take screenshot
npx agent-browser close               # Close browser
```

**Important**: Element refs (like @e5) come from `snapshot -i` output. Always take a snapshot before interacting.

## Workflow

### Phase 1: QR Code Login Handling

1. Navigate to Xiaohongshu creator page
2. Check if login is required (look for QR code)
3. If QR code detected:
   - **IMPORTANT**: Tell user to scan the QR code with Xiaohongshu app
   - Wait for login to complete (page will redirect)
   - Verify login success

### Phase 2: Upload Images and Create Note

1. Parse markdown/content to get images and text
2. Upload images (up to 18 images per note)
3. Fill in title and description
4. Add tags if specified
5. Save as draft OR publish (based on user preference)

## Step 1: Parse Content

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

If user provides images directly, use those paths.

## Step 2: Open Xiaohongshu Creator Platform

```bash
npx agent-browser open "https://creator.xiaohongshu.com/publish/publish"
```

Then take a snapshot:
```bash
npx agent-browser snapshot -i
```

## Step 3: Handle Login (QR Code)

Check the snapshot for login elements. If you see a QR code or login prompt:

1. **Tell the user**: "Please scan the QR code with your Xiaohongshu app to log in."
2. Wait for the user to confirm login, or poll for page changes:
```bash
npx agent-browser wait --text "上传图片" --timeout 120000
```

This waits up to 2 minutes for the "上传图片" (upload images) button to appear.

After login, take a new snapshot:
```bash
npx agent-browser snapshot -i
```

## Step 4: Upload Images

On the publish page, find the image upload area and upload images:

1. Take snapshot to get current element refs
2. Find the upload input element (usually a file input)
3. Upload each image:
```bash
npx agent-browser upload @e<ref> "/path/to/image1.jpg"
```

**For multiple images**, you can upload them comma-separated:
```bash
npx agent-browser upload @e<ref> "/path/to/img1.jpg,/path/to/img2.jpg,/path/to/img3.jpg"
```

Wait for uploads to complete:
```bash
npx agent-browser wait 3000
```

## Step 5: Fill Title and Content

After uploading images, fill in the note details:

1. Take snapshot to find title and content fields
2. Fill title (usually limited to certain characters):
```bash
npx agent-browser fill @e<title_ref> "Your Note Title"
```

3. Fill content/description:
```bash
npx agent-browser fill @e<content_ref> "Your note content here..."
```

## Step 6: Add Tags (Optional)

If tags are needed:
```bash
npx agent-browser click @e<add_tag_ref>
npx agent-browser fill @e<tag_input_ref> "tag1"
npx agent-browser press Enter
```

## Step 7: Save as Draft (MANDATORY DEFAULT)

**⚠️ CRITICAL: ALWAYS save as draft unless user EXPLICITLY requests immediate publishing.**

### Default Action: Save as Draft
Look for "存草稿" (save draft) button and click it:
```bash
npx agent-browser click @e<draft_button_ref>
```

### Only If User Explicitly Requests Publishing
ONLY click "发布" button if user said phrases like "直接发布", "立即发布", "publish now":
```bash
npx agent-browser click @e<publish_button_ref>
```

**When in doubt, ALWAYS save as draft.**

## Step 8: Verify and Report

Take final snapshot to verify success:
```bash
npx agent-browser snapshot -i
```

Report to user:
- "Draft saved successfully. Please review on Xiaohongshu before publishing."
- Or: "Note published successfully!"

## Critical Rules

1. **🚨 NEVER AUTO-PUBLISH** - ALWAYS save as draft by default. Only publish if user EXPLICITLY says "直接发布/立即发布/publish now"
2. **ALWAYS handle QR login** - Xiaohongshu requires login, notify user clearly
3. **Wait for user to scan QR code** - Don't proceed until login is confirmed
4. **Image limits** - Xiaohongshu allows 1-18 images per note
5. **Content limits** - Title: ~20 chars suggested, Content: ~1000 chars max
6. **Take snapshots frequently** - Page state changes, always get fresh refs
7. **Confirm draft saved** - After saving, verify success and tell user to review on Xiaohongshu app

## Example Flow

User: "发布这些图片到小红书: /path/to/photo1.jpg, /path/to/photo2.jpg, 标题是'周末好去处'"

```bash
# 1. Open creator page
npx agent-browser open "https://creator.xiaohongshu.com/publish/publish"

# 2. Take snapshot
npx agent-browser snapshot -i

# 3. If QR code visible, tell user and wait
# [TELL USER]: "Please scan the QR code with your Xiaohongshu app to log in."
npx agent-browser wait --text "上传图片" --timeout 120000

# 4. Take new snapshot after login
npx agent-browser snapshot -i

# 5. Upload images (find file input ref from snapshot)
npx agent-browser upload @e<ref> "/path/to/photo1.jpg,/path/to/photo2.jpg"
npx agent-browser wait 3000

# 6. Take snapshot, fill title
npx agent-browser snapshot -i
npx agent-browser fill @e<title_ref> "周末好去处"

# 7. ⚠️ ALWAYS save as draft (NOT publish!)
npx agent-browser click @e<draft_ref>  # Click "存草稿" button

# 8. Report success
# [TELL USER]: "草稿已保存！请在小红书 App 中查看并确认后再发布。"
```

**Note**: Even though user said "发布", we save as draft first. Only use "发布" button if user explicitly says "直接发布" or "立即发布".

## Troubleshooting

### QR Code Timeout
If user takes too long to scan:
- Re-take snapshot
- Check if still on login page
- Offer to restart the process

### Upload Failed
- Check image file exists and is valid format (jpg, png, gif, webp)
- Check file size (Xiaohongshu has limits)
- Try uploading one at a time

### Element Not Found
- Always take a fresh snapshot before interacting
- Page structure may have changed, look for similar elements
- Use `snapshot -i --json` for detailed structure

## Supported Content

| Type | Details |
|------|---------|
| Images | JPG, PNG, GIF, WebP (1-18 images) |
| Title | Up to ~20 characters recommended |
| Content | Up to ~1000 characters |
| Tags | Up to 5 tags recommended |

## Best Practices

### 为什么用 agent-browser 而非 Playwright MCP?

1. **CLI 优先**: 直接命令行调用，无需 MCP 服务器
2. **快速 snapshot**: 获取页面结构和元素引用
3. **简单文件上传**: `upload` 命令直接支持文件路径

### QR 码登录处理

1. **及时通知用户**: 发现 QR 码立即告知用户
2. **耐心等待**: 给用户足够时间扫码
3. **确认登录成功**: 检查页面跳转或元素变化

### 图片上传效率

- 多图可以一次性上传（逗号分隔路径）
- 每次上传后等待几秒确保完成
- 上传后取新 snapshot 继续操作
