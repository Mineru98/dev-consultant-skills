# Extension Architecture Patterns

Best practices for Chrome Extension architecture in Manifest V3.

## Component Architecture

### Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     Chrome Browser                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────┐     ┌─────────────────┐     ┌───────────┐ │
│  │   Popup     │     │ Service Worker  │     │  Options  │ │
│  │   (UI)      │◄───►│  (Background)   │◄───►│   (UI)    │ │
│  └─────────────┘     └────────┬────────┘     └───────────┘ │
│                               │                             │
│                               ▼                             │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                   Content Scripts                        ││
│  │            (Injected into web pages)                     ││
│  └─────────────────────────────────────────────────────────┘│
│                               │                             │
│                               ▼                             │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                      Web Page                            ││
│  └─────────────────────────────────────────────────────────┘│
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## Project Structure

### Recommended Layout

```
extension/
├── manifest.json
├── background.js
├── popup/
│   ├── popup.html
│   ├── popup.js
│   └── popup.css
├── options/
│   ├── options.html
│   ├── options.js
│   └── options.css
├── content/
│   ├── content.js
│   └── content.css
├── lib/
│   ├── storage.js
│   ├── messaging.js
│   └── utils.js
├── icons/
│   ├── icon16.png
│   ├── icon32.png
│   ├── icon48.png
│   └── icon128.png
└── _locales/
    └── en/
        └── messages.json
```

### Module Organization

```javascript
// lib/storage.js
export const Storage = {
  async get(key) {
    const result = await chrome.storage.sync.get([key]);
    return result[key];
  },

  async set(key, value) {
    await chrome.storage.sync.set({ [key]: value });
  },

  async getAll() {
    return chrome.storage.sync.get(null);
  },

  async clear() {
    await chrome.storage.sync.clear();
  }
};

// lib/messaging.js
export const Messaging = {
  async sendToBackground(type, payload) {
    return chrome.runtime.sendMessage({ type, payload });
  },

  async sendToTab(tabId, type, payload) {
    return chrome.tabs.sendMessage(tabId, { type, payload });
  },

  async sendToActiveTab(type, payload) {
    const [tab] = await chrome.tabs.query({ active: true, currentWindow: true });
    return chrome.tabs.sendMessage(tab.id, { type, payload });
  }
};
```

## Message Passing Patterns

### Request-Response Pattern

```javascript
// background.js
const handlers = {
  GET_DATA: async (payload) => {
    const data = await Storage.get(payload.key);
    return { success: true, data };
  },

  SAVE_DATA: async (payload) => {
    await Storage.set(payload.key, payload.value);
    return { success: true };
  },

  FETCH_EXTERNAL: async (payload) => {
    const response = await fetch(payload.url);
    const data = await response.json();
    return { success: true, data };
  }
};

chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  const handler = handlers[message.type];

  if (handler) {
    handler(message.payload, sender)
      .then(sendResponse)
      .catch((error) => sendResponse({ success: false, error: error.message }));
    return true; // Keep channel open
  }
});

// popup.js or content.js
async function getData(key) {
  const response = await chrome.runtime.sendMessage({
    type: 'GET_DATA',
    payload: { key }
  });

  if (response.success) {
    return response.data;
  }
  throw new Error(response.error);
}
```

### Event Broadcasting Pattern

```javascript
// background.js
function broadcast(type, data) {
  // To all extension pages
  chrome.runtime.sendMessage({ type, data }).catch(() => {});

  // To all content scripts
  chrome.tabs.query({}, (tabs) => {
    tabs.forEach((tab) => {
      chrome.tabs.sendMessage(tab.id, { type, data }).catch(() => {});
    });
  });
}

// On settings change
chrome.storage.onChanged.addListener((changes) => {
  broadcast('SETTINGS_CHANGED', changes);
});

// Listen in popup/content
chrome.runtime.onMessage.addListener((message) => {
  if (message.type === 'SETTINGS_CHANGED') {
    updateUI(message.data);
  }
});
```

### Port-based Communication

```javascript
// Long-lived connection
// popup.js
const port = chrome.runtime.connect({ name: 'popup' });

port.onMessage.addListener((message) => {
  console.log('Received:', message);
});

port.postMessage({ type: 'INIT' });

// background.js
const connections = new Map();

chrome.runtime.onConnect.addListener((port) => {
  connections.set(port.name, port);

  port.onMessage.addListener((message) => {
    handlePortMessage(port, message);
  });

  port.onDisconnect.addListener(() => {
    connections.delete(port.name);
  });
});
```

