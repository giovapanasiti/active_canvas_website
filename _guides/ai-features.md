---
layout: docs
title: AI Features
description: Set up and use AI content generation, image creation, and screenshot-to-code in ActiveCanvas.
order: 3
---

# AI Features

ActiveCanvas includes built-in AI capabilities for generating text/HTML, creating images, and converting screenshots to production code. All AI features are powered by [RubyLLM](https://github.com/crmne/ruby_llm) and support OpenAI, Anthropic, and OpenRouter as providers.

---

## API key setup

ActiveCanvas needs at least one API key to enable AI features. You can configure keys in three ways:

### Option 1: Admin UI

Navigate to **Admin > Settings > AI** and enter your API keys directly. Keys are encrypted at rest in the database using your Rails `secret_key_base`.

### Option 2: Environment variables

Set the standard environment variables that RubyLLM expects:

```bash
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."
export OPENROUTER_API_KEY="sk-or-..."
```

Then enter these same values in the Admin UI so ActiveCanvas can read them.

### Option 3: Rails credentials

Store keys in your encrypted credentials file:

```bash
bin/rails credentials:edit
```

```yaml
active_canvas:
  openai_api_key: sk-...
  anthropic_api_key: sk-ant-...
  openrouter_api_key: sk-or-...
```

Then reference them in the Admin UI or set them programmatically:

```ruby
ActiveCanvas::Setting.ai_openai_api_key = Rails.application.credentials.dig(:active_canvas, :openai_api_key)
ActiveCanvas::Setting.ai_anthropic_api_key = Rails.application.credentials.dig(:active_canvas, :anthropic_api_key)
```

> **Tip:** You only need keys for the providers you plan to use. One provider is enough to get started.

---

## Syncing models

After configuring API keys, you need to sync the list of available models from your providers. This fetches model metadata (names, capabilities, token limits) and stores it locally.

### Via rake task

```bash
bin/rails active_canvas:sync_models
```

Output:

```
Configured providers: openai, anthropic
Syncing models...
Done! Synced 47 models.

Breakdown:
  Text models:   38
  Image models:  3
  Vision models: 6
```

### Via Admin UI

Navigate to **Admin > Settings > AI** and click the **Sync Models** button.

### Listing models

To see all synced models:

```bash
bin/rails active_canvas:list_models
```

Models are categorized as **text** (chat/generation), **image** (DALL-E), and **vision** (models that accept image input). Default models are:

| Type | Default model |
|------|---------------|
| Text | `gpt-4o-mini` |
| Image | `dall-e-3` |
| Vision | `gpt-4o` |

You can change defaults in **Admin > Settings > AI**.

---

## Chat / Generate

The **Generate** feature lets you describe what you want and receive streaming HTML output. It is accessible from the AI panel in the editor's left sidebar.

### How it works

1. Select a model (or use the default)
2. Type a prompt like "Create a pricing table with 3 tiers and a highlighted popular plan"
3. The AI streams HTML back in real time using Server-Sent Events (SSE)
4. The generated code is inserted into the editor canvas

### Framework-aware prompts

ActiveCanvas automatically includes your configured CSS framework in the system prompt. The AI knows to:

- Use **Tailwind CSS v4** utility classes when Tailwind is configured
- Use **Bootstrap 5** components and grid when Bootstrap is configured
- Use **inline styles** with modern CSS when no framework is set

This means generated code is always production-ready for your specific setup.

### Element vs page mode

The Generate feature operates in two modes:

- **Page mode** -- Generates a complete section or component to insert into the page
- **Element mode** -- Modifies or enhances an existing selected element. The current HTML is sent as context so the AI can build on what is already there.

---

## Image generation

The **Image** tab in the AI panel uses DALL-E to generate images from text descriptions.

### How it works

1. Switch to the **Image** tab
2. Enter a description (e.g., "A minimalist illustration of a rocket launching from a laptop")
3. Click **Generate**
4. The image is created via DALL-E, downloaded, and stored in your **media library** via Active Storage
5. Drag the generated image from the media library onto your page

Generated images are tagged with `ai_generated: true` in their metadata, along with the original prompt.

### Storage

Images are stored using whatever Active Storage backend you have configured (local disk, S3, GCS, Azure). They are full media library assets -- you can reuse them across any page.

---

## Screenshot to code

The **Screenshot** tab converts an uploaded image into production HTML.

### How it works

1. Switch to the **Screenshot** tab
2. Upload a screenshot (PNG, JPEG, WebP, or GIF)
3. Optionally add extra instructions (e.g., "Use a dark color scheme" or "Make the buttons rounded")
4. The screenshot is sent to a vision model (default: `gpt-4o`) which analyzes the layout
5. The AI returns clean HTML that matches the screenshot's layout, colors, and typography
6. The code is inserted into your editor

### Supported formats

| Format | Max size |
|--------|----------|
| PNG | 10 MB |
| JPEG / JPG | 10 MB |
| WebP | 10 MB |
| GIF | 10 MB |

The screenshot is validated with magic byte checking to prevent spoofed file types.

---

## Rate limiting

AI requests are rate-limited to prevent abuse. The default limit is **30 requests per minute per IP address**.

```ruby
ActiveCanvas.configure do |config|
  # Allow 60 AI requests per minute per IP
  config.ai_rate_limit_per_minute = 60
end
```

When the limit is hit, the API returns a `429 Too Many Requests` response. The editor shows a user-friendly message.

---

## Server vs direct mode

ActiveCanvas supports two AI connection modes, configurable in **Admin > Settings > AI**:

### Server mode (default)

All AI requests are routed through your Rails server. This is the recommended mode because:

- API keys stay on the server and are never exposed to the browser
- You can enforce rate limiting and authentication
- All requests go through your existing infrastructure

### Direct mode

AI requests are made directly from the browser to the provider API. This can be useful if:

- Your server has strict request timeout limits that interfere with streaming
- You want to reduce server load

> **Warning:** Direct mode exposes your API keys to the browser. Only use this in trusted environments with authenticated admin users.

---

## Timeout configuration

AI streaming responses can take 30 seconds or more for complex prompts. If your server terminates requests too early, you may see incomplete responses.

### ActiveCanvas stream timeouts

```ruby
ActiveCanvas.configure do |config|
  # Maximum total time for an AI stream (default: 5 minutes)
  config.ai_stream_timeout = 5.minutes

  # Maximum time between chunks before considering the stream stalled (default: 30 seconds)
  config.ai_stream_idle_timeout = 30.seconds

  # Maximum response size (default: 1 MB)
  config.ai_max_response_size = 1.megabyte
end
```

### Puma configuration

If you are using Puma, increase the worker timeout to accommodate long-running AI requests:

```ruby
# config/puma.rb
worker_timeout 120  # 2 minutes (default is 60)
```

### Reverse proxy settings

If you have Nginx or another reverse proxy in front of your Rails app, increase its timeout settings:

```nginx
# nginx.conf
location /canvas/admin/ai/ {
    proxy_read_timeout 300s;
    proxy_send_timeout 300s;
    proxy_buffering off;
    proxy_cache off;
}
```

For Apache:

```apache
ProxyTimeout 300
```

---

## Disabling AI features

To completely disable all AI features:

```ruby
ActiveCanvas.configure do |config|
  config.enable_ai_features = false
end
```

Individual features (text, image, screenshot) can be toggled independently in **Admin > Settings > AI** without changing your initializer.

---

## Next steps

- **[Visual Editor](/guides/visual-editor.html)** -- Learn the editor interface where AI features live
- **[Configuration](/guides/configuration.html)** -- Full reference for AI-related config options
- **[Tailwind CSS](/guides/tailwind-css.html)** -- How AI-generated Tailwind code gets compiled for production
