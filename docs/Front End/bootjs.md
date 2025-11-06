---
title: Loading the Framework
excerpt: ''
deprecated: false
hidden: false
metadata:
  title: ''
  description: ''
  robots: index
next:
  description: ''
---
# Boot.js

Boot.js is the first JavaScript file loaded when a page initializes. It serves as the core bootloader responsible for setting up the entire framework, managing dependencies, handling caching, and orchestrating the loading sequence of all subsequent packages and modules.

## Overview

The bootloader is a sophisticated system that handles:

- **Framework Initialization**: Sets up core utilities and global objects
- **Dependency Management**: Loads libraries, core modules, and application code
- **Caching System**: Implements intelligent file caching with version control
- **Error Handling**: Provides comprehensive error screens and recovery mechanisms
- **Platform Detection**: Handles differences between web and mobile (PhoneGap) environments
- **Configuration Management**: Loads and processes application configuration

## Core Components

### 1. Built-in Utilities

Boot.js includes several essential utilities that are immediately available:

#### UUID Generation

```javascript
// Generate standard UUID
Math.uuid(); // "550e8400-e29b-41d4-a716-446655440000"

// Generate UUID with specific length and radix
Math.uuid(8, 16); // "098F4D35"

// Fast UUID generation
Math.uuidFast(); // Optimized version

// Compact UUID generation
Math.uuidCompact(); // Alternative implementation
```

#### EJS Template Engine

Complete EJS (Embedded JavaScript) templating system for dynamic content generation:

```javascript
// Create template from string
var template = new EJS({text: '<h1><%= title %></h1>'});
var html = template.render({title: 'Hello World'});

// Load template from element
var template = new EJS({element: 'template-id'});

// Load template from URL
var template = new EJS({url: '/templates/mytemplate.ejs'});
```

#### Platform Detection

```javascript
// Check if running in PhoneGap/Cordova
if (isPhoneGap()) {
    // Mobile app environment
    console.log('Running in mobile app');
} else {
    // Web browser environment
    console.log('Running in web browser');
}
```

#### Enhanced Storage

Extended localStorage with object support:

```javascript
// Store objects and arrays
localStorage.setObject('user', {name: 'John', age: 30});

// Retrieve objects with fallback
var user = localStorage.getObject('user', {}); // Returns {} if not found
var settings = localStorage.getObject('settings', []); // Returns [] if not found
```

### 2. Bootloader Class

The main bootloader is instantiated as `window._bootloader` and provides:

#### Configuration and Initialization

```javascript
// Initialize bootloader with configuration
var bootloader = new window.bootloader(config);
bootloader.init();
```

#### Key Properties

- `version`: Bootloader version (currently 4)
- `splashDisabled`: Controls splash screen behavior
- `config`: Application configuration object
- `publicconf`: Public configuration loaded from server
- `settings`: User settings and preferences
- `versions`: Version tracking for cached files

#### Core Methods

**Logging System**

```javascript
bootloader.log('Message'); // Logs with timestamp and prefix
bootloader.submitLog(callback); // Submits logs to server
```

**Language and Localization**

```javascript
bootloader.getLang(); // Returns current language code
bootloader.loc('key', data); // Gets localized string
```

**Configuration Loading**

```javascript
bootloader.loadConf({
    success: function() { /* Configuration loaded */ },
    error: function(err) { /* Handle error */ }
});
```

### 3. File and Cache Management

#### Intelligent Caching System

The bootloader implements a sophisticated caching system with:

- **Hash-based Versioning**: Files are cached with MD5 hashes for integrity
- **Length Verification**: File size validation before loading cached content
- **Automatic Invalidation**: Clears cache when versions don't match
- **Offline Support**: Falls back to cached content when network unavailable

#### File Loading Process

```javascript
bootloader.loadFile(fileconf, function(code, error) {
    if (code) {
        // File loaded successfully
        console.log('File loaded:', code.length, 'bytes');
    } else {
        // Handle error
        console.error('Failed to load file:', error);
    }
});
```

#### Storage Abstraction

```javascript
// Set data (uses NativeStorage on mobile, localStorage on web)
bootloader.store.set('key', data, function(success) {
    console.log('Data saved:', success);
});

// Get data with default value
bootloader.store.get('key', function(data) {
    console.log('Retrieved data:', data);
}, defaultValue);

// Delete data
bootloader.store.delete('key', function(success) {
    console.log('Data deleted:', success);
});
```

### 4. Loading Sequence

The bootloader follows a specific loading sequence:

1. **Initialization**
   - Load cached settings and configuration
   - Set up device detection and platform-specific behavior
   - Initialize file system and storage

2. **Configuration Loading**
   - Fetch public configuration from server
   - Process cached vs. fresh configuration
   - Validate configuration integrity

3. **Library Loading**
   - Load core libraries (jQuery, utilities, etc.)
   - Process CSS and JavaScript files
   - Handle font loading

4. **Core Framework Loading**
   - Load core application framework
   - Initialize template system
   - Set up global modules

5. **Application Loading**
   - Load application-specific code
   - Initialize application entry points
   - Complete framework initialization

### 5. Error Handling

#### Error Screen System

