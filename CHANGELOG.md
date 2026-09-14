<!--
Copyright © The Daybrite Project
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Changelog

## 0.4.3

The first release from its own repository. The crate moved out of `daybrite/day`
(`crates/day-lite`) with its history, and [docs/lite.md](docs/lite.md) came with it. The dynamic
piece registry that miniapps build their UI through stays in day, behind day-pieces'
`dyn-registry` feature.

- Tested against day 0.4.
- Updated: rquickjs 0.13 and the oxc crates 0.150.
- Changed: `rust-version` is 1.96, the version the oxc crates require. The 1.89 the crate
  inherited from day's workspace was never enough to build it.
- Unchanged: the embedding API (`Host`, `Store`, `LiteApp`, the `net` and `prefs` bridges) and the
  headless test runner, `run_tests`.
