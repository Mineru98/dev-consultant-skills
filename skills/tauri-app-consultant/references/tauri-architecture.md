# Tauri Architecture Patterns

Reference guide for Tauri v2 application architecture.

## Core Concepts

### Tauri Command Pattern

Commands are Rust functions callable from JavaScript:

```rust
#[tauri::command]
fn greet(name: &str) -> String {
    format!("Hello, {}!", name)
}

// Async command
#[tauri::command]
async fn fetch_data(url: String) -> Result<String, String> {
    // async operation
    Ok(data)
}

// With state
#[tauri::command]
async fn get_count(state: State<'_, AppState>) -> i32 {
    *state.count.lock().unwrap()
}
```

### Frontend Invocation

```typescript
import { invoke } from '@tauri-apps/api/core';

// Simple call
const greeting = await invoke<string>('greet', { name: 'World' });

// With error handling
try {
    const data = await invoke<Data>('fetch_data', { url });
} catch (error) {
    console.error('Command failed:', error);
}
```

## State Management

### Tauri State (Backend)

```rust
use std::sync::Mutex;
use tauri::Manager;

struct AppState {
    db: Mutex<SqliteConnection>,
    counter: Mutex<i32>,
}

fn main() {
    tauri::Builder::default()
        .manage(AppState {
            db: Mutex::new(connection),
            counter: Mutex::new(0),
        })
        .invoke_handler(tauri::generate_handler![commands...])
        .run(tauri::generate_context!())
        .expect("error running app");
}
```

### Frontend State (React)

```typescript
// Zustand store
import { create } from 'zustand';

interface AppStore {
    items: Item[];
    loading: boolean;
    fetchItems: () => Promise<void>;
}

const useAppStore = create<AppStore>((set) => ({
    items: [],
    loading: false,
    fetchItems: async () => {
        set({ loading: true });
        const items = await invoke<Item[]>('get_items');
        set({ items, loading: false });
    },
}));
```

## Data Persistence

### SQLite with sqlx

```rust
use sqlx::sqlite::SqlitePool;

async fn setup_db() -> SqlitePool {
    let db_path = app_handle
        .path()
        .app_data_dir()
        .unwrap()
        .join("data.db");

    SqlitePool::connect(&format!("sqlite:{}?mode=rwc", db_path.display()))
        .await
        .unwrap()
}

#[tauri::command]
async fn create_item(
    title: String,
    pool: State<'_, SqlitePool>
) -> Result<Item, String> {
    sqlx::query_as!(
        Item,
        "INSERT INTO items (title) VALUES (?) RETURNING *",
        title
    )
    .fetch_one(pool.inner())
    .await
    .map_err(|e| e.to_string())
}
```

### Tauri State for Settings

```rust
use tauri_plugin_store::StoreBuilder;

fn main() {
    tauri::Builder::default()
        .plugin(tauri_plugin_store::Builder::default().build())
        .run(tauri::generate_context!())
}
```

```typescript
import { Store } from '@tauri-apps/plugin-store';

const store = new Store('settings.json');
await store.set('theme', 'dark');
const theme = await store.get<string>('theme');
```

## Event System

### Backend to Frontend

```rust
use tauri::Emitter;

#[tauri::command]
fn start_process(app: tauri::AppHandle) {
    std::thread::spawn(move || {
        // Long operation
        app.emit("progress", 50).unwrap();
        // More work
        app.emit("complete", result).unwrap();
    });
}
```

```typescript
import { listen } from '@tauri-apps/api/event';

const unlisten = await listen('progress', (event) => {
    console.log('Progress:', event.payload);
});

// Cleanup
unlisten();
```

### Frontend to Backend

```rust
use tauri::Listener;

fn main() {
    tauri::Builder::default()
        .setup(|app| {
            app.listen("user-action", |event| {
                println!("Received: {:?}", event.payload());
            });
            Ok(())
        })
}
```

## Window Management

### Creating Windows

```rust
use tauri::Manager;

#[tauri::command]
async fn open_settings(app: tauri::AppHandle) -> Result<(), String> {
    tauri::WebviewWindowBuilder::new(
        &app,
        "settings",
        tauri::WebviewUrl::App("settings.html".into())
    )
    .title("Settings")
    .inner_size(600.0, 400.0)
    .build()
    .map_err(|e| e.to_string())?;
    Ok(())
}
```

### Window Events

```typescript
import { getCurrentWindow } from '@tauri-apps/api/window';

const appWindow = getCurrentWindow();

await appWindow.onCloseRequested(async (event) => {
    const confirmed = await confirm('Are you sure?');
    if (!confirmed) {
        event.preventDefault();
    }
});
```

## Error Handling

### Rust Error Types

```rust
use thiserror::Error;

#[derive(Error, Debug)]
enum AppError {
    #[error("Database error: {0}")]
    Database(#[from] sqlx::Error),

    #[error("Not found: {0}")]
    NotFound(String),

    #[error("Invalid input: {0}")]
    Validation(String),
}

impl From<AppError> for String {
    fn from(err: AppError) -> Self {
        err.to_string()
    }
}

#[tauri::command]
async fn get_item(id: i32) -> Result<Item, AppError> {
    // Returns AppError which converts to String for JS
}
```

### Frontend Error Handling

```typescript
async function safeInvoke<T>(cmd: string, args?: object): Promise<T | null> {
    try {
        return await invoke<T>(cmd, args);
    } catch (error) {
        console.error(`Command ${cmd} failed:`, error);
        // Show user-friendly error
        return null;
    }
}
```

## Cross-Platform Considerations

### Path Handling

```rust
use tauri::Manager;

fn get_data_path(app: &tauri::AppHandle) -> PathBuf {
    app.path().app_data_dir().unwrap()
}

// Platform-specific
#[cfg(target_os = "windows")]
fn platform_specific() {
    // Windows-specific code
}

#[cfg(target_os = "macos")]
fn platform_specific() {
    // macOS-specific code
}

#[cfg(target_os = "linux")]
fn platform_specific() {
    // Linux-specific code
}
```

### Keyboard Shortcuts

```typescript
import { register } from '@tauri-apps/plugin-global-shortcut';

// Platform-aware shortcut
const modifier = navigator.platform.includes('Mac') ? 'Cmd' : 'Ctrl';
await register(`${modifier}+S`, () => {
    // Save action
});
```

## Security

### Capability Permissions

```json
// src-tauri/capabilities/main.json
{
    "identifier": "main",
    "windows": ["main"],
    "permissions": [
        "core:default",
        "fs:allow-app-data-dir-read",
        "fs:allow-app-data-dir-write",
        "sql:default"
    ]
}
```

### Input Validation

```rust
#[tauri::command]
fn process_input(input: String) -> Result<String, String> {
    // Validate input
    if input.is_empty() {
        return Err("Input cannot be empty".into());
    }
    if input.len() > 1000 {
        return Err("Input too long".into());
    }
    // Process validated input
    Ok(sanitized)
}
```

## Performance Tips

1. **Async Commands**: Use async for I/O operations
2. **Batch Operations**: Combine multiple DB operations
3. **Lazy Loading**: Load data on demand
4. **Connection Pooling**: Reuse database connections
5. **Frontend Caching**: Cache frequently accessed data
