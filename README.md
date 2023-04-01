# Tauri + Rusqlite

A minimal example of how to set up a backend SQLite store with Tauri.

I'm using [Rusqlite](https://docs.rs/rusqlite/latest/rusqlite/) to provide a nicer wrapper for SQLite.

## Setting up the database

In `setup()` inside [main.rs](./src-tauri/src/main.rs), we initialize the database and put the `Connection` inside Tauri state.

`initialize_database()` creates the expected path for our `.sqlite` file, using `app_handle.path_resolver().app_data_dir()`. It uses `PRAGMA user_version` to track the DB version. `0` means newly-created, and `1` means created. Further versions can be added as the schema changes. `upgrade_database_if_needed()` checks `user_version` and upgrades inside a transaction, to maintain database integrity.

[state.rs](./src-tauri/src/state.rs) sets up the Tauri app state to hold the `Connection` object, to be re-used across the app. It provides some trait methods to grab the `Mutex` for the connection, run an operation and release it again.

## Accessing the `Connection` from commmands

Inside Tauri commands we can supply a `app_handle: AppHandle` argument, which will be automatically populated by the Tauri framework. From there we can call our `db()` trait method on `app_handle` to do database operations. Anything returned from the closure will be passed through.

```rust
let my_result = app_handle.db(|db: &Connection| /* Do something with the DB connection and return a value */);
```

I pass the `AppHandle` through to any other modules that need it, so they also have access to the `Connection`. You can also call `app_handle.clone()` in case you run into ownership issues.

## Recommended IDE Setup

- [VS Code](https://code.visualstudio.com/) + [Tauri](https://marketplace.visualstudio.com/items?itemName=tauri-apps.tauri-vscode) + [rust-analyzer](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer)
