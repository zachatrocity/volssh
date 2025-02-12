# SSH Sync Utility

A modern, developer-friendly utility for bidirectional directory synchronization over SSH with Docker-like volume mapping syntax.

## Overview

volssh is a command-line utility that enables seamless bidirectional synchronization of directories between local and remote machines over SSH. It combines the simplicity of Docker's volume mapping syntax with the power of SSH, offering a modern approach to remote development and file synchronization.

## Key Features

- **Docker-like Volume Mapping**: Simple, familiar syntax
  ```bash
  volssh user@host -v /local/path:/remote/path
  ```

- **Interactive Shell Mode**: Work directly on the remote machine while files sync
  ```bash
  volssh user@host -v /local/src:/app/src -i
  ```

- **Real-time Bidirectional Sync**: Changes are reflected instantly in both directions

- **Intelligent Sync Engine**:
  - Smart batching of file operations
  - Efficient conflict resolution
  - Compression and caching
  - Network-aware optimizations

- **Rich Interactive Features**:
  - Status dashboard (Ctrl+D)
  - Live diff viewer (Ctrl+L)
  - Network monitor (Ctrl+N)
  - Volume manager (Ctrl+V)
  - Quick actions menu (Ctrl+Space)

## Installation

```bash
cargo install volssh  # Coming soon
```

## Basic Usage

1. **Simple Directory Sync**:
   ```bash
   volssh user@host -v /local/project:/remote/project
   ```

2. **Multiple Volume Mapping**:
   ```bash
   volssh user@host -v /src:/app/src -v /config:/etc/app
   ```

3. **Interactive Mode with Sync**:
   ```bash
   volssh user@host -v /local:/remote -i
   ```

4. **With Custom SSH Options**:
   ```bash
   volssh user@host -v /local:/remote -i -- -p 2222 -A
   ```

## Development Status

This project is currently in the planning and design phase. We are focusing on:

1. Core architecture and sync engine design
2. Interactive features and user experience
3. Performance optimizations
4. Testing strategy

## Documentation

- [Architecture](./ARCHITECTURE.md)
- [Features](./FEATURES.md)
- [Comparison with Other Tools](./COMPARISON.md)
- [Interactive Features](./docs/interactive.md)
- [Sync Engine](./docs/sync-engine.md)
- [Testing Strategy](./docs/testing.md)

## Why Another Sync Tool?

While tools like `sshfs` and `rsync` exist, they each have limitations:

- `sshfs` requires FUSE and can be slow for many small files
- `rsync` is not real-time and requires manual syncing
- Neither offers a Docker-like volume mapping experience

SSH Sync combines the best aspects of these tools while adding modern features developers expect:

- No system requirements beyond SSH
- Real-time bidirectional sync
- Intuitive Docker-like syntax
- Rich interactive features
- Development-focused workflow

## Contributing

Project is in planning phase. Documentation and design feedback welcome!
