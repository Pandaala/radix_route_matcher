# radix_route_matcher

A high-performance route matching library based on Radix Tree (compressed trie) data structure.

[![Crates.io](https://img.shields.io/crates/v/radix_route_matcher.svg)](https://crates.io/crates/radix_route_matcher)
[![Documentation](https://docs.rs/radix_route_matcher/badge.svg)](https://docs.rs/radix_route_matcher)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Features

- **High Performance**: Based on Redis's `rax` implementation in C
- **Multiple Match Modes**: Exact match, longest prefix match, and iterator-based matching
- **Safe Rust API**: High-level safe wrapper around C implementation
- **C ABI Compatible**: Can be used from C/C++ or other FFI-capable languages
- **Unicode Support**: Full UTF-8 support including Chinese characters

## Installation

Add this to your `Cargo.toml`:

```toml
[dependencies]
radix_route_matcher = "0.1"
```

## Usage

### Basic Usage

```rust
use radix_route_matcher::RadixTree;

let mut tree = RadixTree::new().expect("Failed to create tree");

// Insert routes
tree.insert("/api", 1).unwrap();
tree.insert("/api/users", 2).unwrap();
tree.insert("/api/posts", 3).unwrap();

// Create an iterator for prefix matching operations
let iter = tree.create_iter().unwrap();

// Exact match
assert_eq!(tree.find_exact("/api/users"), Some(2));

// Longest prefix match
assert_eq!(tree.longest_prefix(&iter, "/api/users/123"), Some(2));

// Get all matching prefixes
let matches = tree.find_all_prefixes(&iter, "/api/users/123/profile");
assert_eq!(matches, vec![2, 1]); // ["/api/users", "/api"]
```

### Iterator-Style Matching

```rust
use radix_route_matcher::RadixTree;

let mut tree = RadixTree::new().unwrap();
tree.insert("/", 1).unwrap();
tree.insert("/api", 2).unwrap();
tree.insert("/api/v1", 3).unwrap();

let iter = tree.create_iter().unwrap();
let path = "/api/v1/users";
if tree.search(&iter, path) {
    while let Some(idx) = tree.next_prefix(&iter, path) {
        println!("Matched route: {}", idx);
    }
}
```

## Performance

- Insert: O(k) where k is the key length (~447ns per route)
- Query: O(k) where k is the key length (~226ns per query)
- Space: Efficient prefix compression, shared prefixes stored once

## API Reference

### RadixTree

| Method | Description |
|--------|-------------|
| `new()` | Creates a new empty Radix Tree |
| `insert(path, idx)` | Inserts a path with an associated index |
| `find_exact(path)` | Finds the exact match for a path |
| `remove(path)` | Removes a path from the tree |
| `create_iter()` | Creates a new iterator for prefix operations |
| `longest_prefix(iter, path)` | Finds the longest prefix match |
| `search(iter, path)` | Initializes iterator for prefix searching |
| `next_prefix(iter, path)` | Gets the next prefix match |
| `find_all_prefixes(iter, path)` | Returns all matching prefixes |

## C API

This library also exports a C-compatible API for use from other languages:

```c
void* radix_tree_new();
int radix_tree_destroy(void* t);
int radix_tree_insert(void* t, const unsigned char* buf, unsigned long len, int idx);
void* radix_tree_find(void* t, const unsigned char* buf, unsigned long len);
int radix_tree_remove(void* t, const unsigned char* buf, unsigned long len);
void* radix_tree_new_it(void* t);
void* radix_tree_search(void* tree, void* it, const unsigned char* buf, unsigned long len);
int radix_tree_up(void* it, const unsigned char* buf, unsigned long len);
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

This project includes code from the Redis rax implementation by Salvatore Sanfilippo, which is also under a BSD-style license.

## Acknowledgments

- [Redis rax](https://github.com/redis/redis) - The underlying radix tree implementation
