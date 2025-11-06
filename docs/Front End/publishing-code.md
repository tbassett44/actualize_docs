---
title: Publishing Code
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
## Front End Code

This publishes the code based on the git branch you are on and developers must have their developer token registered to each branch they have access to publish to.  Ask [juicy@actualize.earth](mailto:juicy@actualize.earth) if you need help getting your developer token registered to publish a branch.

```
npm run publish [environment]
```

### Bonus

The package loading/publishing system is branch aware as well, meaning that each view can have many different branches of the view, all being able to be published and loaded through the boot.js