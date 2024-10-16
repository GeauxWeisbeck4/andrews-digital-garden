---
id: 01J9QF7AWKA7ZXFJ5TDDB06RXJ
modified: 2024-10-08T21:45:26-04:00
---
The previous chapter showed how nearest-neighbor search allows us to find nearby or close data points, broadening our ability to answer coffee-related questions, such as finding the closest physical location or finding items with similar attributes. In this chapter, we build on the concepts of tree-based data structures and spatial partitioning to further improve our nearest-neighbor searches.

Chapter 8 discussed how we can adapt the algorithmic concepts for finding a specific value to the more general problem of finding nearest neighbors. We also saw how operations become more difficult as we transition from a single dimension to multiple dimensions. Maybe we want to find nearby coffee shops in two-dimensional space or similar friends (based on empathy, willingness to listen, and the ever-important coolness factor). Grids replace the simple arrays, and it’s no longer sufficient to sort along a single dimension.

This chapter introduces two new tree-based data structures: uniform quadtrees and k-d trees. The term _quadtree_ is often used to describe an entire class of two-dimensional data structures, based on the original quadtree proposed by computer scientists Raphael Finkel and Jon Bentley, that partition each two-dimensional node into four subquadrants at each level. We focus on a uniform quadtree such as the structure proposed by researcher and inventor David P. Anderson. This structure has equal-sized subregions that mirror the grid structure, thus building upon our discussions in the previous chapter. In contrast, _k-d trees_, invented by Jon Bentley, use a more flexible binary partitioning scheme that can further adapt to the data and allow us to scale to higher dimensions. Through examining quadtrees and k-d trees, we learn to generalize and modify tree-based data structures—and examine these data structures by comparison with the projects of city planners.

## Quadtrees

While grids provide a convenient data structure for storing two-dimensional data, they come with their own host of complexities. As we saw in the previous chapter, both the overhead and usefulness of grids depend heavily on how finely we partition the space. Using a large number of grid bins (by creating a finely grained grid) takes significant memory and may require us to search many bins. On the other hand, coarsely grained partitioning can result in a large number of points per bin, as in [Figure 9-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-1), resembling a simple linear scan over a large number of individual points.

![A two‐by‐two grid with 11 data points and a single target point.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09001.png)

Figure 9-1: A grid with a small number of bins

We can think of the grids in terms of different approaches to home organization. If we throw all our kitchen equipment into a single massive drawer, it takes forever to find a given item. This is equivalent to having a single large grid bin. To improve matters, suppose we separate utensils into different drawers and cookware into various cupboards depending on their usage. We even put cereals on one shelf and spices on another. This is equivalent to using finer-grained bins. Suddenly things are looking up—we no longer need to search under the frying pans to find the cinnamon. A little additional structure makes our cooking routine significantly easier.

However, we can take the idea too far. Perhaps we have too many utensils and it takes too long to search the packed drawer. We might improve efficiency by splitting the utensils into a drawer for scooping utensils and one for non-scooping utensils. But imagine the overhead of storing each utensil in its own drawer—or worse, allocating a drawer for each potential utensil we might buy. Soon we’re staring at a whole wall of drawers, wondering how many we’ll need to check to find something to beat our eggs. This is what happens when we use grids with overly fine bins.

The solution to this conundrum is to partition space dynamically relative to the data. We only bring in additional structure, and its corresponding overhead, when we need it. We start with a coarse-grained partition of the space. Maybe we wait to allocate a separate drawer for spatulas until we have at least five. Until then we can store them with the ladles and whisks. When we need more granularity, we subpartition our space further. To provide this dynamism, we can turn to _uniform quadtrees_.

A uniform quadtree brings the branching structure of trees to grids. Each node in the tree represents a region of space. The root node represents the entire space covered by the tree and all points contained within this space. Each node is partitioned into four equal-sized quadrants, with a child node for each non-empty quadrant. The term _uniform_ refers to the fact that nodes are partitioned into equal-sized spatial regions and thus partition the space within the node uniformly. [Figure 9-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-2) illustrates a quadtree split.

![The root node of the quadtree shown at the top encompasses the entire space and contains 11 points. Each of the four children contains a uniform region of the root node’s space and all points that fall within that region.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09002.png)

Figure 9-2: A quadtree node can have up to four children representing four equal-sized quadrants of that space.

When discussing the four subtrees, it’s common to label them NorthWest, NorthEast, SouthWest, and SouthEast to reflect their spatial positions in the original region.

Internal quadtree nodes store pointers to up to four children, along with associated metadata such as the number of points in the branch or the region’s spatial bounds (the minimum and maximum values for the node along the x- and y-dimension). Leaf nodes store a list of points that fall within that region, along with any desired metadata. We can use a single data structure for both internal and leaf nodes by keeping both a 2×2 grid of pointers to child nodes (for internal nodes) and an array for points (for leaf nodes). We either set the child entries to `null` in a leaf node or use an empty array for internal nodes.

The following is an example of a composite data structure for the `QuadTreeNode`:

```
QuadTreeNode {
    Boolean: is_leaf
    Integer: num_points
    Float: x_min
    Float: x_max
    Float: y_min
    Float: y_max
    Matrix of QuadTreeNodes: children
    Array of Points: points
}
```

We use a simple composite data structure for the points:

```
Point {
    Float: x
    Float: y
}
```

As in the previous chapter, we could alternatively store the points in arrays or ordered tuples.

Technically, we don’t need to store the spatial bounds of the node explicitly. We can instead derive the bounds for each node from bounds of the root node and the sequence of splits, since each node partitions its points along the middle of each dimension, slicing the space into four predictably sized subregions. Given the root node’s original bounds and the series of branches, we can compute the bounds for any child node precisely. However, precomputing and storing the bounds has one distinct advantage: at any given node we can simply look up the bounds instead of deriving them, making implementing our search algorithms significantly easier. I’ve often found that the value of storing this type of additional spatial information outweighs the additional memory costs.

The power of quadtrees is that branching at each level (where there are enough points) effectively creates an adaptive, hierarchical grid. [Figure 9-3](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-3) shows an example of one quadtree’s spatial partitioning.

![A visualization of a four‐level quadtree. Lines indicate the splits at each level.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09003.png)

Figure 9-3: The spatial partition created by a quadtree

