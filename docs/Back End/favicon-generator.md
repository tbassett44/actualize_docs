---
title: Favicon Generator
excerpt: _img/favicon/favicon.php
deprecated: false
hidden: false
metadata:
  title: ''
  description: ''
  robots: index
next:
  description: ''
---
The favicon Generator is used to create images with a number if passed to the URL. This is useful in browser tabs to indicate that new activities / notifications have happened in the browser tab.

The API for this works like

```
https://img.actualize.earth/favicon/favicon.php?src=[URL]&count=[integer]
```

EG

<https://img.actualize.earth/favicon/favicon.php?src=https://one-earth.s3.amazonaws.com/static/a_logo.png&count=5>

[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/9788f44900786023dcb867452205b78e408aa5d290768f6da3d69d3ed4b90aa7-image.png",
        null,
        null
      ],
      "align": "center",
      "sizing": "50px"
    }
  ]
}
[/block]


<br />

OR  
<https://img.actualize.earth/favicon/favicon.php?src=https://one-earth.s3.amazonaws.com/static/a_logo.png>

[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/6a1fc3b371d6fd8617acf4b2220ab1c4407d1acc07d9251a3a034681984aefe3-image.png",
        null,
        null
      ],
      "align": "center",
      "sizing": "50px"
    }
  ]
}
[/block]