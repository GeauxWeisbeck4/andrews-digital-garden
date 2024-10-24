---
id: 01J9QF9BH6YDJ6WCH0C5F3BS7D
modified: 2024-10-08T21:46:32-04:00
---
This chapter introduces _caches_, data structures that seek to alleviate high data access costs by storing some data closer to the computation. As we have seen throughout the previous chapters, the cost of data access is a critical factor in the efficiency of our algorithms. This is important not just to how we organize the data in storage but also to the type of storage we use. We say data is _local_ when it is stored closer to the processor and thus quickly available for processing. If we copy some of the data from more expensive, distant locations to local storage, we can read the data significantly faster.

For example, we might use caches to accelerate access to web pages. Loading a web page consists of querying a server for the information contained on the page, transferring that data to your local computer, and rendering this information visually. If the web page contains large elements, such as images or videos, we may need to transfer a lot of data. To reduce this cost, browsers cache commonly accessed data. Instead of redownloading the logo of our favorite site every time we access the page, the browser keeps this image in a local cache on the computer’s hard drive so that it can quickly read it from disk.

How should we choose which data to load into our cache? Obviously, we can’t store everything. If we could, we wouldn’t need the cache in the first place—we’d just copy the entire data set into the closest memory. There are a range of caching strategies whose effectiveness depends, as always, on the use case. In this chapter, we combine data structures we’ve previously explored to build one such strategy, the least recently used (LRU) cache, which can greatly facilitate the operations of our favorite coffee shop. We also briefly discuss a few other caching strategies for comparison.

## Introducing Caches

Up to this point, we’ve treated all of the computer’s storage equally, a single bookshelf from which we can grab any item we need with approximately the same amount of effort. However, data storage is not this straightforward. We can think of the storage as being like a large, multistory library. We have shelves of popular books arranged near the entrance to satisfy the majority of patrons, while we relegate the musty stacks of old computer science journals to the basement. We might even have a warehouse offsite where rarely used books are stored until a patron specifically requests them. Want that resource on the PILOT programming language? Odds are, it won’t be in the popular picks section.

In addition to paying attention to how we organize data within our program, we need to consider _where_ the data is stored. Not all data storage is equal. In fact, for different mediums, there is often a tradeoff among the size of storage available, its speed, and its cost. Memory on the CPU itself (registers or local caches) is incredibly fast but can only hold a very limited amount of data. A computer’s random-access memory (RAM) provides a larger space at slower speeds. Hard drives are significantly larger, but also slower than RAM. Calling out to a network allows access to a huge range of storage, such as the whole internet, but incurs corresponding overhead. When dealing with very large data sets, it might not be possible to load the entirety of the data into memory, which can have dramatic impact on the algorithms’ performance. It’s important to understand how these tradeoffs impact the performance of algorithms.

Compare this to our own morning coffee-making routine. While it would be ideal to have thousands of varieties at our fingertips, our apartment can store only a limited amount of coffee. Down the street is a coffee distributor stocked with hundreds of varieties, but do we really want to walk there every time we want to make a new cup of coffee? Instead, we store small quantities of our preferred coffees in our apartment. This local storage speeds up the production of our morning brew and lets us enjoy our first cup before venturing out into the world.

