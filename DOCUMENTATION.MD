## Overview
This library provides a comprehensive C++ implementation of a Binary Search Tree (BST) with integrated probability and statistical operations. It supports both set and multiset operations with logical operations, statistical calculations, and advanced data structure features.

---

## Architecture

### Core Components

1. **Binary Search Tree (BST)** - Template-based BST implementation
2. **Node** - Generic node structure for BST
3. **Multiset** - Set/Multiset wrapper with statistical operations

---

## 1. Binary Search Tree (BST)

### Header: `bst.h`

#### Node Class

```cpp
template<typename DataType> 
class node
```

**Members:**
- `DataType data` - Data stored in node
- `uint32_t counter` - Number of occurrences (for multiset support)
- `node<DataType>* left` - Left child pointer
- `node<DataType>* right` - Right child pointer

**Public Methods:**
- `node(void)` - Default constructor
- `node(const DataType&, const uint32_t counter=1)` - Parameterized constructor

#### BST Class

```cpp
template <typename DataType>
class bst
```

**Private Members:**
- `node<DataType>* root` - Root of the tree
- `uint32_t nodes_count` - Number of unique nodes
- `uint32_t total_count` - Total count of all elements (including duplicates)

**Private Methods (Tree Operations):**
- `remove_tree_tour(node<DataType>*)` - Recursively delete all nodes
- `get_node(const DataType&)` - Search for a node
- `remove_node_tour(const DataType&, node<DataType>*)` - Remove specific node
- `collect_tree_vector_tour()` - Collect nodes in in-order to vector
- `balance_tour()` - Create balanced tree from sorted vector
- `height_tour()` - Calculate tree height recursively
- `copy_tree()` - Deep copy tree structure
- `max()` - Find maximum element
- `min()` - Find minimum element

**Public Methods:**

| Method | Description |
|--------|-------------|
| `bst(void)` | Default constructor - initializes empty tree |
| `bst(const bst<DataType>&)` | Copy constructor |
| `operator=(const bst<DataType>&)` | Assignment operator |
| `~bst(void)` | Destructor - cleans up all nodes |
| `insert(const DataType&, uint32_t counter=1)` | Insert element with count |
| `adjust_count(const DataType&, uint32_t &new_counter)` | Update element count |
| `search(const DataType&)` | Search for element |
| `remove(const DataType&)` | Remove element |
| `remove_tree(void)` | Delete entire tree |
| `get_nodes_count(void)` | Get number of unique nodes |
| `get_total_count(void)` | Get total element count |
| `display()` | In-order traversal display |
| `balance(void)` | Rebalance tree to minimize height |
| `height(void)` | Get tree height |

---

## 2. Multiset Class

### Header: `multiset.h`

```cpp
template<typename DataType>
class multiset
```

**Enumerations:**
```cpp
enum mode { SET=0, MULTISET=1 };
```

**Private Member:**
- `bst<DataType> tree` - Underlying BST structure
- `bool mode` - SET or MULTISET mode

### Logical Operations

#### Set Operations

| Method | Description |
|--------|-------------|
| `intersect(const multiset&)` | Returns common elements (minimum counts) |
| `unite(const multiset&)` | Returns union (maximum counts) |
| `operator-(const multiset&)` | Set difference (A - B) |
| `operator+(const multiset&)` | Symmetric difference addition |

**Implementation Details:**
- **Intersection**: For each element in both sets, takes minimum count
- **Union**: For each element, takes maximum count
- **Difference**: Subtracts counts; includes unique elements from first set
- **Addition**: Union plus double-counted intersection elements

#### Comparison Operations

| Method | Returns |
|--------|---------|
| `operator>=(const multiset&)` | True if superset or equal |
| `operator<=(const multiset&)` | True if subset or equal |
| `operator==(const multiset&)` | True if identical |

### Insertion/Removal Methods

```cpp
bool insert(const DataType& d, uint32_t c=1)
bool remove(const DataType& d)
bool search(const DataType&)
uint32_t count(const DataType&)
bool adjust_count(const DataType&, uint32_t &new_count)
bool clear(void)
bool replace(const DataType& old_data, const DataType& new_data)
```

**Behavior:**
- In **SET mode**: Only one copy of each element allowed
- In **MULTISET mode**: Multiple copies allowed with counter

### Statistical Functions

#### Central Tendency

| Method | Description | Formula |
|--------|-------------|---------|
| `average(void)` | Arithmetic mean | Σ(data × count) / total_count |
| `average(const double& power)` | Powered average | Σ(data^power × count) / total_count |
| `Mode(void)` | Element with highest count | |

