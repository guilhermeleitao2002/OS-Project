# TecnicoFS - Extended Features Implementation

## Overview
TecnicoFS is a simplified user-mode filesystem implemented as a library. This project extends its functionality with four major features: support for multi-block files, external file copying, thread-safe concurrency, and a blocking destroy function. Below are the implementation details for each feature.

---

## Features

### 1. Multi-block File Support
- **Objective**: Remove the single-block limitation for files.
- **Implementation**:
  - **Direct Blocks**: Each file's i-node contains **10 direct block pointers** for the first 10 blocks.
  - **Indirect Block**: For files exceeding 10 blocks, the i-node points to an **indirect block** (stored in the data region). This block contains indices to additional data blocks.
  - **Block Size**: Fixed at **1 KB** (`BLOCK_SIZE` macro).
  - **Directory Limitation**: The root directory (`/`) remains restricted to a single block.
  - **Data Structures**:
    - i-node entries include direct block indices and an optional indirect block index.
    - Indirect blocks store `int` indices of data blocks.

---

### 2. Export to External Filesystem
- **Objective**: Copy a file from TecnicoFS to the host OS's filesystem.
- **Function**: `int tfs_copy_to_external_fs(char const *source_path, char const *dest_path);`
- **Behavior**:
  - If `dest_path` does not exist, the file is created.
  - If `dest_path` exists, its content is overwritten.
  - Returns `0` on success, `-1` on error (e.g., source file not found).
- **Implementation**:
  - Reads the entire content of the TecnicoFS file (using `tfs_read`).
  - Uses POSIX functions (`open`, `write`, etc.) to create/write the external file.

---

### 3. Thread-safe Concurrency
- **Objective**: Enable safe concurrent access from multi-threaded clients.
- **Implementation**:
  - **Synchronization**: Fine-grained locking using `pthread_mutex` or `pthread_rwlock`.
  - **Design**:
    - Critical sections (e.g., i-node table, open file table) are protected by locks.
    - Avoid global locks to maximize parallelism (e.g., per-i-node or per-data-structure locks).
  - **Test Programs**:
    - At least **three client programs** demonstrating concurrent operations (e.g., simultaneous reads/writes, file creation).

---

### 4. Blocking Destroy Function
- **Objective**: Ensure `tfs_destroy_after_all_closed()` waits until all files are closed.
- **Function**: `int tfs_destroy_after_all_closed();`
- **Behavior**:
  - Blocks if any files are open; proceeds with destruction once all are closed.
  - Uses **condition variables** (`pthread_cond`) to avoid busy waiting.
- **Implementation**:
  - Maintains a counter for open files.
  - Signals the condition variable when files are closed (`tfs_close`).
  - Thread-safe and deadlock-free.

---

## Build and Testing
- **Dependencies**: POSIX-compliant system, `pthread` library.
- **Compilation**:
  - Use the provided `Makefile` (run `make` to build the library and test programs).
- **Testing**:
  - Execute the three concurrent client programs to validate thread safety.
  - Verify multi-block file support by creating files larger than 10 KB.
  - Test `tfs_copy_to_external_fs` by exporting files and checking their content externally.
