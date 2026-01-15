# Tauri Commands Reference

Complete guide to implementing Tauri commands for frontend-backend communication.

## Command Basics

### Simple Command

```rust
#[tauri::command]
fn greet(name: &str) -> String {
    format!("Hello, {}!", name)
}
```

```typescript
import { invoke } from '@tauri-apps/api/core';

const greeting = await invoke<string>('greet', { name: 'World' });
```

### Async Command

```rust
#[tauri::command]
async fn fetch_data(url: String) -> Result<String, String> {
    let response = reqwest::get(&url)
        .await
        .map_err(|e| e.to_string())?;

    response.text()
        .await
        .map_err(|e| e.to_string())
}
```

### Command with State

```rust
use tauri::State;

struct AppState {
    counter: std::sync::Mutex<i32>,
}

#[tauri::command]
fn increment(state: State<'_, AppState>) -> i32 {
    let mut counter = state.counter.lock().unwrap();
    *counter += 1;
    *counter
}
```

### Command with AppHandle

```rust
#[tauri::command]
fn open_window(app: tauri::AppHandle) -> Result<(), String> {
    tauri::WebviewWindowBuilder::new(
        &app,
        "settings",
        tauri::WebviewUrl::App("settings.html".into())
    )
    .title("Settings")
    .build()
    .map_err(|e| e.to_string())?;

    Ok(())
}
```

## Parameter Types

### Primitive Types

```rust
#[tauri::command]
fn calculate(
    text: String,       // String (owned)
    count: i32,         // Integer
    enabled: bool,      // Boolean
    ratio: f64,         // Float
) -> String {
    // ...
}
```

### Complex Types

```rust
use serde::{Deserialize, Serialize};

#[derive(Debug, Deserialize)]
struct CreateInput {
    title: String,
    description: Option<String>,
    tags: Vec<String>,
}

#[derive(Debug, Serialize)]
struct Item {
    id: i64,
    title: String,
}

#[tauri::command]
fn create_item(input: CreateInput) -> Result<Item, String> {
    // ...
}
```

```typescript
// Frontend call
const item = await invoke<Item>('create_item', {
    input: {
        title: 'New Item',
        description: 'Optional desc',
        tags: ['tag1', 'tag2']
    }
});
```

### Optional Parameters

```rust
#[tauri::command]
fn search(
    query: String,
    limit: Option<i32>,
    offset: Option<i32>,
) -> Vec<Item> {
    let limit = limit.unwrap_or(10);
    let offset = offset.unwrap_or(0);
    // ...
}
```

```typescript
// All variations work
await invoke('search', { query: 'test' });
await invoke('search', { query: 'test', limit: 20 });
await invoke('search', { query: 'test', limit: 20, offset: 40 });
```

## Return Types

### Result Type

```rust
#[tauri::command]
fn might_fail(input: String) -> Result<String, String> {
    if input.is_empty() {
        return Err("Input cannot be empty".into());
    }
    Ok(format!("Processed: {}", input))
}
```

```typescript
try {
    const result = await invoke<string>('might_fail', { input: '' });
} catch (error) {
    // error is the String from Err()
    console.error('Failed:', error);
}
```

### Complex Result

```rust
#[derive(Debug, Serialize)]
struct ApiResponse<T> {
    data: T,
    meta: ResponseMeta,
}

#[derive(Debug, Serialize)]
struct ResponseMeta {
    total: i64,
    page: i32,
}

#[tauri::command]
fn get_paginated(page: i32) -> Result<ApiResponse<Vec<Item>>, String> {
    Ok(ApiResponse {
        data: items,
        meta: ResponseMeta { total: 100, page },
    })
}
```

### Void Return

```rust
#[tauri::command]
fn log_event(event: String) {
    println!("Event: {}", event);
    // No return value
}
```

```typescript
await invoke('log_event', { event: 'button_clicked' });
// Returns undefined
```

## CRUD Commands Pattern

```rust
// Create
#[tauri::command]
async fn create_item(
    input: CreateItemInput,
    state: State<'_, AppState>
) -> Result<Item, String> {
    state.repo.create(input).await.map_err(|e| e.to_string())
}

// Read All
#[tauri::command]
async fn get_items(state: State<'_, AppState>) -> Result<Vec<Item>, String> {
    state.repo.find_all().await.map_err(|e| e.to_string())
}

// Read One
#[tauri::command]
async fn get_item(id: i64, state: State<'_, AppState>) -> Result<Item, String> {
    state.repo
        .find_by_id(id)
        .await
        .map_err(|e| e.to_string())?
        .ok_or_else(|| format!("Item {} not found", id))
}

// Update
#[tauri::command]
async fn update_item(
    id: i64,
    input: UpdateItemInput,
    state: State<'_, AppState>
) -> Result<Item, String> {
    state.repo.update(id, input).await.map_err(|e| e.to_string())
}

// Delete
#[tauri::command]
async fn delete_item(id: i64, state: State<'_, AppState>) -> Result<(), String> {
    state.repo.delete(id).await.map_err(|e| e.to_string())
}
```

## Registering Commands

```rust
fn main() {
    tauri::Builder::default()
        .manage(AppState::new())
        .invoke_handler(tauri::generate_handler![
            // CRUD commands
            create_item,
            get_items,
            get_item,
            update_item,
            delete_item,
            // Other commands
            get_settings,
            update_settings,
            open_window,
        ])
        .run(tauri::generate_context!())
        .unwrap();
}
```

## Frontend Wrapper

```typescript
// src/lib/tauri.ts
import { invoke } from '@tauri-apps/api/core';

// Type-safe wrapper
export const api = {
    items: {
        getAll: () => invoke<Item[]>('get_items'),
        getOne: (id: number) => invoke<Item>('get_item', { id }),
        create: (input: CreateItemInput) => invoke<Item>('create_item', { input }),
        update: (id: number, input: UpdateItemInput) =>
            invoke<Item>('update_item', { id, input }),
        delete: (id: number) => invoke<void>('delete_item', { id }),
    },
    settings: {
        get: () => invoke<Settings>('get_settings'),
        update: (settings: Settings) => invoke<Settings>('update_settings', { settings }),
    },
};

// Usage
const items = await api.items.getAll();
const item = await api.items.create({ title: 'New' });
```

## Error Handling Pattern

```typescript
// Generic error handler
async function safeInvoke<T>(
    command: string,
    args?: Record<string, unknown>
): Promise<{ data: T | null; error: string | null }> {
    try {
        const data = await invoke<T>(command, args);
        return { data, error: null };
    } catch (error) {
        return { data: null, error: String(error) };
    }
}

// Usage
const { data: items, error } = await safeInvoke<Item[]>('get_items');
if (error) {
    showErrorToast(error);
} else {
    setItems(items);
}
```
