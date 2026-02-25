---
layout: docs
title: Authentication
description: Configure authentication for the ActiveCanvas admin interface using Devise, custom controllers, HTTP Basic, or custom logic.
order: 5
---

# Authentication

ActiveCanvas provides flexible authentication options for securing the admin interface. This guide covers every supported approach.

---

## Default behavior

> **Warning:** By default, the ActiveCanvas admin panel is **open to everyone** in development. In production, ActiveCanvas raises a `SecurityError` and returns a 403 Forbidden response if no authentication is configured.

This is intentional -- it lets you get started quickly in development while ensuring you never accidentally deploy an unprotected admin panel.

The production check triggers when **both** of these are true:

- `authenticate_admin` is not set
- `admin_parent_controller` is still `"ActionController::Base"`

Configure at least one of these before deploying.

---

## Option 1: Devise integration

If your app uses [Devise](https://github.com/heartcombo/devise), this is the simplest approach. Point `authenticate_admin` at your Devise authentication method:

```ruby
# config/initializers/active_canvas.rb
ActiveCanvas.configure do |config|
  config.authenticate_admin = :authenticate_user!
end
```

This calls `authenticate_user!` as a `before_action` on every admin controller. Unauthenticated users are redirected to Devise's sign-in page.

### Restricting to admin users

If only certain users should access the CMS, use a lambda:

```ruby
ActiveCanvas.configure do |config|
  config.authenticate_admin = -> {
    authenticate_user!
    redirect_to main_app.root_path, alert: "Not authorized" unless current_user.admin?
  }
end
```

Or with Pundit/CanCanCan:

```ruby
ActiveCanvas.configure do |config|
  config.authenticate_admin = -> {
    authenticate_user!
    authorize :active_canvas, :admin?
  }
end
```

---

## Option 2: Custom parent controller

If your app already has an admin base controller that handles authentication, you can tell ActiveCanvas to inherit from it:

```ruby
ActiveCanvas.configure do |config|
  config.admin_parent_controller = "Admin::ApplicationController"
end
```

ActiveCanvas admin controllers will inherit all `before_action` callbacks, authentication logic, and helper methods from your parent controller. This is the cleanest approach if you have an existing admin namespace.

Your parent controller might look like:

```ruby
# app/controllers/admin/application_controller.rb
class Admin::ApplicationController < ApplicationController
  before_action :authenticate_user!
  before_action :require_admin!

  private

  def require_admin!
    redirect_to root_path, alert: "Not authorized" unless current_user&.admin?
  end
end
```

When `admin_parent_controller` is set to something other than `"ActionController::Base"`, ActiveCanvas skips its own authentication checks entirely -- it trusts that your parent controller handles everything.

---

## Option 3: HTTP Basic Auth

For simple deployments or staging environments, HTTP Basic Auth provides quick protection without a user model:

```ruby
ActiveCanvas.configure do |config|
  config.authenticate_admin = :http_basic_auth
  config.http_basic_user = "admin"
  config.http_basic_password = Rails.application.credentials.active_canvas_password
end
```

Set the password in your Rails credentials:

```bash
bin/rails credentials:edit
```

```yaml
active_canvas_password: your-secure-password-here
```

Or use an environment variable:

```ruby
config.http_basic_password = ENV["ACTIVE_CANVAS_ADMIN_PASSWORD"]
```

> **Note:** HTTP Basic Auth credentials are compared using `ActiveSupport::SecurityUtils.secure_compare` to prevent timing attacks.

---

## Option 4: Lambda-based auth

For full control, pass a lambda or proc. It runs in the context of the controller, so you have access to `request`, `session`, `cookies`, and `params`:

```ruby
ActiveCanvas.configure do |config|
  config.authenticate_admin = -> {
    # Token-based authentication
    token = request.headers["X-Admin-Token"] || params[:admin_token]
    head :unauthorized unless token == ENV["ADMIN_TOKEN"]
  }
end
```

Another example with IP restriction:

```ruby
ActiveCanvas.configure do |config|
  config.authenticate_admin = -> {
    allowed_ips = %w[192.168.1.0/24 10.0.0.0/8]
    unless allowed_ips.any? { |cidr| IPAddr.new(cidr).include?(request.remote_ip) }
      head :forbidden
    end
  }
end
```

---

## Protecting public pages

By default, published pages are accessible to anyone. If you need to restrict access (e.g., for an intranet or members-only site), use `authenticate_public`:

```ruby
ActiveCanvas.configure do |config|
  # Require login to view any published page
  config.authenticate_public = :authenticate_user!
end
```

Or with custom logic:

```ruby
ActiveCanvas.configure do |config|
  config.authenticate_public = -> {
    unless current_user&.active_subscription?
      redirect_to main_app.pricing_path, alert: "Subscribe to access this content"
    end
  }
end
```

You can also use a custom parent controller for public pages:

```ruby
ActiveCanvas.configure do |config|
  config.public_parent_controller = "Members::ApplicationController"
end
```

---

## Combining approaches

You can mix and match different strategies for admin and public access:

```ruby
ActiveCanvas.configure do |config|
  # Admin: inherit from your admin base controller
  config.admin_parent_controller = "Admin::ApplicationController"

  # Public pages: require a logged-in user
  config.authenticate_public = :authenticate_user!

  # Current user method (used for version tracking)
  config.current_user_method = :current_user
end
```

### Priority order

ActiveCanvas resolves authentication in this order:

1. If `admin_parent_controller` is set to something other than `"ActionController::Base"`, the parent controller handles everything. `authenticate_admin` is ignored.
2. If `authenticate_admin` is `:http_basic_auth`, HTTP Basic Auth is used with the configured credentials.
3. If `authenticate_admin` is a Symbol, it is called as a method on the controller (e.g., `:authenticate_user!`).
4. If `authenticate_admin` is a Proc or Lambda, it is executed with `instance_exec` in the controller context.
5. If nothing is configured, development allows access; production raises `SecurityError`.

---

## Current user tracking

ActiveCanvas uses the `current_user_method` to identify who made changes. This is used for:

- **Version history** -- showing which user created each version
- **AI request tracking** -- logging who triggered AI operations

```ruby
ActiveCanvas.configure do |config|
  # Default: :current_user (works with Devise out of the box)
  config.current_user_method = :current_user

  # Custom method name
  config.current_user_method = :current_admin
end
```

If your authentication system does not expose a user object (e.g., HTTP Basic Auth), version tracking will record the change without a user association.

---

## Next steps

- **[Getting Started](/guides/getting-started.html)** -- Initial setup guide
- **[Configuration](/guides/configuration.html)** -- Full reference for all config options
- **[AI Features](/guides/ai-features.html)** -- AI features also respect authentication settings
