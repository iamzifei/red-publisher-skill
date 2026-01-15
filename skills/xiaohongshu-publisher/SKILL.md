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

## 🔐 Multi-Account Support (多账号支持)

This skill supports multiple Xiaohongshu accounts with easy switching.

### Auth State File Location
Each account has its own auth state file:
```
~/.agent-browser/xiaohongshu-auth-<account_name>.json
```

Examples:
```
~/.agent-browser/xiaohongshu-auth-default.json    # Default account
~/.agent-browser/xiaohongshu-auth-work.json       # Work account
~/.agent-browser/xiaohongshu-auth-personal.json   # Personal account
```

### Account Management Commands

#### List All Saved Accounts
```bash
ls ~/.agent-browser/xiaohongshu-auth-*.json 2>/dev/null | sed 's/.*xiaohongshu-auth-\(.*\)\.json/\1/'
```

#### Add New Account
```bash
# 1. Open browser (without loading existing state)
npx agent-browser open "https://creator.xiaohongshu.com/publish/publish"

# 2. Scan QR code to login

# 3. Save with account name
npx agent-browser state save ~/.agent-browser/xiaohongshu-auth-<account_name>.json
```

#### Switch Account (Login with Different Account)
```bash
# Load specific account's auth state
npx agent-browser --state ~/.agent-browser/xiaohongshu-auth-<account_name>.json open "https://creator.xiaohongshu.com/publish/publish"
```

#### Delete Account
```bash
rm ~/.agent-browser/xiaohongshu-auth-<account_name>.json
```

### Account Selection Logic

When user wants to publish, determine which account to use:

1. **User specifies account**: "用工作账号发布" → use `xiaohongshu-auth-work.json`
2. **User says "切换账号"**: List available accounts and let user choose
3. **No account specified**: Use `xiaohongshu-auth-default.json`
4. **No saved accounts**: Prompt for QR login and save as `default`

### Trigger Phrases for Account Operations

| User Says | Action |
|-----------|--------|
| "用XX账号发布" / "使用XX账号" | Load `xiaohongshu-auth-XX.json` |
| "切换账号" / "换个账号" / "switch account" | List accounts, let user choose |
| "添加账号" / "新账号" / "add account" | QR login and save with new name |
| "删除账号" / "remove account" | Delete specified auth file |
| "列出账号" / "list accounts" | Show all saved accounts |

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

### Phase 0: Account Selection (多账号选择)

1. **Determine which account to use**:
   - If user specifies: "用XX账号" → `account_name = "XX"`
   - If user says "切换账号": List accounts and ask user to choose
   - Otherwise: `account_name = "default"`

2. **Check if account exists**:
   ```bash
   ls ~/.agent-browser/xiaohongshu-auth-${account_name}.json 2>/dev/null
   ```

3. **List available accounts** (if user asks or needs to choose):
   ```bash
   echo "已保存的账号："
   ls ~/.agent-browser/xiaohongshu-auth-*.json 2>/dev/null | while read f; do
     name=$(basename "$f" | sed 's/xiaohongshu-auth-\(.*\)\.json/\1/')
     echo "  - $name"
   done
   ```

### Phase 1: Login Handling (with Multi-Account Support)

1. **Try to load saved auth state for selected account**:
   ```bash
   AUTH_FILE=~/.agent-browser/xiaohongshu-auth-${account_name}.json
   if [ -f "$AUTH_FILE" ]; then
     npx agent-browser --state "$AUTH_FILE" open "https://creator.xiaohongshu.com/publish/publish"
   else
     npx agent-browser open "https://creator.xiaohongshu.com/publish/publish"
   fi
   ```

2. **Check if login is still required** (take snapshot, look for QR code or login form)

3. **If already logged in**: Proceed to Phase 2

