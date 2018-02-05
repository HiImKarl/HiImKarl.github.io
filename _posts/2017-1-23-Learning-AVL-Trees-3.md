---
layout: single
title: "AVL Trees -- 3/4: Allowing Upwards Traversal"
---

# Parent Pointer

We can add a pointer to our node's parent in its definition. The root node's parent pointer will point to null. Here's our new node definition:

```cpp
template <typename T>
struct Node {
	typedef T value_type;
	Node(value_type value)
		: value(value), left(nullptr), right(nullptr), parent(nullptr), height(0) 
		{}
	value_type value;
	size_t height;
	Node *left;
	Node *right;
	Node *parent;
};
```

# Insertion
We no longer need to keep track of the path we traversed to get to the place we are inserting the node. We can define a function that grabs the address of the pointer to the node we are examining.

```cpp
template <typename T>
Node<T> *Insert(Node<T> *root, typename Node<T>::value_type val) 
{
	// create a new node to insert into the tree
	Node<T> *new_node = new Node<T>(val);

	// if root is null the new node becomes the root node
	if (!root) return new_node;

	// find out where to put the new node
	// keep track of the route moved into the tree
	Node<T> **indirect = &root;
	while (*indirect) {
		// update the parent of the new_node
		new_node->parent = *indirect;
		if ((*indirect)->value < val) indirect = &(*indirect)->right;
		else indirect = &(*indirect)->left;
	}

	// place the new node into tree
	*indirect = new_node;
	// update the height of the tree, applying rotations if necessary
	while (new_node->parent) {
		util::check_for_rotations(util::pointer_to_node(new_node));
		new_node = new_node->parent;
	}

	// need to the deal with the root node seperately
	indirect = &root;
	util::check_for_rotations(indirect);
	return root;
}
```

Here is the function that grabs the adress of the pointer to a node, provided that the node is not the root. Notice that we have to compare the node to it's parent's children _pointers_, instead of their values, since we can have the case where a node's children both have the exact same value as its parent (because of rotations).

```cpp
// assuming node is not null and it has a parent
template <typename T>
Node<T> **pointer_to_node(Node<T> *node)
{
	Node<T> *parent = node->parent;
	if (parent->left == node) return &(parent->left);
	else return &(parent->right);
} 

```

# Erasing Nodes

As you might imagine, the ```Node<T> Find(typename Node<T>::value_type val)``` remains unchanged. However, we can now pass a pointer to the node we want to erase, allowing us to use ```Node<T> Find(typename Node<T>::value_type val)``` to find a node for the value we want to delete, and verify that it exists, before passing it to erase.

We will define a function called ```void delete_replacement(Node<T> **replacement, Node<T> *dangling_branch)``` to clean up the (replacement) node we are removing. Notice that we have to pass it the dangling branch since it may be the left/right child, depending on which way we walked to find the replacement.

```cpp
template <typename T>
Node<T> *Erase(Node<T> *target) 
{
	Node<T> *walk;
	// find a replacement 
	if (target->left) {
		walk = target->left;
		while (walk->right) {
			walk = walk->right;
		}
		// replace the value and then delete replacement node
		target->value = walk->value;
		delete_replacement(&walk, walk->left);
	} else if (target->right) {
		walk = target->right;
		while (walk->left) 
			walk = walk->left;
		target->value = walk->value;
		delete_replacement(&walk, walk->right);
	} else if (!target->parent) {
		// if the tree is just the single node, return null
		delete target;
		return nullptr;
	} else {
		walk = target;
		delete_replacement(&walk, (Node<T> *)nullptr);
	}

	// update the new heights, applying rotations if necessary
	// traverse until the root node is met
	while (walk->parent) {
		util::check_for_rotations(util::pointer_to_node(walk));
		walk = walk->parent;
	}

	// check if the root node requires a rotation
	Node<T> **root_indirect = &walk;
	util::check_for_rotations(root_indirect);
	return walk;
}
```

This is what our ```delete_replacement``` function looks like:

```cpp
template <typename T>
void delete_replacement(Node<T> **replacement, Node<T> *dangling_branch)
{
	Node<T> **node_ptr = util::pointer_to_node(*replacement);
	Node<T> *parent = (*replacement)->parent;
	(*node_ptr) = dangling_branch;
	if (dangling_branch) dangling_branch->parent = parent;
	delete (*replacement);
	*replacement = parent;
}
```

# Iteration

# Verification

Since the functionality of the tree is the same, we can use the verification framework that we created in [part2](https://www.google.com), along with additional tests for iteration.
