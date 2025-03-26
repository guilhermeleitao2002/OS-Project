# TecnicoFS - Projects Summary

## Overview
Two projects extending the TecnicoFS filesystem:  
1. **Project 1** adds core features like multi-block files and thread-safe operations.  
2. **Project 2** transforms TecnicoFS into a concurrent client-server system.  

---

## Project 1: Extended Features  
### Key Additions  
1. **Multi-Block Files**  
   - Files use **10 direct blocks** + **1 indirect block** for >10 blocks (1 KB/block).  
2. **External File Export**  
   - `tfs_copy_to_external_fs` copies TecnicoFS files to the host OS.  
3. **Thread-Safety**  
   - Fine-grained locking (`pthread_mutex`/`rwlock`) for concurrent client threads.  
4. **Blocking Destroy**  
   - `tfs_destroy_after_all_closed` blocks until all files are closed (uses `pthread_cond`).  

---

## Project 2: Client-Server Architecture  
### Key Enhancements  
1. **Server Process**  
   - Launched via `tfs_server <pipe>`, handles client requests via **named pipes**.  
2. **Session Management**  
   - Clients connect via `tfs_mount`; server assigns unique `session_id` (max `S` sessions).  
3. **Concurrent Sessions**  
   - **Worker threads** process requests per session using synchronized producer-consumer buffers.  
4. **Enhanced Blocking Destroy**  
   - `tfs_shutdown_after_all_closed` triggers server termination after all clients disconnect.  

### Protocol  
- **Structured Messages**: Fixed-format requests/responses with `OP_CODE` (e.g., `tfs_open`, `tfs_write`).  
- **Single-Threaded Clients**: Sequential interaction with the server.  

---

## Build & Testing  
- **Compilation**: Use `make` with provided `Makefile`.  
- **Tests**:  
  - **Project 1**: Multi-threaded clients, large file tests, external copy validation.  
  - **Project 2**: `client_server_simple_test.c` (single-session) and multi-client concurrency tests.  
