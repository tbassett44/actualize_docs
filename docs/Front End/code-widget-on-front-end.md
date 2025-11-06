---
title: Code Widget on Front End
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
This project can be built as a standalone JavaScript widget that can be embedded into any web application.

## Building the Widget

```bash
# Build the widget bundle
npm run build:widget
# or
bun run build:widget
```

This will create two files in the `dist-widget` directory:

- `event-management-widget.js` - The widget JavaScript bundle
- `event-management-widget.css` - The widget styles

## Integration

### Basic HTML Integration

```html
<!DOCTYPE html>
<html>
<head>
  <!-- Include the widget CSS -->
  <link rel="stylesheet" href="./dist-widget/event-management-widget.css">
</head>
<body>
  <!-- Container where the widget will be mounted -->
  <div id="app-container"></div>

  <!-- Include the widget JavaScript -->
  <script src="./dist-widget/event-management-widget.js"></script>
  
  <script>
    // Mount the widget
    window.EventManagementWidget.mount({
      containerId: 'app-container',
      eventId: 'YOUR_EVENT_ID',
      onReady: function() {
        console.log('Widget is ready!');
      },
      onError: function(error) {
        console.error('Widget error:', error);
      }
    });
  </script>
</body>
</html>
```

### JavaScript Integration

```javascript
// Mount the widget
EventManagementWidget.mount({
  containerId: 'app-container',
  eventId: 'EHESJ8MXV249Q', // Optional: Your event ID
  onPageLoad: (pageId, element, timestamp) => {
    console.log('Page loaded:', pageId, element, timestamp);
    // Your form builder integration logic here
  },
  onReady: () => {
    console.log('Widget mounted successfully');
  },
  onError: (error) => {
    console.error('Error mounting widget:', error);
  }
});

// Later, if you need to unmount
EventManagementWidget.unmount('app-container');
```

## Configuration Options

### `mount(config)`

Mounts the widget into a container element.

**Parameters:**

- `config.containerId` (string, required): The ID of the container element
- `config.eventId` (string, optional): The event ID to use throughout the app
- `config.basePath` (string, optional): Base path for routing (future feature)
- `config.onReady` (function, optional): Callback when widget is mounted
- `config.onError` (function, optional): Callback when an error occurs

**Example:**

```javascript
EventManagementWidget.mount({
  containerId: 'widget-root',
  eventId: 'EHESJ8MXV249Q',
  onReady: () => console.log('Ready!'),
  onError: (err) => console.error(err)
});
```

### `unmount(containerId)`

Unmounts the widget from a container.

**Parameters:**

- `containerId` (string, required): The ID of the container element

**Example:**

```javascript
EventManagementWidget.unmount('widget-root');
```

## Features

The widget includes the full event management application with:

- Dashboard and Overview
- Ticket Management
- Ticket Orders and Search
- Affiliates Management
- Promotions and Flyers
- Payouts and Banking
- Support Interface
- Comprehensive Settings
- Theme Switching (Light/Dark mode)

## Routing

The widget includes built-in React Router for navigation between different sections. All routes are handled internally:

- `/` - Overview
- `/dashboard` - Dashboard
- `/tickets` - Ticket Management
- `/ticket-orders` - Ticket Orders
- `/affiliates` - Affiliates
- `/promotions` - Promotions
- `/flyers` - Flyers
- `/payouts` - Payouts
- `/bank` - Banking
- `/support` - Support
- `/preview` - Preview
- `/settings/*` - Settings pages

## Styling

The widget is completely self-contained with all styles bundled. It includes:

- Responsive design
- Dark mode support
- Tailwind CSS styling
- Custom design system
- All component styles

## Testing

A complete example HTML file is provided in `widget-example.html`. Open it in a browser to test the widget integration.

## Production Deployment

1. Build the widget: `npm run build:widget`
2. Upload `dist-widget/event-management-widget.js` and `dist-widget/event-management-widget.css` to your CDN
3. Include them in your application
4. Call `EventManagementWidget.mount()` to render the widget

## Browser Support

- Modern browsers with ES6+ support
- React 18+
- No IE11 support

## Notes

- The widget manages its own state and routing
- Multiple instances can be mounted on the same page using different container IDs
- The widget is responsive and works on mobile devices
- All API calls use the authentication system from `src/lib/utils.ts`