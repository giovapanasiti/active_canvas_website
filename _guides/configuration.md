---
layout: docs
title: Configuration
description: Complete reference for every ActiveCanvas configuration option.
order: 4
---

# Configuration

All ActiveCanvas settings are configured through an initializer file. The install generator creates this file automatically, but you can also create it manually.

---

## Overview

Configuration lives in `config/initializers/active_canvas.rb`:

```ruby
ActiveCanvas.configure do |config|
  # Your settings here
end
```

Settings are read at boot time. Changes require a server restart to take effect. Some settings (like CSS framework and AI keys) can also be changed at runtime through the Admin UI via the `ActiveCanvas::Setting` model.

---

## Full option reference

### Authentication

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `authenticate_admin` | Symbol / Lambda / Proc | `nil` | Authentication callback for admin pages. Set to a method name symbol (e.g., `:authenticate_user!`) or a lambda/proc for custom logic. Use `:http_basic_auth` for HTTP Basic Auth. |
| `authenticate_public` | Symbol / Lambda / Proc / nil | `nil` | Authentication callback for public pages. Same interface as `authenticate_admin`. Set to `nil` to allow public access. |
| `current_user_method` | Symbol | `:current_user` | The method name used to retrieve the current user. Used by AI features, version tracking, and audit logging. |
| `admin_parent_controller` | String | `"ActionController::Base"` | Parent controller class for admin controllers. Set to your app's admin base controller (e.g., `"Admin::ApplicationController"`) to inherit authentication and other before_actions. |
| `public_parent_controller` | String | `"ActionController::Base"` | Parent controller class for public-facing page controllers. |
| `http_basic_user` | String | `nil` | Username for HTTP Basic Auth. Only used when `authenticate_admin = :http_basic_auth`. |
| `http_basic_password` | String | `nil` | Password for HTTP Basic Auth. Only used when `authenticate_admin = :http_basic_auth`. |

### CSS framework

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `css_framework` | Symbol | `:tailwind` | Default CSS framework. Options: `:tailwind`, `:bootstrap5`, `:none`. Can be overridden in Admin > Settings. |

### Media uploads

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `enable_uploads` | Boolean | `true` | Enable or disable file uploads in the media library. |
| `max_upload_size` | Integer | `10.megabytes` | Maximum file size for uploads in bytes. |
| `allow_svg_uploads` | Boolean | `false` | Allow SVG file uploads. Disabled by default due to XSS risks with inline SVG. |
| `storage_service` | Symbol / nil | `nil` | Active Storage service name. Set to `nil` to use the default service, or specify a named service like `:amazon` or `:google`. |
| `public_uploads` | Boolean | `false` | When `true`, uploaded files are served via public URLs. When `false`, signed URLs are used for access control. |

### Editor features

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `enable_ai_features` | Boolean | `true` | Enable AI content generation, image creation, and screenshot-to-code in the editor. |
| `enable_code_editor` | Boolean | `true` | Show the Monaco code editor panel in the visual editor. |
| `enable_asset_manager` | Boolean | `true` | Show the media library / asset manager in the editor. |

### Page settings

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `autosave_interval` | Integer | `60` | Auto-save interval in seconds. Set to `0` to disable auto-save. |
| `max_versions_per_page` | Integer | `50` | Maximum number of versions to keep per page. Older versions are pruned automatically. Set to `0` for unlimited. |

### Security

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `sanitize_content` | Boolean | `true` | Sanitize HTML content on save. Strips potentially dangerous tags and attributes. |

### AI security

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `ai_rate_limit_per_minute` | Integer | `30` | Maximum AI requests per minute per IP address. |
| `ai_stream_timeout` | Duration | `5.minutes` | Maximum total duration for an AI streaming response. |
| `ai_stream_idle_timeout` | Duration | `30.seconds` | Maximum time between chunks before a stream is considered stalled. |
| `ai_max_response_size` | Integer | `1.megabyte` | Maximum size of an AI streaming response. |
| `max_screenshot_size` | Integer | `10.megabytes` | Maximum size for screenshot uploads (base64 encoded). |

---

## Example configuration

Here is a complete configuration block showing all options with their defaults:

```ruby
# config/initializers/active_canvas.rb
ActiveCanvas.configure do |config|
  # ==> Authentication
  # Use Devise authentication for admin
  config.authenticate_admin = :authenticate_user!

  # Leave public pages open
  config.authenticate_public = nil

  # Method to get current user (for version tracking)
  config.current_user_method = :current_user

  # Inherit from your admin base controller
  # config.admin_parent_controller = "Admin::ApplicationController"

  # HTTP Basic Auth (alternative to Devise)
  # config.authenticate_admin = :http_basic_auth
  # config.http_basic_user = "admin"
  # config.http_basic_password = Rails.application.credentials.active_canvas_password

  # ==> CSS Framework
  config.css_framework = :tailwind  # :tailwind, :bootstrap5, or :none

  # ==> Media Uploads
  config.enable_uploads = true
  config.max_upload_size = 10.megabytes
  config.allow_svg_uploads = false
  config.storage_service = nil        # nil = default Active Storage service
  config.public_uploads = false

  # ==> Editor Features
  config.enable_ai_features = true
  config.enable_code_editor = true
  config.enable_asset_manager = true

  # ==> Page Settings
  config.autosave_interval = 60       # seconds (0 = disabled)
  config.max_versions_per_page = 50   # 0 = unlimited

  # ==> Security
  config.sanitize_content = true

  # ==> AI Security
  config.ai_rate_limit_per_minute = 30
  config.ai_stream_timeout = 5.minutes
  config.ai_stream_idle_timeout = 30.seconds
  config.ai_max_response_size = 1.megabyte
  config.max_screenshot_size = 10.megabytes
end
```

---

## Runtime settings via Admin UI

Some settings can be changed at runtime through **Admin > Settings** without restarting the server. These are stored in the `active_canvas_settings` table:

- CSS framework and Tailwind theme configuration
- AI API keys (encrypted at rest)
- Default AI models
- AI feature toggles (text, image, screenshot)
- AI connection mode (server vs direct)
- Global CSS and JavaScript
- Homepage page selection

Runtime settings take precedence over initializer values for options that support both.

---

## Accessing configuration in code

You can read the current configuration anywhere in your application:

```ruby
# Read initializer settings
ActiveCanvas.config.css_framework
# => :tailwind

ActiveCanvas.config.max_upload_size
# => 10485760

# Read runtime settings
ActiveCanvas::Setting.css_framework
# => "tailwind"

ActiveCanvas::Setting.ai_default_text_model
# => "gpt-4o-mini"

# Check feature availability
ActiveCanvas.config.ai_available?
# => true

ActiveCanvas.config.tailwind_compilation_available?
# => true
```

---

## Next steps

- **[Authentication](/guides/authentication.html)** -- Detailed guide for every authentication approach
- **[AI Features](/guides/ai-features.html)** -- AI-specific configuration and setup
- **[Tailwind CSS](/guides/tailwind-css.html)** -- CSS framework configuration and compilation
