---
layout: docs
title: Getting Started
description: Get ActiveCanvas up and running in your Rails app in under 5 minutes.
order: 1
---

# Getting Started

Get ActiveCanvas -- the AI-powered CMS engine -- installed and running in your Rails application in under five minutes.

---

## Prerequisites

Before you begin, make sure you have:

- **Ruby 3.1+** (check with `ruby -v`)
- **Rails 8.0+** (check with `rails -v`)
- **Active Storage** configured (the install generator will prompt you if needed)
- A working Rails application with a database

---

## Step 1: Add the gem

Add ActiveCanvas to your `Gemfile`:

```ruby
gem "active_canvas"
```

Then install it:

```bash
bundle install
```

---

## Step 2: Run the install generator

ActiveCanvas ships with an interactive setup wizard that walks you through every option:

```bash
bin/rails generate active_canvas:install
```

The wizard will:

1. **Copy database migrations** -- creates tables for pages, page types, page versions, media, AI models, and settings
2. **Choose a CSS framework** -- Tailwind CSS (recommended), Bootstrap 5, or none
3. **Configure AI features** -- optionally set up API keys for OpenAI, Anthropic, or OpenRouter
4. **Create the initializer** -- generates `config/initializers/active_canvas.rb` with your chosen settings
5. **Mount the engine** -- adds the route to your `config/routes.rb`

If you prefer to skip the wizard and do things manually, the initializer template and migration files are in the gem source.

---

## Step 3: Run migrations

If you didn't run migrations during the install wizard, do it now:

```bash
bin/rails db:migrate
```

This creates the following tables:

| Table | Purpose |
|-------|---------|
| `active_canvas_pages` | Stores page content, metadata, and compiled CSS |
| `active_canvas_page_types` | Defines categories like "Landing Page" or "Blog Post" |
| `active_canvas_page_versions` | Tracks every content revision |
| `active_canvas_media` | Media library records (files stored via Active Storage) |
| `active_canvas_ai_models` | Cached list of available AI models |
| `active_canvas_settings` | Key-value store for admin-configurable settings |
| `active_canvas_partials` | Reusable content partials |

---

## Step 4: Visit the admin panel

Start your Rails server and navigate to the admin interface:

```bash
bin/rails server
```

Then open your browser to:

```
http://localhost:3000/canvas/admin
```

> **Note:** By default, the admin panel is open to everyone in development. In production, ActiveCanvas will raise a `SecurityError` if authentication is not configured. See the [Authentication guide](/guides/authentication.html) to lock it down.

---

## Step 5: Create a page type

Before creating pages, you need at least one **page type**. Page types are categories like "Landing Page", "Blog Post", or "Documentation Page".

1. In the admin panel, navigate to **Page Types**
2. Click **New Page Type**
3. Enter a name (e.g., "Landing Page") and optional description
4. Save

---

## Step 6: Create your first page

1. Go to **Pages** in the admin panel
2. Click **New Page**
3. Fill in the title, slug, and select your page type
4. Click **Save** to create the page
5. Click **Open Editor** to launch the visual editor

You are now in the GrapeJS-powered visual editor. Drag blocks from the sidebar, edit text inline, and use the code editor panel for fine-grained control.

---

## Step 7: Publish and view

When you are ready to make the page public:

1. In the admin panel, set the page status to **Published**
2. Visit the public URL:

```
http://localhost:3000/canvas/your-page-slug
```

Published pages are served with compiled CSS (no CDN dependency), proper meta tags, and clean markup.

---

## What's next?

Now that you have ActiveCanvas running, explore these guides to get the most out of it:

- **[Visual Editor](/guides/visual-editor.html)** -- Learn the editor interface, block types, and responsive preview
- **[AI Features](/guides/ai-features.html)** -- Set up AI content generation, image creation, and screenshot-to-code
- **[Configuration](/guides/configuration.html)** -- Full reference for every config option
- **[Authentication](/guides/authentication.html)** -- Secure the admin panel with Devise, HTTP Basic, or custom logic
- **[Tailwind CSS](/guides/tailwind-css.html)** -- Understand how Tailwind compilation works in production

---

## Troubleshooting

**Migrations fail with Active Storage errors**

Make sure Active Storage is installed first:

```bash
bin/rails active_storage:install
bin/rails db:migrate
```

**Admin panel shows "Authentication not configured"**

This happens in production when `authenticate_admin` is not set. See the [Authentication guide](/guides/authentication.html).

**Pages show unstyled content**

Check that your CSS framework is configured correctly in **Admin > Settings**. If using Tailwind, make sure the `tailwindcss-rails` gem is installed for production compilation.
