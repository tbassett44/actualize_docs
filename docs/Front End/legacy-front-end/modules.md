---
title: Modules
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
Modules are re-usable components that can be easily added within views to add complex functionality.  Modules are "lazy loaded" into views through the <dependencies></dependencies> section of the view.

Module code is found in the `phi` project in the `code/module` directory.  They have 3 files that get combined into 1 in production.  When combined, a hash is generated of the file, which is then compared against the current version in the app (cached), to determine if it need to re-load or not.

EG

```html
<dependencies>
module/moment
module/infinitescroll
</dependencies>
```