Imagine the successive partitions of a quadtree and how we search them as the interactive geographical search software in a sci-fi thriller. The protagonists crowd the command room and stare at a large screen representing the entire city. Tense music plays. As new information pours in, the operator selects a quadrant from the screen. Someone says, “Zoom in there and enhance,” and the operator does so. Regardless of the dialogue, this operation is the equivalent to descending a level in the quadtree. In an instant, the command room’s screen displays a subset of the city. The entire range under display is a single quadrant of the previous level. Everyone stares at the new geo-subset intently, before they select a further subquadrant and zoom in again. The search concludes when our heroes have found the transmitter closest to the target points.

As with other trees, we can add a wrapper data structure around the uniform quadtree to simplify bookkeeping:

```
QuadTree {
    QuadTreeNode: root
}
```

The root node is created as an empty node with the correct dimensions at the time the tree itself is created, so we do not need to worry about its being null.

### Building Uniform Quadtrees

We build these magical quadtrees by recursively dividing our allocated space into smaller and smaller subregions. Since data can contain arbitrarily close or even duplicate points, we need additional logic to determine when to stop subdividing and designate a node with multiple points as a leaf. At each level, we check whether we need to make the current node an internal node (with child nodes) or a leaf node (with a list of points). There are different mechanisms we can use for this test, but here are the most common:

1. Are there enough points to justify a split? If we have too few points, the cost of checking the distance to the child nodes is higher than exhaustively checking each of the points. It’s not worth the overhead.
2. Are the spatial bounds large enough to justify a split? What if we have 10 points in exactly the same location? We could split and split and split without ever partitioning the points. It’s a waste of time and memory.
3. Have we hit a maximum depth? This provides an alternate check to prevent us from wasting time and memory on excessive subdivision by capping how deep our tree can be.

We can visualize this process as an unconventional city planner’s attempt to divide up the land such that each parcel has a building. Unacquainted with modern geographical partitioning techniques, the planner always divides regions into four equal-sized quadrants. Each time they look at a plot of land, they ask, “Are there too many buildings on this land?” and “Is this plot big enough to divide up further? I can’t sell a 2-foot by 2-foot lot. People would laugh.” The third criterion (maximum depth) represents how much subdividing the planner is willing to do before they give up. After four levels, the planner might call it “good enough” and move on. If the criteria for stopping aren’t met, the planner sighs, mumbles “Really? Again?” and subdivides the plot.

When partitioning a level, we divide the current space into four equal quadrants, partition the points according to quadrant, and recursively examine each one. If we set the minimum of points to 1 and the maximum depth to 4 (including the root), we’d construct the tree in [Figure 9-4](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-4). As shown by the illustration, we can save memory by only storing the non-empty children for each node. If a quadrant doesn’t have a child, we can set its pointer to `null`.

![The figure shows a quadtree with four levels. Each leaf node contains a single point.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09004.png)

Figure 9-4: An example quadtree with four levels

The code for bulk construction of quadtrees is very similar to the code presented in the next section for adding points. In fact, iteratively adding points to an empty quadtree is a good approach for constructing it.

### Adding Points

Since quadtrees are dynamic data structures, we can efficiently add points while maintaining the tree structure. We start with the wrapper function for the `QuadTree` data structure:

```
QuadTreeInsert(QuadTree: tree, Float: x, Float: y):
    IF x < tree.root.x_min OR x > tree.root.x_max:
        return False
    IF y < tree.root.y_min OR y > tree.root.y_max:
        return False
    QuadTreeNodeInsert(tree.root, x, y)
    return True
```

This wrapper guarantees we are always calling `QuadTreeNodeInsert` with a non-null node. The code also checks that the inserted point falls within the quadtree’s bounds. This is critical, as uniform quadtrees use equal-sized bins and cannot be dynamically resized. All points must fall within the root node’s spatial bounds. The code returns `False` if the point is out of range, but, depending on the implementation, you might want to use another mechanism such as returning an error or throwing an exception.

As shown in the following code, adding points to a node consists of traversing the tree to find the new point’s location. This search can end in one of two ways: at a leaf node or at an internal dead end. If we terminate the search at a leaf node, we can add the new point there. Depending on our splitting criteria (spatial bounds, max depth, and number of points), we might need to split the node into subnodes. If we end at an internal dead end, then we have found a path that previously didn’t contain any points. We can create the appropriate node.

```
QuadTreeNodeInsert(QuadTreeNode: node, Float: x, Float: y):
  ❶ node.num_points = node.num_points + 1
 
    # Determine into which child bin the point should go.
  ❷ Float: x_bin_size = (node.x_max - node.x_min) / 2.0
    Float: y_bin_size = (node.y_max - node.y_min) / 2.0
    Integer: xbin = Floor((x - node.x_min) / x_bin_size)
    Integer: ybin = Floor((y - node.y_min) / y_bin_size)

    # Add the point to the correct child.
  ❸ IF NOT node.is_leaf:
      ❹ IF node.children[xbin][ybin] == null:
            node.children[xbin][ybin] = QuadTreeNode(
                  node.x_min + xbin * x_bin_size,
                  node.x_min + (xbin + 1) * x_bin_size,
                  node.y_min + ybin * y_bin_size,
                  node.y_min + (ybin + 1) * y_bin_size)
        QuadTreeNodeInsert(node.children[xbin][ybin], x, y)
        return

    # Add the point to a leaf node and split if needed.
  ❺ node.points.append(Point(x, y))
  ❻ IF we satisfy the conditions to split:
        node.is_leaf = False
      ❼ FOR EACH pt IN node.points:
            QuadTreeNodeInsert(node, pt.x, pt.y)
      ❽ node.num_points = (node.num_points -
                           length(node.points))
        node.points = []
```

The code starts by incrementing `num_points` to represent the new point ❶. The function then determines which of the four bins the new point falls into by computing the size of the bins and using that to map the x and y indices to either `0` or `1` ❷. If the node is not a leaf, the code needs to recursively add the point to the correct child ❸. It starts by checking whether the child exists. If not (if the child’s pointer is `null`), it creates the child ❹. We use the fact that both `xbin` and `ybin` are either `0` or `1` to simplify the logic. Instead of enumerating all four cases, we can compute the child’s bounds with arithmetic. Finally, at leaf nodes, the code inserts the points directly into the node ❺.

We aren’t done yet, though. The code needs to check whether the splitting conditions are met ❻; if so, it splits the current leaf node. Luckily, we can reuse the same insertion function for splitting as we did to add a point. The code marks the node as a non-leaf (`node.is_leaf = False`) and reinserts the points one at a time using a `FOR` loop ❼. Because the current node is no longer a leaf node, the reinserted points now fall through to the correct children, with new children created as needed. However, since we used the function twice for each point, we have to correct `num_points` to avoid double-counting the reinserted points ❽ (due to the counter increment at ❶). The code also clears the list of points at the newly internal node.