4. **If login required** (no saved state or state expired):
   - Open page without state: `npx agent-browser open "https://creator.xiaohongshu.com/publish/publish"`
   - Switch to QR code login if needed (use JavaScript to click QR toggle)
   - **TELL USER**: "请使用小红书 App 扫描二维码登录"
   - Wait for login to complete
   - **Ask user for account name** (if not specified): "请为此账号起个名字（如：work, personal, default）"
   - **SAVE AUTH STATE** after successful login:
     ```bash
     mkdir -p ~/.agent-browser
     npx agent-browser state save ~/.agent-browser/xiaohongshu-auth-${account_name}.json
     ```
   - Tell user: "✅ 登录成功！账号「${account_name}」已保存，下次可直接使用。"

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

## Step 2: Open Xiaohongshu Creator Platform (with Multi-Account)

**Determine account and load auth state:**
```bash
# Set account name (default if not specified by user)
ACCOUNT_NAME="${account_name:-default}"
AUTH_FILE=~/.agent-browser/xiaohongshu-auth-${ACCOUNT_NAME}.json

# Check if auth state file exists and load it
if [ -f "$AUTH_FILE" ]; then
  echo "使用账号: $ACCOUNT_NAME"
  npx agent-browser --state "$AUTH_FILE" open "https://creator.xiaohongshu.com/publish/publish"
else
  echo "账号 $ACCOUNT_NAME 未登录，需要扫码"
  npx agent-browser open "https://creator.xiaohongshu.com/publish/publish"
fi
```

Then take a snapshot:
```bash
npx agent-browser snapshot -i
```

## Step 3: Handle Login (with Multi-Account Support)

Check the snapshot for login elements or upload button.

### If "上传图片" button visible → Already logged in, skip to Step 4

### If login required (QR code or login form visible):

1. **Switch to QR code login** (if showing SMS form):
```bash
npx agent-browser eval "const img = document.querySelector('img.css-wemwzq'); if(img) { img.click(); 'clicked'; }"
```

2. **Tell the user**: "请使用小红书 App 扫描二维码登录"

3. Wait for login to complete:
```bash
npx agent-browser wait --text "上传图片" --timeout 120000
```

4. **Ask for account name if not specified**:
   - If user didn't specify account name, ask: "请为此账号起个名字（如：work, personal）,或直接回复「default」"

5. **Save auth state with account name**:
```bash
mkdir -p ~/.agent-browser
ACCOUNT_NAME="${account_name:-default}"
npx agent-browser state save ~/.agent-browser/xiaohongshu-auth-${ACCOUNT_NAME}.json
```

6. **Tell user**: "✅ 登录成功！账号「${ACCOUNT_NAME}」已保存。下次使用此账号发布时无需扫码。"

7. Take a new snapshot:
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
2. **👥 SUPPORT MULTI-ACCOUNT** - Use account name in auth file: `xiaohongshu-auth-<account>.json`
3. **🔐 TRY SAVED AUTH STATE FIRST** - Always try loading account's auth file before asking user to scan QR
4. **💾 SAVE AUTH STATE WITH ACCOUNT NAME** - After successful QR login, ask for account name and save
5. **🔄 HANDLE ACCOUNT SWITCHING** - When user says "切换账号", list accounts and let them choose
6. **ALWAYS handle QR login** - If no saved state or state expired, notify user clearly to scan QR
7. **Wait for user to scan QR code** - Don't proceed until login is confirmed
8. **Image limits** - Xiaohongshu allows 1-18 images per note
9. **Content limits** - Title: ~20 chars suggested, Content: ~1000 chars max
10. **Take snapshots frequently** - Page state changes, always get fresh refs
11. **Confirm draft saved** - After saving, verify success and tell user to review on Xiaohongshu app

## Example Flows

### Example 1: Basic Publish (Default Account)

User: "发布这些图片到小红书: /path/to/photo1.jpg, /path/to/photo2.jpg, 标题是'周末好去处'"

