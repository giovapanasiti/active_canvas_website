---
layout: docs
title: Visual Editor
description: Learn how to use the ActiveCanvas visual editor to build pages with drag-and-drop blocks, code editing, and more.
order: 2
---

# Visual Editor

The ActiveCanvas editor is a full-featured visual page builder powered by [GrapeJS](https://grapesjs.com/). It runs entirely within your Rails application -- no external services, no iframes to third-party tools.

---

## Editor interface overview

The editor is divided into three main areas:

| Area | Location | Purpose |
|------|----------|---------|
| **Canvas** | Center | Live preview of your page. Click to select elements, drag to reorder. |
| **Left panel** | Left sidebar | Block library, layers tree, and AI assistant. |
| **Right panel** | Right sidebar | Style properties, settings, and component options for the selected element. |
| **Toolbar** | Top bar | Device preview, undo/redo, code editor toggle, and save/publish controls. |

Click any element on the canvas to select it. Selected elements show resize handles and a blue outline. Use the layers panel on the left to navigate complex nested structures.

---

## Block types

Drag blocks from the left panel onto the canvas to build your page. ActiveCanvas includes these built-in block categories:

### Layout blocks
- **Section** -- Full-width container with configurable padding and background
- **Container** -- Centered max-width wrapper
- **Columns** -- Multi-column layout (2, 3, or 4 columns with responsive breakpoints)
- **Div** -- Generic block-level container

### Content blocks
- **Text** -- Rich text with inline editing (headings, paragraphs, lists)
- **Image** -- Single image with alt text, link, and sizing options
- **Video** -- Embedded video (YouTube, Vimeo, or self-hosted)
- **Link** -- Styled anchor element
- **List** -- Ordered or unordered list

### Component blocks
- **Button** -- Call-to-action button with style variants
- **Form** -- Contact or lead capture form with configurable fields
- **Table** -- Data table with editable rows and columns
- **Blockquote** -- Styled quote block
- **Separator** -- Horizontal rule with style options

### Media blocks
- **Image gallery** -- Grid or carousel image layout
- **Icon** -- SVG icon from the built-in set
- **Iframe** -- Embed external content

Each block is a fully editable HTML component. You can customize every aspect through the style panel or switch to the code editor for full control.

---

## Toolbar and properties panel

### Top toolbar

The top toolbar provides quick access to:

- **Device preview** -- Switch between desktop, tablet (768px), and mobile (320px) viewports
- **Undo / Redo** -- Step through your edit history
- **Component outline** -- Toggle element outlines for easier selection
- **Code editor** -- Open the Monaco-powered code panel
- **Full-screen** -- Expand the editor to fill the browser window

### Properties panel (right sidebar)

When an element is selected, the right panel shows:

- **Styles tab** -- Spacing (margin/padding), typography, background, borders, shadows, and layout (flexbox/grid) controls
- **Settings tab** -- Element-specific attributes like `href` for links, `src` for images, `alt` text, CSS classes, and custom attributes
- **Animations tab** -- Entrance animations and scroll-triggered effects

All style changes update the canvas in real time.

---

## Code editor panel

For developers who prefer working directly with code, ActiveCanvas includes a **Monaco-powered code editor** (the same editor that powers VS Code).

Toggle it from the toolbar or press the code editor button. The panel shows three tabs:

### HTML
Edit the raw HTML structure of your page. Changes sync to the visual canvas instantly.

```html
<section class="py-24 bg-gray-50">
  <div class="max-w-6xl mx-auto px-6">
    <h2 class="text-4xl font-bold text-center mb-12">
      Our Features
    </h2>
    <div class="grid grid-cols-3 gap-8">
      <!-- Feature cards here -->
    </div>
  </div>
</section>
```

### CSS
Add custom styles scoped to your page. These styles are included alongside your framework CSS.

```css
.custom-gradient {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}
```

### JavaScript
Add page-specific JavaScript that runs when the page loads. Useful for interactive components, form handling, or analytics events.

```javascript
document.querySelectorAll('.accordion-toggle').forEach(btn => {
  btn.addEventListener('click', () => {
    btn.nextElementSibling.classList.toggle('hidden');
  });
});
```

The code editor can be disabled globally by setting `config.enable_code_editor = false` in your initializer.

---

## Asset manager

The built-in asset manager connects to your **media library** (backed by Active Storage). Open it by clicking the image icon in the toolbar or by double-clicking any image element.

Features:

- **Browse** all uploaded media with thumbnail previews
- **Upload** new files by dragging them into the panel or clicking the upload button
- **Search** media by filename
- **Drag and drop** images directly from the asset manager onto the canvas
- **Delete** unused media files

Supported file types: JPEG, PNG, GIF, WebP, AVIF, and PDF. SVG uploads are disabled by default for security but can be enabled with `config.allow_svg_uploads = true`.

The asset manager can be toggled with `config.enable_asset_manager = false`.

---

## Auto-save

ActiveCanvas automatically saves your work at a configurable interval. The default is **60 seconds**.

How it works:

1. The editor tracks changes since the last save
2. After the configured interval, if there are unsaved changes, it sends the content to the server
3. A new **page version** is created with each save, so you can always roll back
4. The save status is shown in the toolbar (Saved / Unsaved changes / Saving...)

To change the interval, update your initializer:

```ruby
ActiveCanvas.configure do |config|
  # Auto-save every 30 seconds
  config.autosave_interval = 30

  # Disable auto-save (0 = off, manual save only)
  config.autosave_interval = 0
end
```

Version history is kept up to `max_versions_per_page` (default: 50). Older versions are pruned automatically.

---

## Responsive preview

The editor supports three viewport sizes for responsive design:

| Device | Width | Icon |
|--------|-------|------|
| Desktop | Full width | Monitor |
| Tablet | 768px | Tablet |
| Mobile | 320px | Phone |

Switch between devices using the toolbar buttons. The canvas resizes to match the selected viewport, letting you see exactly how your page will look on each device.

When using Tailwind CSS, you can use responsive utility prefixes (`md:`, `lg:`, etc.) in the code editor to create fully responsive layouts. The visual preview updates immediately.

---

## Component-level AI toolbar

When you select a component on the canvas, an AI toolbar appears alongside the standard editing options. This gives you quick access to AI-powered editing without opening the full AI panel.

Available actions:

- **Improve** -- Ask the AI to enhance the selected component's content or structure
- **Rewrite** -- Generate a new version of the selected element with different wording
- **Expand** -- Add more detail or sections to the selected content
- **Simplify** -- Reduce complexity while keeping the key message

The AI respects your configured CSS framework and generates code accordingly. See the [AI Features guide](/guides/ai-features.html) for full details on setting up and using AI capabilities.

---

## Keyboard shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl/Cmd + S` | Save page |
| `Ctrl/Cmd + Z` | Undo |
| `Ctrl/Cmd + Shift + Z` | Redo |
| `Delete / Backspace` | Delete selected element |
| `Ctrl/Cmd + C` | Copy selected element |
| `Ctrl/Cmd + V` | Paste element |
| `Escape` | Deselect current element |

---

## Next steps

- **[AI Features](/guides/ai-features.html)** -- Set up AI content generation for the editor
- **[Tailwind CSS](/guides/tailwind-css.html)** -- Understand how Tailwind is compiled for production
- **[Configuration](/guides/configuration.html)** -- Customize editor behavior and feature toggles