## Storage Patterns

### Settings Manager

```javascript
// lib/settings.js
const DEFAULT_SETTINGS = {
  enabled: true,
  theme: 'auto',
  notifications: true,
  autoSync: false
};

export const Settings = {
  async load() {
    const stored = await chrome.storage.sync.get('settings');
    return { ...DEFAULT_SETTINGS, ...stored.settings };
  },

  async save(settings) {
    await chrome.storage.sync.set({ settings });
  },

  async update(partial) {
    const current = await this.load();
    const updated = { ...current, ...partial };
    await this.save(updated);
    return updated;
  },

  async reset() {
    await this.save(DEFAULT_SETTINGS);
    return DEFAULT_SETTINGS;
  }
};
```

### Data Cache Pattern

```javascript
// lib/cache.js
export const Cache = {
  async get(key, maxAge = 3600000) { // 1 hour default
    const result = await chrome.storage.local.get([key]);
    const cached = result[key];

    if (cached && Date.now() - cached.timestamp < maxAge) {
      return cached.data;
    }
    return null;
  },

  async set(key, data) {
    await chrome.storage.local.set({
      [key]: {
        data,
        timestamp: Date.now()
      }
    });
  },

  async invalidate(key) {
    await chrome.storage.local.remove([key]);
  },

  async clear() {
    await chrome.storage.local.clear();
  }
};
```

## Error Handling

### Centralized Error Handler

```javascript
// lib/errors.js
export class ExtensionError extends Error {
  constructor(code, message, details = {}) {
    super(message);
    this.code = code;
    this.details = details;
  }
}

export const ErrorCodes = {
  STORAGE_ERROR: 'STORAGE_ERROR',
  NETWORK_ERROR: 'NETWORK_ERROR',
  PERMISSION_DENIED: 'PERMISSION_DENIED',
  INVALID_INPUT: 'INVALID_INPUT'
};

export function handleError(error) {
  console.error('[Extension Error]', error);

  if (error instanceof ExtensionError) {
    // Known error
    return { success: false, error: error.message, code: error.code };
  }

  // Unknown error
  return { success: false, error: 'An unexpected error occurred' };
}
```

### Safe Async Wrapper

```javascript
// lib/utils.js
export async function safeAsync(fn) {
  try {
    const result = await fn();
    return { success: true, data: result };
  } catch (error) {
    return handleError(error);
  }
}

// Usage
const result = await safeAsync(() => Storage.get('key'));
if (result.success) {
  console.log(result.data);
} else {
  console.error(result.error);
}
```

## Content Script Patterns

### Isolated World Communication

```javascript
// content.js
// Inject script into page context
function injectScript(file) {
  const script = document.createElement('script');
  script.src = chrome.runtime.getURL(file);
  script.onload = () => script.remove();
  (document.head || document.documentElement).appendChild(script);
}

// Communication via custom events
window.addEventListener('extension-data', (event) => {
  const data = event.detail;
  chrome.runtime.sendMessage({ type: 'PAGE_DATA', data });
});

// injected.js (in web_accessible_resources)
window.dispatchEvent(new CustomEvent('extension-data', {
  detail: { pageVariable: window.someData }
}));
```

### DOM Observer Pattern

```javascript
// content.js
function observeDOM(selector, callback) {
  // Check for existing elements
  document.querySelectorAll(selector).forEach(callback);

  // Watch for new elements
  const observer = new MutationObserver((mutations) => {
    mutations.forEach((mutation) => {
      mutation.addedNodes.forEach((node) => {
        if (node.nodeType === 1) {
          if (node.matches(selector)) {
            callback(node);
          }
          node.querySelectorAll?.(selector).forEach(callback);
        }
      });
    });
  });

  observer.observe(document.body, {
    childList: true,
    subtree: true
  });

  return () => observer.disconnect();
}

// Usage
const cleanup = observeDOM('.target-class', (element) => {
  enhanceElement(element);
});
```

## Security Best Practices

1. **Minimal Permissions** - Request only what you need
2. **Input Validation** - Validate all external input
3. **Content Security Policy** - Use strict CSP
4. **No Remote Code** - Never eval() or remote scripts
5. **Secure Storage** - Don't store sensitive data in sync storage
6. **HTTPS Only** - Use secure connections
7. **Sanitize DOM** - Prevent XSS in content scripts
