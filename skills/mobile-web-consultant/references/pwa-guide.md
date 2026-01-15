# PWA Implementation Guide

Comprehensive guide for Progressive Web App development.

## PWA Requirements

### Core Requirements

1. **HTTPS** - Secure context required
2. **Service Worker** - For offline and caching
3. **Web App Manifest** - For installation
4. **Responsive** - Works on all devices
5. **Offline Capable** - Basic offline functionality

### Web App Manifest

```json
// manifest.json
{
  "name": "My PWA App",
  "short_name": "MyPWA",
  "description": "A progressive web application",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#3b82f6",
  "orientation": "portrait-primary",
  "icons": [
    {
      "src": "/icons/icon-72.png",
      "sizes": "72x72",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-96.png",
      "sizes": "96x96",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-128.png",
      "sizes": "128x128",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-144.png",
      "sizes": "144x144",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-152.png",
      "sizes": "152x152",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/icons/icon-384.png",
      "sizes": "384x384",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any maskable"
    }
  ],
  "screenshots": [
    {
      "src": "/screenshots/home.png",
      "sizes": "1080x1920",
      "type": "image/png",
      "form_factor": "narrow"
    }
  ]
}
```

### HTML Head Tags

```html
<head>
  <!-- Manifest -->
  <link rel="manifest" href="/manifest.json">

  <!-- Theme color -->
  <meta name="theme-color" content="#3b82f6">

  <!-- iOS -->
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="default">
  <meta name="apple-mobile-web-app-title" content="MyPWA">
  <link rel="apple-touch-icon" href="/icons/icon-152.png">

  <!-- Splash screens for iOS -->
  <link rel="apple-touch-startup-image" href="/splash/splash.png">

  <!-- Windows -->
  <meta name="msapplication-TileImage" content="/icons/icon-144.png">
  <meta name="msapplication-TileColor" content="#3b82f6">
</head>
```

## Service Worker

### Registration

```javascript
// main.js
if ('serviceWorker' in navigator) {
  window.addEventListener('load', async () => {
    try {
      const registration = await navigator.serviceWorker.register('/sw.js');
      console.log('SW registered:', registration.scope);
    } catch (error) {
      console.error('SW registration failed:', error);
    }
  });
}
```

### Basic Service Worker

```javascript
// sw.js
const CACHE_NAME = 'app-v1';
const STATIC_ASSETS = [
  '/',
  '/index.html',
  '/styles.css',
  '/app.js',
  '/icons/icon-192.png',
  '/offline.html'
];

// Install - cache static assets
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then((cache) => cache.addAll(STATIC_ASSETS))
      .then(() => self.skipWaiting())
  );
});

// Activate - clean old caches
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys()
      .then((keys) => Promise.all(
        keys
          .filter((key) => key !== CACHE_NAME)
          .map((key) => caches.delete(key))
      ))
      .then(() => self.clients.claim())
  );
});

// Fetch - serve from cache, fallback to network
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request)
      .then((cached) => cached || fetch(event.request))
      .catch(() => caches.match('/offline.html'))
  );
});
```

## Caching Strategies

### Cache First (Static Assets)

```javascript
// Best for: CSS, JS, images, fonts
self.addEventListener('fetch', (event) => {
  if (event.request.destination === 'style' ||
      event.request.destination === 'script' ||
      event.request.destination === 'image') {
    event.respondWith(
      caches.match(event.request)
        .then((cached) => cached || fetchAndCache(event.request))
    );
  }
});

async function fetchAndCache(request) {
  const response = await fetch(request);
  const cache = await caches.open(CACHE_NAME);
  cache.put(request, response.clone());
  return response;
}
```

### Network First (API Data)

```javascript
// Best for: API calls, dynamic content
self.addEventListener('fetch', (event) => {
  if (event.request.url.includes('/api/')) {
    event.respondWith(
      fetch(event.request)
        .then((response) => {
          const clone = response.clone();
          caches.open('api-cache').then((cache) => {
            cache.put(event.request, clone);
          });
          return response;
        })
        .catch(() => caches.match(event.request))
    );
  }
});
```

