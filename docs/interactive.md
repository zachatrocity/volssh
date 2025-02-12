# Interactive Features

This document details the interactive features and user interface elements of SSH Sync.

## Status Dashboard (Ctrl+D)

The main dashboard provides a real-time overview of sync status and operations.

```
╭─ Remote Sync Dashboard ──────────────────────────────────────╮
│ Host: dev-server.example.com                                 │
│ Uptime: 2h 15m                                              │
│                                                             │
│ Volumes:                                                    │
│ ├─ /local/src → /remote/app/src                            │
│ │  ├─ Status: Active                                        │
│ │  ├─ Files: 234 (↑12 ↓3)                                  │
│ │  └─ Last Sync: 2s ago                                    │
│ │                                                           │
│ └─ /local/config → /remote/etc/app                         │
│    ├─ Status: Active                                        │
│    ├─ Files: 12 (↑1 ↓0)                                    │
│    └─ Last Sync: 45s ago                                   │
│                                                             │
│ Network:                                                    │
│ ├─ Bandwidth: ↑2.1MB/s ↓1.3MB/s                           │
│ └─ Latency: 45ms                                           │
│                                                             │
│ [q] Close  [r] Force Resync  [p] Pause/Resume              │
╰─────────────────────────────────────────────────────────────╯
```

Implementation:
```rust
pub struct Dashboard {
    volumes: Vec<VolumeStatus>,
    network_stats: NetworkStats,
    uptime: Duration,
}

impl Dashboard {
    pub fn render(&self) -> String {
        let mut output = String::new();
        self.render_header(&mut output);
        self.render_volumes(&mut output);
        self.render_network(&mut output);
        self.render_footer(&mut output);
        output
    }
}
```

## Live Diff Viewer (Ctrl+L)

Interactive diff viewer for examining and resolving changes.

```
╭─ Live Diff: /app/src/main.rs ─────────────────────────────╮
│ @@ -15,7 +15,7 @@                                         │
│  pub struct Config {                                      │
│-     timeout: u64,                                        │
│+     timeout: Duration,                                   │
│      max_retries: u32,                                    │
│  }                                                        │
│                                                           │
│ [j/k] Navigate  [y] Accept  [n] Reject  [q] Close        │
╰───────────────────────────────────────────────────────────╯
```

Features:
- Syntax highlighting
- Side-by-side comparison
- Inline diff markers
- Quick navigation
- Accept/reject changes

```rust
pub struct DiffViewer {
    current_file: PathBuf,
    changes: Vec<Change>,
    cursor: usize,
}

impl DiffViewer {
    pub fn navigate(&mut self, direction: Direction) {
        match direction {
            Direction::Next => self.cursor = min(self.cursor + 1, self.changes.len()),
            Direction::Prev => self.cursor = self.cursor.saturating_sub(1),
        }
    }
}
```

## Network Monitor (Ctrl+N)

Real-time network performance visualization.

```
╭─ Network Monitor ───────────────────────────────────────────╮
│                                                            │
│ Bandwidth Usage                                            │
│ ▁▂▃▅▂▇▂▁▁▃▅▆▇▆▅▃▂▁▁▂▃▄▅▆▇                                │
│                                                            │
│ Active Transfers:                                          │
│ ├─ main.rs ↑ 2.1MB/s [===========▶    ] 75%              │
│ └─ config.json ↓ 1.3MB/s [=============▶] 90%            │
│                                                            │
│ Connection Quality: Excellent                              │
│ Latency: 45ms                                             │
│                                                            │
│ [t] Throttle  [r] Reset Stats  [q] Close                  │
╰────────────────────────────────────────────────────────────╯
```

Features:
- Real-time bandwidth graph
- Transfer progress bars
- Connection quality metrics
- Throttling controls

```rust
pub struct NetworkMonitor {
    history: VecDeque<NetworkSample>,
    active_transfers: Vec<Transfer>,
    quality_metrics: QualityMetrics,
}

impl NetworkMonitor {
    pub fn update(&mut self) {
        self.collect_metrics();
        self.update_history();
        self.calculate_quality();
    }
}
```

## Volume Manager (Ctrl+V)

