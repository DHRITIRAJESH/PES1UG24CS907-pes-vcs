# PES-VCS Lab Report

## Name: Dhriti Rajesh

## SRN: PES1UG24CS907

---

# Phase 1: Object Storage

## Screenshot 1A: Object Tests

![1A](https://github.com/user-attachments/assets/8bee160a-9436-4bfe-9332-6ee82f01d235)

### Explanation

This output verifies:

* Blob storage works correctly
* Deduplication ensures identical content is stored only once
* Integrity checks confirm data correctness

---

## Screenshot 1B: Object Store Structure

![1B](https://github.com/user-attachments/assets/a15e7d4d-cc50-440b-8255-d945c4eb446d)

### Explanation

* Objects are stored in `.pes/objects/`
* Files are split into subdirectories using first 2 hash characters
* This prevents large directory overload and improves efficiency

---

# Phase 2: Tree Objects

## Screenshot 2A: Tree Tests

![2A](https://github.com/user-attachments/assets/81ab12c3-42d4-41a2-a07a-cf0b178f17d8)

### Explanation

* Tree serialization and parsing works correctly
* Roundtrip conversion preserves all entries
* Deterministic output ensures consistent hashing

---

## Screenshot 2B: Raw Tree Object

![2B](https://github.com/user-attachments/assets/8359b3d1-2876-47ca-a73c-f39bb41e9ff5)

### Explanation

* Displays binary representation of a tree object
* Contains file modes, names, and object hashes
* Demonstrates internal storage structure similar to Git

---

# Phase 3: Index (Staging Area)

## Screenshot 3A: Init → Add → Status

![3A](https://github.com/user-attachments/assets/4c5f6e63-9252-4a3b-a361-876cbc88ff3d)

### Explanation

* Files successfully added to staging area
* `pes status` correctly shows staged files
* No unstaged or untracked files present

---

## Screenshot 3B: Index File

![3B](https://github.com/user-attachments/assets/1f2076c8-49f5-4e59-81f5-e68223ef0c61)

### Explanation

* `.pes/index` is human-readable
* Format:

  ```
  <mode> <hash> <mtime> <size> <path>
  ```
* Stores metadata for all staged files

---

# Phase 4: Commits and History

## Screenshot 4A: Commit Log

![4A-1](https://github.com/user-attachments/assets/b433b0f2-f99f-4256-81a6-bb1045134b86)
![4A-2](https://github.com/user-attachments/assets/73c9aff6-0222-477e-9108-d87376b65696)

### Explanation

* Displays commit history
* Each commit includes:

  * Hash
  * Author
  * Timestamp
  * Message
* Demonstrates linked structure of commits

---

## Screenshot 4B: Object Growth

![4B](https://github.com/user-attachments/assets/7f717cf6-55e4-4f9a-b31a-257fc3453184)

### Explanation

* Object store grows as commits increase
* New blobs, trees, and commits are added
* Unchanged files reuse existing objects

---

## Screenshot 4C: References

![4C](https://github.com/user-attachments/assets/5b512272-60c7-45db-bb87-eb9155d1d301)

### Explanation

* `HEAD` points to current branch
* Branch file stores latest commit hash
* Demonstrates Git-like reference mechanism

---

# Final Integration Test

![Final-1](https://github.com/user-attachments/assets/8ec09b85-cec6-4e88-b143-cd872fccf663)
![Final-2](https://github.com/user-attachments/assets/a774870e-8553-4f2c-afa4-222b1b4a21df)

### Explanation

* Full workflow is functional:

  * Initialization
  * Adding files
  * Committing changes
  * Viewing history
* All integration tests pass successfully

---

# Analysis Questions

## Q5.1: Checkout Implementation

To implement `pes checkout <branch>`:

* Update `.pes/HEAD` to point to the new branch
* Read commit hash from `.pes/refs/heads/<branch>`
* Load tree corresponding to that commit
* Replace working directory files accordingly

### Complexity

* Must safely overwrite files
* Must handle directory structures
* Risk of losing uncommitted changes

---

## Q5.2: Detecting Dirty Working Directory

Steps:

1. Compare index entries with working directory
2. Check file existence, modification time, and size
3. If mismatch occurs → file is modified

### Conflict Case

If a modified file also differs in target branch → checkout must be blocked

---

## Q5.3: Detached HEAD

Detached HEAD occurs when HEAD points directly to a commit.

### Behavior

* New commits can still be created
* These commits are not tracked by any branch

### Recovery

* Create a new branch from that commit:

  ```
  pes branch new_branch <commit_hash>
  ```

---

## Q6.1: Garbage Collection

### Algorithm

1. Start from all branch heads
2. Traverse commits recursively
3. Mark all reachable objects

### Data Structure

* Use a hash set to track visited objects

### Deletion

* Remove objects not present in reachable set

### Estimate

* ~100,000 commits → traversal of commits, trees, and blobs

---

## Q6.2: GC Race Condition

### Problem

* Garbage collector deletes an object
* Commit process is about to reference it

### Result

* Repository corruption

### Solution

* Use locking mechanisms
* Delay deletion
* Ensure safe mark-and-sweep process

---

# Conclusion

This project demonstrates:

* Content-addressable storage
* Efficient version control using hashing
* Tree-based filesystem representation
* Atomic file operations
* Git-like commit history and references

---