### Stale While Revalidate

```javascript
// Best for: News, social feeds
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.match(event.request).then((cached) => {
        const fetched = fetch(event.request).then((network) => {
          cache.put(event.request, network.clone());
          return network;
        });
        return cached || fetched;
      });
    })
  );
});
```

## IndexedDB with localbase

### Setup

```javascript
import Localbase from 'localbase';

const db = new Localbase('myapp');

// Disable console logs in production
db.config.debug = false;
```

### CRUD Operations

```javascript
// Create
await db.collection('items').add({
  id: crypto.randomUUID(),
  title: 'New Item',
  createdAt: new Date().toISOString()
});

// Read all
const items = await db.collection('items').get();

// Read one
const item = await db.collection('items').doc({ id: '123' }).get();

// Update
await db.collection('items').doc({ id: '123' }).update({
  title: 'Updated Title'
});

// Delete
await db.collection('items').doc({ id: '123' }).delete();

// Delete collection
await db.collection('items').delete();
```

### Offline Sync Pattern

```javascript
class SyncManager {
  constructor() {
    this.pendingQueue = [];
    this.db = new Localbase('myapp');
  }

  async addToQueue(action) {
    await this.db.collection('syncQueue').add({
      id: crypto.randomUUID(),
      action,
      timestamp: Date.now()
    });
  }

  async processQueue() {
    if (!navigator.onLine) return;

    const pending = await this.db.collection('syncQueue').get();

    for (const item of pending) {
      try {
        await this.syncItem(item.action);
        await this.db.collection('syncQueue').doc({ id: item.id }).delete();
      } catch (error) {
        console.error('Sync failed:', error);
      }
    }
  }

  async syncItem(action) {
    // Send to server
    await fetch('/api/sync', {
      method: 'POST',
      body: JSON.stringify(action)
    });
  }
}

// Listen for online event
window.addEventListener('online', () => {
  syncManager.processQueue();
});
```

## Install Prompt

```javascript
let deferredPrompt;

window.addEventListener('beforeinstallprompt', (e) => {
  e.preventDefault();
  deferredPrompt = e;
  showInstallButton();
});

function showInstallButton() {
  const btn = document.getElementById('install-btn');
  btn.style.display = 'block';
  btn.addEventListener('click', async () => {
    deferredPrompt.prompt();
    const { outcome } = await deferredPrompt.userChoice;
    console.log('Install outcome:', outcome);
    deferredPrompt = null;
    btn.style.display = 'none';
  });
}

window.addEventListener('appinstalled', () => {
  console.log('App installed');
  deferredPrompt = null;
});
```

## Offline Page

```html
<!-- offline.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Offline - MyPWA</title>
  <style>
    body {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      font-family: system-ui, sans-serif;
      text-align: center;
      padding: 2rem;
    }
    .icon { font-size: 4rem; margin-bottom: 1rem; }
    h1 { margin-bottom: 0.5rem; }
    p { color: #666; margin-bottom: 1.5rem; }
    button {
      padding: 0.75rem 1.5rem;
      background: #3b82f6;
      color: white;
      border: none;
      border-radius: 0.5rem;
      font-size: 1rem;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <div class="icon">📶</div>
  <h1>You're Offline</h1>
  <p>Please check your internet connection and try again.</p>
  <button onclick="location.reload()">Try Again</button>
</body>
</html>
```

## Performance Checklist

- [ ] Service Worker registered
- [ ] Static assets cached
- [ ] Offline page available
- [ ] Web App Manifest complete
- [ ] All icon sizes provided
- [ ] HTTPS enabled
- [ ] Lighthouse PWA audit passed
- [ ] Install prompt handled
- [ ] Update flow implemented
