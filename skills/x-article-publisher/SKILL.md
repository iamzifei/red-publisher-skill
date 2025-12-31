---
name: x-article-publisher
description: Publish Markdown articles to X (Twitter) Articles editor with proper formatting. Use when user wants to publish a Markdown file/URL to X Articles, or mentions "publish to X", "post article to Twitter", "X article", or wants help with X Premium article publishing. Handles cover image upload and converts Markdown to rich text automatically.
---

# X Article Publisher

Publish Markdown content to X (Twitter) Articles editor, preserving formatting with rich text conversion.

## Prerequisites

- Playwright MCP for browser automation
- User logged into X with Premium Plus subscription
- Python 3.9+ with dependencies: `pip install Pillow pyobjc-framework-Cocoa`

## Scripts

Located in `~/.claude/skills/x-article-publisher/scripts/`:

### parse_markdown.py
Parse Markdown and extract structured data:
```bash
python parse_markdown.py <markdown_file> [--output json|html] [--html-only]
```
Returns JSON with: title, cover_image, content_images (with block_index for positioning), html, total_blocks

### copy_to_clipboard.py
Copy image or HTML to system clipboard:
```bash
# Copy image (with optional compression)
python copy_to_clipboard.py image /path/to/image.jpg [--quality 80]

# Copy HTML for rich text paste
python copy_to_clipboard.py html --file /path/to/content.html
```

## Workflow

**Strategy: "先文后图" (Text First, Images Later)**

For articles with multiple images, paste ALL text content first, then insert images at correct positions using block index.

1. Parse Markdown with Python script → get title, images with block_index, HTML
2. Navigate to X Articles editor
3. Upload cover image (first image)
4. Fill title
5. Copy HTML to clipboard (Python) → Paste with Cmd+V
6. Insert content images at positions specified by block_index
7. Save as draft (NEVER auto-publish)

## Step 1: Parse Markdown (Python)

Use `parse_markdown.py` to extract all structured data:

```bash
python ~/.claude/skills/x-article-publisher/scripts/parse_markdown.py /path/to/article.md
```

Output JSON:
```json
{
  "title": "Article Title",
  "cover_image": "/path/to/first-image.jpg",
  "content_images": [
    {"path": "/path/to/img2.jpg", "block_index": 5, "after_text": "context for debugging..."},
    {"path": "/path/to/img3.jpg", "block_index": 12, "after_text": "another context..."}
  ],
  "html": "<p>Content...</p><h2>Section</h2>...",
  "total_blocks": 45
}
```

**Key fields:**
- `block_index`: The image should be inserted AFTER block element at this index (0-indexed)
- `total_blocks`: Total number of block elements in the HTML
- `after_text`: Kept for reference/debugging only, NOT for positioning

Save HTML to temp file for clipboard:
```bash
python parse_markdown.py article.md --html-only > /tmp/article_html.html
```

## Step 2: Open X Articles Editor

```
browser_navigate: https://x.com/compose/articles
```

**重要**: 页面加载后会显示草稿列表，不是编辑器。需要：

1. **等待页面加载完成**: 使用 `browser_snapshot` 检查页面状态
2. **立即点击 "create" 按钮**: 不要等待 "添加标题" 等编辑器元素，它们只有点击 create 后才出现
3. **等待编辑器加载**: 点击 create 后，等待编辑器元素出现

```
# 1. 导航到页面
browser_navigate: https://x.com/compose/articles

# 2. 获取页面快照，找到 create 按钮
browser_snapshot

# 3. 点击 create 按钮（通常 ref 类似 "create" 或带有 create 标签）
browser_click: element="create button", ref=<create_button_ref>

# 4. 现在编辑器应该打开了，可以继续上传封面图等操作
```

**注意**: 不要使用 `browser_wait_for text="添加标题"` 来等待页面加载，因为这个文本只有在点击 create 后才出现，会导致超时。

If login needed, prompt user to log in manually.

## Step 3: Upload Cover Image

1. Click "添加照片或视频" button
2. Use browser_file_upload with the cover image path (from JSON output)
3. Verify image uploaded

## Step 4: Fill Title

- Find textbox with "添加标题" placeholder
- Use browser_type to input title (from JSON output)

## Step 5: Paste Text Content (Python Clipboard)

Copy HTML to system clipboard using Python, then paste:

```bash
# Copy HTML to clipboard
python ~/.claude/skills/x-article-publisher/scripts/copy_to_clipboard.py html --file /tmp/article_html.html
```

Then in browser:
```
browser_click on editor textbox
browser_press_key: Meta+v
```

This preserves all rich text formatting (H2, bold, links, lists).

## Step 6: Insert Content Images (Block Index Positioning)

**关键改进**: 使用 `block_index` 精确定位，而非依赖文字匹配。

### 定位原理

粘贴 HTML 后，编辑器中的内容结构为一系列块元素（段落、标题、引用等）。每张图片的 `block_index` 表示它应该插入在第 N 个块元素之后。

### 操作步骤

1. **获取所有块元素**: 使用 browser_snapshot 获取编辑器内容，找到 textbox 下的所有子元素
2. **按索引定位**: 根据 `block_index` 点击对应的块元素
3. **粘贴图片**: 复制图片到剪贴板后粘贴

For each content image (from `content_images` array):

