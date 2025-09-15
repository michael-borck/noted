Of course. Here is a comprehensive specification and architecture document based on our conversation, designed to guide the development of the `noted` toolkit.

---

# **`noted` — System Architecture & Specification**

### **Tagline: Plain Text Productivity Intelligence**

## 1. Overview & Philosophy

`noted` is **not another note-taking app**. It is a lightweight, Unix-style intelligence layer designed to enhance existing plain-text productivity and development workflows.

Instead of replacing proven tools like `zk`, `task`, or `git`, `noted` fills the gaps between them. It provides smart linking, similarity-based discovery, and cross-tool health checks, all driven by a pipe-friendly Command-Line Interface (CLI).

The system is built on a "documentation-first" principle, bundling guides and best practices to help users master a powerful plain-text workflow. While designed for notes, its intelligence features extend seamlessly to any text-based repository, including source code, linking documentation, tasks, and code files.

### Core Principles
* [cite_start]**Integrate, Don’t Replace:** `noted` complements existing tools rather than creating a new monolith[cite: 503].
* [cite_start]**Unix Philosophy:** It is composed of small, single-purpose scripts that are pipe-friendly and work with text streams[cite: 280, 504].
* [cite_start]**Documentation First:** The toolkit ships with comprehensive, offline-accessible guides for workflows and best practices[cite: 505].
* [cite_start]**Minimal Maintenance:** The system consists of a few auditable scripts and a core binary, avoiding complex dependencies[cite: 506].
* [cite_start]**Progressive Adoption:** Users can start by using only the documentation and adopt the CLI tools as needed[cite: 507].

***

## 2. Target Platform & MVP Scope

* **Target Platform:** The initial build will target Unix-like systems, specifically **Ubuntu Linux** and **macOS**.
* **MVP Scope:** A frictionless Minimum Viable Product that delivers the core "gap-filling" functionality. This includes:
    * The core `noted` CLI with the `related`, `resolve`, `doctor`, and `docs` commands.
    * The Rust binary for similarity analysis.
    * Basic configuration file support.
    * Bundled documentation for core workflows.

***

## 3. Architecture

`noted`'s architecture is modular, consisting of lightweight wrappers coordinating with a powerful core binary and external tools.

* [cite_start]**Shell Script Wrappers:** The main `noted` command is a Bash script that acts as a dispatcher, providing a consistent user interface for all sub-commands[cite: 76, 188].
* **Core Rust Binary:** A compiled, high-performance binary handles computationally intensive tasks like similarity scoring (TF-IDF/cosine similarity) and indexing. [cite_start]This avoids the slowness of shell scripts for these operations[cite: 78, 292].
* **External Tool Integration:** `noted` seamlessly wraps and enhances existing command-line tools:
    * [cite_start]**`zk`:** Used optionally for manual backlink discovery and graph generation[cite: 255].
    * [cite_start]**`task` / `todo.sh`:** The standard for `todo.txt` task management, enhanced by `noted`'s smart linking[cite: 138, 288].
    * [cite_start]**Deduplication Utilities:** Wraps tools like `fdupes` or `jdupes` for an optional `noted import` command[cite: 248, 295].
    * [cite_start]**`git`:** The recommended backend for versioning, synchronization, and conflict resolution[cite: 177, 297].
* **Metadata Layer:** A hidden `.noteindex/` directory within the notes folder stores all metadata as human-readable, pipe-delimited text files. [cite_start]This avoids database lock-in and keeps the system transparent[cite: 162, 170, 293].

### Gap-Filling Value Proposition
`noted` is justified by the functionality it provides that is missing when using standard tools alone.

| Gap | Existing Tools Lack | `noted` Provides |
| :--- | :--- | :--- |
| **Similarity Search** | `zk` and `grep` find exact matches but not conceptual similarity. | `noted related` finds similar notes and code files based on content. |
| **Smart Linking** | `todo.txt` tasks can't automatically resolve keyword links to files. | `noted resolve` maps `note:keyword` to a specific file path. |
| **Cross-Tool Health** | No unified way to check for broken links, duplicates, or format errors across notes and tasks. | `noted doctor` runs a comprehensive health check on the entire system. |
| **Unified Workflows** | Documentation for combining these tools is scattered. | `noted docs` provides bundled, offline guides. |
| **Code Intelligence** | No easy way to link a task in `todo.txt` to a specific function in a source file. | `noted resolve` and `noted tasks-for` bridge tasks, docs, and code. |

