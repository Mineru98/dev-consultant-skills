# Manifest V3 Guide

Complete reference for Chrome Extension Manifest V3 development.

## Manifest Structure

### Minimal Manifest

```json
{
  "manifest_version": 3,
  "name": "My Extension",
  "version": "1.0.0",
  "description": "A brief description"
}
```

### Complete Manifest

```json
{
  "manifest_version": 3,
  "name": "__MSG_extName__",
  "version": "1.0.0",
  "description": "__MSG_extDescription__",
  "default_locale": "en",

  "icons": {
    "16": "icons/icon16.png",
    "32": "icons/icon32.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  },

  "action": {
    "default_popup": "popup/popup.html",
    "default_icon": {
      "16": "icons/icon16.png",
      "32": "icons/icon32.png"
    },
    "default_title": "Click to open"
  },

  "background": {
    "service_worker": "background.js",
    "type": "module"
  },

  "content_scripts": [
    {
      "matches": ["https://*.example.com/*"],
      "js": ["content/content.js"],
      "css": ["content/content.css"],
      "run_at": "document_idle"
    }
  ],

  "permissions": [
    "storage",
    "activeTab",
    "alarms"
  ],

  "optional_permissions": [
    "tabs",
    "bookmarks"
  ],

  "host_permissions": [
    "https://*.example.com/*"
  ],

  "options_ui": {
    "page": "options/options.html",
    "open_in_tab": false
  },

  "web_accessible_resources": [
    {
      "resources": ["images/*"],
      "matches": ["https://*.example.com/*"]
    }
  ],

  "content_security_policy": {
    "extension_pages": "script-src 'self'; object-src 'self'"
  },

  "commands": {
    "toggle-feature": {
      "suggested_key": {
        "default": "Ctrl+Shift+Y",
        "mac": "Command+Shift+Y"
      },
      "description": "Toggle the feature"
    }
  }
}
```

## Permissions

### Common Permissions

| Permission | Use Case |
|------------|----------|
| `storage` | Save extension data |
| `activeTab` | Access current tab on click |
| `tabs` | Access tab URLs and titles |
| `alarms` | Schedule periodic tasks |
| `notifications` | Show desktop notifications |
| `contextMenus` | Add right-click menu items |
| `scripting` | Programmatic injection |
| `webRequest` | Observe network requests |
| `cookies` | Access browser cookies |

### Host Permissions

```json
{
  "host_permissions": [
    "https://www.example.com/*",
    "https://*.example.com/*",
    "*://*.example.com/*",
    "<all_urls>"
  ]
}
```

### Optional Permissions

```json
{
  "optional_permissions": ["tabs", "bookmarks"],
  "optional_host_permissions": ["https://*.google.com/*"]
}
```

```javascript
// Request at runtime
async function requestPermission() {
  const granted = await chrome.permissions.request({
    permissions: ['tabs'],
    origins: ['https://*.google.com/*']
  });

  if (granted) {
    console.log('Permission granted');
  }
}
```

## Service Worker (Background)

### Basic Setup

```javascript
// background.js
console.log('Service worker started');

// Install event
chrome.runtime.onInstalled.addListener((details) => {
  if (details.reason === 'install') {
    // First install
    chrome.storage.sync.set({ settings: defaultSettings });
  } else if (details.reason === 'update') {
    // Extension updated
    console.log('Updated to', chrome.runtime.getManifest().version);
  }
});

// Startup event
chrome.runtime.onStartup.addListener(() => {
  console.log('Browser started');
});
```

### Message Handling

```javascript
// Listen for messages
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  console.log('Message from:', sender.tab ? sender.tab.url : 'extension');

  switch (message.type) {
    case 'GET_DATA':
      handleGetData(message.payload).then(sendResponse);
      return true; // Keep channel open for async

    case 'SAVE_DATA':
      handleSaveData(message.payload);
      sendResponse({ success: true });
      break;

    default:
      sendResponse({ error: 'Unknown message type' });
  }
});

async function handleGetData(payload) {
  const data = await chrome.storage.sync.get([payload.key]);
  return data;
}

function handleSaveData(payload) {
  chrome.storage.sync.set({ [payload.key]: payload.value });
}
```

### Alarms (Periodic Tasks)

```javascript
// Create alarm
chrome.alarms.create('periodic-check', {
  periodInMinutes: 60
});

// Listen for alarm
chrome.alarms.onAlarm.addListener((alarm) => {
  if (alarm.name === 'periodic-check') {
    performCheck();
  }
});
```