```bash
# 1. Copy image to clipboard (with compression)
python ~/.claude/skills/x-article-publisher/scripts/copy_to_clipboard.py image /path/to/img.jpg --quality 85
```

```
# 2. Click the block element at block_index
# Example: if block_index=5, click the 6th block element (0-indexed)
browser_click on the element at position block_index in the editor

# 3. Paste image
browser_press_key: Meta+v

# 4. Wait for upload (use short time, returns immediately when done)
browser_wait_for textGone="正在上传媒体" time=2
```

### 定位策略

在 browser_snapshot 返回的结构中，编辑器内容通常是：
```yaml
textbox [ref=xxx]:
  generic [ref=block0]:  # block_index 0
    - paragraph content
  heading [ref=block1]:   # block_index 1
    - h2 content
  generic [ref=block2]:  # block_index 2
    - paragraph content
  ...
```

要在 `block_index=5` 后插入图片：
1. 找到编辑器 textbox 下的第 6 个子元素（索引从0开始）
2. 点击该元素
3. 粘贴图片

**注意**: 每插入一张图片后，后续图片的实际位置会偏移。建议按 `block_index` **从大到小**的顺序插入图片，这样先插入的图片不会影响后续图片的索引。

### 反向插入示例

如果有3张图片，block_index 分别为 5, 12, 27：
1. 先插入 block_index=27 的图片
2. 再插入 block_index=12 的图片
3. 最后插入 block_index=5 的图片

这样每次插入都不会影响前面已经定位好的位置。

## Step 7: Save Draft

1. Verify content pasted (check word count indicator)
2. Draft auto-saves, or click Save button if needed
3. Click "预览" to verify formatting
4. Report: "Draft saved. Review and publish manually."

## Critical Rules

1. **NEVER publish** - Only save draft
2. **First image = cover** - Upload first image as cover image
3. **Rich text conversion** - Always convert Markdown to HTML before pasting
4. **Use clipboard API** - Paste via clipboard for proper formatting
5. **Block index positioning** - Use block_index for precise image placement
6. **Reverse order insertion** - Insert images from highest to lowest block_index
7. **H1 title handling** - H1 is used as title only, not included in body

## Supported Formatting

- H2 headers (## )
- Blockquotes (> )
- Code blocks (``` ... ```) - converted to blockquotes since X doesn't support `<pre><code>`
- Bold text (**)
- Hyperlinks ([text](url))
- Ordered lists (1. 2. 3.)
- Unordered lists (- )
- Paragraphs

## Example Flow

User: "Publish /path/to/article.md to X"

```bash
# Step 1: Parse Markdown
python ~/.claude/skills/x-article-publisher/scripts/parse_markdown.py /path/to/article.md > /tmp/article.json
python ~/.claude/skills/x-article-publisher/scripts/parse_markdown.py /path/to/article.md --html-only > /tmp/article_html.html
```

2. Navigate to https://x.com/compose/articles
3. Upload cover image (browser_file_upload for cover only)
4. Fill title (from JSON: `title`)
5. Copy & paste HTML:
   ```bash
   python ~/.claude/skills/x-article-publisher/scripts/copy_to_clipboard.py html --file /tmp/article_html.html
   ```
   Then: browser_press_key Meta+v
6. For each content image, **in reverse order of block_index**:
   ```bash
   python copy_to_clipboard.py image /path/to/img.jpg --quality 85
   ```
   - Click block element at `block_index` position
   - browser_press_key Meta+v
   - Wait until upload complete
7. Verify in preview
8. "Draft saved. Please review and publish manually."

## Best Practices

### 为什么用 block_index 而非文字匹配？

1. **精确定位**: 不依赖文字内容，即使多处文字相似也能正确定位
2. **可靠性**: 索引是确定性的，不会因为文字相似而混淆
3. **调试方便**: `after_text` 仍保留用于人工核验

### 为什么用 Python 而非浏览器内 JavaScript？

1. **本地处理更可靠**: Python 直接操作系统剪贴板，不受浏览器沙盒限制
2. **图片压缩**: 上传前压缩图片 (--quality 85)，减少上传时间
3. **代码复用**: 脚本固定不变，无需每次重新编写转换逻辑
4. **调试方便**: 脚本可单独测试，问题易定位

### 等待策略

**关键理解**: `browser_wait_for` 的 `textGone` 参数会在文字消失时**立即返回**，`time` 只是最大等待时间，不是固定等待时间。

- ❌ 保守等待: `time=5` 或 `time=10`，如果上传只需2秒，剩余时间全浪费
- ✅ 短间隔轮询: `time=2`，条件满足立即返回，最多等2秒

```
# 正确用法：短 time 值，条件满足立即返回
browser_wait_for textGone="正在上传媒体" time=2

# 错误用法：固定长时间等待
browser_wait_for time=5  # 无条件等待5秒，浪费时间
```

**原则**: 不要预设"需要等多久"，而是设置一个合理的最大值，让条件检测尽快返回。

### 图片插入效率

每张图片的浏览器操作从5步减少到2步：
- 旧: 点击 → 添加媒体 → 媒体 → 添加照片 → file_upload
- 新: 点击段落 → Meta+v

### 封面图 vs 内容图

- **封面图**: 使用 browser_file_upload（因为有专门的上传按钮）
- **内容图**: 使用 Python 剪贴板 + 粘贴（更高效）
