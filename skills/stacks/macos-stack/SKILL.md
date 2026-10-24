---
name: macos-stack
description: Use when creating a native macOS app, or when choosing its UI, persistence, or architecture. Swift, SwiftUI, Observation, SwiftData or GRDB. Leave an existing project stack alone. This is not an iOS stack.
---

# macOS stack

Use this for a new native macOS app, or when choosing its UI, persistence, or architecture. An existing project keeps its stack unless the user asks to migrate. A webpage or HTTP API uses the `web-stack` skill. A game uses the `game-stack` skill. This is not an iOS stack.

Prefer a fully native app unless there is a strong reason to share one UI codebase with other platforms. Prefer native macOS APIs over a cross-platform abstraction. Use Xcode and the latest stable Swift toolchain.

## Stack

- Swift, SwiftUI, and Observation (`@Observable`)
- Structured concurrency: async/await, Task, and actors where they fit
- Swift Package Manager
- URLSession, Codable, and Keychain for credentials
- AppKit where SwiftUI does not expose the capability
- The native framework that owns the system feature, such as Foundation, SwiftData, Core Graphics, AVFoundation, Metal, Vision, Network, or Accessibility

## Persistence

SwiftData for straightforward local persistence. SQLite with GRDB when the app has substantial local data, complex queries, migrations, performance requirements, or needs explicit database control.

## Architecture

Feature-oriented. State stays with the feature that owns it. Start with SwiftUI and Observation. Add a service or repository when it is a real boundary. Inject dependencies without a DI framework.

TCA only when shared state, navigation, effects, or testing are complex enough to need it. Combine only when an Apple API requires it, or a reactive stream is actually the right model. Skip TCA, Combine, and Clean Architecture layers otherwise.
