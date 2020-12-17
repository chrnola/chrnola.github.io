---
layout: post
title: December 2020 LeetCode Challenge Day 16 - Validate Binary Search Tree
---

Well, [today's challenge](https://leetcode.com/explore/item/3568) is:

> Given the `root` of a binary tree, _determine if it is a valid binary search tree (BST)_.
>
> A **valid BST** is defined as follows:
> * The left subtree of a node contains only nodes with keys **less than** the node's key.
> * The right subtree of a node contains only nodes with keys **greater than** the node's key.
> * Both the left and right subtrees must also be binary search trees.

# First Attempt

At first I thought to try a recursive breadth-first search.
I started off by passing the current node's value down on the recursive call and validating that the left and right children were less and greater than it, respectively.

```csharp
public class Solution
{
    private bool isLess(TreeNode node, int parentVal)
    {
        if (node == null)
            return true;

        return
            node.val < parentVal
            && this.isLess(node.left, node.val)
            && this.isGreater(node.right, node.val);
    }

    private bool isGreater(TreeNode node, int parentVal)
    {
        if (node == null)
            return true;

        return
            node.val > parentVal
            && this.isLess(node.left, node.val)
            && this.isGreater(node.right, node.val);
    }

    public bool IsValidBST(TreeNode root)
    {
        if (root.left == null && root.right == null)
            return true;

        return
            this.isLess(root.left, root.val)
            && this.isGreater(root.right, root.val);
    }
}
```

However, this doesn't work since I'm only comparing each node against its immediate parent.
The requirement, however, dictates that _every_ child node in a subtree must have a value that is greater/less than the root node's value.
Therefore by evaluating nodes without the context of _all_ of their parents, I can't possibly satisfy that requirement.

This example demonstrates why my first approach doesn't work:

```
  5
 / \
4   6
   / \
  3   7
```

My first solution would have deemed this example to be valid.
However the 3 node causes this tree to be invalid.
While 3 is less than 6, it is not greater than 5.

We're going to need more context.

# Second Attempt

For my next attempt, I took a cue from my functional programming roots and tried to pass a list of all the parent nodes down.
My thinking was that if I had all of this context, I could be sure that I was validating the _entire_ subtree.

```csharp
public class Solution
{
    public bool IsValidBST(TreeNode root, List<TreeNode> lessThan = null, List<TreeNode> greaterThan = null)
    {
        if (root == null)
            return true;

        if (lessThan == null)
            lessThan = new List<TreeNode>();

        if (greaterThan == null)
            greaterThan = new List<TreeNode>();

        bool leftValid = lessThan.TrueForAll((lt) => root.val < lt.val);
        bool rightValid = greaterThan.TrueForAll((gt) => root.val > gt.val);

        // When taking the left path, add my value to the lessThan list
        var newLT = new List<TreeNode>(lessThan);
        newLT.Add(root);

        // When taking the right path, add my value to the greaterThan list
        var newGT = new List<TreeNode>(greaterThan);
        newGT.Add(root);

        return
            leftValid
            && rightValid
            && this.IsValidBST(root.left, newLT, greaterThan)
            && this.IsValidBST(root.right, lessThan, newGT);
    }
}
```

So this solution works, but uses more memory than necessary.

# Third Attempt

I eventually realized that validating the current node against _each_ of its parents individually was unnecessary.
It would be sufficient instead to track the acceptable range of values for the current node.
This range could be _derived_ from all previous parents, but wouldn't require as much space.
So on this attempt I switched to tracking the minimum of the upper bounds and the maximum of the lower bounds.

```csharp
public class Solution
{
    public bool IsValidBST(TreeNode root, int? min = null, int? max = null)
    {
        if (root == null)
            return true;

        int newMax = max == null ? root.val : Math.Min(max.Value, root.val);
        int newMin = min == null ? root.val : Math.Max(min.Value, root.val);

        return
            (min == null || root.val > min.Value)
            && (max == null || root.val < max.Value)
            && this.IsValidBST(root.left, min, newMax)
            && this.IsValidBST(root.right, newMin, max);
    }
}
```

Final results: top 50% runtime, top 77% memory.

This has been a...humbling experience thus far. :)

# Why not depth-first search?

About halfway through I considered switching to a depth-first search, but I'm not sure that it would make much of a difference.
Perhaps if the invalid nodes were to always appear at the deepest level of the tree, then depth-first would be a better choice.
However the problem makes no guarantees about the depths of the invalid nodes.
