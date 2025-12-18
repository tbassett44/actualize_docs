---
title: Imageviewer
deprecated: false
hidden: false
metadata:
  robots: index
---
A fullscreen image viewer module built on top of PhotoSwipe that supports single images and galleries with zoom, pan, swipe navigation, and sharing capabilities.

## Basic Usage

### Single Image

```javascript
new modules.imageviewer({
    img: {
        pic: { id: 'abc123', ar: 1.5 },  // Image object with id and aspect ratio
        type: 'full'                      // Size to load: 'small', 'full', etc.
    },
    ele: $('#thumbnail')  // Optional: Element to animate from
});
```

### Gallery (Multiple Images)

```javascript
new modules.imageviewer({
    data: {
        list: {
            'id1': { pic: { id: 'abc123', ar: 1.5 } },
            'id2': { pic: { id: 'def456', ar: 0.75 } },
            'id3': { pic: { id: 'ghi789', ar: 1.0 } }
        },
        order: ['id1', 'id2', 'id3']  // Display order
    },
    index: 0,             // Starting image index
    ele: $('#thumbnail')  // Optional: Element to animate from
});
```

## Constructor Options

| Option    | Type     | Required | Description                                     |
| --------- | -------- | -------- | ----------------------------------------------- |
| `img`     | Object   | Yes*     | Single image object (use this OR `data`)        |
| `data`    | Object   | Yes*     | Gallery data with `list` and `order` properties |
| `index`   | Number   | No       | Starting index for gallery (default: 0)         |
| `ele`     | jQuery   | No       | Thumbnail element for open/close animation      |
| `spin`    | Boolean  | No       | Show loading spinner while image loads          |
| `onClose` | Function | No       | Callback when viewer is closed                  |

> *Either `img` or `data` is required

## Image Object Structure

```javascript
{
    pic: {
        id: 'image_id',     // Image ID for _.getImg()
        ar: 1.5,            // Aspect ratio (width/height)
        type: 'full'        // Optional: size type
    },
    ar: 1.5,                // Override aspect ratio
    type: 'full'            // Override size type ('small', 'full', 'images')
}
```

## Features

| Feature                | Description                                             |
| ---------------------- | ------------------------------------------------------- |
| **Pinch Zoom**         | Two-finger pinch to zoom on touch devices               |
| **Double-tap Zoom**    | Double-tap to zoom in/out (3x on mobile, 2x on desktop) |
| **Swipe Navigation**   | Swipe left/right to navigate gallery                    |
| **Arrow Navigation**   | Click arrows on desktop to navigate                     |
| **Keyboard Support**   | `←` `→` arrows to navigate, `Esc` to close              |
| **Share**              | Share image via native share dialog (PhoneGap)          |
| **Download**           | Save image to device                                    |
| **Orientation Unlock** | Automatically unlocks screen rotation                   |

## Methods

| Method      | Description                     |
| ----------- | ------------------------------- |
| `destroy()` | Closes and cleans up the viewer |

## Dependencies

* **PhotoSwipe** - Core gallery functionality
* **modules.tools** - Image URL resolution (`getImg`)
* **modules.alertdelegate** - Share/download menu
* **modules.saveimage** - Image download functionality
* **TweenLite** (optional) - Animation for `init2` mode

## Example with Callback

```javascript
new modules.imageviewer({
    img: {
        pic: { id: 'photo123', ar: 1.33 }
    },
    ele: $('.photo-thumbnail'),
    onClose: function() {
        console.log('Viewer closed');
        // Refresh UI, analytics, etc.
    }
});
```

## Internal Methods

These methods are used internally but may be useful for advanced use cases:

| Method           | Description                                              |
| ---------------- | -------------------------------------------------------- |
| `init()`         | Initializes the viewer (called automatically)            |
| `getPic(img)`    | Extracts and normalizes the picture object from an image |
| `ensureArrows()` | Shows/hides navigation arrows based on current position  |
| `bindMore()`     | Binds the more options menu (share/download)             |
| `onKeyup(e)`     | Keyboard event handler                                   |

## Notes

* The viewer automatically hides the status bar on mobile devices
* Screen orientation is unlocked while viewing, then locked again on close
* On desktop, clicking the image toggles between fit and 2x zoom
* On mobile, double-tap toggles between fit and 3x zoom
