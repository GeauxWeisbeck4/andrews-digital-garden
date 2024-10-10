---
id: 01J9QF6CZ2C6CPV8V11EGAQTKP
modified: 2024-10-08T21:44:55-04:00
---
In this chapter, we look at what happens as we consider multidimensional values and targets. The data structures we’ve examined so far have all shared a common constraint—they organize data based on a single value. Many real-world problems involve multiple important dimensions, and we need to extend our data structures to handle searches over such data.

This chapter starts by introducing nearest-neighbor search, which will serve as our motivating use case for multidimensional data. As we will see, the generality of nearest-neighbor search makes it very flexible and applicable to a wide range of spatial and non-spatial problems. It can help us find the cup of coffee closest to our current location or the brand best suited to our tastes.

We then introduce the grid data structure and show how it facilitates nearest-neighbor search over two dimensions, using spatial relationships within the data to prune out infeasible regions of the search space. We briefly discuss how these approaches can be extended to more than two dimensions. We will also see how these data structures fall short, providing the motivation for further spatial data structures.

## Introducing Nearest-Neighbor Search

As its name implies, _nearest-neighbor search_ consists of finding a particular data point closest to a given search target—for example, the coffee shop nearest our current location. Formally, we define nearest-neighbor search as follows:

> Given a set of _N_ data points _X_ = {_x_1, _x_2, … , _x_N}, a target value _x_’, and a distance function _dist_(_x_,_y_), find the point _x__i_ ∈ _X_ that minimizes _dist_(_x_’,_x__i_).

Nearest-neighbor search is closely related to the target value search we used to motivate binary search in Chapter 2. Both algorithms search for a specific data point within a set of data. The key difference lies in the success criteria. Whereas binary search tests for an exact match within a data set, which may or may not be present, nearest-neighbor search is only concerned with finding the closest match.

This framing makes nearest-neighbor search useful for many types of multiple-dimensional data. We could be searching a map for nearby coffee shops, a list of historical temperatures for days similar to the current date, or a list of “close” misspellings of a given word. As long as we can define a distance between the search target and other values, we can find nearest neighbors.

In past chapters, we primarily considered targets that are individual numeric values, like the data stored in binary search trees and heaps. While we sometimes included auxiliary data, the targets themselves remained simple. In contrast, nearest-neighbor search is most interesting when dealing with multidimensional data, which may be stored in a variety of other data structures such as arrays, tuples, or composite data structures. Later in this chapter, we look at example two-dimensional search problems and their targets. For now, though, let’s introduce a basic algorithm for this search.

### Nearest-Neighbor Search with Linear Scan

As a baseline algorithm for nearest-neighbor search, we start with a modified version of the linear scan algorithm from Chapter 2. The linear scan algorithm isn’t particularly exciting; you can implement it with a simple loop in most programming languages. Yet, because of its simplicity, linear scan provides a good starting point from which to examine more complex and efficient algorithms.

Consider the problem of nearest-neighbor search with numbers using the absolute distance: _dist_(_x_,_y_) = |_x_ – _y_|. Given a list of numbers and a search target, we want to find the closest number on the list. Perhaps we wake up in a new city and need to find our first cup of coffee in the morning. The hotel’s concierge provides a list of coffee shops on the same street, along with a helpful map. Not recognizing any of the businesses, we resolve to prioritize expedience and visit the coffee shop closest to the hotel.

We can visualize this search with a number line shown in [Figure 8-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-1). The points represent different coffee shops and their location with respect to the start of the map, while the X represents our hotel with a location of 2.2 miles along the street.

![A number line with seven candidate neighbors and one target point. The target point sits at 2.2 with the closest neighbor at 2.6.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/f08001.png)

Figure 8-1: A one-dimensional nearest-neighbor search represented as a number line

In a program, the points in [Figure 8-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-1) might represent unsorted values within an array. However, visualizing these values along a real-valued number line has two advantages in the context of nearest-neighbor search. First, it clarifies the importance of distance: we can see the gaps between our target value and each data point. Second, it helps us generalize the techniques beyond a single dimension, as we’ll see in the next section.

For now, the linear scan proceeds through each data point, as shown in [Figure 8-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-2), computing the distance for the current data point and comparing it to the minimum distance found so far. Here we consider the points in sorted order since they are already along the number line, but linear scan does not require a particular ordering. It uses the data’s ordering within the list.

![The linear scan computes each data point’s distance from the target point. A series of number lines shows each pair of points and their corresponding distance.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/f08002.png)

Figure 8-2: A linear scan through the data points in a one-dimensional nearest-neighbor search

In the first comparison in [Figure 8-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-2), we find a point at distance 1.8. This becomes our best option so far, our _candidate nearest neighbor_. It might not be a _good_ neighbor—1.8 miles is a bit far to walk for our morning cup of joe—but it’s the best we’ve seen. The next two steps discover better candidates at distances 1.2 and 0.4, respectively. Alas, the remaining four comparisons don’t produce a better candidate; the point at distance 0.4 remains the closest we’ve found. In the end, the algorithm returns that third point on our number line as the nearest neighbor. We head to the coffee shop, confident we’re heading to the closest one on the street.

