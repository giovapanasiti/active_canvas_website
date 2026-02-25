---
layout: docs
title: Tailwind CSS
description: Learn how ActiveCanvas handles Tailwind CSS compilation for production-ready pages without CDN dependency.
order: 6
---

# Tailwind CSS

ActiveCanvas uses a two-phase approach to Tailwind CSS: the CDN for instant editor previews, and local compilation for production-ready output. This gives you the best of both worlds -- fast editing with zero-bloat deployment.

---

## How it works

### In the editor (development)

The visual editor loads Tailwind CSS via the **browser CDN** (`@tailwindcss/browser@4`). This means:

- Every Tailwind utility class is available instantly
- No build step needed while editing
- Changes preview in real time
- The full Tailwind v4 feature set is supported

### On save (compilation)

When a page is saved, ActiveCanvas compiles the page's HTML through the **local Tailwind CLI**. The compiler:

1. Scans the page content for used Tailwind classes
2. Generates a minimal CSS file containing only those classes
3. Stores the compiled CSS in the `compiled_css` column on the page record

### In production (serving)

Published pages are served with the **compiled CSS inlined** in a `<style>` tag. No CDN, no external requests, no unused CSS. The result is a fast, self-contained page.

```
Editor: CDN (all classes available)
    |
    v
  Save: Tailwind CLI compiles used classes only
    |
    v
Production: Inline <style> with minimal CSS
```

---

## Setup

### 1. Install the tailwindcss-ruby gem

ActiveCanvas uses the `tailwindcss-ruby` gem for compilation. The install generator will offer to add it for you, or you can do it manually:

```ruby
# Gemfile
gem "tailwindcss-rails"
```

```bash
bundle install
```

The `tailwindcss-rails` gem bundles the Tailwind CLI binary -- no Node.js required.

### 2. Configure the framework

In your initializer (or via Admin > Settings):

```ruby
ActiveCanvas.configure do |config|
  config.css_framework = :tailwind
end
```

### 3. Verify compilation is available

You can check if Tailwind compilation is working:

```ruby
ActiveCanvas.config.tailwind_compilation_available?
# => true (if tailwindcss-ruby is installed and framework is :tailwind)

ActiveCanvas::TailwindCompiler.available?
# => true
```

---

## How compilation works

When a page is saved, the `TailwindCompiler` runs the following process:

1. Creates a temporary directory
2. Writes the page HTML to a temp file
3. Generates an input CSS file with:
   - The Tailwind v4 import (`@import "tailwindcss"`)
   - A `@source` directive pointing at the HTML file
   - Custom theme values (colors, fonts) from admin settings
4. Runs the Tailwind CLI with `--minify`
5. Reads the compiled output and stores it on the page

The compiler runs per-page, so each page gets its own minimal CSS bundle. A typical compiled output is 5-15 KB for a full landing page (compared to 300+ KB for the full Tailwind CDN).

### Example input CSS

The compiler generates this input for the Tailwind CLI:

```css
@import "tailwindcss";
@source "/tmp/active_canvas_tailwind/input.html";
@theme { --color-brand: #e8b84b; }
@theme { --font-display: "Sora", sans-serif; }
```

---

## Custom theme configuration

You can extend the Tailwind theme with custom colors and fonts through the Admin UI at **Admin > Settings > Tailwind**.

### Custom colors

Add brand colors that are available as Tailwind utilities:

```ruby
# Configured via Admin > Settings > Tailwind
# Stored as JSON in the settings table
{
  "theme": {
    "extend": {
      "colors": {
        "brand": "#e8b84b",
        "brand-dark": "#c49a2a",
        "accent": "#ef4444"
      }
    }
  }
}
```

These become available as `bg-brand`, `text-brand-dark`, `border-accent`, etc.

### Custom fonts

Add font families that work with Tailwind's `font-*` utilities:

