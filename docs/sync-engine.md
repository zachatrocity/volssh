# Sync Engine Design

This document details the design and implementation of SSH Sync's synchronization engine.

## Core Components

```
┌─────────────────┐     ┌──────────────┐     ┌─────────────────┐
│  File Watcher   │────▶│ Sync Queue   │────▶│  Transfer Mgr   │
└─────────────────┘     └──────────────┘     └─────────────────┘
         │                                            │
         │                      ┌────────────────┐    │
         └──────────────────────│ Cache Manager  │◀───┘
                               └────────────────┘
```

## File Watching

```rust
pub struct FileWatcher {
    path: PathBuf,
    ignore_patterns: Vec<String>,
    event_tx: mpsc::Sender<FileEvent>,
    watcher: notify::RecommendedWatcher,
}

pub enum FileEvent {
    Create(PathBuf),
    Modify(PathBuf),
    Delete(PathBuf),
    Rename(PathBuf, PathBuf),
}

impl FileWatcher {
    async fn handle_event(&mut self, event: notify::Event) {
        match event.kind {
            EventKind::Create(_) => self.handle_create(event.paths),
            EventKind::Modify(_) => self.handle_modify(event.paths),
            EventKind::Delete(_) => self.handle_delete(event.paths),
            EventKind::Rename(from, to) => self.handle_rename(from, to),
        }
    }
}
```

## Sync Queue

```rust
pub struct SyncQueue {
    events: VecDeque<FileEvent>,
    batch_size: usize,
    batch_interval: Duration,
    priority_events: VecDeque<FileEvent>,
}

impl SyncQueue {
    pub fn push(&mut self, event: FileEvent) {
        if self.is_priority(&event) {
            self.priority_events.push_back(event);
        } else {
            self.events.push_back(event);
        }
    }

    pub fn take_batch(&mut self) -> Vec<FileEvent> {
        let mut batch = Vec::new();
        
        // First take priority events
        while let Some(event) = self.priority_events.pop_front() {
            batch.push(event);
        }
        
        // Then take regular events up to batch_size
        while batch.len() < self.batch_size {
            if let Some(event) = self.events.pop_front() {
                batch.push(event);
            } else {
                break;
            }
        }
        
        batch
    }
}
```

## Transfer Manager

```rust
pub struct TransferManager {
    ssh_client: Box<dyn SshClient>,
    compression: CompressionLevel,
    bandwidth_limit: Option<u64>,
    concurrent_transfers: usize,
}

impl TransferManager {
    pub async fn transfer_batch(&mut self, batch: Vec<FileEvent>) -> Result<()> {
        let mut tasks = Vec::new();
        
        for event in batch {
            let task = self.transfer_file(event);
            tasks.push(task);
            
            if tasks.len() >= self.concurrent_transfers {
                futures::future::join_all(tasks).await?;
                tasks.clear();
            }
        }
        
        // Handle remaining tasks
        if !tasks.is_empty() {
            futures::future::join_all(tasks).await?;
        }
        
        Ok(())
    }
}
```

## Cache Manager

```rust
pub struct CacheManager {
    cache_dir: PathBuf,
    max_size: usize,
    entries: HashMap<PathBuf, CacheEntry>,
}

pub struct CacheEntry {
    checksum: String,
    last_modified: SystemTime,
    size: usize,
}

impl CacheManager {
    pub fn should_transfer(&self, path: &Path) -> bool {
        if let Some(entry) = self.entries.get(path) {
            let metadata = fs::metadata(path).ok()?;
            let current_checksum = self.calculate_checksum(path);
            
            entry.checksum != current_checksum ||
            entry.last_modified != metadata.modified().ok()?
        } else {
            true
        }
    }
}
```

## Conflict Resolution

