---
id: 01J9QFDPDC6JG6CH1TF4J4ZAJY
modified: 2024-10-08T21:49:46-04:00
---
Throughout this book, we examined a range of different data structures, how they impact the algorithms that use them, and whether they can aid us in our search for coffee. We showed how the organization of the data can lead to significant decreases in computational cost or changes to an algorithm’s behavior. We looked at the tradeoffs between different representations and why they matter. In doing so, we tried to provide an intuitive foundation for how to think about data structures.

Understanding each data structure’s motivation, construction, uses, and tradeoffs is critical in order to use them in the development of efficient solutions. If you randomly choose a data structure that looks “good enough,” you can run into worst-case scenarios and terrible performance. Below we revisit some core themes from the preceding chapters in order to highlight some of the questions that every computer science practitioner should ask when selecting a data structure.

## What Is the Impact of the Data’s Structure?

Starting with binary search in Chapter 2, we saw how adding even a small amount of structure to the data can greatly impact the efficiency of algorithms. Structure within the data allows us to efficiently access values, aggregate computations, or prune out regions of the search space. As in the case of binary search, this structure can be as simple as putting the data in sorted order. This single change allows us to cut the worst-case runtime, changing its relationship to the number of values from linear to logarithmic. Similarly, organizing our coffee pantry can optimize our coffee making experience in different ways—most often in reducing the time needed to make our first cup.

Binary search trees, tries, quadtrees, and k-d trees showed us how to further facilitate pruning during a search. Tree-based data structures provide an explicit branching organization that allows us to prune out large regions of the search space with simple tests. We encode the bounds of the data into the structure of the trees and the nodes themselves. Further, the branching nature of the data allows us to clearly visualize the question we ask at each level: “Given bounds on the points lying below this node in a tree, could the point of interest be in this subtree?”

Even if we don’t actively optimize the data’s organization for the algorithm at hand, its arrangement can profoundly impact the behavior and efficiency of our algorithms, as stacks and queues demonstrate. Switching from a stack to a queue, for example, moves a search from depth-first to breadth-first. In the extreme case, the structure of the data requires the development of completely new algorithmic approaches: graphs’ connection-based structure drives a range of new algorithms to search, sort, and perform other computations.

## Do We Need Dynamic Data Structures?

Dynamic data structures dramatically increase the flexibility and adaptability of our approaches. Using these structures means we’re no longer constrained to preallocating blocks of memory that may turn out to be too small for the task at hand. Instead, we can link locations throughout the memory with pointers, allowing our data structures to grow and shrink as necessary. Most importantly, dynamic data structures allow us to continually grow our coffee log and store multiple coffee shop locations in our geographic grid cells.

Dynamic data structures provide the foundation for some of the most exciting, interesting, and powerful algorithms in computer science. Almost every data structure described in this book takes advantage of pointers (or a related linkage) to organize the data across different blocks of memory. We used pointers to link the nodes in a binary search tree, create linked lists in both our grid cells and hash table bins, and represent the very structure of graphs.

The tradeoff for this power and flexibility is additional complexity when accessing the data. In an array, we can look up any item given its index. However, this direct approach doesn’t work once pointers are involved. Our search for a particular piece of data must follow the chain of pointers through the memory, whether across the nodes of a linked list, down the nodes in a tree, or across the nodes in a graph. Depending on the arrangement of these pointers (linked list versus search tree), we may make the operations more or less efficient for the task at hand. We always need to understand how the algorithms will use the structure. It’s not enough just to buy a piece of fancy coffee-making equipment; we need to understand how to use it.

## What Is the Amortized Cost?

When considering whether to use any data structure, it’s important to take into account both the cost of constructing the data structure and the savings it will enable. Sorting an array or constructing a binary search tree can be more expensive than scanning through the data to find a single value. It’s almost always the case that searching once through each data point is more efficient than constructing an auxiliary data structure. Yet the math changes as soon as we move beyond a single search.

Sorted arrays, binary search trees, and other data structures are useful because they reduce the cost of _all_ future searches. If we pay a one-time _N_ log2(_N_) cost to sort an array of integers, we can then perform as many log2(_N_) binary searches as we want. We win because we amortize the cost of sorting the data over many future searches. In the same way, sorting cartons of milk in our refrigerator by expiration date can save us precious seconds during retrieval.

## How Can We Adapt Data Structures to a Specific Problem?

Basic data structures provide not only a set of tools that are useful in their own right but also a foundation on which to build more adaptive and specialized approaches. With the trie, we examined how to extend the branching structure of the binary search tree to a higher branching factor that enables efficient search over strings. We saw how linked lists provide a second level of flexibility to handle collisions in a hash table or multiple items in a grid cell.

Spatial data structures provide an excellent example of our ability to adapt, combine, and refine data structures. Combining the spatial partitioning of grids with a tree-based structure gives us the adaptive structure of a quadtree. However, both grids and quadtrees break down in higher dimensions. We saw how k-d trees adapt spatial data structures to split along a single dimension at each branch, not only helping the structure scale to higher dimensions but also improving its pruning power. As we consider new coffee-related problems, such as matching logos or optimizing the parameters of our brewing equipment, we should reexamine and potentially adapt the approaches in our toolbox to the specifics of the problem.