Caches are a step before accessing expensive data, as shown in [Figure 11-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c11.xhtml#figure11-1). Before calling out to a remote server or even accessing our local hard drive, we check whether we already have the data stored locally in the cache. If we do, we access the data directly, and cheaply, from the cache. When we find the data in the cache, that’s a _cache hit_. We can stop the lookup and avoid calling out to the more expensive storage. If the data isn’t in the cache, we have a _cache miss_. We give a defeated sigh and proceed with the more expensive access.

![The cache sits between the algorithm and expensive data store. Arrows labeled “query” point from the algorithm to the cache and from the cache to the data store. Arrows labeled “reply” point the other direction.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c11/f11001.png)

Figure 11-1: A cache sits between the algorithm and a slow, expensive data store.

Picture this in the context of a busy coffee counter. The daily menu contains 10 coffees ranging in popularity from the mega-popular house brew to the rarely purchased mint-cinnamon-pumpkin explosion. At the far end of the counter sits a coffee station with 10 pots of coffee, each on its own heater. Upon receiving an order, the barista walks to the coffee station and fills a cup with the appropriate coffee. After miles of walking each day, the barista asks the owner to install another heater right beside the register so they can give their tired feet a rest. The owner agrees but asks, “Which coffee will you keep near the register?” This question weighs on the barista’s mind and they try a few strategies.

The barista quickly realizes that it isn’t just a question of which coffee, but also when they should change it. They can store different coffees locally at different points of the day. They try keeping the house brew in the cache all day, moving the decaf closer at night, and swapping out the current selection for the house brew when there hasn’t been a customer in the last 10 minutes. Some strategies work and others, especially anything involving caching mint-cinnamon-pumpkin explosion, fail miserably. If the barista chooses correctly, most orders result in a cache hit, and they can avoid the long walk. If they choose incorrectly, the local coffee is useless for most orders.

The barista could further improve the situation by installing a second heater by the register. Instead of storing a single flavor nearby, they can now store two. The cost, however, is that this second heater uses up precious counter real estate. Both computers and coffee shops have a limited amount of local space. We cannot store every coffee next to the register, and we cannot store the entire internet in our computer’s RAM. If we make the cache too large, we will need to use slower storage for it. This is the fundamental tradeoff with a cache—memory used versus the speedup provided.

When the cache fills up, we determine which data to keep and which data to replace. We say the replaced data is _evicted_ from the cache. Perhaps the triple-caffeinated blend replaces the decaf during a fall morning rush. By evicting the decaf from the nearby heater, the barista saves hours of walking to retrieve the trendier blend. Different caching approaches use different eviction strategies to determine which data to replace, from counting how often the data has been accessed thus far to making predictions about whether the data will be used again in the near future.

## LRU Eviction and Caches

The _least recently used (LRU) cache_ stores recently used information. LRU caches are a simple and common approach that provide a good illustration of the types of tradeoffs we face in using caches. The intuition behind this caching strategy is that we are likely to re-access information that we recently needed, which fits well with our web-browsing example above. If we commonly access the same sets of pages over and over, it pays to store some of the (unchanging) elements of these pages locally. We don’t need to re-request the site’s logo with each visit. LRU caches consist of a set amount of storage filled with the most recently accessed items. The cache starts evicting older items—those least recently used—when the cache is full.

If our barista decides to treat the heater closest to the register as an LRU cache, they keep the last ordered coffee on that heater. When they receive an order for something different, they take the current cached coffee back to the coffee station, place it on the heater there, and retrieve the new coffee. They bring this new coffee back to the register, use it to fulfill the order, and leave it there.

If every customer orders something different, this strategy just adds overhead. Instead of walking to the end of the counter with a cup, the barista is now making the same journey with a pot of coffee—probably complaining under their breadth about the length of the counter or resentfully judging customers’ tastes: “Who orders mint-cinnamon-pumpkin explosion?” However, if most customers order in clumps of similar coffees, this strategy can be hugely effective. Three customers ordering regular coffee in a row means two round trips saved. The advantage can compound with the effect of customer interaction. After seeing the person in front of them order the pumpkin concoction, the next customer makes the (questionable) decision to copy their predecessor, saying, “I’ll have the same.”

This is exactly the scenario we can run into when browsing our favorite website. We move from one page of the site to another page of the same site. Certain elements, such as logos, will reappear on subsequent pages, leading to a significant savings from keeping those items in the cache.

### Building an LRU Cache

Caching with the LRU eviction policy requires us to support two operations: looking up an arbitrary element and finding the least recently used element (for eviction). We must be able to perform both operations quickly. After all, the purpose of a cache is to accelerate lookups. If we have to scan through an entire unstructured data set to check for a cache hit, we are more likely to add overhead than to save time.

We can construct an LRU cache using two previously explored components: a hash table and a queue. The hash table allows us to perform fast lookups, efficiently retrieving any items that are in the cache. The queue is a FIFO data structure that allows us to track which item hasn’t been used recently. Instead of scanning through each item and checking the timestamp, we can dequeue the _earliest_ item in the queue, as shown in the following code. This provides an efficient way to determine which data to remove.

```
LRUCache {
    HashTable: ht
    Queue: q
    Integer: max_size
    Integer: current_size
 }
```

Each entry in the hash table stores a composite data structure containing at least three entries: the key, the corresponding value (or data for that entry), and a pointer to the corresponding node in the cache’s queue. This last piece of information is essential, because we need a way to look up and modify entries in the queue. As in [Listing 11-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c11.xhtml#listing11-1), we store those pieces of information directly in the value entry of the hash table’s nodes.

```
CacheEntry {
    Type: key
    Type: value
    QueueListNode: node
}
```

Listing 11-1: The data structure for a cache entry containing the data’s key, value, and a link to its node in the queue

[Figure 11-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c11.xhtml#figure11-2) shows the resulting diagram of how these pieces fit together. The diagram looks a bit complex but becomes easier to understand when we break it down into two parts. On the left is the hash table. As in Chapter 10, each hash value corresponds to a linked list of entries. The value for these entries is the `CacheEntry` data structure in [Listing 11-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c11.xhtml#listing11-1). On the right-hand side is the queue data structure, which stores the entry’s keys. A single pointer links across these data structures, pointing from the `CacheEntry` data structure to the queue node with the corresponding key.

Of course, the real-world layout of the components in [Figure 11-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c11.xhtml#figure11-2) throughout the computer’s memory is even messier than it appears in the diagram, because the nodes do not actually reside next to each other.

![A hash table on the left includes an entry for each cache entry with pointers to an additional CacheEntry node. Each of the CacheEntry nodes has a single pointer that links to an entry in the queue on the right.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c11/f11002.png)

Figure 11-2: An LRU cache implemented as a combination of a hash table and queue

In [Listing 11-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c11.xhtml#listing11-2), we define a single lookup function, `CacheLookup`, that returns the value for a given lookup key. If there is a cache hit, the lookup function returns the value directly and updates how recently that data was accessed. If there is a cache miss, the function fetches the data via the expensive lookup, inserts it into the cache, and, if necessary, removes the oldest piece of data.

```
CacheLookup(LRUCache: cache, Type: key):
  ❶ CacheEntry: entry = HashTableLookup(cache.ht, key)

    IF entry == null:
      ❷ IF cache.current_size >= cache.max_size:
            Type: key_to_remove = Dequeue(cache.q)
            HashTableRemove(cache.ht, key_to_remove)
            cache.current_size = cache.current_size - 1

      ❸ Type: data = retrieve data for the key from 
                     the slow data source.

      ❹ Enqueue(cache.q, key)
        entry = CacheEntry(key, data, cache.q.back)
      ❺ HashTableInsert(cache.ht, key, entry)
        cache.current_size = cache.current_size + 1
    ELSE:
        # Reset this key's location in the queue.
      ❻ RemoveNode(cache.q, entry.node)
      ❼ Enqueue(cache.q, key)

        # Update the CacheEntry's pointer.
      ❽ entry.node = cache.q.back
    return entry.value
```

Listing 11-2: Code for looking up an item by its key

This code starts by checking whether `key` occurs in our cache table ❶. If so, we have a cache hit. Otherwise, we have a cache miss.

We deal with the case of a cache miss first. If the hash table returns `null`, we need to fetch the data from the more expensive data store. We store this newly retrieved value in the cache, evicting the (oldest) item from the front of the queue. We do this in three steps. First, if the cache is full ❷, we dequeue the key of the oldest item from the queue and use it to remove the corresponding entry in the hash table. This completes the eviction of the oldest item. Second, we retrieve the new data ❸. Third, we insert the (`key`, `data`) pair into the cache. We enqueue `key` into the back of the queue ❹. Then we create a new hash table entry with the new `key`, `data`, and pointer to the key’s corresponding location in the queue (using the queue’s `back` pointer). We store this entry in the hash table ❺.

The final block of code handles the case of a cache hit. When we see a cache hit, we want to move the element for this key from its current position to the back of the queue. After all, we’ve just seen it. It should now be the last element we want to discard. We move this element in two steps. First, we remove it from the queue with the `RemoveNode` function, using the pointer to the node ❻. Second, we re-enqueue the key at the back of the queue ❼ and update the pointer to that queue node ❽.

We can picture this update operation in the context of a line of customers in a coffee shop. If a customer leaves the line, they lose their position. When they rejoin the line in the future, they do so at the back.

### Updating an Element’s Recency

In order to support the `RemoveNode` operation in [Listing 11-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c11.xhtml#listing11-2), we need to change our queue to support _updating_ an element’s position. We need to move recently accessed items from their current position in the queue to the back in order to indicate that they were the most recently accessed. First, we modify our queue implementation from Chapter 4 to use a doubly linked list, which will allow us to efficiently remove items from the middle of the queue:

```
QueueListNode {
    Type: value
    QueueListNode: next
    QueueListNode: prev
}
```

As illustrated in [Figure 11-3](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c11.xhtml#figure11-3), the `next` field refers to the node directly behind the current node (the next node after this one to be dequeued), where the `prev` field refers to the node preceding this one.

![A queue with elements 31, 41, 5, 92, and 65. Each node contains a single number and links to both the next and previous number in the queue.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c11/f11003.png)

Figure 11-3: A queue implemented as a doubly linked list

Second, we modify the enqueue and dequeue operations accordingly. For both enqueue and dequeue, we add extra logic to update the previous pointers. The primary change from Chapter 4 is in how we set the `prev` pointer:

```
Enqueue(Queue: q, Type: value):
    QueueListNode: node = QueueListNode(value)
    IF q.back == null:
        q.front = node
        q.back = node
    ELSE:
        q.back.next = node
      ❶ node.prev = q.back
        q.back = node
```

In the enqueue operation, the code needs to set the new node’s `prev` pointer to the previous last element `q.back` ❶.

```
Dequeue(Queue: q):
    IF q.front == null:
        return null

    Type: value = q.front.value
    q.front = q.front.next
    IF q.front == null:
        q.back = null
    ELSE:
      ❶ q.front.prev = null
    return value
```

The dequeue operation sets the `prev` pointer of the new front node to `null` to indicate that there is no node in front of it ❶.

Finally, we add the `RemoveNode` operation, which provides the ability to remove a node from the middle of our queue. This is where the use of a doubly linked list helps. By keeping pointers in both directions, we do not need to scan the entire queue to find the entry before the current one. The code for `RemoveNode` adjusts the pointers into the node from the adjacent nodes in the linked list (`node.prev` and `node.next`), the front of the queue (`q.front`), and the back of the queue (`q.back`).

```
RemoveNode(Queue: q, QueueListNode: node):
    IF node.prev != null:
        node.prev.next = node.next
    IF node.next != null:
        node.next.prev = node.prev
    IF node == q.front:
       q.front = q.front.next
    IF node == q.back:
       q.back = q.back.prev
```

This code contains multiple `IF` statements to handle the special cases of adding or removing from either end of the queue.

One question remains: How can we efficiently find the element we want to remove and reinsert? We can’t afford to scan through the entire queue while searching for the node to update. Supporting a cache hit requires speedy lookups. In the case of our cache above, we maintain a pointer directly from the cache’s entry to the element in the queue. This allows us to access, remove, and reinsert the element in constant time by following the pointers.

## Other Eviction Strategies

Let’s consider how three of the many alternate eviction strategies—most recently used, least frequently used, and predictive eviction—compare to LRU eviction. The goal isn’t to provide an in-depth analysis of these methods but rather to help develop your intuition about the types of tradeoffs that arise when picking a caching strategy. The optimal caching strategy for a particular scenario will depend on the specifics of that scenario. Understanding the tradeoffs will help you pick the best strategy for your use case.

LRU is a good eviction strategy when we see bursts of common usage among certain cache items but expect the distribution of cached items to change over time. The local pot of coffee near the register described earlier is an excellent example. When someone orders a new type of coffee, the barista walks to the end of the counter, returns the old blend, retrieves the new blend, and leaves that carafe by the register.

Let’s contrast that with evicting the _most recently used (MRU)_ element. You can picture this in the context of an ebook reader that prefetches and caches books that you might enjoy. Since it is unlikely that we will read a book twice in a row, it might make sense to discard the recently completed book from our cache in order to free up space for a new one. While MRU can be a good approach for items with infrequent repetition, the same eviction would create constant tragedy in our coffee pantry. Imagine if we were forced to evict our favorite, go-to blend anytime we brewed a cup.

Instead of considering _when_ the item was last accessed, we could also track how many times it has been accessed in total. The _least frequently used (LFU)_ eviction strategy discards the element with the smallest count. This approach is advantageous when our cached population remains stable, such as with the tried-and-true coffee varieties we keep at home. We don’t evict one of our three go-to varieties simply because we recently tried the seasonal blend at our local coffeehouse. The items in the cache have a proven track record and are there to stay—at least, until we find a new favorite. Unfortunately, when preferences change, it can take a while for newly popular items to accumulate enough counts to make it into our cache. If we encounter a new brew that is better than anything in our home collection, we’d need to sample individual cups of the coffee many times, making numerous trips to the coffee shop, before finally buying beans for our own pantry.

The _predictive evictio__n_ strategy provides a forward-looking approach that tries to predict what elements will be needed in the future. Instead of relying on simple counts or timestamps, we can build a model to predict what cache entries are most likely to be accessed in the future. The effectiveness of this cache depends on the accuracy of the model. If we have a highly accurate model, such as predicting that we will switch from our go-to java blends to a seasonal fall blend only in October and November, we greatly improve the hit rate. However, if the model is inaccurate, perhaps associating each month with the first coffee we consumed that month last year, we can find ourselves reliving that onetime mistake of drinking mint-cinnamon-pumpkin lattes in August. Another downside of predictive eviction is that it adds complexity to the cache itself. It is no longer sufficient to track simple counts or timestamps; instead, the cache must learn a model.

## Why This Matters

Caches can alleviate some of the access cost when dealing with expensive storage mediums. Instead of calling out to the expensive storage, caches store a portion of this data in a closer and faster location. If we choose the correct caching strategy, we can save significant amounts of time by pulling data from the cache instead of the slower locations. This makes caches a common and powerful tool in our computational toolbox. They are used throughout the real world, from web browsers to library shelves to coffee shops.

Caches also illustrate several key concepts. First, they highlight the potential tradeoffs to consider when accessing data from different mediums. If we could store everything in the fastest memory, we wouldn’t need caches. Unfortunately, this usually isn’t possible; our algorithms will need to access larger, slower data stores, and it’s worth understanding the potential cost this adds. In the next chapter, we introduce the B-tree, a tree-based data structure that reduces the number of data accesses needed. This optimization helps reduce the overall cost when we cannot fit our data in nearby, fast memory.

Second, caches revisit the data structure–tuning question that we saw in previous chapters. Both the size of the cache and the eviction strategy are parameters that can have massive impact on your cache’s performance. Consider the problem of selecting the size of the cache. If the cache is too small, it might not store enough to provide a benefit. The single nearby pot of coffee in our café only helps so much. On the other hand, too large a cache will take important memory resources that might be needed for other parts of our algorithm.

Finally, and most importantly for this book’s purposes, caches illustrate how we can combine basic data structures, such as the hash table and the queue, to provide more complex and impactful behaviors. In this case, by using pointers to tie hash table entries to nodes in a queue, we can efficiently track which entry in our cache should be removed next.