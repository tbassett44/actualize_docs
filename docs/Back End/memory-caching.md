---
title: Memory Caching
excerpt: Redis
deprecated: false
hidden: false
metadata:
  robots: index
---
<br />

# Redis Cache Layer

The caching system uses Redis with persistent connections (`pconnect`) for connection pooling across requests. It supports both direct TTL values (seconds) and named TTL keys from `cache_settings::$map`.

***

## Configuration

Redis connection is configured in `phi::$conf['cache']['redis']`:

```php
[
    'host'     => 'localhost',     // Redis server host
    'port'     => 6379,            // Redis server port  
    'password' => 'optional_pass'  // Optional authentication
]
```

***

## Core Methods

### Basic Operations

```php
// Get cached value
$data = cache::get('user:U123:account');

// Save with TTL (int seconds or named key)
cache::save('user:U123:account', $userData, 1800);
cache::save('user:U123:account', $userData, 'user:account');

// Delete single key
cache::clear('user:U123:account');

// Flush entire cache (use with caution!)
cache::clearAll();
```

### Cache-Aside Pattern (Recommended)

The `cache::remember()` method eliminates repetitive get/check/save boilerplate:

```php
// Basic usage with callback
$user = cache::remember("user:$id:account", function() use ($db, $id) {
    return db2::findOne($db, 'user', ['_id' => $id]);
}, 'user:account');

// With secure key hashing (for tokens)
$auth = cache::remember("token:$token:auth", $callback, 'token:auth', [
    'key' => ['index' => 1]  // Hash segment at index 1
]);

// Force fresh fetch, strip sensitive fields
$data = cache::remember($key, $callback, 3600, [
    'removeFields' => ['password', 'secret.key']
], true);
```

**Parameters:**

* `$key` - Cache key string
* `$dataOrCallback` - Callable for cache miss, or direct value
* `$ttl` - Seconds (int) or named key from `cache_settings::$map`
* `$secureOpts` - Optional: `['key' => ['index' => N]]` to hash key segment, `['removeFields' => [...]]` to strip fields
* `$forceNoCache` - If `true`, bypass cache and fetch fresh

**Returns:** Cached data includes `_cached => true` flag when served from cache.

***

## TTL Named Keys

Defined in `api/cache.settings.php`:

| Key                 | TTL (seconds) | Duration |
| ------------------- | ------------- | -------- |
| `user:scopes`       | 1800          | 30 min   |
| `user:roles`        | 1800          | 30 min   |
| `user:subscription` | 1800          | 30 min   |
| `user:settings`     | 1800          | 30 min   |
| `user:account`      | 1800          | 30 min   |
| `token:auth`        | 1800          | 30 min   |
| `app:config`        | 1800          | 30 min   |
| `app:settings`      | 1800          | 30 min   |
| `plan:settings`     | 1800          | 30 min   |
| `subscription_info` | 604800        | 7 days   |
| `app_text`          | 604800        | 7 days   |
| `app_config`        | 604800        | 7 days   |
| `version:code`      | 604800        | 7 days   |
| `place`             | 604800        | 7 days   |
| `quote`             | 86400         | 24 hours |

***

## Pattern-Based Clearing

Use SCAN-based pattern clearing for bulk invalidation (non-blocking, production-safe):

```php
// Clear by glob pattern
cache::clearPattern('user:U123:*');

// Convenience methods
cache::clearUserCache('U123');        // Clear user:U123:*
cache::clearAllTokens();              // Clear all token:*
cache::clearAppCache('app1');         // Clear app:app1:*
cache::clearAppCache();               // Clear all app:*
cache::clearAllUserCache();           // Clear all user:*
cache::clearPlanCache('plan1');       // Clear plan:plan1
cache::clearPlanCache();              // Clear all plan:*
cache::clearPlaceCache('place1');     // Clear place:place1
cache::clearPlaceCache();             // Clear all place:*
```

***

## Cache Invalidation via schema.json

The most powerful cache invalidation pattern is declarative: define `clearChache` hooks in `schema.json` field definitions. When `formbuilder::update()` modifies a record, the hook automatically clears the relevant cache keys.

### How It Works

1. **Declare the hook** in `schema.json` under a field's `hooks.onUpdate`:

```json
{
    "token": {
        "fields": {
            "revoked": {
                "type": "bool",
                "hooks": {
                    "onUpdate": {
                        "clearChache": {
                            "cacheKeys": ["token:[id]:auth"],
                            "cacheSecurityOpts": {
                                "key": { "index": 1 }
                            }
                        }
                    }
                }
            }
        }
    }
}
```

2. **Update via formbuilder** - the hook triggers automatically:

```php
// In core::logout() - revoking a token
formbuilder::update('token', [
    'id' => $r['qs']['token'],
    'revoked' => true
], false, 1);
// ↑ This triggers the onUpdate hook, which clears "token:{token_id}:auth"
```

3. **The hook processor** in `formbuilder::runHooks()` handles `clearChache`:

```php
case 'clearChache':
    if (isset($opts['cacheKeys'])) {
        foreach ($opts['cacheKeys'] as $cv) {
            // Replaces [id], [uid], etc. with actual values from $d['current']
            $key = phi::renderTemplate($cv, phi::convertArray($d['current']));
            cache::clear($key, $opts['cacheSecurityOpts'] ?? false);
        }
    }
    break;
```

### Hook Options

| Option              | Description                                             |
| ------------------- | ------------------------------------------------------- |
| `cacheKeys`         | Array of cache key patterns with `[field]` placeholders |
| `cacheSecurityOpts` | Optional security options for key hashing (see below)   |

**Template Variables:** Use `[fieldname]` syntax in `cacheKeys` to substitute values from the current record:

* `[id]` → record ID
* `[uid]` → user ID field
* Any dot-notation path: `[user.id]`, `[from.type]`

**Security Options:** When caching sensitive data like tokens, hash the key segment:

```json
"cacheSecurityOpts": {
    "key": { "index": 1 }  // Hash the segment at index 1 (0-based after splitting by ":")
}
```

### More Examples

**User collection** - clear user cache on any update:

```json
{
    "user": {
        "fields": {
            "id": {
                "hooks": {
                    "onUpdate": {
                        "clearChache": {
                            "cacheKeys": ["user:[id]:*"]
                        }
                    }
                }
            }
        }
    }
}
```

**Subscription changes** - clear multiple related caches:

```json
{
    "subscription": {
        "fields": {
            "status": {
                "hooks": {
                    "onUpdate": {
                        "clearChache": {
                            "cacheKeys": [
                                "user:[uid]:subscription",
                                "user:[uid]:scopes"
                            ]
                        }
                    }
                }
            }
        }
    }
}
```

***

## Best Practices

1. **Use named TTL keys** - Centralize TTL values in `cache_settings::$map` for consistency
2. **Use `cache::remember()`** - Cleaner than manual get/check/save pattern
3. **Declarative invalidation** - Prefer `schema.json` hooks over manual `cache::clear()` calls
4. **Hash sensitive keys** - Use `secureOpts['key']` for tokens and credentials
5. **Pattern clearing** - Use `clearPattern()` for bulk invalidation, not `clearAll()`
6. **Check `_cached` flag** - Useful for debugging cache hits vs misses

***

## Files Reference

| File                        | Purpose                                    |
| --------------------------- | ------------------------------------------ |
| `api/cache.php`             | Cache class with all methods               |
| `api/cache.settings.php`    | TTL named keys map                         |
| `_manage/schema.json`       | Collection schemas with cache hooks        |
| `api/class/formbuilder.php` | Hook processor (`runHooks`, `clearChache`) |
