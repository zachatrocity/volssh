# Features

## Core Features

### 1. Volume Mapping

Docker-like syntax for mapping directories:
```bash
# Basic mapping
volssh user@host -v /local/path:/remote/path

# Multiple volumes
volssh user@host -v /src:/app/src -v /config:/etc/config

# With specific options
volssh user@host -v /local:/remote:cached,ignore=node_modules
```

Volume Options:
- `cached`: Prioritize local reads for performance
- `ignore`: Specify patterns to ignore
- `one-way`: Sync in one direction only
- `readonly`: Prevent modifications

### 2. Interactive Shell Mode

Launch an interactive shell while maintaining sync:
```bash
volssh user@host -v /local:/remote -i
```

Features:
- Full terminal emulation
- Real-time sync status display
- Keyboard shortcuts for common operations
- Split view showing sync activity

### 3. Sync Engine

Intelligent file synchronization:
- Real-time bidirectional sync
- Smart batching of operations
- Conflict detection and resolution
- Network-aware transfer optimization
- Checksums for change verification
- Compression for efficient transfers

## Interactive Features

### 1. Status Dashboard (Ctrl+D)
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

### 2. Live Diff Viewer (Ctrl+L)
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

### 3. Network Monitor (Ctrl+N)
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

### 4. Volume Manager (Ctrl+V)
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

### 5. Quick Actions Menu (Ctrl+Space)
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

## Keyboard Shortcuts

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

## Configuration

Configuration can be specified via:
1. Command line flags
2. Environment variables
3. Config file (~/.config/volssh/config.yaml)

Example configuration:
```yaml
defaults:
  sync_interval: 1s
  compression: true
  ignore_patterns:
    - "*.tmp"
    - "node_modules/"
    - ".git/"

hosts:
  dev-server:
    hostname: dev.example.com
    volumes:
      - local: /local/src
        remote: /app/src
        options:
          cached: true
          ignore:
            - "*.log"
      - local: /local/config
        remote: /etc/app
        options:
          readonly: true

performance:
  max_concurrent_transfers: 4
  compression_level: 6
  cache_size: "1GB"
  network_throttle: "10MB/s"
```

## Advanced Features

### 1. Conflict Resolution
- Auto-merge compatible changes
- Interactive resolution for conflicts
- Configurable resolution strategies
- Version history for conflicts

### 2. Performance Optimization
- Smart batching of small files
- Delta transfers for large files
- Compression based on file type
- Connection pooling
- Local caching

### 3. Security
- Respect SSH config
- Support for SSH keys
- Encrypted local cache
- Secure file permissions

### 4. Monitoring
- Transfer statistics
- Performance metrics
- Error logging
- Activity history
