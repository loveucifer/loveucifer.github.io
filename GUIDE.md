# Blog Guide

A minimal Hugo blog with multi-theme support, LaTeX math, and image support.

## Quick Start

### Run Locally
```bash
hugo server
```
Visit `http://localhost:1313`

### Build for Production
```bash
hugo
```

### Create a New Post
```bash
hugo new posts/my-new-post.md
```
Edit the file in `content/posts/`

---

## Writing Posts

Create `content/posts/my-post.md`:

```markdown
---
title: My Post Title
date: 2026-01-15
draft: false
---

Your content here...

## Section Heading
```

**Front Matter Fields:**

| Field | Description |
|-------|-------------|
| `title` | Post title |
| `date` | Publication date (YYYY-MM-DD) |
| `draft` | Set to `true` to hide from production |
| `description` | Meta description for SEO |

---

## Markdown Features

### Text Formatting
- **bold** with `**text**`
- *italic* with `*text*`
- `code` with backticks

### Code Blocks
````markdown
```python
print("hello world")
```
````

### Images
Place images in `static/images/` then reference:
```markdown
![alt text](/images/photo.jpg)
```

### Links
```markdown
[link text](https://example.com)
```

### Blockquotes
```markdown
> This is a quote
```

### Lists
```markdown
- item 1
- item 2
  - nested item
```

---

## LaTeX Math

Site uses MathJax for rendering.

**Inline math:** `$E = mc^2$`

**Block math:**
```markdown
$$
\int_{0}^{\infty} e^{-x^2} dx = \frac{\sqrt{\pi}}{2}
$$
```

**Examples:**

| Syntax | Output |
|--------|--------|
| `$x^2$` | x² |
| `$\frac{a}{b}$` | a/b fraction |
| `$\sqrt{x}$` | √x |
| `$\sum_{i=1}^n$` | summation |
| `$\alpha, \beta, \gamma$` | greek letters |

---

## Themes

Switch themes using the buttons in the top-left corner:

- **BW** - Black & white (default)
- **Temple** - Temple OS style (light blue)
- **White** - Light mode

Theme preference is saved to localStorage.

---

## Comments

Comments powered by [Giscus](https://giscus.app) (GitHub Discussions).

Already configured for this repo - comments appear at the bottom of each post.

---

## Deployment

Push to GitHub - CI automatically builds and deploys to GitHub Pages.

---

## File Structure

```
.
├── content/
│   └── posts/         # Blog posts (*.md)
├── layouts/           # Hugo templates
├── static/
│   └── images/       # Images (put your images here)
├── hugo.toml          # Hugo config
└── public/           # Build output (generated)
```

---

## Tips

- Posts go in `content/posts/`
- Use `draft: true` to hide from production
- Put images in `static/images/` folder
- Commit and push - CI handles the rest