[Figure 9-5](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-5) shows what happens if we add two points to the tree. The inserted open-circle point causes one node to split before triggering the max-depth condition. The resulting leaf node holds two points. The inserted open-square point adds a single new child node corresponding to its parent NorthWest quadrant. The code does not split again because of the minimum number of points condition.

As noted in the previous section, we can use this approach to construct a uniform quadtree from a set of points. First, we create an empty root node with the necessary spatial bounds. Then we incrementally add each point from our set. Since splits are always based at the halfway point along each dimension, the tree structure does not change due to the order in which we insert the points.

![A figure showing the addition of two points to the uniform quadtree. The open circle is inserted in the root’s NorthWest Child and that node’s SouthWest child. The previous leaf now has two points and is split again. Both points are in the same resulting node, which is a leaf due to the depth constraint.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09005.png)

Figure 9-5: An example of adding two points, indicated by a shaded circle and shaded square, to a quadtree

### Removing Points

Removing points from a node follows a similar, but more complex, process as inserting points. We delete the point from the list in the leaf node. We can then proceed back up the tree, removing splits that are no longer warranted by our criteria. This can involve recursively extracting the points from each of the node’s children (which may themselves have children) and merging them into a single list at the next leaf node.

One additional difficulty is determining which point to delete. As with grids, the user might insert arbitrarily close or even duplicate points. In the code below, we delete the first matching point in the leaf node’s list. Due to floating-point errors (rounding due to a floating-point variable’s limited precision), we also cannot use a direct equality test. Therefore, we use a helper function to find a point that is close enough. We can reuse the `approx_equal` function from Listing 8-3 for this test.

We also use a helper function to collapse nodes that no longer meet our splitting criteria. The code collapses a node with children and returns an array with all the subtree’s points:

```
QuadTreeNodeCollapse(QuadTreeNode: node):
  ❶ IF node.is_leaf:
        return node.points

  ❷ FOR i IN [0, 1]:
        FOR j IN [0, 1]:
            IF node.children[i][j] != null:
                Array: sub_pts = QuadTreeNodeCollapse(node.children[i][j])
                FOR EACH pt IN sub_pts:
                    node.points.append(pt)
                node.children[i][j] = null
  ❸ node.is_leaf = True
  ❹ return node.points
```

The code starts by checking whether the current node is already a leaf and, if so, directly returning the array of points ❶. Otherwise, the node is internal, and the code needs to aggregate the individual data points from each child. The code loops through each of the four children, checking whether they are `null` and, if not, recursively calling `QuadTreeNodeCollapse` to aggregate the points ❷. The function finishes by setting the current node to be a leaf ❸ and returning the points ❹.

With that helper function, we can move on to the deletion function. We start with the wrapper.

```
QuadTreeDelete(QuadTree: tree, Float: x, Float: y):
    IF x < tree.root.x_min OR x > tree.root.x_max:
        return False
    IF y < tree.root.y_min OR y > tree.root.y_max:
        return False
    return QuadTreeNodeDelete(tree.root, x, y)
```

The wrapper function starts by checking that the point lies within the bounds of the tree and thus could be in the tree. If so, it calls the recursive deletion function.

The recursive deletion code proceeds down the tree as though searching for the point. When it reaches a leaf, it deletes the point if it exists. It then returns up the tree, collapsing nodes as needed. The function returns `True` if a point was deleted.

```
QuadTreeNodeDelete(QuadTreeNode: node, Float: x, Float: y):
  ❶ IF node.is_leaf:
        Integer: i = 0
      ❷ WHILE i < length(node.points):
          ❸ IF approx_equal(node.points[i].x, node.points[i].y, x, y):
                remove point i from node.points
                node.num_points = node.num_points - 1
                return True
            i = i + 1
        return False

    # Determine into which child bin the point to be removed would go.
  ❹ Float: x_bin_size = (node.x_max - node.x_min) / 2.0
    Float: y_bin_size = (node.y_max - node.y_min) / 2.0
    Integer: xbin = Floor((x - node.x_min) / x_bin_size)
    Integer: ybin = Floor((y - node.y_min) / y_bin_size)

  ❺ IF node.children[xbin][ybin] == null:
        return False
      
  ❻ IF QuadTreeNodeDelete(node.children[xbin][ybin], x, y):
        node.num_points = node.num_points - 1

      ❼ IF node.children[xbin][ybin].num_points == 0:
            node.children[xbin][ybin] = null

      ❽ IF node no longer meets the split conditions
            node.points = QuadTreeNodeCollapse(node)
        return True
  ❾ return False
```

The code starts by checking whether the recursion has reached a leaf node ❶. If so, it iterates through the array of points ❷, checking whether each one matches the target point ❸. If the code finds a matching point, it removes it from the array, decrements the count of points in this node, and returns `True` to indicate a successful deletion. Note that only a single matching point is ever deleted. If the code does not find a matching point at the leaf, it returns `False`.

The search then progresses down into the bin into which the target point would be assigned ❹. The code checks whether the correct child exists and, if not, returns `False` to indicate the point is not in the tree ❺. Otherwise, the code recursively calls `QuadTreeNodeDelete` on the child ❻.

## NOTE

As in Chapter 8, due to the numerical precision for floating-point values, we might need to extend the check of the child bin to handle cases where the point falls right on the split threshold. For simplicity of illustration, we will just check a single bin for now. But additional checks may be needed for production code.

If the recursive call to `QuadTreeNodeDelete` returns `True`, then the code has deleted a point from one of its children. It updates the count of points and checks whether the child is empty ❼. If so, it deletes the child node. Then the code checks whether the current node continues to meet the criteria for being an internal node ❽. If not, it collapses the node. The code returns `True` to indicate the deletion was successful. If the recursive call did not return `True`, then no point was deleted. The function finishes by returning `False` ❾.

### Searching Uniform QuadTrees

We begin our search of the quadtree at the root node. At each node, we first ask whether the node could contain anything closer than our current candidate. If so, for an internal node we explore the children recursively, and for a leaf node we test the points directly. However, if we determine that the current node could _not_ contain our nearest neighbor, we can prune the search by ignoring not only that node but its entire subtree.

We check the compatibility of points within a node using the same test we applied to grid cells in Chapter 8. We have similar information at our disposal: the location of the point and the bounding box of the region. As described in the previous chapter, the distance computation is:

![g09001](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/g09001.png)

where

IF _x_ < _x_min THEN _x_dist = _x_min – _x_