```ruby
{
  "theme": {
    "extend": {
      "fontFamily": {
        "display": ["Sora", "sans-serif"],
        "body": ["Nunito Sans", "sans-serif"]
      }
    }
  }
}
```

These enable `font-display` and `font-body` utility classes.

> **Note:** Custom theme values are applied during compilation via `@theme` directives. They are also injected into the CDN preview in the editor so what you see matches what gets compiled.

---

## Bootstrap 5 alternative

If you prefer Bootstrap, ActiveCanvas supports it as a first-class option:

```ruby
ActiveCanvas.configure do |config|
  config.css_framework = :bootstrap5
end
```

With Bootstrap 5:

- The editor loads Bootstrap from the CDN (`bootstrap@5.3.3`)
- AI generates Bootstrap-compatible HTML (grid system, components, utility classes)
- Published pages include the Bootstrap CDN link
- No local compilation step (Bootstrap is served from CDN in both editor and production)

---

## No framework option

For maximum control, you can run without any CSS framework:

```ruby
ActiveCanvas.configure do |config|
  config.css_framework = :none
end
```

With no framework:

- No CSS is loaded in the editor beyond what you add manually
- AI generates HTML with inline styles
- You can add your own CSS via the code editor or global CSS settings
- Published pages include only your custom styles

---

## Bulk recompile

When you update your Tailwind theme configuration (colors, fonts, etc.), existing pages still have CSS compiled with the old theme. You need to recompile them.

### When to recompile

- After changing custom colors or fonts in Admin > Settings > Tailwind
- After upgrading the `tailwindcss-rails` gem (which may include a new Tailwind version)
- After modifying global CSS that affects Tailwind's output

### How to trigger

From the **Admin > Settings > Tailwind** page, click the **Recompile All Pages** button. This queues a background job that recompiles every published page.

Programmatically:

```ruby
# Recompile a single page
page = ActiveCanvas::Page.find(1)
compiled_css = ActiveCanvas::TailwindCompiler.compile_for_page(page)
page.update(compiled_css: compiled_css)

# Recompile all published pages
ActiveCanvas::Page.published.find_each do |page|
  ActiveCanvas::CompileTailwindJob.perform_later(page.id)
end
```

---

## Troubleshooting

### "tailwindcss-ruby gem is not installed"

The compiler cannot find the Tailwind CLI binary. Make sure `tailwindcss-rails` is in your Gemfile and you have run `bundle install`.

```bash
bundle list | grep tailwind
```

### Compiled CSS is empty

This usually means the HTML content has no Tailwind classes, or the content is blank. Check:

```ruby
page = ActiveCanvas::Page.find(1)
page.content.present?  # Should be true
```

### Classes are missing from compiled output

The Tailwind CLI only includes classes it finds in the HTML. If you are generating classes dynamically with JavaScript (e.g., `classList.add('hidden')`), those classes will not be in the compiled CSS.

Workaround: Add a comment in the HTML with the dynamic classes so the compiler picks them up:

```html
<!-- Tailwind safelist: hidden block opacity-0 opacity-100 transition-opacity -->
```

### Compilation is slow

Tailwind compilation typically takes 200-500ms per page. If it is significantly slower:

- Check that you are using the latest `tailwindcss-rails` gem
- Ensure the temp directory (`/tmp`) has fast I/O
- For bulk recompilation, use background jobs (the default) rather than inline compilation

### Styles look different in editor vs production

The editor uses the Tailwind CDN (full framework), while production uses compiled CSS (only used classes). If a class appears in production but was not compiled:

1. Check that the class is present in the page's HTML content
2. Recompile the page after making changes
3. Verify theme customizations are applied correctly

---

## Next steps

- **[Visual Editor](/guides/visual-editor.html)** -- Using Tailwind classes in the visual editor
- **[AI Features](/guides/ai-features.html)** -- AI generates framework-aware Tailwind code
- **[Configuration](/guides/configuration.html)** -- All CSS framework configuration options
