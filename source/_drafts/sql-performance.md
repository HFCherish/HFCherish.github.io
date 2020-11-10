---
title: sql performance
toc: true
date: 2020-07-22 14:37:30
tags:
	- database
---

[sql performance explained](https://use-the-index-luke.com/sql/anatomy)

- [ ] Read source code of `HashMap` in c#
- [ ] Read source code of `LinkedList` in c#
- [ ] Unlike the index, the table data is stored in a heap structure and is not sorted at all?????  Heap is either min-heap or max-heap, all is partially sorted.

# Developers need to know this?

Sql separate what & how.

However, developers needs to know how. Because the access path is what influence the performance most, and developers instead of DBAs know it.

# Structure

**Doubly-linked list: (leaf nodes)**

* manage the logic order between leaf nodes, which is redundancy of unordered table data.
* a leaf node is saved in a database block or page. All blocks are of the same size, typically a few kilobytes.
* a leaf node contains several ordered index entries.
* Each index entry consists of the indexed columns (the key, column 2) and refers to the corresponding table row (via `ROWID` or `RID`)
*  the index order is maintained on two different levels: the index entries within each leaf node, and the leaf nodes among each other using a doubly linked list.

![leaf nodes](https://use-the-index-luke.com/static/fig01_01_index_leaf_nodes.en.MMHwYDFb.png)

**B-Tree (branch nodes)**

* a balanced search tree

> The tree traversal is a very efficient operation—so efficient that I refer to it as the *first power of indexing*.
>
> That is primarily because of the tree balance, which allows accessing all elements with the same number of steps, and secondly because of the logarithmic growth of the tree depth.
>
>  That means that the tree depth grows very slowly compared to the number of leaf nodes. Real world indexes with millions of records have a tree depth of four or five. A tree depth of six is hardly ever seen. 

![b-tree structure](https://use-the-index-luke.com/static/fig01_02_tree_structure.en.BdEzalqw.png)

![tree traversal](https://use-the-index-luke.com/static/fig01_03_tree_traversal.en.niC7Q5jq.png)

# Slow Indexes

> An index lookup requires three steps: **(1) the tree traversal; (2) following the leaf node chain; (3) fetching the table data.** The tree traversal is the only step that has an upper bound for the number of accessed blocks—the index depth. The other two steps might need to access many blocks—they cause a slow index lookup.

The Oracle database has three distinct operations that describe a basic index lookup:

1. INDEX UNIQUE SCAN: performs the tree traversal only. (unique constraint)
2. INDEX RANGE SCAN: erforms the tree traversal *and* follows the leaf node chain to find all matching entries. (non-unique constraint)
3. TABLE ACCESS BY INDEX ROWID: This operation is (often) performed for every matched record from a preceding index scan operation.

# Questions

1. How's the tree traversal going? especially on each node, branch node, leaf node?
2. If we traverse the branch node & the leaf node in the same way, and the branch node & the leaf node contain the same count of entries, then they are the same for step (1) and step (2)???