[Listing 8-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#listing8-1) shows the code for a linear scan using an arbitrary distance function. We use floating-point values for the one-dimensional case but can extend the function to multiple dimensions by using composite data structures or other representations.

```
LinearScanClosestNeighbor(Array: A, Float: target, Function: dist):
    Integer: N = length(A)
  ❶ IF N == 0:
        return null

  ❷ Float: candidate = A[0]
    Float: closest_distance = dist(target, candidate)

    Integer: i = 1
  ❸ WHILE i < N:
        Float: current_distance = dist(target, A[i])
      ❹ IF current_distance < closest_distance:
            closest_distance = current_distance
            candidate = A[i]
        i = i + 1
  ❺ return candidate
```

Listing 8-1: The code for a linear scan nearest-neighbor algorithm

The code starts by checking whether the array is empty and, if so, returning `null` ❶, since there is no closest point. The code then picks the first item in the array as the initial candidate nearest neighbor and computes the distance from that point to the target ❷. This information provides a starting point for our search: we compare all future points against the best candidate and distance so far. The remainder of the code uses a `WHILE` loop to iterate over the remaining elements in the array ❸, computing the distance to the target and comparing that to the best distance found so far. The code updates the best candidate and best distance found whenever it finds a closer candidate ❹, then returns the closest neighbor ❺.

Beyond providing a simple implementation of nearest-neighbor search, the linear scan algorithm also trivially supports different distance functions or even higher-dimensional points. First, let’s look at some example problems in this two-dimensional space.

### Searching Spatial Data

Imagine that you are multiple hours into a cross-country road trip and desperately need a coffee refill. Panic floods your mind as you realize that you haven’t mapped out the optimal coffee shops along your route. You take a few deep breaths, pull out the map shown in [Figure 8-3](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-3), and locate numerous towns with known coffee establishments. Prioritizing expedience over quality, you vow to find the closest café.

![A two‐dimensional map of towns. The target point lies in the middle left near the towns of Gridville, Cartesian, and Fort Fortran.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/f08003.png)

Figure 8-3: A map as an example of two-dimensional data

The data consists of two-dimensional points—towns with _x_, _y_ coordinates. These data points can be stored as an ordered tuple (_x_, _y_); a small, fixed-size array [_x_, _y_]; or even a composite data structure for two-dimensional spatial points:

```
Point {
    Float: x
    Float: y
}
```

When determining which town is closest, we’ll focus on just the straight-line distance to the coffee shop. In any real-world navigation task, we’d also need to consider obstacles standing between us and our coffee. For now, though, let’s just consider the Euclidean distance to the coffee shops. If our current point is (_x_1_, y_1) and the coffee shop is at (_x_2_, y_2), then the distance is:

![G08001](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/G08001.png)

We could use the linear-scan algorithm in [Listing 8-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#listing8-1). The algorithm computes the distance from our target to each candidate point, as shown in [Figure 8-4](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-4).

![An illustration of the distance computations from our target point to each of the 11 towns on the map. The distances are represented by dashed lines from the target point to the town.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/f08004.png)

Figure 8-4: A linear scan nearest-neighbor search computes the distance from the target to each candidate point.

The point with the smallest distance to the target, shown in [Figure 8-5](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-5), is the target’s nearest neighbor. The dashed line represents the distance to the closest point, and the dotted circle shows the area of our map that is closer than (or equal to) the closest point. No other points lie closer than the nearest neighbor.

![The map with a dashed line from the target point to the nearest neighbor and a dotted circle indicating the space that falls within this radius.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/f08005.png)

Figure 8-5: The point with the smallest distance to the target is that target’s nearest neighbor.

As we’ve seen multiple times, though, this type of linear scan search quickly becomes inefficient as the number of points increases. If the current edition of the _Coffee Lover’s Guide to Roadside Coffee_ lists 100,000 coffee shops, it would be needlessly time-consuming to check each one.

We shouldn’t need to look at every single data point in two-dimensional space. Some points are too far away to matter. We would never consider Alaskan coffee shops when driving through Florida. This is not to disparage Alaskan cafés—I’m sure there are plenty that equal their Floridian peers in terms of taste and quality. It’s simply a matter of expedience. We can’t survive an hour without our coffee, let alone a multi-day drive. If we are driving through northern Florida, we need to focus on northern Floridian coffee establishments.

As we saw in binary search, we can often use structure within the data to help eliminate large numbers of candidates. We can even adapt binary search to find nearest neighbors in one-dimensional space. Unfortunately, a simple sort will not help in the two-dimensional case. If we sort and search either x or y dimensions, as shown in [Figure 8-6](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-6), we get the wrong answer—the closest neighbor in the one-dimensional space is not the same as the closest two-dimensional neighbor.

We need to use information from all relevant dimensions to make accurate pruning decisions. A point close to our target along a single dimension might be staggeringly far away in other dimensions. If we sort our list of coffee shops by latitude, our search for locations near our current latitude in northern Florida might return quite a few “close” results from Houston. Similarly, if we sort by longitude, we might be swamped with entries from Cleveland. We need to explore new approaches, adapted from our experience with one-dimensional data but also making use of the structure inherent in higher dimensions.

![On the left, a projection of the map points from Figure 8‐3 onto the Y‐axis. On the right, a projection of the points onto the X‐axis. In both cases, the closest neighbor in one dimension is not the same as the closest neighbor in two dimensions.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/f08006b.png)

Figure 8-6: Projecting the data to one dimension along either the y-axis (left) or x-axis (right) removes important spatial information about the other dimension.

## Grids

_Grids_ are data structures for storing two-dimensional data. Like arrays, grids consist of a fixed set of _bins_, or _cells_. Since we are initially covering two-dimensional data, we use a two-dimensional arrangement of bins and index each bin with two numbers, _xbin_ and _ybin_, representing the bin numbers along the x-axis and y-axis respectively. [Figure 8-7](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-7) shows an example grid.

![A two‐by‐two grid over the map. Bins are labeled 0 and 1 along both the X and Y dimension, resulting in a total of four quadrants.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/f08007.png)

Figure 8-7: A 2×2 grid of spatial data points

Unlike arrays, we can’t restrict each bin to hold a single value. Grid cells are defined by spatial bounds—a high and low edge along each dimension. No matter how finely we draw the grid, multiple points might fall within the same cell, so we need our bins to store multiple elements apiece. Each bin stores a list of _all_ data points that fall within that bin’s range.

We can visualize the difference between grids and arrays as different forms of refrigerator storage. The egg carton is an array with one individual space for each egg. In contrast, the vegetable drawer is like a grid bin. It contains multiple items of the same type, all vegetables. We might stuff a single drawer with twenty-five onions. The egg carton, on the other hand, contains only a fixed number of eggs, each in its designated place. While vegetable drawers may generate intense debates about where to correctly store tomatoes or cucumbers, the bounds of a grid cell are defined with mathematical precision.

Grids use the points’ coordinates to determine their storage, allowing us to use the data’s spatial structure to limit our searches. To see how this is possible, we first need to consider the details of the grid’s structure.

### Grid Structure

The top-level data structure for our grid contains some extra bookkeeping information. As shown in [Figure 8-8](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-8), we need to include multiple pieces of information along each dimension. In addition to the number of bins along the x- and y-dimensions, we must track the spatial bounds along each dimension. We use `x_start` and `x_end` to indicate the minimum and maximum values of x included in our grid. Similarly, `y_start` and `y_end` capture the spatial bounds for y.

![The grid spans x_start to x_end along the x‐axis and y_start to y_end along the y‐axis. the grid has 6 bins along each dimension, and the resulting bin widths are shown.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/f08008.png)

Figure 8-8: A grid with specified starting and ending values along each dimension

We can derive some top-level information from the number of bins and the spatial bounds, but we often want to store that additional information about the grid for convenience. Precomputing the width of the bins along each dimension simplifies later code:

```
x_bin_width = (x_end – x_start) / num_x_bins
y_bin_width = (y_end – y_start) / num_y_bins
```

Other useful information might include the total number of points stored in the grid or the number of empty bins. We can track all this information in a composite data structure. For two-dimensional data, our typical data structure would look like this:

```
Grid {
    Integer: num_x_bins
    Integer: num_y_bins
    Float: x_start
    Float: x_end
    Float: x_bin_width
    Float: y_start
    Float: y_end
    Float: y_bin_width
    Matrix of GridPoints: bins
}
```

For a fixed-size grid, we can map from a point’s spatial coordinates to the grid’s bin using a simple mathematical computation:

```
xbin = Floor((x – x_start) / x_bin_width)
ybin = Floor((y – y_start) / y_bin_width)
```

The switch from “one bin, one value” to “spatial partitioning” has important consequences beyond index mapping. It means that we can no longer store the data as a set of fixed bins in the computer’s memory. Each square could contain an arbitrary number of data points. Each grid square needs its own internal data structure to store its points. One common and effective data structure for storing points within the bin is a linked list like the one in [Figure 8-9](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-9).

![A grid shown as a list of four bins (a flattened matrix), each pointing to the start of a linked list of points. The first three bins contain three points, and the final bin (xbin = 1, ybin = 1) includes two points.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/f08009.png)

Figure 8-9: A representation of the data structure used to store points in a grid

Each bin stores the pointer to the head of a linked list, which contains all the points in that bin. We accomplish this with another, internal data structure to store individual points:

```
GridPoint {
    Float: x
    Float: y
    GridPoint: next
}
```

Alternatively, we could use the `LinkedListNode` data structure from Chapter 3 and store a pair to represent the _x_, _y_ coordinates.

### Building Grids and Inserting Points

We construct a grid for our data set by allocating an empty grid data structure and iteratively inserting points using a single `FOR` loop over the data points. The high-level structure of the grid itself (the spatial bounds and number of bins along each dimension) is fixed at time of creation and does not change with the data added.

As shown in [Listing 8-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#listing8-2), inserting a point consists of finding the correct bin and prepending the new point to the beginning of the linked list for that bin.

```
GridInsert(Grid: g, Float: x, Float: y):
  ❶ Integer: xbin = Floor((x - g.x_start) / g.x_bin_width)
    Integer: ybin = Floor((y - g.y_start) / g.y_bin_width)
 
    # Check that the point is within the grid.
  ❷ IF xbin < 0 OR xbin >= g.num_x_bins:
        return False
    IF ybin < 0 OR ybin >= g.num_y_bins:
        return False
    
    # Add the point to the front of the list. 
  ❸ GridPoint: next_point = g.bins[xbin][ybin]
    g.bins[xbin][ybin] = GridPoint(x, y)
    g.bins[xbin][ybin].next = next_point

  ❹ return True
```

Listing 8-2: A function to insert a new point into a grid

The code first computes the x and y bins for the new point ❶ and confirms that the new point falls within a valid bin ❷. While it’s always important to confirm that you are accessing an in-bounds array index whenever using an array, spatial data structures present additional concerns. We might not be able to predefine a fixed, finite grid that works for every possible future point. Therefore, it is important to consider what happens when previously unseen points fall outside the range covered by your spatial data structure. In this example, we return a Boolean to indicate whether or not the point could be inserted ❹. However, you might prefer other mechanisms, such as throwing an exception, depending on the programming language.

Once we have determined that point does fit within our grid, the code finds the appropriate bin. The code prepends the new point to the front of the list, creating a new list if the bin was previously empty ❸. The function concludes by returning `True` ❹.

### Deleting Points

We can use a similar approach to insertion for deleting points from a grid. One additional difficulty is determining which point in the bin to delete. In many use cases, the user might insert arbitrarily close or even duplicate points into the grid. For example, if we are storing a list of ground coffees available to purchase, we might insert multiple points for a single coffee shop. Ideally, we use other identifying information, such as the name or ID number of the coffee, to determine which of the points to delete. In this section, we present the simple and general approach of deleting the first matching point in our linked list.

Due to the limited precision of floating-point variables, we also might not be able to use a direct equality test. In [Listing 8-3](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#listing8-3), we use a helper function to find a point that is close enough. The `approx_equal` function returns `True` if both points are within a threshold distance in both dimensions.

```
approx_equal(Float: x1, Float: y1, Float: x2, Float: y2):
    IF abs(x1 – x2) > threshold:
        return False
    IF abs(y1 – y2) > threshold:
        return False
    return True
```

Listing 8-3: Code to check whether two data points, represented as a pair of floating-point numbers, are equal

The code checks each dimension independently and compares the distance to a threshold. The threshold will depend on the use case and the numerical precision of your programming language. Generally, we want these thresholds to be just large enough to account for the float’s numerical precision.

Deletion consists of finding the correct bin, traversing the linked list until we find a match, and removing the match by splicing it out of the list. Our delete function returns `True` if a point was found and deleted and `False` otherwise.

```
GridDelete(Grid: g, Float: x, Float: y):
  ❶ Integer: xbin = Floor((x - g.x_start) / g.x_bin_width)
    Integer: ybin = Floor((y - g.y_start) / g.y_bin_width)
 
    # Check that the point is within the grid.
  ❷ IF xbin < 0 OR xbin >= g.num_x_bins:
        return False
    IF ybin < 0 OR ybin >= g.num_y_bins:
        return False
     
    # Check if the bin is empty.
  ❸ IF g.bins[xbin][ybin] == null:
        return False

    # Find the first matching point and remove it.
  ❹ GridPoint: current = g.bins[xbin][ybin]
    GridPoint: previous = null
    WHILE current != null:
      ❺ IF approx_equal(x, y, current.x, current.y):
          ❻ IF previous == null:
                g.bins[xbin][ybin] = current.next
            ELSE:
                previous.next = current.next
            return True
      ❼ previous = current
        current = current.next
    return False
```

The code first computes the x and y bins for the new point ❶ and confirms that the new point falls within a valid bin ❷. Next it checks whether the target bin is empty ❸, returning `False` if it is.

## NOTE

While the code above checks a single bin for simplicity, we could theoretically see (extremely rare) edge cases where the deleted point lies right on the bin’s boundary. We could further account for the limited precision of floats in that case with additional checks.

If there are points to check, the code iterates through the list ❹. Unlike the code for insertion, we track both the current node and the previous one so that we can splice the target node out of the list. The code uses the `approx_equal` helper function from [Listing 8-3](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#listing8-3) to test each point ❺. When it finds a matching point, it splices it out of the list, taking care to correctly handle the special case of the first node in the list ❻, and returns `True`. Thus, only the _first_ matching point in the list is removed. If the current point does not match, the search continues to the next node in the list ❼. If the search finishes the entire list, the function returns `False` to indicate that no matching node was deleted.

## Searches Over Grids

Now that we’ve learned how to construct grid data structures, let’s use them to improve our nearest-neighbor searches. First, we examine the problem of pruning grid cells that are too far away, which will allow us to avoid unnecessary computations within grid cells. We then consider two basic searches: a linear scan over all the bins and an expanding search.

### Pruning Bins

The grid’s spatial structure allows us to limit how many points we need to check, excluding those outside the range we’re interested in (northern Florida). Once we have a candidate neighbor and its associated distance, we can use that distance to _prune bins_. Before checking the points in a bin, we ask whether _any_ point within the bin’s spatial bounds could be closer than the current best distance. If not, we can ignore the bin.

Determining whether _any_ point within a bin lies within a given distance from our target point may sound like a daunting task. However, if we are using Euclidean distance ![i08001](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/i08001.png), which we can encapsulate in this simple helper function:

```
euclidean_dist(Float: x1, Float: y1, Float: x2, Float: y2):
    return sqrt((x1-x2)*(x1-x2) + (y1-y2)*(y1-y2))
```

then the test boils down to simple mathematics. We start by finding the closest possible point in the grid cell and use that for our pruning test. Specifically, if the closest possible point in the grid cell is further than our current best candidate, there is no reason to check any of the actual points stored in the bin. They all must be further away. If a target point falls within the cell—that is, if its x and y values are within the cell’s x and y ranges, respectively—the distance to the cell (and thus the closest possible point) is zero.

If the point falls outside the cell, then the closest possible point in the cell must lie on the edge of the cell. [Figure 8-10](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-10) shows a variety of points outside the grid cell and the corresponding closest points within the cell. For points outside the grid cell, we need to compute the distance to the closest edge point.

![Eight target points shown as gray circles and their corresponding closest points within the cell. The closest points to targets outside the cell are on the cell’s perimeter.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/f08010.png)

Figure 8-10: Points outside a grid cell (gray circles) and the corresponding closest points within the cell (solid circles)

We can compute the Euclidean distance between a point and the nearest edge of a grid’s bin by considering each dimension independently. We find the minimum distance needed to shift the x value within the bin’s range and the minimum distance to shift the y value within the bin’s range. For the grid bin (_xbin_, _ybin_), the minimum and maximum x and y dimensions are:

```
x_min = x_start + xbin * x_bin_width
x_max = x_start + (xbin + 1) * x_bin_width
y_min = y_start + ybin * y_bin_width
y_max = y_start + (ybin + 1) * y_bin_width
```

We can compute the distance as follows (in the case of Euclidean distance):

![g08002](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/g08002.png)

where

IF _x_ < _x___min_ THEN _x_dist _= x___min_ − _x_

IF _x___min_ ≤ _x_ ≤ _x___max_ THEN _x_dist = 0

IF _x_ > _x___max_ THEN _x_dist = _x_ – _x___max_

and

IF _y_ < _y_min_ THEN _y_dist = _y_min_ − _y_

IF _y_min_ ≤ _y_ ≤ _y_max_ THEN _y_dist = 0

IF _y_ > _y_max_ THEN _y_dist = _y_ − _y_max_

If the minimum distance to any possible point in the bin is greater than that of our current closest point, then nothing in the bin could replace the current closest point. We can ignore the entire bin!

The code for computing the minimum distance from a point to a bin can be encapsulated into the following helper function. This function implements the mathematical logic above.

```
MinDistToBin(Grid: g, Integer: xbin, Integer: ybin, Float: x, Float: y):
    # Check that the bin is valid.
  ❶ IF xbin < 0 OR xbin >= g.num_x_bins:
        return Inf
    IF ybin < 0 OR ybin >= g.num_y_bins:
        return Inf

  ❷ Float: x_min = g.x_start + xbin * g.x_bin_width
    Float: x_max = g.x_start + (xbin + 1) * g.x_bin_width
    Float: x_dist = 0
    IF x < x_min:
      x_dist = x_min - x
    IF x > x_max:
      x_dist = x - x_max

  ❸ Float: y_min = g.y_start + ybin * g.y_bin_width
    Float: y_max = g.y_start + (ybin + 1) * g.y_bin_width
    Float: y_dist = 0
    IF y < y_min:
      y_dist = y_min - y
    IF y > y_max:
      y_dist = y - y_max
    return sqrt(x_dist*x_dist + y_dist*y_dist)
```

Listing 8-4: A helper function that computes the closest distance from a target point to a given bin

The code starts by checking that the bin indices are valid ❶. In this example, we use an infinite distance to indicate that the function’s caller has referenced an invalid bin. This logic allows us to use this lookup function in pruning computations that might ask about invalid bins. However, this may lead to confusion: Why is the function returning any distance for an invalid bin? Depending on the usage, it might be preferable to throw an error indicating that the bin indices are invalid. Either way, the function’s behavior should be clearly documented for users.

The remainder of the code proceeds through the distance logic above for the x and y dimensions (❷ and ❸, respectively). The code computes the minimum and maximum values for the bin, compares them with the point’s value along that dimension, and computes the distance.

To visualize this distance test, imagine that a raucous game of catch sends a ball over our fence into the yard of our friendly, but exceedingly lazy, neighbor. They will return the ball, of course, but without exerting any more effort than absolutely necessary. What is the shortest distance they need to throw the ball in order for it to (just barely) return to our yard? If their longitude already falls within the bounds of our yard, they will throw in a pure north or south direction so as not to add any unnecessary east/west distance. In the end, their throw always lands exactly on the fence such that it falls back into our property. Our neighbor may be lazy, but they have some impressive throwing skills.

### Linear Scan Over Bins

The simplest approach to searching a grid would iterate through all the grid’s bins using a linear scan and only check those that could contain a potential nearest neighbor. This is not a particularly good algorithm, but it provides an easy introduction to working with and pruning bins.

The linear search algorithm simply applies the aforementioned minimum distance test to each bin before checking its contents:

```
GridLinearScanNN(Grid: g, Float: x, Float: y): 
  ❶ Float: best_dist = Inf
    GridPoint: best_candidate = null

    Integer: xbin = 0
  ❷ WHILE xbin < g.num_x_bins:
        Integer: ybin = 0
        WHILE ybin < g.num_y_bins:

            # Check if we need to process the bin.
          ❸ IF MinDistToBin(g, xbin, ybin, x, y) < best_dist:

                # Check every point in the bin's linked list.
                GridPoint: current = g.bins[xbin][ybin]
              ❹ WHILE current != null:
                    Float: dist = euclidean_dist(x, y, current.x, current.y)
                  ❺ IF dist < best_dist:
                        best_dist = dist
                        best_candidate = current
                    current = current.next
            ybin = ybin + 1
        xbin = xbin + 1
  ❻ return best_candidate
```

Listing 8-5: A nearest-neighbor search that uses a linear scan over a grids bin with pruning tests on each bin.

The code starts by setting the best distance to infinity to indicate that no best point has been found so far ❶. Then the algorithm scans through each bin using a pair of nested `WHILE` loops that iterate over the x and y bins ❷. Before checking the individual points in the bin, the code performs a minimum distance test to check whether _any_ point in the bin could be a better neighbor ❸. If the bin may contain better neighbors, the code uses a third `WHILE` loop to iterate through the linked list of points in the bin ❹. It tests the distance to each point and compares it with the best distance found so far ❺. The function completes by returning the best candidate found, which may be `null` if the grid is empty ❻.

The algorithm in [Listing 8-5](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#listing8-5) allows us to prune out entire bins, along with all the points they contain, whenever we determine that minimum distance to any point in the bin is greater than the distance to the best point seen so far. If we have a large number of points per bin, this can lead to significant savings. However, if the grid is sparsely populated, we might end up paying more to check each of the bins than we would have if we’d checked each point individually.

Unlike the `GridInsert` function in [Listing 8-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#listing8-2), our linear scan works with target points inside or outside the grid’s spatial bounds. `GridLinearScanNN` does not need to map the target point to a bin and therefore does not care if the target is on the grid itself. It will still return the nearest neighbor from the grid (or `null` if the grid is empty). This provides an additional level of flexibility to our nearest-neighbor search that can be useful when encountering new, non-typical targets.

### Ideal Expanding Search over Bins

While the linear scan algorithm allows us to prune out entire bins based on their minimum distance to our target point, we’re still not using all the spatial information at our disposal. We waste a significant amount of computation by testing bins that are far from our target point. We can do better if we prioritize bins by their proximity to their target, first searching the bins closest to our target point and halting the search when the remaining bins are further than the nearest neighbor we have found so far. We call this an _expanding search_, because we effectively expand out from the bin containing the target point until we have found the nearest neighbor.

To visualize this improved scanning method, imagine a panicked search for our car keys in the morning. We start with the area (comparable to a grid cell) where the car keys would be if we had stored them correctly. We inspect every inch of the kitchen counter before admitting that we must have misplaced the keys. Spiraling out to other parts of the house (that is, other bins), we check nearby locations, such as the coffee table and the floor, before venturing further and further away. This search continues, exploring less and less likely locations, until we find the keys mysteriously sitting in the sock drawer.

For an example of an expanding scan in action, consider our map overlaid with a four-by-four grid, as shown in [Figure 8-11](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-11). We find the closest bin to our target point by asking, “Into which bin does our target point fall?” and using the grid index-mapping equations. Since it is possible that our target point falls outside the grid, we might also need to shift the computed bin indices into the valid range. In [Figure 8-11](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-11), the target point is in the third bin up in the leftmost column (_xbin_ = 0 and _ybin_ = 2 in our notation).

![The map points from figure 8‐3 placed in a four‐by‐four grid.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/f08011.png)

Figure 8-11: A 4×4 grid of two-dimensional points

We can start our search in the target point’s bin and test every point in that bin. As long as the bin isn’t empty, we are guaranteed to find our first _candidate_ nearest neighbor, as shown in [Figure 8-12](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-12). Unfortunately, since we’re not organizing or sorting the points within each bin, we can’t do better in this case than a linear scan through that bin’s points. Of course, if the initial bin is empty, we must progress our search outward to neighboring bins, incrementally trying further and further bins until we find one containing a data point to be our candidate nearest neighbor.

![The four‐by‐four grid of map points from Figure 8‐11 with a dashed line to the candidate nearest neighbor. The first candidate falls within the same bin.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/f08012.png)

Figure 8-12: An initial candidate nearest neighbor found in the same bin as the target point

Once we obtain this initial candidate for nearest neighbor, we are still not done. The candidate is just that—a candidate. It’s possible there could be a closer point in one of the adjacent bins. This is more likely if our target point is near the edge of a bin. In [Figure 8-13](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-13), the dashed circle represents the space of all points that are closer to or at the same distance from the current candidate. Any other point that falls within the circle could be the true nearest neighbor. The shaded grid cells are those that overlap this region.

![The four‐by‐four grid of map points from Figure 8‐11 with a dotted circle showing the region of space with distance closer than or equal to the current candidate neighbor. Four grid cells overlapping this circle are shaded, indicating that they could contain a closer neighbor.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/f08013.png)

Figure 8-13: A candidate nearest neighbor and the gridcells that could contain points closer to the target

To visualize why we need to continue to check other bins, imagine you want to determine the closest person to you at an outdoor party. You’re telling a particularly embarrassing story, involving the use of spoiled milk in your coffee, and want to make sure that only the intended audience hears you. Your best friend standing by the house might appear to be closest to you, but, if you’re near your fence, you also need to consider people on the other side. If your neighbor is planting flowers along their side of the fence, they might actually be closer and hear all of the humiliating details. You can’t discount them because there is a fence in the way. This is why we always check neighboring bins—and why you should always be careful about telling embarrassing stories in public.

We continue to expand out the search to include _all_ nearby bins until we can guarantee that no possible point in the remaining bins could be closer than our candidate nearest neighbor. Once we have checked all the bins within the radius of our nearest-neighbor candidate, we can ignore any bins beyond that. We don’t even need to check their distance.

The tradeoff for this improved grid search is algorithmic complexity. Instead of scanning across every one of the bins—an algorithm we could implement with a nested pair of `FOR` loops—the optimized search spirals out from a single bin, exploring further and further away until we can prove that none of the unexplored bins could contain a better neighbor. This requires additional logic in the search order (outward spiral), bounds checking (avoiding testing bins off the edge of the grid), and termination criteria (knowing when to stop). The next section presents a simple example of an expanding search for illustrative purposes.

### Simplified Expanding Search

Let’s consider a simplified (and non-optimized) version of an expanding search that moves outward in a diamond-shaped pattern. Instead of executing a perfect spiral out from the initial bin, the search uses an increasing distance from an initial bin. For simplicity of implementation, we will use a Manhattan distance on the grid indices that counts the steps between grid cells:

_d_ = |_xbin_1 − _xbin_2| + |_ybin_1 − _ybin_2|

While this search pattern is unlikely to be efficient for grids with drastically different bin widths along each dimension, it provides an easy-to-follow illustration.

[Figure 8-14](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-14) shows the first four iterations of the search. During the first iteration in [Figure 8-14](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-14)(a), we search the bin containing the target point (zero steps away). During the next iteration in [Figure 8-14](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-14)(b), we search all bins a single step away. On each subsequent iteration, we search all the bins that are one step further out.

![Four iterations of an example expanding search. In the first iteration, a single bin containing the target point is shaded. In the second iteration, four bins, all one step away, are shaded. In the third iteration, eight bins, all two steps away, are shaded.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/f08014.png)

Figure 8-14: The first four iterations of a simplified expanding search on a grid

We start with a helper function that checks whether any points within a specified bin are closer to our target point (`x`, `y`) than a given `threshold`. This function encodes our linear scan through the bin’s points. If there is at least one point closer than `threshold`, the function returns the closest such point. The use of a threshold value will allow us to use the helper function to compare the bin’s points to the best candidates from other bins.

```
GridCheckBin(Grid: g, Integer: xbin, Integer: ybin, 
             Float: x, Float: y, Float: threshold):
    # Check that it is a valid bin and within the pruning threshold.
  ❶ IF xbin < 0 OR xbin >= g.num_x_bins:
        return null
    IF ybin < 0 OR ybin >= g.num_y_bins:
        return null

    # Check each of the points in the bin one by one.
    GridPoint: best_candidate = null
  ❷ Float: best_dist = threshold
    GridPoint: current = g.bins[xbin][ybin]
  ❸ WHILE current != null:
      ❹ Float: dist = euclidean_dist(x, y, current.x, current.y)
        IF dist < best_dist:
            best_dist = dist
            best_candidate = current
        current = current.next
  ❺ return best_candidate
```

Listing 8-6: A helper function that returns the closest point in a bin to the target point as long as it is below the given threshold

The code starts with a safety check that we are accessing a valid bin ❶. If not, it returns `null` to indicate that there are no valid points. The code then uses a `WHILE` loop to iterate through each point in the bin ❸, computing its distance from the target point, comparing it to the best distance seen so far, and saving it as the new best candidate if it is closer ❹. The code finishes by returning the closest point ❺. Since the code initially set `best_dist` to the `threshold` value before checking any points ❷, it will only mark points with a distance less than `threshold` as new candidates. The function returns `null` if none of the bin’s points are closer than `threshold`.

The code for performing the expanding search works by iterating through a different number of steps and checking all bins that can be reached in that number of steps. As in previous searches, we track the best candidate seen so far. The search concludes after iteration _d_ if there are no valid grid cells _d_ steps away that could contain a closer neighbor.

```
GridSearchExpanding(Grid: g, Float: x, Float: y):
    Float: best_d = Inf
    GridPoint: best_pt = null

  ❶ # Find the starting x and y bins for our search.
    Integer: xb = Floor((x - g.x_start) / g.x_bin_width)
    IF xb < 0:
        xb = 0
    IF xb >= g.num_x_bins:
        xb = g.num_x_bins - 1

    Integer: yb = Floor((y - g.y_start) / g.y_bin_width)
    IF yb < 0:
        yb = 0
    IF yb >= g.num_y_bins:
        yb = g.num_y_bins - 1

    Integer: steps = 0
    Boolean: explore = True
  ❷ WHILE explore:
        explore = False

      ❸ Integer: xoff = -steps
        WHILE xoff <= steps:
          ❹ Integer: yoff = steps - abs(xoff)
          ❺ IF MinDistToBin(g, xb + xoff, yb - yoff, x, y) < best_d:
              ❻ GridPoint: pt = GridCheckBin(g, xb + xoff, yb - yoff, 
                                             x, y, best_d)
                IF pt != null:
                    best_d = euclidean_dist(x, y, pt.x, pt.y)
                    best_pt = pt
              ❼ explore = True

          ❽ IF (MinDistToBin(g, xb + xoff, yb + yoff, x, y) < best_d
                AND yoff != 0):
                GridPoint: pt = GridCheckBin(g, xb + xoff, yb + yoff, 
                                             x, y, best_d)
                IF pt != null:
                    best_d = euclidean_dist(x, y, pt.x, pt.y)
                    best_pt = pt
              ❾ explore = True

            xoff = xoff + 1
        steps = steps + 1
    return best_pt
```

This code starts by finding the closest bin within the grid to our target point, taking care to map targets outside the grid to their closest bin in the grid ❶. The resulting bin (`xb`, `yb`) will be the starting point for the search. By mapping bins outside the grid to a valid bin, the function can return the nearest neighbor for target points that lie outside the grid itself.

The code then uses a `WHILE` loop to explore outward from this initial bin by increasing amounts ❷. The variable `steps` tracks the distance used for the current iteration. The `WHILE` loop is conditioned on the variable `explore`, which indicates that the next iteration may include a valid bin and we should thus continue exploring at the next value of `steps`. As we will see shortly, the `WHILE` loop terminates as soon as it completes a full iteration where _none_ of the bins visited could have held a closer neighbor.

Within the main `WHILE` loop, the code iterates across the different x-index offsets from `-steps` to `steps` as though scanning horizontally across the grid ❸. The total number of steps in the x-direction and y-direction are fixed by `steps`, so the code can programmatically compute the remaining number of steps to use in the (positive or negative) y-direction ❹. Starting with the negative y-direction, the code uses `MinDistToBin` from [Listing 8-4](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#listing8-4) to check whether the bin indices are valid and, if so, determine the distance to that bin ❺. It can skip any bins that are invalid or too far away. If the bin could contain a closer point than our current candidate, the code uses `GridCheckBin` from [Listing 8-6](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#listing8-6) to check for such a point ❻. Whenever a closer point has been found, the code saves it as the new closest candidate and updates its estimate of the closest distance. The second `IF` block performs the same checks in the positive y-direction as long as the y-offset is not zero (in which case we have already checked the bin in the negative y-direction) ❽.

During an iteration of the outer `WHILE` loop ❷, the code resets `explore` to `False`. It later updates `explore` to `True` if any of the calls to `MinDistToBin` indicate that a bin could contain a closer neighbor (❼ and ❾). Thus, the outer loop continues until it reaches a number of steps where every bin is either further than `best_d` or lies off the grid (and is therefore invalid). While other termination criteria may provide more exact tests and terminate earlier, we use this rule in the code due to its simplicity.

## The Importance of Grid Size

The size of our grid’s bins has a massive impact on the efficiency of our search. The larger our bins, the more points we may need to check per bin. Remember that our grid searches still do a linear scan through the points within each visited bin. However, partitioning the grid into finer bins has tradeoffs both in terms of memory and the number of empty bins we may encounter. As we shrink the size of the grid’s bins, we often need to search more individual bins before we even find the first candidate nearest neighbor and the cost of checking bins increases.

[Figure 8-15](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-15) shows an extreme case where the grid is too fine.

![The map points from figure 8‐3 placed in a 17‐by‐17 grid. There are now 36 bins that are closer than the nearest neighbor.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/f08015.png)

Figure 8-15: A fine grid in which most of the bins are empty

In [Figure 8-15](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-15), we must search 36 bins in order to find the nearest neighbor. This is clearly more expensive than the example in [Figure 8-13](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-13), where we only needed to check four bins and two individual points. Sadly, it might even be more expensive than the linear scan search, which checked every one of the 11 data points.

Consider this in the context of our search for coffee shops. If we partition the space too finely, such as in 1 m by 1 m squares, we’ll be facing a grid that contains mostly empty bins. If we partition the space more coarsely, such as 5 km by 5 km squares, we might bucket entire cities and their multitudes of coffee shops in a single bin while still leaving (to our utmost horror) a large number of bins nearly or completely empty.

The optimal grid size often depends on multiple factors, including the number of points and their distribution. More complex techniques, such as non-uniform grids, can be used to dynamically adapt to the data. In the next chapter, we will consider several tree-based data structures that dynamically enable this type of adaptation.

## Beyond Two Dimensions

The grid-based techniques developed for two dimensions can be scaled to higher dimensional data as well. We might need to search a multi-floor office building for the closest available conference room. We can search for nearest neighbors in three-dimensional data by incorporating the _z_ coordinate into our distance computation:

![g08003](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/g08003.png)

Or, more generally, we can define Euclidean distance over _d_-dimensional data as:

![g08004](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/g08004.png)

where _x_i[_d_] is the _d_th dimension of the _i_th data point.

Higher-dimensional data comes with another challenge for the grid-based approach we’ve considered in this chapter: it requires us to partition the space along more dimensions. The space required to store such data structures explodes quickly as we consider higher dimensions. For data with _D_ dimensions and _K_ bins per dimension, we need _K__D_ individual bins! This can require a huge amount of memory. [Figure 8-16](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-16) shows a three-dimensional example, a 5×5×5 grid, which already includes a large number of individual bins.

Worse, as we increase the number of grid bins, we’re likely increasing the percentage of empty bins. Checking those empty bins is wasted work. For this reason, grids aren’t ideal for higher-dimensional problems. In the next chapter, we will introduce a better approach for scaling to higher-dimensional data—the k-d tree.

While it is difficult to think of an everyday spatial problem using more than three dimensions, we can use our nearest-neighbor formulation on data beyond spatial points. In the next section, we will see how nearest-neighbor search can be used to help us find similar coffee or days with similar weather conditions.

![A five‐by‐five‐by‐five grid for three‐dimensional points.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/f08016.png)

Figure 8-16: A grid of three-dimensional points

## Beyond Spatial Data

Spatial data, such as locations on a map, provides a simple visual example for both nearest-neighbor search and grids themselves. We’re accustomed to thinking about locations in terms of proximity, since we regularly ask ourselves questions like “What is the closest gas station?” or “Where is the closest hotel to the conference center?” Yet the nearest-neighbor problem extends beyond spatial data.

Let’s consider the critical problem of selecting the next-best brand of coffee to purchase when our favorite brand is out of stock. To find something similar to our favorite brew, we might consider what attributes we liked about that coffee, such as strength or acidity level, and then look for other coffees with similar attributes. We can extend nearest-neighbor search to find these “close” coffees. To do so, we first record every coffee we have ever sampled in a coffee log, noting such properties as strength and acidity, as shown in [Figure 8-17](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c08.xhtml#figure8-17).

Over the years, we build a comprehensive mapping of the coffee landscape. Performing a nearest-neighbor search on this data allows us to find varieties of coffee similar to a target value. Looking for a strong, low-acidity brew to fuel the hurried work before a tight deadline? We can picture exactly the coffee we want, that sublime brew we had once in Hawai’i. Unfortunately, the upcoming deadline does not provide enough slack to justify a quick trip to Hawai’i. But have no fear! We use our thorough analysis of the coffee’s attributes, as captured in our coffee log, to define a search target and then search for a local brand that might be similar enough.

![On the left is a two‐dimensional plot of data points with one axis labeled strength and the other acidity. On the right is the same points with a grid overlaid.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/f08017b.png)

Figure 8-17: An example of coffee attributes as two-dimensional data (left) and those points in a grid (right)

To perform this search, we just need a way to compute distances for attributes like coffee strength or acidity. The nearest-neighbor algorithm relies on our ability to distinguish “near” versus “far” neighbors. While it is possible to define distance measures over other types, such as strings, we restrict our discussion in this chapter to real-valued attributes for consistency.

With spatial data points, we have simple standard measures of the distances between two points (_x_1_, y_1) and (_x_2_, y_2), such as the Euclidean distance used earlier_._ But the optimal distance measure for any problem will depend on the problem itself. When evaluating brands of coffee, we might want to weight the attributes differently in different situations. Before our impending deadline, caffeine content takes precedence over factors such as the acidity level.

One common distance measure for non-spatial data is weighted Euclidean distance:

![g08005](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c08/g08005.png)

where _x__i_[_d_] is the _d_th dimension of the _i_th data point and _w__d_ is the weighting for the _d_th dimension. This formulation allows us to weight the impact of the different dimensions. In this case, we might set the weight of caffeine content to twice that of acidity, skewing the search toward coffees that are similarly caffeinated. We can even vary the weights per search.

Of course, our search makes no guarantees as to the suitability of other aspects of the coffee. We’re only measuring proximity along the specified dimensions. If we are searching for an everyday coffee by matching only strength and acidity, then we do not consider roast level, batch size, growing conditions, caffeine content, or even the concentration of nutrients in the soil. If the nearest neighbor turns out to be decaf, our search wouldn’t account for this travesty. We’d be left with substandard coffee and tears of disappointment. It is important to make sure your distance computation takes into consideration all the dimensions of interest.

## Why This Matters

Nearest-neighbor search allows us to find points that are “close” to some target value, whether spatial or non-spatial. From an algorithmic point of view, nearest-neighbor search moves us from searching for an exact target to searching based on distance metrics. The details of search get more complex as we step away from one-dimensional data sets into the realm of multidimensional data. As we saw with the shift from arrays to grids, this extension opens a range of new questions in terms of how we organize and search the data. It’s no longer possible to consider a simple ordering, as we did with a binary search for one-dimensional data. We need to adapt our data structures to a new type of multidimensional structure. Grids provide a new way to structure data based on aggregating points within the same spatial regions into the same bin.

At the same time, grids illustrate a different structure than the one-bucket, one-value structure we have seen with arrays. Grids use linked-list or other internal data structures to store multiple values per bin, a technique we will reuse in future chapters. By using this structure, grids also introduce a new tradeoff to consider—the size of the bins. By increasing the size of the bins, we can shift cost from evaluating many small bins to scanning through a large number of points per bin. Choosing the right number of bins is an example of the common task of _tuning_ our data structure for the specific problem at hand.

In the next chapter, we’ll take spatial partitioning further by combining the adaptive properties of trees with the spatial properties of grids. In doing so, we’ll address some of the major drawbacks of grids—and make the search for a good cup of coffee significantly more efficient.