#### Dispersion Measures

| Method | Description | Formula |
|--------|-------------|---------|
| `variance(void)` | Variance | E[X²] - (E[X])² |
| `standard_deviation(void)` | Square root of variance | √variance |
| `range(void)` | Max - Min | max_element - min_element |

#### Probability Functions

| Method | Returns |
|--------|---------|
| `pmf(void)` | Probability Mass Function as `vector<double>` |
| `cmf(void)` | Cumulative Mass Function as `vector<double>` |

**PMF Calculation**: Each element's count divided by total_count
**CMF Calculation**: Running cumulative sum of PMF

### Display & Utility Methods

```cpp
void display(void)                           // Output set in format: {(data,count) ...}
bool set_mode(const int &wanted_mode)       // Switch between SET/MULTISET
bool remove_duplicates(void)                // Convert to SET (all counts=1)
multiset normalize(void)                    // Standardize: (x - μ) / σ
```

---

## Implementation Details

### Implementation File: `multiset.cpp`

#### Tree Traversal Tours

**Private Tour Methods** (Internal traversal helpers):

1. **`intersection_tour()`** - In-order traversal finding common elements
2. **`union_tour()`** - In-order traversal for union operation
3. **`unique_elements_tour()`** - Find elements unique to source tree
4. **`difference_tour()`** - Calculate set difference with adjusted counts
5. **`contain_tour()`** - Check if source is subset of other
6. **`equality_tour()`** - Verify element-by-element equality
7. **`avg_tour()`** - Accumulate weighted sum for average
8. **`powered_avg_tour()`** - Accumulate powered sum
9. **`variance_tour()`** - Accumulate for variance calculation
10. **`Mode_tour()`** - Find element with max count
11. **`pmf_tour()`** - Collect counts for PMF
12. **`cmf_tour()`** - Calculate cumulative sums
13. **`addition_tour()`** - Add counts from intersection
14. **`apply_function_tour()`** - Apply custom function to all elements
15. **`merge()`** - Merge two sorted element arrays

#### Helper Functions

```cpp
template<typename DataType>
bool remove_duplicates_function(DataType&d1, uint32_t&c1, 
                                DataType*v1, uint32_t*v2, const uint32_t)
// Sets count to 1 for all elements

template<typename DataType>
bool standardize_element(DataType&d1, uint32_t&c1, 
                         DataType*v1, uint32_t*v2, const uint32_t)
// Standardizes: (element - mean) / std_dev
```

---

## Algorithm Complexity Analysis

### Binary Search Tree Operations

| Operation | Time Complexity | Space |
|-----------|-----------------|-------|
| Insert | O(log n) avg, O(n) worst | O(1) |
| Search | O(log n) avg, O(n) worst | O(1) |
| Delete | O(log n) avg, O(n) worst | O(1) |
| Balance | O(n) | O(n) |
| Height | O(n) | O(n) recursive |

### Multiset Operations

| Operation | Complexity |
|-----------|------------|
| Intersection | O(n₁ log n₂) where n₁,n₂ are set sizes |
| Union | O(n₁ + n₂ + merge) |
| Difference | O(n₁ log n₂) |
| Statistical calculations | O(n) |
| Equality/Containment | O(n) |

---

## Usage Example

```cpp
multiset<int> set1(MULTISET);
multiset<int> set2(MULTISET);

// Insert elements
set1.insert(5, 3);  // Add 5 three times
set1.insert(10, 2);
set2.insert(5, 2);
set2.insert(10, 3);

// Set operations
multiset<int> intersection = set1.intersect(set2);
multiset<int> unionSet = set1.unite(set2);
multiset<int> difference = set1 - set2;

// Statistics
double avg = set1.average();
double var = set1.variance();
double stdev = set1.standard_deviation();
int mode = set1.Mode();

// Probability functions
vector<double> pmf = set1.pmf();
vector<double> cmf = set1.cmf();

// Comparison
if (set1 >= set2) { /* set1 is superset */ }
if (set1 == set2) { /* sets are equal */ }
```

---

## Key Features

✅ **Template-based generics** - Works with any comparable data type
✅ **Dual mode** - Supports both SET and MULTISET operations
✅ **BST-based** - Efficient in-order traversal and balanced operations
✅ **Statistical analysis** - Complete probability and statistics suite
✅ **Memory management** - Proper allocation/deallocation with destructors
✅ **Deep copy semantics** - Copy constructor and assignment operator support
✅ **Flexible operations** - Composable set operations with overloaded operators
