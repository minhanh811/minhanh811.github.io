---
layout: post
title: "Enable TSL For Redis"
nav: blog
---

## Problem

My Ruby on Rails app gets this error
```
OpenSSL::SSL::SSLError: SSL_connect returned=1 errno=0 state=error: certificate verify failed
```

## Reason

Redis requires TLS to connect.

## Solution

Use OpenSSL::SSL::VERIFY_NONE for Redis client.

```ruby
# config/initializers/sidekiq.rb
Sidekiq.configure_server do |config|
  config.redis = { ssl_params: { verify_mode: OpenSSL::SSL::VERIFY_NONE } }
end

Sidekiq.configure_client do |config|
  config.redis = { ssl_params: { verify_mode: OpenSSL::SSL::VERIFY_NONE } }
end
```

```
Redis.new(url: 'url', driver: :ruby, ssl_params: { verify_mode: OpenSSL::SSL::VERIFY_NONE })
```