The bootloader provides comprehensive error handling with user-friendly screens:

```javascript
// Show error screen with specific type
bootloader.showErrorScreen('timeout', 'Custom message');

// Hide error screen
bootloader.hideErrorScreen();
```

#### Error Types

- **timeout**: Network connectivity issues
- **code**: JavaScript execution errors
- **blank_conf**: Invalid configuration
- **api_issue**: Server-side problems

#### Recovery Mechanisms

- **Retry Button**: Reloads the application
- **Contact Support**: Submits error logs to support
- **Cache Clearing**: Automatically clears problematic cached files

### 6. Template Processing

#### Template System

```javascript
// Process templates from combined string
bootloader.processTemplates(templateString);

// Templates are stored in global templates object
window.templates['template_name'].render(data);
```

#### Template Format

Templates are stored in a special format separated by `@@@`:

```
@@@template_name@@@<h1><%= title %></h1>@@@another_template@@@<p><%= content %></p>@@@
```

### 7. Development vs Production

#### Development Mode

- Loads individual files for easier debugging
- Includes cache-busting timestamps
- Supports hot reloading
- Provides detailed logging

#### Production Mode

- Loads combined/minified files
- Implements aggressive caching
- Optimized for performance
- Minimal logging

## Configuration Structure

### Public Configuration Object

```javascript
{
    "vars": {}, // Global variables
    "libraries": {
        "key": "libraries",
        "hash": "abc123",
        "url": "https://api.example.com/dist/libraries.js",
        "combined": true
    },
    "core": {
        "key": "core", 
        "hash": "def456",
        "url": "https://api.example.com/dist/core.js",
        "combined": true
    },
    "loader": {
        "key": "loader",
        "hash": "ghi789", 
        "url": "https://api.example.com/dist/loader.js",
        "combined": true
    },
    "app": {
        "key": "app",
        "hash": "jkl012",
        "url": "https://api.example.com/dist/app.js", 
        "combined": true
    }
}
```

### File Configuration Object

```javascript
{
    "key": "unique_identifier",
    "hash": "md5_hash_of_file",
    "url": "https://api.example.com/file.js",
    "urls": {
        "js": ["file1.js", "file2.js"],
        "css": ["styles.css"],
        "font": ["fonts.css"]
    },
    "combined": true, // Single combined file vs multiple files
    "type": "dna|json|jsonp", // File format type
    "entry": {
        "scope": "window_object",
        "function": "init_function"
    }
}
```

## Platform-Specific Features

### Mobile (PhoneGap/Cordova)

- **NativeStorage**: Persistent storage using native APIs
- **File System**: Local file caching using Cordova File plugin
- **Device Detection**: Access to device information and capabilities
- **Splash Screen**: Native splash screen management
- **Status Bar**: Platform-specific status bar control

### Web Browser

- **localStorage**: Fallback storage mechanism
- **IndexedDB**: File caching using Dexie.js
- **Service Workers**: (Future enhancement for offline support)
- **Progressive Web App**: PWA-ready architecture

## Best Practices

### 1. Configuration Management

- Always validate configuration before processing
- Implement proper error handling for configuration failures
- Use version hashing for cache invalidation

### 2. Performance Optimization

- Minimize bootloader size for faster initial load
- Implement progressive loading for large applications
- Use intelligent caching to reduce network requests

### 3. Error Recovery

- Provide clear error messages to users
- Implement automatic retry mechanisms
- Log errors for debugging and monitoring

### 4. Development Workflow

- Use development mode for debugging
- Test both cached and fresh loading scenarios
- Validate cross-platform compatibility

## Troubleshooting

### Common Issues

1. **Infinite Loading**
   - Check network connectivity
   - Verify configuration URL accessibility
   - Clear application cache

2. **JavaScript Errors**
   - Check browser console for specific errors
   - Verify all dependencies are loaded
   - Test in development mode

3. **Cache Issues**
   - Clear bootloader cache: `bootloader.setVersionInfo('delete')`
   - Force fresh configuration load
   - Check file hash mismatches

4. **Platform-Specific Problems**
   - Verify PhoneGap plugins are installed
   - Check platform-specific permissions
   - Test on actual devices vs simulators

## Integration Examples

### Basic Integration

```html
<!DOCTYPE html>
<html>
<head>
    <script>
        window.app_conf = {
            api: 'api.example.com',
            appid: 'your_app_id',
            version: '1.0.0'
        };
    </script>
    <script src="code/js/boot.js"></script>
    <script>
        var bootloader = new window.bootloader();
        bootloader.init();
    </script>
</head>
<body>
    <!-- Application content -->
</body>
</html>
```

### Advanced Configuration

```javascript
var bootloader = new window.bootloader({
    // Pre-loaded configuration
    vars: {custom_var: 'value'},
    libraries: {/* library config */},
    core: {/* core config */}
});

// Custom initialization
bootloader.init();

// Monitor loading progress
bootloader.log = function(msg) {
    console.log('Custom log:', msg);
    // Send to analytics, etc.
};
```

Boot.js is the foundation of the entire framework, providing robust initialization, caching, and error handling capabilities that ensure reliable application loading across all supported platforms.