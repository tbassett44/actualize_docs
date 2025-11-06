---
title: Getting Started
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
[This project](https://github.com/tbassett44/event-zen-dash)  demonstrates a powerful hybrid development workflow that combines the rapid prototyping capabilities of Lovable.dev with the precision of local IDE development enhanced by AI coding assistants. Start by using [Lovable](https://lovable.dev/) 's natural language interface to quickly scaffold your widget architecture, implement core features, and iterate on UI/UX designs through conversational prompts. Once you've established the foundation, connect your Lovable project to GitHub to enable bidirectional sync—changes made in Lovable automatically push to your repository, while local commits sync back to Lovable in real-time.

For complex coding tasks that require deeper architectural changes, detailed refactoring, or intricate business logic, pull the repository to your local machine and leverage [VSCode](https://vscode.dev/)  with the [Augment AI](https://www.augmentcode.com/)  plugin. This combination gives you the best of both worlds: Augment's context-aware code completion and generation capabilities work seamlessly with your local development environment, allowing you to tackle sophisticated implementation challenges while maintaining full version control. The local development server (npm run dev) provides instant feedback, and you can test both the standalone app and widget builds (npm run build:dev for development or npm run build for production).

The workflow becomes circular and iterative—use Lovable for rapid feature additions and UI adjustments, switch to local development with Augment for complex algorithmic work or architectural decisions, then push your changes back to GitHub where they automatically sync to Lovable. This approach maximizes productivity by matching the tool to the task: conversational AI for speed and exploration, local IDE with AI assistance for precision and complexity. The widget architecture documented in WIDGET-ARCHITECTURE.md ensures that any AI assistant (whether Lovable, Augment, or future tools) has comprehensive context about callbacks, routing, state management, and styling conventions, making handoffs between development environments seamless.

![](https://files.readme.io/86fba28d4fdd15354aee38cbab26f8eaecbf3816139efffbeed6da7abc508804-image.png)

![](https://files.readme.io/7f7e5103255f4e1e93e7bfa94e8db4a8d3eb08c2fbd16c48e699f2a38ab90bb9-image.png)

<br />

# View Widgets - Architecture & Implementation Guide

## Overview

This is a self-contained widget that can be embedded into any web application. It uses a UMD build format for maximum compatibility and includes its own routing, state management, and styling.

## Build System

### Build Commands

```bash
# Development build
npm run build:dev
bun run build:dev

# Production build (default)
npm run build
bun run build
```

### Build Outputs

* **Production**: `dist/widget.js` and `dist/widget.css` (minified, UMD format)
* **Development**: Same output but with readable code for debugging
* **Library Mode**: UMD format exposing `EventManagementWidget` global
* **Externals**: React and ReactDOM are externalized (must be provided by host)

### Vite Configuration (`vite.config.ts`)

* **Mode-specific builds**: Development vs production configurations
* **CSS Processing**: Tailwind → PostCSS → Autoprefixer → cssnano (production)
* **CSS Prefixing**: `vite-css-prefixer` adds `event-widget-` prefix to avoid conflicts
* **Bundle format**: UMD with named exports
* **Globals**: Expects `React` and `ReactDOM` from host application

## Integration Method

```javascript
// Mount with configuration object
widget.mount({
  containerId: 'widget-container',
  eventId: '123',
  initialRoute: '/dashboard',
  onRouteChange: (route) => console.log('Route changed:', route),
  onAction: (action, data, element) => console.log('Action:', action, data)
});

// Unmount by container ID
widget.unmount('widget-container');
```

<br />

## Configuration Options

### WidgetConfig Interface (src/index.ts)

```typescript
type WidgetConfig = {
  containerId: string;           // Required: DOM element ID
  eventId?: string;               // Optional: Event ID for data fetching
  initialRoute?: string;          // Optional: Starting route (e.g., '/dashboard')
  onRouteChange?: (route: string) => void;  // Optional: Route change callback
  onAction?: (action, data, element) => void; // Optional: Action callback
  [key: string]: unknown;         // Additional properties passed to App
};
```

### Extended Config (src/lib/app-widget.tsx)

```typescript
interface WidgetConfig {
  containerId: string;
  eventId?: string;
  basePath?: string;
  initialRoute?: string;
  onReady?: () => void;           // Called when widget is mounted
  onError?: (error: Error) => void; // Called on mount errors
  onAction?: (action, data, element) => void;
  onRouteChange?: (route: string) => void;
}
```

## Callbacks

### 1. onRouteChange

**Purpose**: Notify parent application when widget route changes

**Signature**: `(route: string) => void`

**Implementation**: Uses `useLocation` hook within a `RouteChangeListener` component

**Triggered**: On every route change within the widget

**Example**:

```javascript
widget.mount({
  containerId: 'app',
  onRouteChange: (route) => {
    console.log('Widget navigated to:', route);
    // Update parent app state, sync URL, track analytics, etc.
  }
});
```

### 2. onAction

**Purpose**: Handle custom actions triggered by widget elements

**Signature**: `(action: string, data: Record<string, any>, element: HTMLElement) => void`

**Implementation**: Provided via `WidgetContext`, accessed via `useWidgetAction` hook

**Usage in components**:

```javascript
const handleAction = useWidgetAction();
<button onClick={handleAction} data-action="ticket-purchased" data-ticket-id="123">
  Buy Ticket
</button>
```

**Example**:

```javascript
widget.mount({
  containerId: 'app',
  onAction: (action, data, element) => {
    if (action === 'ticket-purchased') {
      // Handle ticket purchase in parent app
      console.log('Ticket purchased:', data.ticketId);
    }
  }
});
```

### 3. onReady

**Purpose**: Called when widget successfully mounts

**Signature**: `() => void`

**Available in**: `src/lib/app-widget.tsx` only

### 4. onError

**Purpose**: Called when widget mount fails

**Signature**: `(error: Error) => void`

**Available in**: `src/lib/app-widget.tsx` only

## Routing Architecture

### Router Type: MemoryRouter

* **No URL changes**: Routes are internal to the widget
* **Parent app control**: Use `onRouteChange` to sync with parent app
* **Initial route**: Set via `initialRoute` config option

### Route Change Detection

Implemented via `RouteChangeListener` component in `src/lib/app-widget.tsx`:

```typescript
const RouteChangeListener = () => {
  const location = useLocation();
  
  useEffect(() => {
    onRouteChange?.(location.pathname);
  }, [location]);
  
  return null;
};
```

## State Management

### 1. PageStateContext (`src/contexts/PageStateContext.tsx`)

**Purpose**: Cache page state to preserve UI state during navigation

**API**:

```typescript
const { getCachedState, setCachedState, clearCachedState } = usePageState('pageKey');
```

**Implementation**: Uses `useRef` with `Map` for persistence across renders

**Use case**: Preserve scroll position, form data, filters when navigating

### 2. WidgetContext (`src/contexts/WidgetContext.tsx`)

**Purpose**: Provide `onAction` callback throughout component tree

**API**:

```typescript
const handleAction = useWidgetAction();
```

**Implementation**: React Context + custom hook for easy access

**Use case**: Trigger parent app callbacks from any component

### 3. React Query (`@tanstack/react-query`)

**Purpose**: Data fetching and caching

**Configuration**:

```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,  // 5 minutes
      refetchOnWindowFocus: false
    }
  }
});
```

## Styling Architecture

### Design System

**Location**: `src/index.css` and `tailwind.config.ts`

**Critical Rules**:

1. Always use HSL colors
2. Use semantic tokens (CSS variables) instead of direct colors
3. No `text-white`, `bg-white`, `text-black`, etc.
4. Theme everything via design system

### CSS Variables Pattern

```css
:root {
  --primary: 220 90% 56%;        /* HSL values */
  --background: 0 0% 100%;
  --foreground: 222.2 84% 4.9%;
}

.dark {
  --background: 222.2 84% 4.9%;
  --foreground: 210 40% 98%;
}
```

### Usage in Components

```tsx
// ❌ WRONG
<div className="bg-white text-black">

// ✅ CORRECT
<div className="bg-background text-foreground">
```

### CSS Processing Pipeline

1. Write Tailwind classes
2. Tailwind processes into CSS
3. PostCSS applies autoprefixer
4. `vite-css-prefixer` adds `event-widget-` prefix (production only)
5. cssnano minifies (production only)

## File Structure

### Entry Points

* **`src/index.ts`**: Main entry point for UMD build, simple mount/unmount
* **`src/lib/app-widget.tsx`**: Alternative entry with advanced features
* **`src/App.tsx`**: Main React application component

### Key Files

* **`vite.config.ts`**: Build configuration
* **`src/index.css`**: Design system CSS variables
* **`tailwind.config.ts`**: Tailwind configuration
* **`src/widget-styles.css`**: Widget-specific styles
* **`src/contexts/PageStateContext.tsx`**: Page state caching
* **`src/contexts/WidgetContext.tsx`**: Action callback context
* **`src/components/Layout.tsx`**: Main layout wrapper

### Pages Directory

All page components in `src/pages/`:

* Overview, Dashboard, Tickets, TicketOrders, etc.
* Settings pages in `src/pages/settings/`
* NotFound for 404 handling

## Dependencies

### Core Dependencies

* **react** & **react-dom**: ^18.3.1 (externalized in build)
* **react-router-dom**: ^6.30.1 (routing)
* **@tanstack/react-query**: ^5.83.0 (data fetching)
* **tailwindcss**: CSS framework
* **@radix-ui/\***: UI component primitives

### Build Dependencies

* **vite**: Build tool
* **vite-css-prefixer**: Prefix CSS to avoid conflicts
* **autoprefixer**: CSS vendor prefixing
* **cssnano**: CSS minification

## Development vs Production

### Development Mode

* Readable code output
* Source maps enabled
* No CSS minification
* No CSS prefixing
* Faster build times

### Production Mode

* Minified output
* Tree-shaking enabled
* CSS prefixing with `event-widget-`
* CSS minification via cssnano
* Optimized bundle size

## Multiple Widget Instances

### Instance Tracking

Both entry points support multiple instances:

```typescript
// src/index.ts uses WeakMap
const roots = new WeakMap<HTMLElement, Root>();

// src/lib/app-widget.tsx uses Map
const instances = new Map<string, WidgetInstance>();
```

### Mounting Multiple Instances

```javascript
widget.mount({ containerId: 'widget-1', eventId: 'event-1' });
widget.mount({ containerId: 'widget-2', eventId: 'event-2' });
```

Each instance maintains its own:

* Routing state
* Page state cache
* React Query cache
* Event handlers

## Authentication & API

### Token Management (`src/lib/utils.ts`)

```typescript
export function getAuthToken(): string | null {
  // Development: Use dev token
  if (isInIframe()) {
    return "ACTZ-ad0d031843529fa8a3cf65b78c51e3d8";
  }
  
  // Production: Get from window.app.user.profile._id
  return (window as any).app?.user?.profile?._id || null;
}
```

### API URL Helper

```typescript
export function appendAuthToUrl(url: string): string {
  const authParam = getAuthQueryParam();
  if (!authParam) return url;
  
  const separator = url.includes('?') ? '&' : '?';
  return `${url}${separator}${authParam}`;
}
```

## Best Practices for AI Editing

### 1. Always Check Context First

* Review `src/index.ts` vs `src/lib/app-widget.tsx` - two different entry points
* Check which files are already in context before reading
* Understand the dual implementation pattern

### 2. Maintain Consistency

* If editing one entry point, consider if the other needs updates
* Keep callback signatures consistent across both implementations
* Maintain the same route structure in both approaches

### 3. Respect Architecture

* Don't replace MemoryRouter with BrowserRouter
* Don't bundle React/ReactDOM (they're externalized)
* Always use semantic tokens for colors (HSL only)
* Keep CSS prefixing configuration intact

### 4. Testing Considerations

* Test both mount methods (element-based and config-based)
* Test multiple simultaneous instances
* Test all callbacks (onRouteChange, onAction, onReady, onError)
* Test with and without React/ReactDOM available

### 5. Build Verification

* Always test both development and production builds
* Verify CSS prefixing in production output
* Check bundle size after changes
* Ensure externals aren't bundled

## Common Pitfalls

1. **Using direct colors instead of semantic tokens**
   * ❌ `className="bg-white text-black"`
   * ✅ `className="bg-background text-foreground"`

2. **Modifying URL directly**
   * Widget uses MemoryRouter, URL changes won't work
   * Use `onRouteChange` callback to inform parent

3. **Not handling both entry points**
   * Changes to routing/callbacks need updates in both `src/index.ts` and `src/lib/app-widget.tsx`

4. **Forgetting CSS prefixing in production**
   * Production CSS is prefixed with `event-widget-`
   * Don't remove or modify the `vite-css-prefixer` configuration

5. **Bundling React/ReactDOM**
   * These must remain externalized
   * Host application must provide them

## Future Considerations

* **HashRouter Migration**: If parent app integration requires URL sync, consider migrating to HashRouter
* **Micro-frontend Architecture**: Current `common-widgets/` suggests multi-widget system
* **State Persistence**: Consider adding localStorage support for PageStateContext
* **Error Boundaries**: Add React error boundaries for graceful failure handling
* **Analytics**: Built-in analytics hooks for tracking widget usage
