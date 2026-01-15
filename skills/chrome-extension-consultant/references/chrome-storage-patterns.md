# Chrome Storage Patterns

Best practices for data storage in Chrome extensions.

## Storage Types Overview

| Type | Limit | Sync | Persists | Use Case |
|------|-------|------|----------|----------|
| `sync` | 100KB | Yes | Yes | User settings |
| `local` | 5MB | No | Yes | Large data, cache |
| `session` | 1MB | No | No | Temp state |

## Basic Operations

### Read/Write

```javascript
// Single key
const result = await chrome.storage.sync.get('key');
const value = result.key;

// Multiple keys
const result = await chrome.storage.sync.get(['key1', 'key2']);
console.log(result.key1, result.key2);

// All keys
const all = await chrome.storage.sync.get(null);

// Write
await chrome.storage.sync.set({ key: value });
await chrome.storage.sync.set({ key1: value1, key2: value2 });

// Remove
await chrome.storage.sync.remove('key');
await chrome.storage.sync.remove(['key1', 'key2']);

// Clear all
await chrome.storage.sync.clear();
```

### Change Listener

```javascript
chrome.storage.onChanged.addListener((changes, areaName) => {
  for (const [key, { oldValue, newValue }] of Object.entries(changes)) {
    console.log(`${areaName}.${key}:`, oldValue, '→', newValue);
  }
});
```

## Storage Manager Pattern

```javascript
class StorageManager {
  constructor(area = 'sync') {
    this.storage = chrome.storage[area];
  }

  async get(key, defaultValue = null) {
    const result = await this.storage.get([key]);
    return result[key] ?? defaultValue;
  }

  async set(key, value) {
    await this.storage.set({ [key]: value });
  }

  async update(key, updater) {
    const current = await this.get(key);
    const updated = updater(current);
    await this.set(key, updated);
    return updated;
  }

  async remove(key) {
    await this.storage.remove(key);
  }

  async clear() {
    await this.storage.clear();
  }

  async getAll() {
    return this.storage.get(null);
  }

  onChange(callback) {
    const listener = (changes, area) => {
      if (area === this.storage.QUOTA_BYTES ? 'sync' : 'local') {
        callback(changes);
      }
    };
    chrome.storage.onChanged.addListener(listener);
    return () => chrome.storage.onChanged.removeListener(listener);
  }
}

// Usage
const syncStorage = new StorageManager('sync');
const localStorage = new StorageManager('local');

await syncStorage.set('theme', 'dark');
const theme = await syncStorage.get('theme', 'light');
```

## Settings Pattern

```javascript
const DEFAULT_SETTINGS = {
  enabled: true,
  theme: 'system',
  fontSize: 14,
  notifications: {
    enabled: true,
    sound: false
  }
};

class SettingsManager {
  constructor() {
    this.STORAGE_KEY = 'settings';
    this.listeners = new Set();
  }

  async load() {
    const result = await chrome.storage.sync.get(this.STORAGE_KEY);
    return this.merge(DEFAULT_SETTINGS, result[this.STORAGE_KEY] || {});
  }

  async save(settings) {
    await chrome.storage.sync.set({ [this.STORAGE_KEY]: settings });
    this.notify(settings);
  }

  async update(path, value) {
    const settings = await this.load();
    this.setPath(settings, path, value);
    await this.save(settings);
    return settings;
  }

  async reset() {
    await this.save(DEFAULT_SETTINGS);
    return DEFAULT_SETTINGS;
  }

  // Deep merge
  merge(target, source) {
    const result = { ...target };
    for (const key of Object.keys(source)) {
      if (source[key] && typeof source[key] === 'object' && !Array.isArray(source[key])) {
        result[key] = this.merge(target[key] || {}, source[key]);
      } else {
        result[key] = source[key];
      }
    }
    return result;
  }

  // Set nested path
  setPath(obj, path, value) {
    const keys = path.split('.');
    let current = obj;
    for (let i = 0; i < keys.length - 1; i++) {
      current = current[keys[i]];
    }
    current[keys[keys.length - 1]] = value;
  }

  // Subscribe to changes
  subscribe(callback) {
    this.listeners.add(callback);
    return () => this.listeners.delete(callback);
  }

  notify(settings) {
    this.listeners.forEach(cb => cb(settings));
  }
}

// Usage
const settings = new SettingsManager();

const currentSettings = await settings.load();
await settings.update('notifications.sound', true);
await settings.reset();

// Subscribe
const unsubscribe = settings.subscribe((newSettings) => {
  console.log('Settings changed:', newSettings);
});
```