IF _x_min ≤ _x_ ≤ _x_max THEN _x_dist = 0

IF _x_ > _x_max THEN _x_dist = _x_ – _x_max

and

IF _y < y_min THEN _y_dist _= y_min _– y_

IF _y_min ≤ _y_ ≤ _y_max THEN _y_dist = 0

IF _y_ > _y_max THEN _y_dist = _y_ – _y_max

In short, we’re checking the distance from our search target to the closest possible point contained in the node’s spatial region. To revisit the example from the last chapter, we’re once again asking how far our lazy neighbor would need to throw a ball in order to just barely return it to our yard. This check only tests whether a point at this distance could exist within the node. We need to explore the node to see what points exist.

Consider searching for the nearest neighbor of the point marked with an X in [Figure 9-6](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-6). This figure shows the same distribution of points as our original map of coffee shop locations in Figure 8-3, where the X is our current location and the other points are nearby cafés.

We start at the root node with a dummy candidate point with infinite distance. We haven’t seen any candidate neighbors yet, so we need something with which to start our search. Using a dummy point with infinite distance allows the algorithm to accept the first point it finds as the new candidate without requiring any special logic. Any point we find is going to be closer than infinitely far away, and every region will contain closer points. In the case of searching for coffee shops, this corresponds to starting with an imaginary café that is infinitely far away. Any real café on our list is guaranteed to be closer than this imaginary point.

![A set of two‐dimensional points. There are 11 data points and 1 query point in the upper left‐hand area of the plot.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09006.png)

Figure 9-6: An example data set for nearest-neighbor search

## NOTE

It’s possible to begin our quadtree search with something other than an infinite-distance dummy candidate, like the first point in the data set (as we did with linear scan in Listing 8-1) or a random point from out data set. All that really matters is that we have some point and distance for use in our comparisons at each node.

Since our (dummy) candidate point has infinite distance, our compatibility test for the root node passes. At least one of the points in the root node _could_ be less than infinitely far away. Although this test is mathematically identical to the one we used for grid cells, it has one big practical difference: the size of the cells we are testing varies at each level of the tree. At higher levels, each node covers a large amount of space. As we descend lower, the spatial bounds tighten.

We prioritize which child node to search first based on the childrens’ proximity to the query point. After all, we ultimately want to find the closest point and prune as much as possible. So we consider the x and y splits and ask “Into which of the four quadrants would our target point fall?” In this case, our query point falls in the NorthWest quadrant, as shown in [Figure 9-7](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-7), so we start there.

We proceed down the pointer to the NorthWest child and find ourselves focusing on a subset of both the space and the points as shown in [Figure 9-8](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-8). The grayed-out nodes represent nodes that have not been explored, and the grayed-out points have not been checked.

![The figure shows the root node with lines to indicate its partition into four quadrants. The query point is shown as an X in the NorthWest quadrant.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09007.png)

Figure 9-7: The query point falls within the NorthWest quadrant of the quadtree’s root node.

![The diagram shows the first three levels of a quadtree. The child corresponding to the root node’s northwest quadrant is circled to indicate that the search is exploring that node.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09008.png)

Figure 9-8: The nearest-neighbor search for the example shown in [Figure 9-6](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-6) starts by searching the subtree corresponding to the NorthWest quadrant of the root node.

Again, our test of spatial compatibility reveals that this node _could_ contain our nearest neighbor. The minimum distance to the any point in the node is less than that of our current (dummy) candidate. We’re at another internal node, which means that the space is further partitioned into four subregions.

Our search continues to the SouthWest child of the internal node in question, as illustrated in [Figure 9-9](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-9). We choose this node because it is the closest to our query point. For the third time, the compatibility test passes. Since we’re at a leaf node, we explicitly check the distance to each point in that node. In this case, there is only a single point, and it is closer than our dummy candidate.

![The diagram shows the first three levels of a quadtree. Three nodes are solid: the root node, the root node’s NorthWest child, and that node’s SouthWest child. The current node in the search is circled.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09009.png)

Figure 9-9: At the second level of the quadtree, our search starts with the closest quadrant to the target point, SouthWest.

We’ve found our first real candidate nearest neighbor! Its distance becomes our minimum distance so far. We can be pickier about all future points. In our running example of finding a nearby coffee shop, this first neighbor represents the closest coffee shop found so far. The distance to this point is the maximum distance that we’ll need to travel to get a cup of coffee. We might find closer coffee shops later in our search, but at least we don’t have to travel infinite miles. The relief is palpable.

Once we’ve tested all the points in a leaf node, we return to the (internal) parent node and check the remainder of the children. Now that we have a real candidate and distance, our pruning tests have power. We check the compatibility of all remaining child quadrants: NorthWest, NorthEast, and SouthEast. Our distance test shows that we can skip the NorthWest quadrant: as shown in [Figure 9-10](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-10), the closest possible point in its spatial bounds is further than the candidate we already have. It can’t possibly contain a better neighbor.

![A diagram showing the four quadrants in the root’s NorthWest child. Lines from the query point to the best node so far indicate the threshold distance. The min distance to the current node’s NorthWest child is also indicated by a longer line.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09010.png)

Figure 9-10: An illustration of the relative distance between the best candidate seen so far and the current node’s NorthWest quadrant

We can also skip the empty NorthEast and SouthEast quadrants. Since they don’t have any points, they can’t have a better neighbor either. Our search can discard both quadrants without a distance test because their pointers will be `null` to indicate that no such child exists. We’ve managed to prune three of the four quadrants in this node as illustrated by the grayed-out quadrants in [Figure 9-11](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-11). The two data points in the NorthWest quadrant also remain gray, because we never tested them.

![The search has returned to the root node’s NorthWest child. Three of the node’s quadrants are grayed out to indicate they were pruned from the search.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09011.png)

Figure 9-11: The nearest-neighbor search is able to skip three of the node’s four quadrants.

Once we’ve finished checking the quadrants within an internal node, we return to its parent and repeat the process. The next closest quadrant is SouthWest, which our pruning test confirms is close enough to possibly contain a better neighbor, as shown in [Figure 9-12](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-12).

Any time we find a child that, according to our simple distance test, _could_ contain a closer point, we proceed down that pathway to check if there are indeed closer neighbors. In this case, our search descends into the SouthWest quadrant.

![An illustration of pruning at the root node. The query point is shown with an X, and a line shows the distance to the best neighbor so far. A second line shows the (smaller) distance to the root’s SouthWest child.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09012.png)

Figure 9-12: A pruning test where the candidate quadrant could include a closer neighbor than the current best point