```bash
# 1. Use default account
ACCOUNT_NAME="default"
AUTH_FILE=~/.agent-browser/xiaohongshu-auth-${ACCOUNT_NAME}.json

# 2. Check for saved auth state and open creator page
if [ -f "$AUTH_FILE" ]; then
  npx agent-browser --state "$AUTH_FILE" open "https://creator.xiaohongshu.com/publish/publish"
else
  npx agent-browser open "https://creator.xiaohongshu.com/publish/publish"
fi

# 3. Take snapshot to check login status
npx agent-browser snapshot -i

# 4. If login required, handle QR code and save with account name
# ... (see Step 3 for details)
npx agent-browser state save ~/.agent-browser/xiaohongshu-auth-${ACCOUNT_NAME}.json

# 5. Upload and fill content
npx agent-browser upload @e<ref> "/path/to/photo1.jpg,/path/to/photo2.jpg"
npx agent-browser fill @e<title_ref> "周末好去处"

# 6. Save as draft
npx agent-browser click @e<draft_ref>
```

### Example 2: Publish with Specific Account

User: "用工作账号发布这些图片到小红书"

```bash
# Use "work" account
ACCOUNT_NAME="work"
AUTH_FILE=~/.agent-browser/xiaohongshu-auth-work.json

npx agent-browser --state "$AUTH_FILE" open "https://creator.xiaohongshu.com/publish/publish"
# ... rest of the flow
```

### Example 3: Switch Account

User: "切换账号" or "换个账号发布"

```bash
# 1. List available accounts
echo "已保存的账号："
ls ~/.agent-browser/xiaohongshu-auth-*.json 2>/dev/null | while read f; do
  name=$(basename "$f" | sed 's/xiaohongshu-auth-\(.*\)\.json/\1/')
  echo "  - $name"
done

# 2. [ASK USER]: "请选择要使用的账号，或输入「新账号」添加新账号"
# 3. Load selected account or proceed with new login
```

### Example 4: Add New Account

User: "添加新账号" or "登录另一个小红书账号"

```bash
# 1. Open without loading any state
npx agent-browser open "https://creator.xiaohongshu.com/publish/publish"

# 2. Handle QR login
# 3. [ASK USER]: "请为此账号起个名字（如：personal, work, shop）"
# 4. Save with new account name
ACCOUNT_NAME="<user_input>"
npx agent-browser state save ~/.agent-browser/xiaohongshu-auth-${ACCOUNT_NAME}.json
# 5. [TELL USER]: "✅ 账号「${ACCOUNT_NAME}」已添加！"
```

**Note**: Even though user said "发布", we save as draft first. Only use "发布" button if user explicitly says "直接发布" or "立即发布".

## Troubleshooting

### Auth State Expired
If saved auth state no longer works:
```bash
# Delete old auth state for specific account
rm ~/.agent-browser/xiaohongshu-auth-<account_name>.json

# Open without state, re-login, and save new state
npx agent-browser open "https://creator.xiaohongshu.com/publish/publish"
# (scan QR code)
npx agent-browser state save ~/.agent-browser/xiaohongshu-auth-<account_name>.json
```

### List All Accounts
```bash
ls ~/.agent-browser/xiaohongshu-auth-*.json 2>/dev/null | \
  sed 's/.*xiaohongshu-auth-\(.*\)\.json/\1/' | \
  while read name; do echo "  - $name"; done
```

### Delete Specific Account
```bash
rm ~/.agent-browser/xiaohongshu-auth-<account_name>.json
```

### Delete All Accounts
```bash
rm ~/.agent-browser/xiaohongshu-auth-*.json
```

### QR Code Timeout
If user takes too long to scan:
- Re-take snapshot
- Check if still on login page
- Offer to restart the process

### QR Code Not Visible (SMS form showing)
Use JavaScript to click the QR code toggle:
```bash
npx agent-browser eval "const img = document.querySelector('img.css-wemwzq'); if(img) { img.click(); 'clicked'; }"
```

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
