# Probability-and-Statistics-Library

## 1) BST Section (`bst.h`, `bst.tpp`)

### Overview
The BST is implemented as two templates:
- `node<DataType>`: tree node storing `data`, a `counter` (multiplicity), and `left/right` pointers.
- `bst<DataType>`: owns the root pointer and maintains:
  - `nodes_count`: number of **unique** elements (nodes)
  - `total_count`: sum of all node counters (total elements including duplicates)

This BST supports:
- insertion with multiplicity
- search
- removal of a key (removes whole node, not “decrement counter”)
- adjusting counter
- balancing by rebuilding from in-order traversal
- height calculation
- min/max lookup
- display in-order

---

### 1.1 `class node<DataType>` (in `bst.h`)

#### Data members (private)
- `DataType data`  
  Stored value/key.
- `uint32_t counter`  
  Count/multiplicity for that value.
- `node<DataType>* left`, `node<DataType>* right`  
  Child pointers.

#### Friends
- `friend class bst<T>`
- `friend class multiset<T>`
This allows `bst` and `multiset` to access node internals directly.

#### `node()`
**Signature:** `node(void);`  
**Behavior (in `bst.tpp`):**
- sets `counter = 0`
- sets `left = NULL`, `right = NULL`
**Notes:** does not initialize `data`.

#### `node(const DataType& value, const uint32_t counter = 1)`
**Signature:** `node(const DataType&, const uint32_t counter = 1);`  
**Behavior:**
- assigns `data=value`, `counter=_c`
- sets children to `NULL`

#### `bool update_parent_link(node<DataType>* dest)`
**Declared only** in `bst.h`; **no definition** found in `bst.tpp` or elsewhere.  
**Intended meaning (inferred):** likely meant to help rewire pointers during deletion/rotation.  
**Status:** not usable unless implemented.

---

### 1.2 `class bst<DataType>` Public API (in `bst.h` + `bst.tpp`)

#### `bst()`
**Signature:** `bst(void);`  
**Behavior:**
- `root=NULL`, `nodes_count=0`, `total_count=0`

#### Copy constructor
**Signature:** `bst(const bst<DataType>&);`  
**Behavior:**
- calls `remove_tree()` (even though a new object should already be empty)
- deep copies using `copy_tree(src.root)`
- copies counts (`nodes_count`, `total_count`)

#### Copy assignment
**Signature:** `bst<DataType>& operator=(const bst<DataType>&);`  
**Behavior:**
- clears current tree via `remove_tree()`
- deep copies `src`
- returns `*this`

#### Destructor
**Signature:** `~bst(void);`  
**Behavior:**
- calls `remove_tree_tour(root)` then resets all fields to 0/null

---

#### `bool insert(const DataType& key, const uint32_t counter = 1)`
**Behavior:**
- If `counter==0`: returns `false`.
- If tree empty: creates root node, sets:
  - `nodes_count=1`
  - `total_count += counter`
- Else:
  - walks BST by comparisons (`<`, `==`)
  - If key exists: increments that node’s counter by `counter`, increments `total_count`
  - If new key: inserts new node, increments `nodes_count` and `total_count` accordingly
**Complexity:** average O(log n), worst O(n)

---

#### `bool adjust_count(const DataType& key, const uint32_t& new_counter)`
**Behavior:**
- Finds node via `get_node()`.
- If not found: returns `false`.
- If `new_counter != 0`:
  - updates `total_count` by subtracting old counter and adding new
  - sets node counter to `new_counter`
- If `new_counter == 0`:
  - deletes the node entirely via `root = remove_node_tour(key, root)`
**Important:** setting to 0 removes the *node*, not just sets count to 0.
**Complexity:** O(height)

---

#### `bool search(const DataType& key) const`
**Behavior:** returns `get_node(key) != NULL`.

---

#### `bool remove(const DataType& key)`
**Behavior:**
- If key exists: `root = remove_node_tour(key, root)` and returns `true`
- Else returns `false`
**Important:** removes the whole node regardless of its counter (does not decrement).
**Complexity:** O(height)

---

#### `bool remove_tree()`
**Behavior:**
- if `root` exists: calls `remove_tree_tour(root)` and returns `true`
- else returns `false`

