---
title: Insets / Spacing on Phones
deprecated: false
hidden: false
metadata:
  robots: index
---
## Overview

The `phone.insets` system provides a unified way to handle safe area insets and device-specific layout constraints for mobile applications. It creates CSS custom properties (variables) that can be used throughout the application to ensure content is properly positioned within the safe areas of various devices.

## &#x20;phone.insets API

### CSS Variables Created

The system would create the following CSS custom properties:

| Variable Name              | Description                       | Example Value |
| -------------------------- | --------------------------------- | ------------- |
| `--safe-area-inset-top`    | Top safe area (status bar, notch) | `44px`        |
| `--safe-area-inset-bottom` | Bottom safe area (home indicator) | `34px`        |
| `--safe-area-inset-left`   | Left safe area (landscape notch)  | `0px`         |
| `--safe-area-inset-right`  | Right safe area (landscape notch) | `0px`         |

<br />

## Usage Examples

### CSS Usage

```css
.header {
    padding-top: var(--safe-area-inset-top);
    background: #007AFF;
}

.content {
    padding-left: var(--safe-area-inset-left);
    padding-right: var(--safe-area-inset-right);
}

.footer {
    padding-bottom: var(--safe-area-inset-bottom);
}

/* Fallback for older browsers */
.header {
    padding-top: 20px; /* fallback */
    padding-top: var(--safe-area-inset-top);
}
/* built in css classes ready to use*/
.mobiletop{
  top:[top_header]px !important;
}
.mobiletop2{
  top:[top_header_20]px !important;
}
.infinitescroll_sticky_container{
   top:[top_header]px !important;
}
.mobilespacer{
   height:[top_header]px !important;
}
.mobilescrollcontent{
   padding-top:[top_header]px !important;
}
.mobilepageresponsive{
   top:[top_header]px !important;
}
.mobileheader{
   padding-top:[top]px !important;
}
.mobilefooter{
   padding-bottom:[bottom]px !important;
}
.keyboardpage{
   bottom:[bottom]px !important;
}
```

<br />

<br />

## Benefits

1. **Unified API**: Single source of truth for safe area handling
2. **CSS Integration**: Seamless integration with CSS custom properties
3. **Device Agnostic**: Works across different device types and orientations
4. **Future Proof**: Easy to extend for new device types
5. **Performance**: Calculated once, used everywhere via CSS variables

<br />