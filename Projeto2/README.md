# TecnicoFS - Client-Server Architecture & Concurrent Access

## Overview
This project extends the TecnicoFS filesystem into a client-server model, enabling concurrent access from multiple clients. Key features include a blocking destroy function and a multi-session server architecture. Below are the implementation details.

---

## Features

### 1. Blocking Destruction Function
- **Objective**: Ensure `tfs_destroy_after_all_closed()` waits until all files are closed.
- **Function**: `int tfs_destroy_after_all_closed();`
- **Implementation**:
  - Uses **condition variables** (`pthread_cond`) to block until no files are open, avoiding busy waiting.
  - Thread-safe with **mutexes** to protect shared state (e.g., open file counter).
  - After destruction, subsequent `tfs_open` calls return `-1` until reinitialized via `tfs_init`.
- **Tests**:
  - Provided test: `lib_destroy_after_all_closed_test.c`.
  - Custom multi-threaded tests (e.g., threads opening/closing files while `tfs_destroy_after_all_closed` runs).

---

### 2. Client-Server Architecture
The TecnicoFS server runs as an autonomous process, handling client requests via **named pipes**.

#### Server Setup
- **Launch Command**: `tfs_server <pipe_name>`
  - Creates a named pipe for client connections.
- **Session Management**:
  - Clients initiate sessions via `tfs_mount`, providing their own response pipe.
  - Server assigns a unique `session_id` (0 to `S-1`, where `S` is the max concurrent sessions).
  - Rejects new sessions if `S` is exceeded.

#### Client API
- **Session Control**:
  - `tfs_mount(char const *client_pipe, char const *server_pipe)`: Establishes a session.
  - `tfs_unmount()`: Terminates the active session, closes pipes.
- **File Operations**:
  - `tfs_open`, `tfs_close`, `tfs_write`, `tfs_read`: Mirror server-side functions.
  - `tfs_shutdown_after_all_closed()`: Triggers server destruction and shutdown.

#### Protocol Design
- **Message Structure**:
  - **OP_CODE** identifies the operation (e.g., `1` for `tfs_mount`, `3` for `tfs_open`).
  - Fixed-size fields (e.g., 40-byte pipe names, `int` for `session_id`).
  - Example: `tfs_write` request includes `OP_CODE=5`, `session_id`, file handle, buffer length, and data.

---

### Implementation Stages

#### Stage 2.1: Single-Session Server
- **Simplifications**:
  - Single-threaded server (`S=1`).
  - Processes requests sequentially.
- **Testing**:
  - Use `client_server_simple_test.c` to validate basic functionality.

#### Stage 2.2: Multi-Session Concurrency
- **Enhancements**:
  - **Worker Threads**: `S` pre-created threads, each managing a `session_id`.
  - **Producer-Consumer Buffers**:
    - Receiver thread (main thread) accepts incoming requests and routes them to worker threads.
    - Workers use synchronized buffers (with `pthread_mutex` and `pthread_cond`) to process requests.
  - **Response Handling**: Workers send results directly to clients via session-specific pipes.
- **Concurrency**:
  - Maximizes parallelism by isolating per-session processing.
  - Thread-safe operations via fine-grained locking.

---

## Build and Testing
- **Dependencies**: POSIX-compliant OS, `pthread`, named pipe support (`mkfifo`).
- **Compilation**:
  - Use provided `Makefile` (run `make` to build server and client binaries).
- **Testing**:
  - **Single-Session**: Run `client_server_simple_test.c`.
  - **Multi-Session**: Launch multiple clients concurrently (e.g., using shell scripts).
  - Validate `tfs_shutdown_after_all_closed` terminates the server gracefully.