---

#### `uint32_t get_nodes_count() const`
Returns number of **unique** nodes.

#### `uint32_t get_total_count() const`
Returns total multiplicity across all nodes.

---

#### `void display(const node<DataType>* ptr = NULL) const`
In-order traversal printing to `std::cout`:
- prints as `(data,counter) ` for each node
- if called with `ptr==NULL`, it uses `root` (if non-null)
**Side effects:** writes to stdout.

---

#### `bool balance()`
**Behavior:**
- Collects an in-order vector of node pointers (`collect_tree_vector_tour`).
- Builds a new balanced tree from that sorted vector (`balance_tour`) by creating **new nodes**.
- Deletes old tree (`remove_tree_tour(root)`)
- Sets `root=temp_root`
**Counts:**
- `nodes_count` remains unchanged.
- `total_count` is updated during deletion and not explicitly recomputed afterward in `bst::balance()`. However, `remove_tree_tour` subtracts old counters and `balance_tour` creates new nodes but does **not** add into `total_count`. This means `total_count` can become incorrect after balancing.
  - (In contrast, `multiset::form_tree` does recompute total_count manually.)
**Complexity:** O(n)

---

#### `int64_t height() const`
**Behavior:**
- returns `height_tour(root)`
- base case returns `-1` for null
- returns max(left,right)+1
**Note:** Implementation uses `return (left_height>right_height)?left_height:right_height +1;` which due to operator precedence is effectively:
- if left > right: returns `left_height`
- else: returns `right_height + 1`
That is **not correct** for height and will undercount in many cases.
**Expected fix** would be `(left_height > right_height ? left_height : right_height) + 1`.

---

### 1.3 BST Internal / Helper Functions (private)

These are used to implement the public API.

#### `node<DataType>* get_node(const DataType&) const`
Iterative search; returns pointer or NULL.

#### `bool remove_tree_tour(node<DataType>* ptr)`
Post-order deletes nodes:
- recursively deletes left and right
- updates `nodes_count--`, `total_count -= ptr->counter`
- deletes ptr
**Side effects:** modifies the whole tree bookkeeping.

#### `node<DataType>* remove_node_tour(const DataType&, node<DataType>* ptr)`
Recursive delete by key:
- navigates left/right by comparisons
- on match:
  - no children: delete node, return NULL
  - one child: delete node, return child
  - two children:
    - finds max in left subtree (in-order predecessor)
    - copies its `data` and `counter` into current node
    - recursively deletes predecessor node
**Bookkeeping:** decrements `nodes_count` and subtracts the deleted node’s counter each time it deletes a node.  
**Concern:** when replacing with predecessor, it copies predecessor’s counter into current node, then deletes predecessor and subtracts its counter from `total_count`. This can cause `total_count` to become inconsistent (because the counter “moved” but `total_count` still subtracts it once).

#### `void collect_tree_vector_tour(const node<DataType>*, node<DataType>** tree_vector, uint32_t& pos) const`
In-order traversal placing node pointers into `tree_vector`.

#### `node<DataType>* balance_tour(const node<DataType>** tree_vector, const uint32_t& start, const uint32_t& end) const`
Recursive rebuild:
- chooses middle element, creates a **new node** from its `(data,counter)`
- recurses to build left/right halves

#### `node<DataType>* copy_tree(const node<DataType>* ptr)`
Deep copy helper: recreates each node recursively.

#### `node<DataType>* max() const`, `node<DataType>* min() const`
Walks right-most / left-most chain.

---

## 2) Set / Multiset Operations (`multiset.h`, `multiset.cpp`)

### Overview
`multiset<DataType>` wraps a `bst<DataType>` and supports both:
- `MULTISET` mode (counts can be > 1)
- `SET` mode (intended counts forced to 1)

It provides:
- insert/remove/search/count
- intersection, union, difference
- multiset sum (`operator+`) which adds counts
- subset/superset and equality comparisons

### Mode enum
```cpp
enum mode { SET=0, MULTISET=1 };
```

---

### 2.1 Public API: constructors / mode / basic CRUD

#### `multiset()`
**Behavior:** sets `mode = MULTISET`.

