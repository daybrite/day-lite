<!--
Copyright © The Daybrite Project
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# day-lite

Run JavaScript and TypeScript miniapps inside a Day app.

A host app (a superapp) embeds this crate and installs miniapps from a git repository, a static
web host, or a local directory. A miniapp's script builds its UI from the same pieces and signals a
compiled Day app uses, so each platform draws it with its own widgets. QuickJS runs the scripts,
and oxc strips TypeScript types as each module loads, so `.ts` files run without a build step. The
host chooses which native capabilities to expose, and each call checks the permissions the user
granted at install.

## Embed it

```toml
[dependencies]
day-lite = { git = "https://github.com/daybrite/day-lite.git" }
```

```rust
use day_lite::{Host, LiteApp, Store};

/// Installs a miniapp from `origin` (an https prefix or a local path) and boots it.
fn open(data_dir: std::path::PathBuf, origin: &str, app_id: &str) -> Result<LiteApp, String> {
    let store = Store::at(data_dir);
    // `install` fetches only the manifest. Show `plan.manifest.req_permissions` to the user, then
    // pass the permissions they accept to `confirm`, which downloads the files.
    let plan = store.install(origin).map_err(|e| e.to_string())?;
    let granted: Vec<String> = plan
        .manifest
        .req_permissions
        .iter()
        .map(|p| p.name.clone())
        .collect();
    plan.confirm(&granted).map_err(|e| e.to_string())?;

    let host = Host::builder().store(store).bridge(day_lite::net()).build();
    host.launch(app_id)
}
```

`LiteApp::surface()` returns the miniapp's UI as a piece you place anywhere in your own tree. Keep
the `LiteApp` alive while that UI is on screen; dropping it shuts the script runtime down.

## Write a miniapp

A miniapp is a `manifest.json` plus its scripts:

```json
{
  "app_id": "org.example.notes",
  "name": "Notes",
  "version": { "code": 1, "name": "1.0" },
  "pages": ["home"],
  "day": { "files": ["app.ts"] }
}
```

```ts
const draft = signal("");
App({});
page("home", () =>
  column(
    label("Notes").font("title"),
    text_field(draft).placeholder("Add a note"),
  ).spacing(8).padding(10),
);
```

[docs/lite.md](docs/lite.md) covers the manifest, the piece and signal API, storage, permissions,
the package store, and what a superapp needs to provide.

## Test a miniapp

A miniapp keeps its tests in `tests/*.test.ts`. `run_tests` runs them against Day's headless mock
toolkit, with storage in a temporary sandbox:

```rust
fn check(dir: &std::path::Path) -> Result<(), String> {
    let failed: Vec<String> = day_lite::run_tests(dir)?
        .into_iter()
        .filter(|o| !o.passed)
        .map(|o| format!("{}: {}: {}", o.module, o.name, o.detail))
        .collect();
    if failed.is_empty() {
        Ok(())
    } else {
        Err(failed.join("\n"))
    }
}
```

## Compatibility and development

This crate requires Rust 1.96 or newer, the version its oxc dependencies require, and declares
compatibility with Day 0.4 in [Cargo.toml](Cargo.toml). It is consumed from Git, not crates.io.
Its Day dependencies use `https://github.com/daybrite/day.git` without a branch, tag, or revision.
Use the same source in your app and keep its `Cargo.lock` to record the resolved revisions. Mixing
Day source URLs or refs can introduce duplicate framework crates and incompatible types.

QuickJS and the bundled SQLite are C, so a build needs a C compiler, plus libclang for rquickjs's
bindgen step. For iOS, Android, and HarmonyOS, the `day` CLI supplies the cross-compile environment
those build scripts need ([docs/lite.md](docs/lite.md) §13).

For a local framework checkout, run `day patch --local ../day` from this repository. `cargo test`
runs the unit tests, a miniapp's own tests through `run_tests`, and two miniapps mounted on the mock
toolkit.

The crate moved out of the [day repository](https://github.com/daybrite/day) with its history in
2026-09. The dynamic piece registry it drives stays there, behind day-pieces' `dyn-registry`
feature.

## Part of Day

This crate is one piece of [Day](https://daybrite.dev), a Rust framework for building apps
out of each platform's own widgets — AppKit, UIKit, Android's Material widgets, GTK 4, Qt 6,
XAML, and ArkUI — from one codebase. When you write `button("Save")`, macOS shows an
`NSButton` and Android shows a Material button. The framework also ships the tooling around
the app: the `day` CLI, a VS Code extension, GitHub CI workflows, localization,
accessibility, and dayscript automation.

New to Day? Start at [daybrite.dev](https://daybrite.dev), or browse the
[source repository](https://github.com/daybrite/day).