***

## 4. Directory & Data Structure

### Managed File Store
* **Notes Directory (`NOTES_DIR`):** Defaults to `~/notes/` but is configurable. This directory contains all notes and the metadata index.
* **File Structure:** The default is a **flat** structure. [cite_start]The year/month folder structure is an optional configuration (`FOLDER_STRUCTURE="year/month"`)[cite: 152, 309].
* **Note Naming:** Note files use a `YYYYMMDD-title-slug.md` format. [cite_start]This ensures chronological sort order and uniqueness while remaining human-readable[cite: 144, 285]. Duplicate titles on the same day are automatically suffixed (e.g., `-2`).

### Task File
* [cite_start]A standard `~/todo.txt` file is used, ensuring full compatibility with the existing `todo.txt` ecosystem[cite: 38, 114].

### Metadata Index (`.noteindex/`)
This hidden directory lives inside `NOTES_DIR` and contains versioned, pipe-delimited text files.

* [cite_start]`links.db`: Caches the resolution of `note:keyword` links from tasks to specific note files to avoid repeated prompts[cite: 162, 289].
* [cite_start]`related.db`: Stores pre-computed similarity scores between notes and code files[cite: 162, 290].
* [cite_start]`dedup.db`: Contains file content hashes to speed up deduplication during imports[cite: 162, 293].

**Atomic Updates:** To prevent corruption, all writes to these files are atomic. [cite_start]Data is written to a temporary file (`.tmp`) which is then renamed to replace the original[cite: 268, 301].

***

## 5. Configuration

`noted` is managed via a simple configuration file.

* **Location:** `~/.config/noted/config`
* **Format:** A simple shell script sourced by the `noted` commands.
* **Key Variables:**
    * `NOTES_DIR="$HOME/notes"`
    * `EDITOR="vim"`
    * `DEDUP_TOOL="jdupes"` (can be `fdupes`, etc.)
    * `FOLDER_STRUCTURE="flat"` (can be `year/month`)

***

## 6. Core Commands (`noted` CLI)

All commands are designed to be pipe-friendly, reading from `stdin` and writing to `stdout` where applicable.

* `noted related [FILE|SLUG]`
    Finds and ranks notes and source code files that are contextually similar to the specified item. Powered by the Rust binary.

* `noted resolve note:KEYWORD`
    Resolves a keyword from a `todo.txt` task to a unique file path. Prompts the user for clarification if the keyword is ambiguous.

* `noted tasks-for [FILE|SLUG]`
    Displays all tasks from `todo.txt` that are linked to the specified note or source code file.

* `noted doctor`
    Performs a comprehensive health check on the system, verifying:
    * Broken `note:keyword` links.
    * Duplicate files.
    * Metadata index integrity.
    * Availability of external tool dependencies.

* `noted docs [GUIDE]`
    An offline documentation browser. Running `noted docs` lists available guides, and `noted docs workflow.md` displays the guide in the user's pager (`less`).

* `noted import /path/to/files`
    A future command to safely import notes from other systems, using the configured `DEDUP_TOOL` to prevent duplicate content.

***

## 7. Future Enhancements

The design allows for a clear, phased evolution beyond the MVP.

* **Phase 1: Core Stability:** Comprehensive test suite, shell completions, and man pages.
* **Phase 2: Intelligence:** Move from TF-IDF to semantic similarity using embeddings for more accurate `related` results.
* **Phase 3: Integration:** Editor plugins (Vim, VSCode) that use the `noted` CLI as a backend.
* [cite_start]**Phase 4: Scale:** An optional migration path from text-based `.db` files to a single SQLite file for users with tens of thousands of notes[cite: 172, 216]. [cite_start]Optional per-file encryption via `gpg` or `age` wrappers[cite: 270, 298].