#### `multiset(const int& wanted_mode)`
**Behavior:**
- if `wanted_mode` is `SET` or `MULTISET`, uses it
- else defaults to `MULTISET`

#### `~multiset()`
**Behavior:** clears underlying tree using `tree.remove_tree()`.

#### `bool set_mode(const int& wanted_mode)`
**Behavior:**
- if `wanted_mode` is valid and different from current `mode`, sets it and returns `true`
- otherwise returns `false`.

#### `bool insert(const DataType& d, const uint32_t c = 1)`
**Behavior:**
- if `c==0`: returns false
- if `MULTISET`: inserts with multiplicity `c`
- if `SET`: inserts only if not already present (inserts with counter 1)

#### `bool remove(const DataType& d)`
Calls `tree.remove(d)`; removes node entirely.

#### `bool clear()`
Calls `tree.remove_tree()` and returns true.

#### `bool search(const DataType& d) const`
Calls `tree.search(d)`.

#### `bool adjust_count(const DataType& d, const uint32_t& c)`
**Behavior:**
- if `c != 0` and mode is MULTISET: forwards to `tree.adjust_count(d,c)`
- if `c != 0` and mode is SET: returns false (no counter changes allowed)
- if `c == 0`: removes the element via `tree.remove(d)`

#### `uint32_t count(const DataType& d) const`
**Intended:** return multiplicity/counter for `d`.  
**Actual code bug:** `return (ptr)?ptr->count:0;` but node field is named `counter`, not `count`. This will not compile as-is.  
**Correct behavior should be:** `ptr ? ptr->counter : 0`.

---

### 2.2 Core set/multiset operations (public)

#### `multiset intersect(const multiset&) const`
**Meaning:** multiset intersection; count for each shared element = `min(countA, countB)`.  
**Implementation:**
- builds a vector of matching nodes by traversing `this->tree` and querying the other tree for each key
- for each match, stores pointer to the node with smaller counter
- builds balanced result tree from vector using `form_tree`
**Complexity:** O(n log m) average due to `get_node` lookups.

#### `multiset unite(const multiset&) const`
**Meaning:** multiset union; count = `max(countA, countB)` for shared elements, plus unique elements from both.  
**Implementation:**
- `union_tour` collects all nodes from *this* plus max-count nodes where overlaps exist
- `unique_elements_tour` collects nodes unique to the other set
- merges the two sorted vectors with `merge`
- builds balanced tree with `form_tree`

#### `multiset operator-(const multiset&) const`
**Meaning (as implemented):**
- For each element in A that is also in B:
  - if `countA > countB`, include it with `countA-countB`
- Also includes elements unique to A unchanged
So this is multiset difference A\B with subtraction of counts.
**Implementation details:**
- creates new nodes for elements where counts are reduced (these are later deleted manually)
- merges reduced-count nodes with elements unique to A

#### `multiset operator+(const multiset&) const`
**Meaning:** multiset “sum”:
- start with union (max counts)
- then add intersection counts again onto those elements (effectively makes shared element counts `countA + countB`)
**Implementation:** `ret_set = unite(src)` then `intersection = intersect(src)` then `addition_tour` adds counters from intersection into ret_set.

#### `bool operator>=(const multiset&) const`
**Meaning:** `this` contains `other` (superset in multiset sense: for every element x, count_this(x) >= count_other(x)).  
**Implementation:** traverses `other` and checks existence + sufficient counter in this tree.

#### `bool operator<=(const multiset&) const`
Implemented as `return mset >= (*this);`

#### `bool operator==(const multiset&) const`
Checks exact equality of counts for each element (and same keys).

---

## 3) Probability & Statistical Calculations (in `multiset`)

These operate on the multiset’s values and their multiplicities.

### 3.1 `DataType average() const`
**Meaning:** weighted mean  
\[
\bar{x} = \frac{\sum x_i \cdot c_i}{\sum c_i}
\]
**Implementation:** sums `data * counter` via `avg_tour`, divides by `tree.total_count`.

### 3.2 `DataType average(const double& power) const`
**Meaning (as implemented):** computes
\[
\frac{\sum (x_i^{power}\cdot c_i)}{\sum c_i}
\]
This is a “powered moment” style average, not a generalized mean.