Interface for managing volume mappings and sync settings.

```
╭─ Volume Manager ───────────────────────────────────────────╮
│ [1] /local/src → /remote/app/src                          │
│     [p] Pause  [r] Resume  [f] Force Sync  [i] Ignore    │
│                                                           │
│ [2] /local/config → /remote/etc/app                       │
│     [p] Pause  [r] Resume  [f] Force Sync  [i] Ignore    │
│                                                           │
│ [a] Add Volume  [d] Remove Volume  [q] Close             │
╰───────────────────────────────────────────────────────────╯
```

Features:
- Add/remove volumes
- Pause/resume sync
- Force sync
- Ignore patterns
- Volume settings

```rust
pub struct VolumeManager {
    volumes: Vec<Volume>,
    selected: Option<usize>,
}

impl VolumeManager {
    pub fn toggle_pause(&mut self, index: usize) {
        if let Some(volume) = self.volumes.get_mut(index) {
            volume.toggle_pause();
        }
    }
}
```

## Quick Actions Menu (Ctrl+Space)

Fast access to common operations.

```
╭─ Quick Actions ─────────────────╮
│ [1] Force Sync All             │
│ [2] Add Ignore Pattern         │
│ [3] Show Logs                  │
│ [4] Network Settings           │
│ [5] Export Config              │
│ [6] Show Pending Changes       │
│                                │
│ Enter number or [q] to cancel  │
╰────────────────────────────────╯
```

Implementation:
```rust
pub struct QuickActions {
    actions: Vec<Action>,
}

impl QuickActions {
    pub async fn execute(&self, index: usize) -> Result<()> {
        if let Some(action) = self.actions.get(index) {
            action.execute().await
        } else {
            Ok(())
        }
    }
}
```

## Keyboard Shortcuts

Global shortcuts available in any context:

| Shortcut    | Action                |
|-------------|----------------------|
| Ctrl+D      | Show Dashboard       |
| Ctrl+L      | Live Diff Viewer     |
| Ctrl+N      | Network Monitor      |
| Ctrl+V      | Volume Manager       |
| Ctrl+Space  | Quick Actions        |
| Ctrl+S      | Toggle Sync          |
| Ctrl+R      | Force Resync         |
| Ctrl+F      | Find Files           |
| Ctrl+H      | Show Help           |

Implementation:
```rust
pub struct KeyboardHandler {
    shortcuts: HashMap<KeyCombo, Action>,
    context: Context,
}

impl KeyboardHandler {
    pub async fn handle_key(&mut self, key: Key) -> Result<()> {
        if let Some(action) = self.shortcuts.get(&key) {
            action.execute(self.context).await
        } else {
            Ok(())
        }
    }
}
```

## Notifications

System for displaying important events and alerts.

```
╭─ Notification ──────────────────────────╮
│ Conflict Detected                       │
│ File: config.json                       │
│ Both local and remote copies modified   │
│                                         │
│ [r] Resolve  [i] Ignore  [d] Dismiss   │
╰─────────────────────────────────────────╯
```

Implementation:
```rust
pub struct NotificationManager {
    notifications: VecDeque<Notification>,
    max_notifications: usize,
}

impl NotificationManager {
    pub fn push(&mut self, notification: Notification) {
        while self.notifications.len() >= self.max_notifications {
            self.notifications.pop_front();
        }
        self.notifications.push_back(notification);
    }
}
```

## Progress Indicators

Various progress indicators for long-running operations:

1. File Transfer:
```
Transferring: large-file.zip
[===========▶    ] 75% (2.1MB/s)
```

2. Initial Sync:
```
Initial Sync Progress:
Files: 234/1000 [=====▶     ] 45%
Size:  1.2GB/4.5GB
```

3. Network Activity:
```
Network Activity:
↑ 2.1MB/s  ↓ 1.3MB/s
```

Implementation:
```rust
pub struct ProgressBar {
    total: u64,
    current: u64,
    width: usize,
    style: ProgressStyle,
}

impl ProgressBar {
    pub fn update(&mut self, progress: u64) {
        self.current = progress;
        self.render();
    }
}