```rust
pub enum ConflictResolution {
    KeepLocal,
    KeepRemote,
    Merge,
    KeepBoth,
    Ask,
}

pub struct ConflictResolver {
    strategy: ConflictResolution,
    merge_tool: Option<String>,
}

impl ConflictResolver {
    pub async fn resolve(&self, conflict: FileConflict) -> Result<Resolution> {
        match self.strategy {
            ConflictResolution::Merge => self.attempt_merge(conflict),
            ConflictResolution::Ask => self.prompt_user(conflict),
            ConflictResolution::KeepLocal => Ok(Resolution::KeepLocal),
            ConflictResolution::KeepRemote => Ok(Resolution::KeepRemote),
            ConflictResolution::KeepBoth => self.keep_both(conflict),
        }
    }
}
```

## Performance Optimizations

### 1. Smart Batching

```rust
impl SyncManager {
    fn optimize_batch(&self, events: Vec<FileEvent>) -> Vec<FileEvent> {
        let mut optimized = Vec::new();
        let mut seen = HashSet::new();
        
        // Process events in reverse to get final state
        for event in events.into_iter().rev() {
            match &event {
                FileEvent::Create(path) | FileEvent::Modify(path) => {
                    if !seen.contains(path) {
                        optimized.push(event);
                        seen.insert(path);
                    }
                }
                FileEvent::Delete(path) => {
                    if !seen.contains(path) {
                        optimized.push(event);
                        seen.insert(path);
                    }
                }
                FileEvent::Rename(from, to) => {
                    if !seen.contains(from) && !seen.contains(to) {
                        optimized.push(event);
                        seen.insert(from);
                        seen.insert(to);
                    }
                }
            }
        }
        
        optimized.reverse();
        optimized
    }
}
```

### 2. Delta Transfers

```rust
impl TransferManager {
    async fn transfer_file_delta(&mut self, path: &Path) -> Result<()> {
        let signature = self.get_remote_signature(path).await?;
        let delta = self.calculate_delta(path, &signature)?;
        
        if delta.size() < self.full_file_size(path) {
            self.transfer_delta(path, delta).await
        } else {
            self.transfer_full_file(path).await
        }
    }
}
```

### 3. Compression

```rust
pub enum CompressionStrategy {
    Always,
    Never,
    Smart {
        min_size: usize,
        max_size: usize,
        mime_types: Vec<String>,
    },
}

impl TransferManager {
    fn should_compress(&self, path: &Path) -> bool {
        match &self.compression_strategy {
            CompressionStrategy::Always => true,
            CompressionStrategy::Never => false,
            CompressionStrategy::Smart { min_size, max_size, mime_types } => {
                let size = fs::metadata(path).map(|m| m.len()).unwrap_or(0);
                let mime = mime_guess::from_path(path).first_or_octet_stream();
                
                size >= *min_size &&
                size <= *max_size &&
                mime_types.iter().any(|m| mime.essence_str().starts_with(m))
            }
        }
    }
}
```

## Error Handling

```rust
pub enum SyncError {
    IoError(std::io::Error),
    SshError(ssh2::Error),
    ConflictError(FileConflict),
    NetworkError(NetworkError),
    CacheError(CacheError),
}

impl SyncManager {
    async fn handle_error(&mut self, error: SyncError) -> Result<()> {
        match error {
            SyncError::IoError(e) => self.handle_io_error(e).await,
            SyncError::SshError(e) => self.handle_ssh_error(e).await,
            SyncError::ConflictError(e) => self.handle_conflict_error(e).await,
            SyncError::NetworkError(e) => self.handle_network_error(e).await,
            SyncError::CacheError(e) => self.handle_cache_error(e).await,
        }
    }
}
```

## Monitoring and Metrics

```rust
pub struct SyncMetrics {
    files_transferred: Counter,
    bytes_transferred: Counter,
    transfer_duration: Histogram,
    errors: Counter,
    conflicts: Counter,
}

impl SyncMetrics {
    pub fn record_transfer(&mut self, size: u64, duration: Duration) {
        self.files_transferred.inc();
        self.bytes_transferred.inc_by(size);
        self.transfer_duration.record(duration);
    }
}
