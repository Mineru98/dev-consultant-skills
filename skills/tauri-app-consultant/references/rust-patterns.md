# Rust Patterns for Tauri

Essential Rust patterns and best practices for Tauri backend development.

## Error Handling

### Result Type

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("Database error: {0}")]
    Database(#[from] sqlx::Error),

    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),

    #[error("Not found: {0}")]
    NotFound(String),

    #[error("Validation failed: {0}")]
    Validation(String),
}

// Convert to String for Tauri commands
impl From<AppError> for String {
    fn from(err: AppError) -> Self {
        err.to_string()
    }
}
```

### The ? Operator

```rust
#[tauri::command]
async fn load_data(path: String) -> Result<Data, AppError> {
    let content = std::fs::read_to_string(&path)?;  // Propagates Io error
    let data: Data = serde_json::from_str(&content)?;  // Propagates Parse error
    Ok(data)
}
```

## Concurrency

### Mutex for Shared State

```rust
use std::sync::Mutex;

pub struct AppState {
    counter: Mutex<i32>,
    cache: Mutex<HashMap<String, Data>>,
}

#[tauri::command]
fn increment(state: State<'_, AppState>) -> i32 {
    let mut counter = state.counter.lock().unwrap();
    *counter += 1;
    *counter
}
```

### RwLock for Read-Heavy Access

```rust
use std::sync::RwLock;

pub struct AppState {
    settings: RwLock<Settings>,
}

#[tauri::command]
fn get_setting(key: String, state: State<'_, AppState>) -> Option<String> {
    let settings = state.settings.read().unwrap();
    settings.get(&key).cloned()
}

#[tauri::command]
fn set_setting(key: String, value: String, state: State<'_, AppState>) {
    let mut settings = state.settings.write().unwrap();
    settings.insert(key, value);
}
```

### Async with Tokio

```rust
use tokio::time::{sleep, Duration};

#[tauri::command]
async fn long_operation() -> Result<String, String> {
    // Simulate async work
    sleep(Duration::from_secs(2)).await;
    Ok("Done".into())
}

// Spawn background task
#[tauri::command]
fn start_background_task(app: tauri::AppHandle) {
    tokio::spawn(async move {
        loop {
            // Background work
            app.emit("heartbeat", ()).unwrap();
            sleep(Duration::from_secs(60)).await;
        }
    });
}
```

## Data Structures

### Serde for Serialization

```rust
use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Item {
    pub id: i64,
    pub title: String,
    pub completed: bool,
    #[serde(with = "chrono::serde::ts_seconds")]
    pub created_at: DateTime<Utc>,
}

#[derive(Debug, Deserialize)]
pub struct CreateItemInput {
    pub title: String,
}

#[derive(Debug, Deserialize)]
pub struct UpdateItemInput {
    pub title: Option<String>,
    pub completed: Option<bool>,
}
```

### Option and Default

```rust
#[derive(Debug, Serialize, Deserialize)]
pub struct Settings {
    #[serde(default = "default_theme")]
    pub theme: String,
    #[serde(default)]
    pub notifications: bool,
}

fn default_theme() -> String {
    "system".into()
}
```

## Database Patterns

### Repository Pattern

```rust
pub struct ItemRepository {
    pool: SqlitePool,
}

impl ItemRepository {
    pub fn new(pool: SqlitePool) -> Self {
        Self { pool }
    }

    pub async fn find_all(&self) -> Result<Vec<Item>, sqlx::Error> {
        sqlx::query_as!(Item, "SELECT * FROM items ORDER BY created_at DESC")
            .fetch_all(&self.pool)
            .await
    }

    pub async fn find_by_id(&self, id: i64) -> Result<Option<Item>, sqlx::Error> {
        sqlx::query_as!(Item, "SELECT * FROM items WHERE id = ?", id)
            .fetch_optional(&self.pool)
            .await
    }

    pub async fn create(&self, input: CreateItemInput) -> Result<Item, sqlx::Error> {
        sqlx::query_as!(
            Item,
            "INSERT INTO items (title) VALUES (?) RETURNING *",
            input.title
        )
        .fetch_one(&self.pool)
        .await
    }

    pub async fn update(&self, id: i64, input: UpdateItemInput) -> Result<Item, sqlx::Error> {
        // Build dynamic update query
        let item = self.find_by_id(id).await?.ok_or(sqlx::Error::RowNotFound)?;

        let title = input.title.unwrap_or(item.title);
        let completed = input.completed.unwrap_or(item.completed);

        sqlx::query_as!(
            Item,
            "UPDATE items SET title = ?, completed = ? WHERE id = ? RETURNING *",
            title, completed, id
        )
        .fetch_one(&self.pool)
        .await
    }

    pub async fn delete(&self, id: i64) -> Result<(), sqlx::Error> {
        sqlx::query!("DELETE FROM items WHERE id = ?", id)
            .execute(&self.pool)
            .await?;
        Ok(())
    }
}
```

### Using Repository in Commands

```rust
#[tauri::command]
async fn get_items(state: State<'_, AppState>) -> Result<Vec<Item>, String> {
    state.repo.find_all().await.map_err(|e| e.to_string())
}

#[tauri::command]
async fn create_item(
    input: CreateItemInput,
    state: State<'_, AppState>
) -> Result<Item, String> {
    state.repo.create(input).await.map_err(|e| e.to_string())
}
```

## Modules Organization

```
src/
├── main.rs           # Entry point
├── commands/         # Tauri commands
│   ├── mod.rs       # Module exports
│   ├── items.rs     # Item commands
│   └── settings.rs  # Settings commands
├── models/          # Data structures
│   ├── mod.rs
│   └── item.rs
├── repositories/    # Database access
│   ├── mod.rs
│   └── item_repo.rs
├── state.rs         # App state
└── error.rs         # Error types
```

### Module Pattern

```rust
// src/commands/mod.rs
mod items;
mod settings;

pub use items::*;
pub use settings::*;

// src/main.rs
mod commands;
mod state;

fn main() {
    tauri::Builder::default()
        .manage(state::AppState::new())
        .invoke_handler(tauri::generate_handler![
            commands::get_items,
            commands::create_item,
            commands::get_settings,
        ])
        .run(tauri::generate_context!())
        .unwrap();
}
```

## Common Crates

```toml
[dependencies]
tauri = { version = "2.0", features = ["..."] }
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
sqlx = { version = "0.7", features = ["runtime-tokio", "sqlite"] }
thiserror = "1"
chrono = { version = "0.4", features = ["serde"] }
uuid = { version = "1", features = ["v4", "serde"] }
log = "0.4"
env_logger = "0.10"
```

## Logging

```rust
use log::{debug, error, info, warn};

#[tauri::command]
async fn process_data(data: String) -> Result<String, String> {
    info!("Processing data: {} bytes", data.len());

    match do_processing(&data) {
        Ok(result) => {
            debug!("Processing successful");
            Ok(result)
        }
        Err(e) => {
            error!("Processing failed: {}", e);
            Err(e.to_string())
        }
    }
}
```