At the next level, four potential quadrants again vie for our attention. Armed with a real candidate point and its corresponding distance, we can aggressively prune our search, as shown in [Figure 9-13](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-13). We check the NorthWest quadrant (and its single point) because it falls within our distance threshold. We can skip the other three quadrants; the NorthEast and SouthWest quadrants are both empty and thus have null child pointers, and we use a distance test to confirm that the SouthEast quadrant is too far away to contain a better neighbor.

This time when we return to the root node, we can prune out the two remaining children, as shown in [Figure 9-14](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-14).

The remote expanses of both the NorthEast and SouthEast quadrants lie well outside our distance threshold.

![The search has progressed down the entire branch corresponding to the root node’s SouthWest child. That child has three of its four quadrants grayed out.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09013.png)

Figure 9-13: While checking the root node’s SouthWest quadrant, the nearest-neighbor search can again skip three of the four subquadrants.

![The search has returned to the root node. Two of the root node’s quadrants are grayed out. Nine of the 11 data points remain gray.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09014.png)

Figure 9-14: Upon returning to the root node of the quadtree, the search can skip two of that node’s quadrants.

### Nearest-Neighbor Search Code

To simplify our implementation of the nearest-neighbor search code, we start with a helper function to compute the distance from the target point (_x_, _y_) to a node. The code for checking the minimum distance to a node is similar to the minimum distance code presented for grids in the last chapter:

```
MinDist(QuadTreeNode: node, Float: x, Float: y):
    Float: x_dist = 0.0
    IF x < node.x_min:
        x_dist = node.x_min – x
    IF x > node.x_max:
        x_dist = x – node.x_max

    Float: y_dist = 0.0
    IF y < node.y_min:
        y_dist = node.y_min – y
    IF y > node.y_max:
        y_dist = y – node.y_max

    return sqrt(x_dist*x_dist + y_dist*y_dist)
```

However, in this case, the code does not need to compute the minimum and maximum bounds for the node, since each node stores them explicitly.

The main search algorithm uses the same recursive formulation described for other tree-based methods. Our implementation of this search algorithm includes a parameter `best_dist` that represents the distance so far. By passing `best_dist` to our search function, we can simplify the pruning logic. If the minimum distance to the current node is greater than the best distance so far, we can terminate that branch of the search. The function then returns a _closer_ point if it finds one and `null` otherwise. It’s important to note that in this implementation, a return value of `null` means that there are no points in the current node closer than `best_dist`.

We use a thin wrapper that passes along the root node and an initial infinite distance:

```
QuadTreeNearestNeighbor(QuadTree: tree, Float: x, Float: y):
    return QuadTreeNodeNearestNeighbor(tree.root, x, y, Inf)
```

Our wrapper function for nearest-neighbor search does not check that the target point falls within the quadtree bounds. This allows us to use the code to find neighbors within the tree for target points that fall outside the tree’s bounds, increasing the usefulness of the code.

Here’s the code for recursively searching the nodes:

```
QuadTreeNodeNearestNeighbor(QuadTreeNode: node, Float: x,
                            Float: y, Float: best_dist):
    # Prune if the node is too far away. 
  ❶ IF MinDist(node, x, y) >= best_dist:
        return null
    Point: best_candidate = null

    # If we are in a leaf, search the points.
  ❷ IF node.is_leaf:
        FOR EACH current IN node.points:
            Float: dist = euclidean_dist(x, y, current.x, current.y)

            IF dist < best_dist:
                best_dist = dist
                best_candidate = current
        return best_candidate

    # Recursively check all 4 children starting with the closest.
  ❸ Float: x_bin_size = (node.x_max - node.x_min) / 2.0 
    Float: y_bin_size = (node.y_max - node.y_min) / 2.0 
    Integer: xbin = Floor((x - node.x_min) / x_bin_size)
    IF xbin < 0:
        xbin = 0
    IF xbin > 1:
        xbin = 1

    Integer: ybin = Floor((y - node.y_min) / y_bin_size)
    IF ybin < 0:
        ybin = 0
    IF ybin > 1:
        ybin = 1

  ❹ FOR EACH i IN [xbin, (xbin + 1) % 2]:
        FOR EACH j IN [ybin, (ybin + 1) % 2]:
            IF node.children[i][j] != null:
                Point: quad_best = QuadTreeNodeNearestNeighbor(
                                       node.children[i][j], 
                                       x, y, best_dist)
              ❺ IF quad_best != null:
                    best_candidate = quad_best
                    best_dist = euclidean_dist(x, y, quad_best.x, 
                                               quad_best.y)
    return best_candidate
```

The function starts with the pruning test, skipping the node and returning `null` if no point could be closer than `best_dist` ❶. The code then checks if it is at a leaf node ❷. If it has reached a leaf, it uses a `FOR` loop to check whether each point is closer than `best_dist`, updating `best_dist` and `best_candidate` if so. At the end of this loop, we return `best_candidate`, which has the value `null` if we haven’t found a new closer point.

The next block of code handles the logic for an internal node. We have only made it this far if the node isn’t a leaf and thus doesn’t have any candidate points. Some basic numerical tests and integer manipulation control the order that the code searches the children, allowing the code to search the closest child first and then expand to the rest of the children. The code starts by computing into which x and y bin the candidate point should fall ❸, adjusting the value so that `xbin` and `ybin` both fall within [0, 1], and thus indicates the closest child node. This adjustment is necessary because, for many internal nodes, our target point will lie completely outside the 2×2 grid represented by the current node.