## Content Scripts

### Static Injection (manifest.json)

```json
{
  "content_scripts": [
    {
      "matches": ["https://*.example.com/*"],
      "exclude_matches": ["https://example.com/admin/*"],
      "js": ["content.js"],
      "css": ["content.css"],
      "run_at": "document_idle",
      "all_frames": false
    }
  ]
}
```

### Programmatic Injection

```javascript
// Using scripting API
chrome.scripting.executeScript({
  target: { tabId: tabId },
  files: ['content.js']
});

// With function
chrome.scripting.executeScript({
  target: { tabId: tabId },
  func: (arg) => {
    console.log('Injected with arg:', arg);
  },
  args: ['hello']
});
```

### Content Script Communication

```javascript
// content.js
// Send to background
chrome.runtime.sendMessage({ type: 'PAGE_DATA', data: pageData }, (response) => {
  console.log('Response:', response);
});

// Listen from background
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.type === 'UPDATE_PAGE') {
    updatePage(message.data);
    sendResponse({ success: true });
  }
});
```

## Storage

### Storage Types

```javascript
// Sync storage (100KB, synced across devices)
await chrome.storage.sync.get(['key']);
await chrome.storage.sync.set({ key: value });

// Local storage (5MB, local only)
await chrome.storage.local.get(['key']);
await chrome.storage.local.set({ key: value });

// Session storage (in-memory, cleared on restart)
await chrome.storage.session.get(['key']);
await chrome.storage.session.set({ key: value });
```

### Storage Change Listener

```javascript
chrome.storage.onChanged.addListener((changes, area) => {
  for (const [key, { oldValue, newValue }] of Object.entries(changes)) {
    console.log(`Storage ${area}: ${key} changed from`, oldValue, 'to', newValue);
  }
});
```

## Action API

### Badge

```javascript
// Set badge text
chrome.action.setBadgeText({ text: '5' });
chrome.action.setBadgeText({ text: '5', tabId: tabId }); // Per tab

// Set badge color
chrome.action.setBadgeBackgroundColor({ color: '#FF0000' });

// Clear badge
chrome.action.setBadgeText({ text: '' });
```

### Dynamic Icon

```javascript
// Set icon
chrome.action.setIcon({
  path: {
    16: 'icons/active-16.png',
    32: 'icons/active-32.png'
  }
});

// Per tab
chrome.action.setIcon({
  tabId: tabId,
  path: 'icons/special.png'
});
```

### Enable/Disable

```javascript
// Disable action for specific tab
chrome.action.disable(tabId);

// Enable
chrome.action.enable(tabId);
```

## Context Menus

```javascript
// Create menu
chrome.contextMenus.create({
  id: 'my-menu',
  title: 'My Extension',
  contexts: ['selection', 'link', 'page']
});

// With submenu
chrome.contextMenus.create({
  id: 'parent',
  title: 'Parent Menu',
  contexts: ['all']
});

chrome.contextMenus.create({
  id: 'child',
  parentId: 'parent',
  title: 'Child Item',
  contexts: ['all']
});

// Handle click
chrome.contextMenus.onClicked.addListener((info, tab) => {
  if (info.menuItemId === 'my-menu') {
    console.log('Selected text:', info.selectionText);
    console.log('Link URL:', info.linkUrl);
  }
});
```

## Keyboard Commands

```json
{
  "commands": {
    "_execute_action": {
      "suggested_key": {
        "default": "Ctrl+Shift+E",
        "mac": "Command+Shift+E"
      }
    },
    "toggle-feature": {
      "suggested_key": {
        "default": "Alt+T"
      },
      "description": "Toggle the feature"
    }
  }
}
```

```javascript
chrome.commands.onCommand.addListener((command) => {
  if (command === 'toggle-feature') {
    toggleFeature();
  }
});
```

## Chrome Web Store Requirements

### Required Assets

| Asset | Size | Format |
|-------|------|--------|
| Icon | 128x128 | PNG |
| Small tile | 440x280 | PNG/JPEG |
| Marquee | 1400x560 | PNG/JPEG |
| Screenshots | 1280x800 or 640x400 | PNG/JPEG |

### Privacy Policy

Required if extension:
- Collects user data
- Uses remote servers
- Accesses personal information

### Listing Requirements

- Clear description
- Accurate screenshots
- Privacy practices disclosure
- Contact information