### 3.3 `DataType variance() const`
**Meaning (as implemented):**
\[
E[X^2] - (E[X])^2
\]
(using counts as weights)
**Implementation:** `variance_tour` accumulates both sums, then normalizes by total_count.

### 3.4 `DataType standard_deviation() const`
Returns `sqrt(variance())`.

### 3.5 `DataType median() const`
**Important:** this does **NOT** compute the statistical median by value/order.  
**Actual behavior:** chooses the node with the **largest counter** (i.e., the **mode**, most frequent value).  
So `median()` is misnamed; it returns the **mode by frequency**.

### 3.6 `DataType range() const`
Returns `max_value - min_value` using BST `max()` and `min()`.

### 3.7 `std::vector<double> pmf() const`
Returns a vector (size = number of unique nodes) containing each node’s probability mass:
- traverses tree in sorted order, collects counters
- divides each by `total_count`
**Output ordering:** in-order sorted by `data`.

### 3.8 `std::vector<double> cmf() const`
Computes cumulative distribution over sorted unique values:
- accumulates running sum of counters in-order
- divides each by `total_count`

### 3.9 `multiset normalize()`
Returns a new multiset where each value is standardized:
\[
z = \frac{x - \text{avg}}{\text{stddev}}
\]
**Implementation approach:**
- computes `avg` and `standard_deviation`
- traverses and updates each `data` in a copied tree via a function pointer (`standardize_element`)
**Note:** normalization changes values but keeps the original counters.

---

## 4) Helper / Internal Functions (Multiset & Free Helper Templates)

### 4.1 Free helper template functions (in `multiset.h`)

#### `remove_duplicates_function`
```cpp
template<typename DataType>
bool remove_duplicates_function(DataType& d1, uint32_t& c1, DataType* v1, uint32_t* v2, const uint32_t);
```
Assigns the couner in each node in the BST to be 1

#### `standardize_element`
```cpp
template<typename DataType>
bool standardize_element(DataType& d1, uint32_t& c1, DataType* v1, uint32_t* v2, const uint32_t);
```
**Actual behavior:**
- expects `v1[0] = mean`, `v1[1] = stddev`
- transforms `d1 = (d1 - v1[0]) / v1[1]`

These are intended to be called via `apply_function_tour`.

---

### 4.2 Multiset private traversal helpers

#### `intersection_tour(...)`
Traverses one tree and finds matches in the other tree, building a vector of pointers to min-count nodes.

#### `union_tour(...)`
Collects max-count nodes for shared keys and collects all unique nodes from the source traversal.

#### `unique_elements_tour(...)`
Collects nodes that are not present in the other tree.

#### `difference_tour(...)`
Collects “reduced count” nodes for A\B where counts reduce (allocates new nodes).

#### `merge(dest, vec1, vec2, v1_c, v2_c)`
Merges two sorted vectors of node pointers into `dest` by comparing `data`.

#### `form_tree(vec, size)`
Creates a balanced `multiset` from a sorted node-pointer vector:
- uses `bst::balance_tour` to build a balanced tree
- sets `nodes_count=size`
- recomputes `total_count` by summing `vec[i]->counter`

#### `contain_tour(...)`, `equality_tour(...)`
Traverse one tree and query the other to validate multiset containment/equality.

#### `avg_tour(...)`, `powered_avg_tour(...)`, `variance_tour(...)`
Accumulate weighted sums.

#### `median_tour(...)`
Selects pointer to node with maximum counter (mode), despite name “median”.

#### `pmf_tour(...)`, `cmf_tour(...)`
Fill vectors with counters (and cumulative counters).

#### `addition_tour(...)`
For each key in result tree, if key exists in intersection tree, adds its counter.

#### `apply_function_tour(...)`
Generic in-order traversal calling a function pointer:
```cpp
bool(*f_ptr)(DataType&, uint32_t&, DataType*, uint32_t*, const uint32_t)
```
Used by:
- `remove_duplicates()` → sets all counters to 1 and forces `total_count = nodes_count`
- `normalize()` → standardizes each data value

---