## Cache Pattern

```javascript
class CacheManager {
  constructor(maxAge = 3600000) { // 1 hour default
    this.maxAge = maxAge;
  }

  async get(key) {
    const result = await chrome.storage.local.get(key);
    const cached = result[key];

    if (!cached) return null;

    if (Date.now() - cached.timestamp > this.maxAge) {
      await this.remove(key);
      return null;
    }

    return cached.data;
  }

  async set(key, data, customMaxAge = null) {
    await chrome.storage.local.set({
      [key]: {
        data,
        timestamp: Date.now(),
        maxAge: customMaxAge || this.maxAge
      }
    });
  }

  async getOrFetch(key, fetcher, maxAge = null) {
    const cached = await this.get(key);
    if (cached !== null) return cached;

    const data = await fetcher();
    await this.set(key, data, maxAge);
    return data;
  }

  async remove(key) {
    await chrome.storage.local.remove(key);
  }

  async clear() {
    await chrome.storage.local.clear();
  }

  async cleanup() {
    const all = await chrome.storage.local.get(null);
    const now = Date.now();
    const expired = [];

    for (const [key, value] of Object.entries(all)) {
      if (value.timestamp && now - value.timestamp > (value.maxAge || this.maxAge)) {
        expired.push(key);
      }
    }

    if (expired.length > 0) {
      await chrome.storage.local.remove(expired);
    }

    return expired.length;
  }
}

// Usage
const cache = new CacheManager();

// Get with fetch fallback
const userData = await cache.getOrFetch('user-data', async () => {
  const response = await fetch('/api/user');
  return response.json();
}, 300000); // 5 min cache

// Periodic cleanup (in service worker)
chrome.alarms.create('cache-cleanup', { periodInMinutes: 60 });
chrome.alarms.onAlarm.addListener(async (alarm) => {
  if (alarm.name === 'cache-cleanup') {
    const removed = await cache.cleanup();
    console.log(`Cleaned up ${removed} expired cache entries`);
  }
});
```

## Session State Pattern

```javascript
class SessionState {
  async get(key, defaultValue = null) {
    const result = await chrome.storage.session.get(key);
    return result[key] ?? defaultValue;
  }

  async set(key, value) {
    await chrome.storage.session.set({ [key]: value });
  }

  async setAll(data) {
    await chrome.storage.session.set(data);
  }

  async clear() {
    await chrome.storage.session.clear();
  }
}

// Usage - temporary state that doesn't persist
const session = new SessionState();

await session.set('currentTab', tabId);
await session.set('lastAction', { type: 'search', query: 'test' });

const currentTab = await session.get('currentTab');
```

## Quota Management

```javascript
async function checkStorageQuota() {
  const sync = await chrome.storage.sync.getBytesInUse(null);
  const local = await chrome.storage.local.getBytesInUse(null);

  return {
    sync: {
      used: sync,
      total: chrome.storage.sync.QUOTA_BYTES,
      percentage: (sync / chrome.storage.sync.QUOTA_BYTES * 100).toFixed(2)
    },
    local: {
      used: local,
      total: chrome.storage.local.QUOTA_BYTES,
      percentage: (local / chrome.storage.local.QUOTA_BYTES * 100).toFixed(2)
    }
  };
}

// Usage
const quota = await checkStorageQuota();
console.log(`Sync storage: ${quota.sync.percentage}% used`);
console.log(`Local storage: ${quota.local.percentage}% used`);
```

## Migration Pattern

```javascript
const CURRENT_VERSION = 2;

async function migrateStorage() {
  const { storageVersion = 1 } = await chrome.storage.local.get('storageVersion');

  if (storageVersion >= CURRENT_VERSION) return;

  // Migration v1 → v2
  if (storageVersion < 2) {
    const oldData = await chrome.storage.sync.get('settings');
    if (oldData.settings) {
      // Transform old format to new format
      const newSettings = {
        ...oldData.settings,
        // Add new fields with defaults
        newFeature: true
      };
      await chrome.storage.sync.set({ settings: newSettings });
    }
  }

  // Future: Migration v2 → v3
  // if (storageVersion < 3) { ... }

  await chrome.storage.local.set({ storageVersion: CURRENT_VERSION });
  console.log(`Storage migrated to version ${CURRENT_VERSION}`);
}

// Run on install/update
chrome.runtime.onInstalled.addListener(async (details) => {
  if (details.reason === 'install' || details.reason === 'update') {
    await migrateStorage();
  }
});
```
