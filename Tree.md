## Tree

### Binary Trees

A binary tree is either empty or it contains a root node and left- and right-subtrees that are also binary trees. 

> 二叉树要么是空的，要么包含一个根节点以及也是二叉树的左子树和右子树。

A binary tree is a nonlinear data structure. 

> 二叉树是非线性结构。

The top node of a tree is called the root. 

> 树的顶部节点被称作根节点。

Any node in a binary tree has at most 2 children. 

> 二叉树任何一个节点最多有两个子节点。

Any node in a binary tree has exactly one parent node (except the root). 

> 二叉树中的任何节点都只有一个父节点（根节点除外）。

### A full binary tree

A full binary tree is a binary tree such that all leaves have the same level, and every non-leaf node has 2 children. 

> 满二叉树是一个二叉树，所有叶子节点都有相同的层级，每一个非叶子节点的的节点都有两个子节点。

### A complete binary tree

A complete binary tree is a binary tree such that every level of the tree has the maximum number of nodes possible except possibly the deepest level, and at the deepest level, the nodes are as far left as possible. 

> 完全二叉树是一个二叉树，树的每一层的节点数都是当前层可能的最大节点数，除了最深一层，并且在最深层，节点尽可能地靠左。

### Binary Tree - Height 

The height of a binary tree T is the number of nodes in the longest path from the root node to a leaf node. 

> 二叉树的树高T是从根节点到叶节点的最长路径中的节点数。

### Binary Tree Traversals

preorder、inorder、postorder

> 先序、中序、后序

Preorder traversal
> 1. Visit the root.
> 2. Perform a preorder traversal of the left subtree.
> 3. Perform a preorder traversal of the right subtree.

Inorder traversal
> 1. Perform an inorder traversal of the left subtree.
> 2. Visit the root.
> 3. Perform an inorder traversal of the right subtree.

Postorder traversal
> 1. Perform a postorder traversal of the left subtree.
> 2. Perform a postorder traversal of the right subtree.
> 3. Visit the root. 

### Implementing a binary tree 

Use an array to store the nodes. mainly useful for complete binary trees.

> 使用数组来存储节点。主要用于完全二叉树。

Use a variant of a linked list where each data element is stored in a node with links to the left and right children of that node.

> 使用链表的变体，其中每个数据元素都存储在一个节点中，并带有指向该节点左右子节点的链接

Instead of a head reference, we will use a root reference to the root node of the tree. 

> 我们将使用对树根节点的根引用，而不是头引用。

### Binary Search Trees

A binary tree T is a binary search tree if
- T is empty, or
- T has two subtrees, T_L and T_R, such that
	- All the values in T_L are less than the value in the root of T,
	- All the values in T_R are greater than the value in the root of T, and
	- T_L and T_R are binary search trees 
	
### Efficiency of insert on a full BST 

A full binary search tree with height H has how many nodes? 

> 高度为H的满二叉树有多少个节点? 

N = 1 + 2 + 4 + ... + 2^(H-1) = 2^H - 1 等比数列

Thus, H = log2(N+1). 

An insert will take O(log N) time on a full BST since we have to examine one node at each level in the worst cast before we find the insert point, and there are H levels.

> 在满二叉搜索树上，插入将花费 O(log N) 时间，因为在找到插入点之前，我们必须在最差强制转换中检查每个级别的一个节点，并且有 H 个级别。

### Efficiency of insert on a BST 

Insert the following numbers in order into a BST: 11 21 39 45 62 83 96 What do you get? 

> 将以下数字按顺序插入 BST：11 21 39 45 62 83 96 你会得到什么？

An insert will take O(N) time in the worst case on an arbitrary BST since there may be up to N levels in a BST with N nodes. 

在任意 BST 的最坏情况下，插入将花费 O(N) 时间，因为在具有 N 个节点的 BST 中可能有多达 N 个级别。

