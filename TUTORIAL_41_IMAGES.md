# Tutorial 41: Working with Images

> **Use images with Claude Code for visual analysis and debugging**

## Overview

Claude Code can analyze images for debugging, UI issues, and documentation.

---

## Part 1: Image Support

### Supported Formats

- PNG
- JPEG
- GIF
- WebP

### Common Use Cases

- Screenshot debugging
- UI/UX analysis
- Diagram interpretation
- Error message analysis

---

## Part 2: Sharing Images

### Drag and Drop

In VS Code extension:
1. Open Claude panel
2. Drag image into chat

### File Reference

```bash
Analyze this screenshot: @path/to/screenshot.png
```

### Paste from Clipboard

In supported IDEs, paste directly.

---

## Part 3: Use Cases

### UI Debugging

```bash
This screenshot shows a layout issue. 
@screenshots/bug.png
What's causing the misalignment?
```

### Error Analysis

```bash
Here's the error I'm seeing:
@screenshots/error.png
How do I fix this?
```

### Design Implementation

```bash
Implement this mockup:
@designs/new-feature.png
Create the React component.
```

---

## Part 4: Best Practices

### Image Quality

- Use clear, readable screenshots
- Crop to relevant area
- Include necessary context

### Combine with Code

```bash
This button doesn't work:
@screenshots/button.png

Here's the current code:
@src/components/Button.tsx
```

### Multiple Images

```bash
Compare before and after:
@screenshots/before.png
@screenshots/after.png
What changed?
```

---

## New in Claude Code v2.1.x

### Image Source Metadata (v2.1.2)

Images now include source metadata for better tracking:

```
[Image #1: screenshot.png, 1920x1080, 256KB]
```

Metadata includes:
- Original filename
- Dimensions
- File size
- Source location (local/clipboard/drag-drop)

### Clickable Image Links (v2.1.2)

Image references in output are now clickable:

```
I've analyzed [Image #1] and [Image #2] from your screenshots.
```

**Terminal behavior:**
- Click `[Image #1]` to open the image in your default viewer
- Requires OSC 8 hyperlink support in terminal
- Supported in iTerm2, Kitty, WezTerm, Windows Terminal

### Cmd+V Paste in iTerm2 (v2.1.0)

Direct clipboard paste support in iTerm2:

```
# In iTerm2, press Cmd+V to paste images
# Claude Code will detect and process the image
```

**Requirements:**
- iTerm2 3.5+ with image paste enabled
- Preferences → General → Magic → Enable Paste

### Image Display Improvements

Claude Code v2.1.x improves image handling:

```json
{
  "images": {
    "maxDisplayWidth": 800,
    "preserveAspectRatio": true,
    "showMetadata": true
  }
}
```

### Image Annotation Response

When analyzing images, Claude can now reference specific locations:

```
Looking at [Image #1]:
- The button at coordinates (245, 180) has incorrect padding
- The header section shows alignment issues at the top-left
- Error message visible in the bottom-right corner
```

### Multiple Image Workflows

Enhanced support for comparing multiple images:

```bash
# Reference numbered images in responses
> Compare these layouts:
> @v1/design.png
> @v2/design.png

# Claude response:
Comparing [Image #1] (v1) with [Image #2] (v2):
1. Header height changed from 60px to 80px
2. Sidebar width reduced in v2
3. New button added in [Image #2] footer
```

---

## Quick Reference

### Sharing Methods

| Method | How |
|--------|-----|
| File reference | `@path/to/image.png` |
| Drag and drop | IDE extension |
| Clipboard | Paste in supported IDEs |
| Cmd+V | iTerm2 direct paste (v2.1.0) |

### Common Tasks

| Task | Example |
|------|---------|
| Debug UI | "What's wrong with this layout?" |
| Implement design | "Create component from mockup" |
| Analyze error | "Fix this error message" |

---

## Sources

- [Common Workflows](https://code.claude.com/docs/en/common-workflows.md)
- [Claude Code Changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)

---

*This tutorial is part of the Claude Code documentation series.*
*Version 2.1.x | Last Updated: January 2026*