## What Are the Memory vs. Runtime Tradeoffs?

The tradeoff of memory and runtime is a classic consideration in computer science. We can often significantly decrease the cost of an algorithm by precomputing and storing additional data. Heaps allow us to efficiently find and extract the minimal (or maximal) element in a list, either within search algorithms or as an auxiliary data structure. The tradeoff is the cost of the heap itself. We use extra memory in a linear relationship to the size of the data we want to store. Similarly, by using extra memory to build a quadtree or k-d tree, we can drastically decrease the runtime of future nearest-neighbor searches.

Even within data structures, this tradeoff holds. We can decrease the collision rate in a hash table by increasing its size. Storing additional information in a linked list allows us to implement a skip list and realize better average performance for searches. Similarly, precomputing the bounds of spatial tree nodes and storing them in the node may allow us to more efficiently test whether we can prune a node.

It is critical to understand these tradeoffs and adapt them to a given project’s environment. Is the video game you are writing going to run on a personal computer, mobile device, or large server in a data center? Low memory environments might call for different approaches than high memory environments. The size of our coffee pantry will impact not only the total amount of coffee we can store but also whether adding brightly colored dividers is worth it. In a large pantry—such as a bedroom converted to a storage area, perhaps—the dividers might help us find coffee faster. In a small pantry, such as the kitchen cabinet, they might just cost us precious shelf space.

## How Can We Tune Our Data Structure?

Some data structures have parameters that greatly impact the performance of operations. The performance of grids on a nearest-neighbor search is highly dependent on the number and granularity of the grid cells. Similarly, the size parameter _k_ of B-trees allows us to adapt the size of each node to our local memory. These parameters almost always depend on the context in which we use the data structure. There is no one perfect setting.

It is important to understand how a data structure’s parameters impact performance and how they depend on the specifics of the problem. In some cases, we can analytically determine what parameter to use. For example, we can choose the size parameter _k_ for a B-tree using information about the size of memory blocks on the device where our code will run. We choose _k_ so that a full B-tree node fits snuggly in the memory block, allowing us to retrieve the maximum amount of data with a single access.

Other times, we may need to empirically test different parameters on real data. One simple approach is to use the data with a range of parameter settings and see which one performs best.

## How Does Randomization Impact Expected Behavior?

When examining binary search trees and hash tables, we noted that both data structures’ worst-case performance could degrade to linear time. If we insert sorted items into a binary search tree or choose a bad hash function for the data, we effectively end up with linked lists. The performance of our data structures isn’t guaranteed to be optimal under all conditions but depends on the data itself. Sometimes the best we can do is improve expected (or average case) runtime.

Understanding the possibility of extreme performance is critical to choosing and tuning the best data structure for your problem. When choosing the parameters of a hash table, we want to select a table size large enough to lower the probability of collisions while avoiding wasted memory. More critical is the choice of hash function, which, for hash tables, requires us to understand the distribution of keys. If the keys themselves have structure, such as consisting only of even numbers, we need to pick a hash function that is robust to that structure. Similarly, if we are organizing a conference for coffee lovers whose last name starts with _K_, the registration table shouldn’t partition attendees by the first letter of their last name.

We can mitigate the impact of pathologically bad data somewhat by randomizing the data structure itself. If we are always adding data into a binary search tree in sorted order, we effectively end up with a linked list. Skip lists provide a technique for intentionally injecting randomness into the level of our list nodes to provide, on average, logarithmic running times. Randomization isn’t a cure-all, though. Skip lists may choose bad heights by chance. In the worst case, like linked lists, skip lists’ performance degrades to be linear in the size of the data. However, the odds that they do so are small, and we can expect them to perform well on average, even in the face of pathologically bad data.

## Why This Matters

There is no one perfect data structure in computer science. It would be fantastic if we could point to a single data structure and say, “Always use X,” but unfortunately, it’s not that simple. All data structures come with their own tradeoffs of complexity, performance, memory usage, and accuracy.

Throughout this book, we examined a sampling of different data structures, their tradeoffs, and how they could impact algorithms. Our coverage was far from exhaustive; there’s a wide range of data structures further optimized for individual algorithms, problems, or domains. For example, red-black trees provide a self-balancing extension of binary search trees, while metric trees provide a different approach to spatial partitioning for higher-dimensional data. Both these approaches, and the hundreds of other impressive data structures out there, come with their own sets of tradeoffs and optimal use cases. We barely scratched the surface of the rich and complex world of data structures.

This book aimed to encourage you to think carefully about how to store and organize your data. As much as specific programming languages or clever algorithms, data structures can have a material impact on the performance, accuracy, and complexity of your programs. It’s important for all computer science practitioners to understand not only the specifics of individual data structures but also how those data structures function in the broader context of the problem they are trying to solve.

Especially if coffee is involved.