We then recursively explore the non-null children using a pair of nested `FOR` loops to iterate over the pairs ❹. Each time we check whether we find a closer point (represented by `quad_best != null)` and, if so, update both `best_candidate` and `best_dist` ❺. At the end of the function, we return `best_candidate`. As in the case of the leaf node, `best_candidate` may be `null` if we haven’t found a point closer than the original `best_dist`.

## k-d Trees

We’ve solved the problem of dynamic splits for two dimensions, so now it’s time to turn our attention to searching for points or nearest neighbors in three or more dimensions using k-d trees. We’ve already seen the disappointment that can result when we don’t account for all the relevant attributes in our coffee. Higher-dimensional problems are common whenever we’re looking for similar points in data sets, such as searching for similar conditions (temperature, pressure, humidity) in a weather data set.

In theory, we could scale quadtrees by simply splitting along more dimensions. _Octtrees_, for example, are three-dimensional versions that split into eight subnodes at each level. Eight-way splits may not seem too bad, but this approach clearly doesn’t scale gracefully with the number of dimensions. If we want to build a tree over _D_-dimensional data, we need to split along all _D_ dimensions at once, giving us 2_D_ children for each internal node. If we were building a data structure over weather data containing temperature, pressure, humidity, precipitation, and wind speed, we would be using five-dimensional points and splitting 32 subtrees at each level! This immense overhead is the same problem we found when scaling grids to higher dimensions.

To effectively scale to higher dimensions, we need to rein in the branching factor. There are a variety of powerful data structures designed to enable efficient proximity search in higher dimensions. One example is the k-d tree, which builds off similar concepts as the quadtree.

### k-d Tree Structure

A _k-d tree_ is a spatial data structure that combines the spatial partitioning of quadtrees with the binary branching factor of a binary search tree, giving us the best of both worlds. Instead of splitting along every dimension at every level, the k-d tree chooses a single dimension and splits the data along that dimension. Each internal node thus partitions the data into exactly two children: a left child whose points fall below (or equal to) the split value, and a right child whose points fall above the split value.

When working with k-d trees, we lose the regular grid-like structure of the uniform quadtree, but, in exchange, we return to the two-way branching factor we know and love from binary search trees. This ability to partition along a single dimension in turn allows the k-d tree to scale to higher-dimensional data sets. We no longer need to split into 2_D_ children at each level. [Figure 9-15](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-15) shows an example k-d tree.

![The root node of the k‐d tree is split along the y‐dimension. Each of the two resulting children is split along the x‐dimension.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09015.png)

Figure 9-15: A k-d tree split along a single dimension at each level

A k-d tree can adjust more flexibly to the structure of the data than quadtrees. We’re not constrained to split in the midpoint of every dimension at once. Instead of partitioning space into 2_D_ equally sized grid cells, we can choose a split dimension and value that are best suited to the data at each node. Each internal node thus stores both the dimension along which we’re partitioning (`split_dim`) and the value (`split_val`), with points assigned to the left child if their value in that one dimension is less than or equal to the node’s split value:

```
pt[split_dim] <= split_val
```

Rather than split along alternating axes, producing partitioning similar to that of the quadtree in [Figure 9-15](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-15), we can tailor our tree structure by choosing splits based on the composition of the data points within the current node, thus allowing us to make splits that will better aid our future searches. We might choose the split value based on the overall bounds of the node, such as splitting in the middle of the widest dimension. Or we could choose the split value based on the distribution of data points, such as splitting at the middle of the range or the median of the points’ values along the split dimension. The difference between these two options is illustrated in [Figure 9-16](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-16).

![The left box shows points partitioned at the middle of the x‐dimension, splitting the space in half. The right box shows points partitioned at the median value along the x‐dimension, resulting in two boxes with an approximately equal number of points but different sizes.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09016b.png)

Figure 9-16: A node split at the middle (left) and median (right)

The flexible structure of the k-d tree means we need to take additional care when dealing with nodes’ spatial bounds. Unlike the uniform quadtree’s square grid cells, the nodes of a k-d tree cover multidimensional rectangles of the overall search space. The width in each dimension can be completely different. Some nodes might be square-ish, while others might be long and thin. We represent a node’s area by explicitly tracking the multidimensional rectangle defining its spatial bounds—the minimum and maximum values in each dimension. Since we can have an arbitrary number of dimensions, we store the bounds in two arrays, `x_min` and `x_max`, where `x_min[d]` represents the lowest value along dimension `d` contained within the current node and `x_max[d]` represents the highest. All points within the node satisfy:

```
x_min[d] <= pt[d] <= x_max[d] FOR ALL d
```

As a result of its complexity, each k-d tree node stores a significant amount of information. While this might initially seem like expensive overhead, as we’ll see in this section, the cost is offset by the power and flexibility of tree itself. As with every other data structure in this book, we are making an explicit tradeoff in terms of memory, data structure complexity, and later algorithmic efficiency.

Here is an example composite data structure for the `KDTreeNode`:

```
KDTreeNode {
    Boolean: is_leaf
    Integer: num_dimensions
    Integer: num_points
    Array of Floats: x_min
    Array of Floats: x_max
    Integer: split_dim
    Float: split_val
    KDTreeNode: left
    KDTreeNode: right
    Array of Arrays: points
}
```

In this case, we use an array to represent each point, allowing it to have arbitrary dimensions.

As with other trees in this book, we can wrap the nodes in an outer data structure such as a `KDTree`.

```
KDTree {
    Integer: num_dimensions
    KDTreeNode: root
}
```

It is helpful to store the number of dimensions in this wrapper data structure to check for consistency during operations such as insertion, deletion, or search. The value of `num_dimensions` is set at time of the k-d tree’s creation and remains fixed for this tree.

This flexibility of choosing a strategy for splitting nodes illustrates the true power of the k-d tree: we’re augmenting the spatial partitioning of quadtrees to further adapt to the data. If our points are clustered, we choose splits that provide the most information by focusing on those regions. [Figure 9-17](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-17) demonstrates this dynamic partitioning.

![The figure of a k‐d tree with horizontal or vertical lines representing the different splits at each level. The subdivisions are all of different dimensions.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09017.png)

Figure 9-17: The spatial partition created by a k-d tree

Consider our ongoing task of locating nearby coffee shops. If we are taking a road trip along Interstate 95 from Florida to Maine, we could preprocess the data to store only those coffee shops within 50 miles of the highway. [Figure 9-18](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-18) shows an example distribution of this shape.

This prefiltering helps limit the search space to only coffee shops near the highway, yet we can make our searches even more efficient by storing these locations in a spatial data structure. We want to narrow our search to the appropriate region along the highway as well. After all, there is no need to check coffee shops in Massachusetts while we are still in South Carolina. We quickly see that a uniform partitioning scheme is far from ideal: our trip covers over 1,500 miles of mostly northward driving. Since we have already filtered to only shops along the highway, we do not gain as much pruning from partitioning uniformly east and west as well. We can increase the amount of pruning, and thus decrease the search cost, by biasing our partitions along the north-south direction.

To give another analogy, if quadtrees are the daily routine of a city planner following rigid regulations to divide up a map, k-d trees represent a city planner with a wider range of tools at their disposal. They’re no longer constrained to divide each plot into perfect squares. Instead, they have the flexibility to partition the space according to the distribution of actual points. Our city planner chooses the widest dimension to split in order to minimize long, narrow zones. They also use the median point along this dimension to provide a balanced tree. The result might not always look as orderly as the quadtrees squares, but it can be much more effective.

![A scatterplot of points in the shape of Route 95.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09018.png)

Figure 9-18: An example of how coffee shops might be distributed along a major highway

### Tighter Spatial Bounds

We can often further improve a spatial tree’s pruning power by tracking the bounding box of all the points falling within a given node instead of the node’s total spatial bounds. This changes our pruning question from “Could a nearest-neighbor candidate exist in the space covered by the node?” to “Could a nearest-neighbor candidate exist in the bounding boxes of actual points within the node?” While this might seem like a small change, depending on what logic we use to split the nodes, we can see significant savings, as shown in [Figure 9-19](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-19).

![The tight bounding box of a k‐d tree node is shown as a dashed box bounding the points within the node.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09019.png)

Figure 9-19: The bounding box of points within a k-d tree node

If we use these tighter bounding boxes during node construction, we can better adapt to the data. We are no longer splitting based on the entire possible spatial region, but rather on the region occupied by actual points. The resulting k-d tree structure looks like the one in [Figure 9-20](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#figure9-20). The black boxes indicate the tight bounding boxes at each node. The gray points and lines are shown to place the nodes’ bounds in the context of the rest of the data set.

![Each node in the k‐d tree is shown as a set of points, and a tight bounding box is laid over the grayed‐out full data set.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/f09020.png)

Figure 9-20: A three-level k-d tree and the bounding boxes for each node

During a search, we use the black boxes (tight bounding boxes) for pruning. As you can see from the figure, the resulting regions can be significantly smaller, allowing us to prune much more aggressively. Since the tight bounding box is smaller, it will often have a greater minimum distance to our query point.

We can use a simple helper function to compute the tight bounding boxes from a set of points represented as arrays ([Listing 9-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#listing9-1)):

```
ComputeBoundingBox(Array of Arrays: pts):
  ❶ Integer: num_points = length(pts)
    IF num_points == 0:
        return Error
    Integer: num_dims = length(pts[0])

  ❷ Array: L = Array of length num_dims
    Array: H = Array of length num_dims
    Integer: d = 0
  ❸ WHILE d < num_dims:
        L[d] = pts[0][d]
        H[d] = pts[0][d]
        d = d + 1

    Integer: i = 1
  ❹ WHILE i < num_points:
        d = 0
        WHILE d < num_dims:
            IF L[d] > pts[i][d]:
                L[d] = pts[i][d]
            IF H[d] < pts[i][d]:
                H[d] = pts[i][d]
            d = d + 1
        i = i + 1
  ❺ return (L, H)
```

Listing 9-1: A helper function to compute the tight bounding box of the points in a node

The code extracts the number of points and number of dimensions from the input data ❶. It then creates new arrays `L` and `H` to hold the low and high bounds, respectively, ❷ and seeds them with the coordinates of the first point in our array ❸. The code then iterates through the remaining points in the array and checks whether any of them fall outside the bounds ❹. If so, it extends the bounds. The code ends by returning the two arrays ❺.

This helper function also illustrates the benefit of prechecking that all our points contain the correct number of dimensions in the `KDTree`’s wrapper function. This check ensures that we do not try to access our invalid array entries for our data points.

Of course, tracking these additional bounds adds significant complexity when we start dealing with dynamic changes. Adding points to a k-d tree can increase the bounds for a given node. Similarly, removing nodes may shrink the bounds.

### Building k-d Trees

Construction of a k-d tree uses a recursive process similar to the one for binary search trees, but with a few major differences. We start with all the data points and divide them into two subsets by picking a splitting dimension and value. This process repeats at each level until we hit our termination criteria: a minimum number of points left at the node, a minimum width, or a max depth. Generally, we use two of these tests, minimum number of points and minimum width, but using at least one of the last two tests is essential to avoid infinite recursion when you have duplicate data points.

We start with a wrapper function to check that our data is valid:

```
BuildKDTree(KDTree: tree, Array of Arrays: pts):
  FOR EACH pt IN pts:
      IF length(pt) != tree.num_dimensions:
          Return an error.
  IF length(pts) > 0:
      tree.root = KDTreeNode()
      RecursiveBuildKDTree(tree.root, tree.num_dimensions, pts)
  ELSE:
      tree.root = null
```

The code starts by checking that all points have the correct dimensionality. It then checks that there are points from which to build the tree. If so, it allocates a new root node (overwriting the previous tree) and recursively builds the k-d tree using the function below. Otherwise, it sets the root to `null` to indicate an empty tree.

The primary difference between k-d tree and quadtree construction is that k-d trees require us to choose a single split dimension at each level. If we are using tight bounding boxes, we also need to compute the bounding boxes over _D_ dimensions. While these changes make the code a little longer (with extra loops over _D_), they don’t add any real complexity. The code for building a k-d tree recursively partitions our set of points among the child nodes until we reach the termination criteria.

```
RecursiveBuildKDTree(KDTreeNode: node, Integer: num_dims,
                     Array of Arrays: pts):
  ❶ node.num_points = length(pts)
    node.num_dimensions = num_dims
    node.left = null
    node.right = null
    node.points = empty array
    node.split_dim = -1
    node.split_val = 0.0
    node.is_leaf = True

    # Compute the bounding box of the points.
  ❷ (node.x_min, node.x_max) = ComputeBoundingBox(pts)

    # Compute the width of the widest dimension.
  ❸ Float: max_width = 0.0
    Integer: d = 0
    WHILE d < node.num_dimensions:
        IF node.x_max[d] - node.x_min[d] > max_width:
            max_width = node.x_max[d] - node.x_min[d]
        d = d + 1

    # If we meet the conditions for a leaf, append the
    # remaining points to the node's point list.
  ❹ IF we do not satisfy the conditions to split:
        FOR EACH pt IN pts:
            node.points.append(pt)
        return

    # Choose split dimension and value.
  ❺ node.split_dim = chosen split dimension
    node.split_val = chosen split value along node.split_dim
    node.is_leaf = False

    # Partition the points into two sets based on
    # the split dimension and value.
    Array of Arrays: left_pts = []
    Array of Arrays: right_pts = []
  ❻ FOR EACH pt IN pts:
        IF pt[node.split_dim] <= node.split_val:
            left_pts.append(pt)
        ELSE:
            right_pts.append(pt)

    # Recursively build the child nodes.
  ❼ node.left = KDTreeNode()
    RecursiveBuildKDTree(node.left, num_dims, left_pts)

    node.right = KDTreeNode()
    RecursiveBuildKDTree(node.right, num_dims, right_pts)
```

This code starts with bookkeeping to fill in the information needed at each node. We record the essential details, such as the number of points and number of dimensions of the current points ❶. Then the function loops over all the points, computing the tight bounding box for the node ❷ using the helper function in [Listing 9-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c09.xhtml#listing9-1).

Once we have the bounds, the code loops over each of the dimensions to find the widest dimension ❸. We use this loop for one of the stopping conditions for the recursion (not splitting a node that is too small). If the node does not meet the conditions to keep splitting, the code stores all the points in a list at the leaf ❹. Otherwise, the code picks a split dimension and a split value for the node ❺. The code iterates over the current set of points, partitioning them into two arrays, `left_pts` and `right_pts`, according to the split dimension and value ❻. Those arrays are used to recursively build the two children nodes ❼.

One approach for choosing `split_dim` and `split_val` is to split along the middle of the widest dimension. The code for this is relatively simple and most of it can be incorporated into the initial block that finds the widest dimension ❸:

```
    Float: max_width = 0.0
    Integer: split_dim = 0
    Integer: d = 0
    WHILE d < node.num_dimensions:
        IF node.x_max[d] - node.x_min[d] > max_width:
            max_width = node.x_max[d] - node.x_min[d]
            split_dim = d
        d = d + 1
```

and then later setting the split dimension and the value at ❺:

```
    node.split_dim = split_dim
    node.split_val = (node.x_min[node.split_dim] + 
                      node.x_max[node.split_dim]) / 2.0
```

Bulk construction of k-d trees has a significant advantage over dynamically inserting and removing points. By considering all the data points during construction, we can better adapt the structure of the tree to the data. We choose splits based on all the data points instead of the subset inserted so far.

### k-d Tree Operations

The basic operations of inserting points, deleting points, and searching the k-d tree follow the same approach as with quadtrees. We start all operations at the top of the tree (root node) and use the split values to navigate down the appropriate branches. The major difference is that, instead of picking which of four quadrants to explore, we use `split_dim` and test against `split_val` to choose one of two children. As the high-level concepts are similar to those presented for the quadtree, we will not go into the code in detail for each one. Instead, let’s look at some of the differences.

1. Insertion When inserting points into a k-d tree node, we use `split_dim` and `split_val` to determine which branch to take. We split leaf nodes if they meet our split condition using the same approach as we would use during bulk construction. Finally, if we are tracking the tight bounding box for each node, we need to update the bounds to account for the new point. Since we are adding points, this update will always increase the size of the bounding box.
    
    ```
        Integer: d = 0
        WHILE d < node.num_dimensions:
            IF x[d] < node.x_min[d]:
                node.x_min[d] = x[d]
            IF x[d] > node.x_max[d]:
                node.x_max[d] = x[d]
            d = d + 1
    ```
    
    This code iterates over each dimension of the new point, checks whether it falls outside the bounding box, and, if so, updates the bounds.
    
2. Deletion When deleting points in a k-d tree, we use `split_dim` and `split_val` to determine which branch to take during our search for the point. After deleting a node, we return to the root of the tree. At each node along the way, we check whether the bounds can be tightened (using the points in a leaf or the two children’s bounding boxes for an internal node). We also check whether internal nodes can be collapsed.
3. Search The key difference in the search operations between a quadtree and a k-d tree is in testing whether we can prune nodes. For example, we can compute the Euclidean distance between a point _x_ and the closest possible point in a node’s (non-uniform, _D_-dimensional) bounding box using an extension of the formula ![i09001](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c09/i09001.png) that we used for quadtrees and grids. We start by competing the minimum distance from the point to the node’s spatial bounds along each individual dimension:

IF _x_[_d_] < _x_min[_d_] THEN _dist_d = _x_min[_d_] − _x_[_d_]

IF _x_min[_d_] ≤ _x_ ≤ _x_max[_d_] THEN _dist_d = 0

IF _x_[_d_] > _x_max[_d_] THEN _dist_d = _x_[_d_] − _x_max[_d_]

1. where _x_[_d_] represents the query point’s _d_th dimension, and _x_min[_d_] and _x_max[_d_] represent the node’s low and high bounds in the _d_th dimension. Then we compute the sum of squared distances along each dimension and take the square root. We can implement this entire computation as a `WHILE` loop over the dimensions:
    
    ```
    KDTreeNodeMinDist(KDTreeNode: node, Point: pt):
        Float: dist_sum = 0.0
        Integer: d = 0
        WHILE d < node.num_dimensions:
            Float: diff = 0.0
            IF pt[d] < node.x_min[d]:
                diff = node.x_min[d] - pt[d]
            IF pt[d] > node.x_max[d]:
                diff = pt[d] - node.x_max[d]
            dist_sum = dist_sum + diff * diff
            d = d + 1
        return sqrt(dist_sum)
    ```
    

Note that k-d trees can be more sensitive to additions and deletions than quadtrees. While both trees can become unbalanced due to the splitting rules and the distribution of points, k-d trees’ splits are chosen based on the data at the time. If we significantly change the distribution of points, the original split values may no longer be good ones. During bulk construction, we can adapt the splits to the data at hand, considering such factors as the depth of the tree, whether it is balanced, and the tightness of the nodes’ spatial bounds. This illustrates another tradeoff we see with data structures—the performance of a structure may degrade as the data changes.

## Why This Matters

The quadtree and k-d trees are examples of how we can combine the power of dynamic data structures with the spatial structure in our search problem. By branching along multiple dimensions at once, quadtrees allow us to adapt the resolution of a grid to the density of data points in a local area. High-density areas result in deeper branches and thus a finer-grained grid. At the same time, retaining a regular grid structure introduces new costs for higher dimensions. Examining how quadtrees, octtrees, and their variants scale across different data sets provides an important lens for how we think about utilizing spatial structure.

A k-d tree represents a combination of the concepts we have built up over the past few chapters to solve the problem of nearest-neighbor search. It solves the problem of having a high branching factor by returning to the core concepts of binary search trees and choosing a single dimension along which to split at each node. In addition, it allows more flexibility to adapt to the structure of the data, increasing pruning power.

Quadtrees and k-d trees aren’t the only data structures for facilitating nearest-neighbor searches. There are a wealth of other tree-based and non-tree-based approaches. The topic of spatial data structures could fill multiple books. As with almost everything in computer science, each approach comes with its own tradeoffs in terms of program complexity, computational cost, and memory usage. For the purpose of this book, quadtrees and k-d trees serve as excellent examples of how we can combine the spatial pruning of nearest-neighbor search with tree-based structures to allow the spatial trees to adapt to the data at hand.