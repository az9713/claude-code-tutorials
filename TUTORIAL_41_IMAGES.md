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

## Quick Reference

### Sharing Methods

| Method | How |
|--------|-----|
| File reference | `@path/to/image.png` |
| Drag and drop | IDE extension |
| Clipboard | Paste in supported IDEs |

### Common Tasks

| Task | Example |
|------|---------|
| Debug UI | "What's wrong with this layout?" |
| Implement design | "Create component from mockup" |
| Analyze error | "Fix this error message" |

---

## Sources

- [Common Workflows](https://code.claude.com/docs/en/common-workflows.md)
