# Probability and Statistics Library (C++)

A C++ probability & statistics library built around a **template-based Binary Search Tree (BST)** and a **set/multiset** abstraction. It supports:

- BST-backed **set / multiset** storage (with duplicate counting)
- Logical operations (intersection, union, difference, symmetric difference)
- Statistical calculations (mean, variance, standard deviation, range, mode)
- Discrete probability utilities (PMF/CMF)
- Utility helpers like normalization (standardization) and duplicate removal

> Documentation reference: see `DOCUMENTATION.MD`.

---

## Contents

- [Features](#features)
- [Project Structure](#project-structure)
- [Quick Start](#quick-start)
- [Core API](#core-api)
  - [BST (`bst.h` / `bst.tpp`)](#bst-bsth--bsttpp)
  - [Multiset (`multiset.h` / `multiset.cpp`)](#multiset-multiseth--multisetcpp)
- [Complexity](#complexity)
- [Notes / Requirements](#notes--requirements)

---

## Features

### Data structures
- **Generic node** storing:
  - `data`
  - `counter` (occurrence count for multiset behavior)
  - `left` / `right` child pointers
- **Template BST** with:
  - insertion, search, deletion
  - balancing
  - height calculation
  - deep copy semantics (copy constructor + assignment operator)

### Set & multiset operations
- **Intersection**: keeps common elements with **minimum** counts
- **Union**: keeps elements with **maximum** counts
- **Difference** (`A - B`): subtracts counts / removes elements based on membership
- **Addition / symmetric-like operation** (`A + B`): union plus additional handling of intersection counts (see docs)

### Statistics
- average / powered average
- variance
- standard deviation
- range
- mode

### Probability (discrete)
- PMF: probability mass function as `std::vector<double>`
- CMF: cumulative mass function as `std::vector<double>`

### Utilities
- switch between SET and MULTISET modes
- remove duplicates (convert to SET)
- normalize / standardize: `(x - μ) / σ`

---

## Project Structure

- `bst.h` / `bst.tpp` — template-based BST implementation
- `multiset.h` / `multiset.cpp` — set/multiset wrapper + probability/statistics operations
- `DOCUMENTATION.MD` — detailed architecture + method list + example usage
- `README.md` — this file

---

## Quick Start

This project appears to be a **header + source** library (not packaged). The typical workflow is:

1. Add the repository (or the needed files) to your project.
2. Include `multiset.h` (and compile/link `multiset.cpp`) in your build.

### Example usage

```cpp
#include "multiset.h"
#include <vector>

int main() {
    multiset<int> set1(MULTISET);
    multiset<int> set2(MULTISET);

    // Insert elements
    set1.insert(5, 3);   // add 5 three times
    set1.insert(10, 2);
    set2.insert(5, 2);
    set2.insert(10, 3);

    // Set operations
    multiset<int> intersection = set1.intersect(set2);
    multiset<int> unionSet     = set1.unite(set2);
    multiset<int> difference   = set1 - set2;

    // Statistics
    double avg   = set1.average();
    double var   = set1.variance();
    double stdev = set1.standard_deviation();
    int mode     = set1.Mode();

    // Probability functions
    std::vector<double> pmf = set1.pmf();
    std::vector<double> cmf = set1.cmf();

    // Comparison
    if (set1 >= set2) { /* set1 is superset */ }
    if (set1 == set2) { /* sets are equal */ }

    return 0;
}
```

> If you want, I can add a `Makefile` or CMake example once you tell me your preferred build system.

---

## Core API

## BST (`bst.h` / `bst.tpp`)

A template-based BST storing `DataType` values and tracking:

- `nodes_count`: number of unique nodes
- `total_count`: total number of items including duplicates (sum of counters)

Common public operations:

- `insert(const DataType&, uint32_t counter=1)`
- `search(const DataType&)`
- `remove(const DataType&)`
- `balance()`
- `height()`
- `display()`
- `get_nodes_count()`
- `get_total_count()`
- deep copy + cleanup (`copy ctor`, `operator=`, destructor)

---

## Multiset (`multiset.h` / `multiset.cpp`)

### Modes

```cpp
enum mode { SET=0, MULTISET=1 };
```

- **SET mode**: only one copy of each element allowed
- **MULTISET mode**: duplicates allowed via internal counters

### Logical operations

- `intersect(const multiset&)`
- `unite(const multiset&)`
- `operator-(const multiset&)` (difference)
- `operator+(const multiset&)` (addition / symmetric-like behavior)

### Comparison

- `operator>=` (superset or equal)
- `operator<=` (subset or equal)
- `operator==` (equality)

### Insert / remove / modify

- `insert(const DataType& d, uint32_t c=1)`
- `remove(const DataType& d)`
- `search(const DataType&)`
- `count(const DataType&)`
- `adjust_count(const DataType&, uint32_t &new_count)`
- `replace(const DataType& old_data, const DataType& new_data)`
- `clear()`

### Statistics

Central tendency:
- `average()`
- `average(const double& power)`
- `Mode()`

Dispersion:
- `variance()`
- `standard_deviation()`
- `range()`

Probability:
- `pmf()`
- `cmf()`

Utilities:
- `display()` (prints like `{(data,count) ...}`)
- `set_mode(...)`
- `remove_duplicates()`
- `normalize()` (standardize)

---

## Complexity

### BST operations
| Operation | Time Complexity | Space |
|---|---:|---:|
| Insert | O(log n) avg, O(n) worst | O(1) |
| Search | O(log n) avg, O(n) worst | O(1) |
| Delete | O(log n) avg, O(n) worst | O(1) |
| Balance | O(n) | O(n) |
| Height | O(n) | O(n) recursion |

### Multiset operations
| Operation | Complexity |
|---|---:|
| Intersection | O(n₁ log n₂) |
| Difference | O(n₁ log n₂) |
| Union | O(n₁ + n₂ + merge) |
| Statistical calculations | O(n) |
| Equality / containment checks | O(n) |

---

## Notes / Requirements

- Requires a C++ compiler with template support (modern compilers recommended).
- `DataType` should be **comparable** (BST ordering depends on comparison operators).

---
