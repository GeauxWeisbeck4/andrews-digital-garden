---
id: 01J9QFB0KZWWMTJ4WTQPD3589M
modified: 2024-10-08T21:47:26-04:00
---
As we saw in the previous chapter, we often need to be cognizant of how our data structures fit into local memory and how to limit retrievals from slower memory. As a data structure grows, it can store more data but might not be able to fit into the fastest memory. This chapter introduces the _Bloom filter_, a data structure that extends the core concepts behind a hash table to limit the amount of memory needed to filter over a large key space.

Bloom filters were invented in 1970 by computer scientist Burton Bloom. A Bloom filter tracks which keys have been inserted while ruthlessly optimizing for both memory usage and execution time. It tries to answer one very simple yes/no question: have we previously seen this key? For example, we might use a Bloom filter to check if a password is on a list of known weak passwords. Alternatively, we can use Bloom filters as prefiltering step before accessing a larger and more comprehensive, but slower, data structure.

In its quest for ultimate efficiency, the Bloom filter uses an approach that can result in false positives. This means that with some non-zero probability, the Bloom filter will indicate that a key has been inserted when it, in fact, has not. In other words, a Bloom filter of bad passwords might occasionally reject a good one.

As we will see, the combination of low memory overhead, fast execution time, and a guaranteed lack of false negatives—cases that indicate a key has not occurred when it actually has—make Bloom filters useful for prefiltering tasks and early checks before accessing a more expensive data structure. This leads to a two-stage lookup similar to the approach for caches in Chapter 11. To check whether a record is in our data set, we first check whether it has been inserted into the Bloom filter. Since the Bloom filter is a compact data structure in fast memory, we can perform this check quickly. If the Bloom filter returns false, we know the record is not in our data set, and we can skip the expensive lookup in the comprehensive data structure. If the Bloom filter returns true, then either we have received a false positive or the record is in our larger data structure, and we perform the full search.

Imagine we’d like to determine whether a given record exists in a large database of medical records before we do a more computationally expensive search. The medical database is huge, includes images and videos, and must be stored across a multitude of large hard drives. Further, given the many millions of records, even the index is too large to fit into local memory. While it will occasionally return a false positive and send us searching for a record that does not exist, the Bloom filter will also help us avoid many pointless searches. Whenever the Bloom filter indicates that a record is not in the data set, we can stop the search immediately!

## Introducing Bloom Filters

At its heart, a Bloom filter is an array of binary values. Each bin tracks whether or not we have ever seen anything mapping to that hash value before. A value of 1 indicates that a given bin has been seen before. A value of 0 indicates that it has not.

Bloom filters can be incredibly useful when we want to easily search for a single value within a large number of values. Imagine this filtering in the context of searching a large, crowded ballroom for a friend. We could spend hours wandering the floor and peering at faces before concluding that our friend is not in attendance. It’s much simpler to ask a knowledgeable event organizer first. We describe our friend to the organizer, who has an excellent memory and can confirm whether anyone matching the description is there. Our description, and the organizer’s mental map, consists of a series of basic attributes. Our friend is tall, wears sneakers, and has glasses.

The event organizer’s answer may still not be 100 percent accurate—we are using general attributes, and multiple people will share those individual descriptors. Sometimes the organizer will give us a false positive, saying, “I saw someone with those characteristics” when our friend isn’t there. But they will never make a false negative statement and tell us our friend isn’t there when they really are. If the organizer has not seen someone with the three characteristics we list, then we can guarantee our friend is not at the event. The response doesn’t need to have zero false positives to help us on average. If the organizer can save us 9 out of every 10 searches, it would be a tremendous win.

Let’s examine how we can extend the hashing techniques we learned Chapter 10 to this prefiltering question. We start with a simple indicator array and examine where it falls short. Then we show how the use of multiple hash functions can provide a more robust filtering scheme.

### Hash Tables of Indicators

Consider the simplest filter, a binary indicator array mapped to by a single hash function. When we insert a key, we compute the hash value and mark the appropriate bin with a 1. When we look up a key, we compute the hash value and check the corresponding bin. It’s simple. It’s elegant. And, unfortunately, it breaks at the slightest hash collision.

Imagine we implement this simple single-hash-function filter for our thousand-page coffee log. Whenever we want to look up a type of coffee in our log, we first ask the filter the simple question, “Have I tried this type of coffee before?” This filter often allows us to avoid binary searching through a thousand pages if we know that we have never tried the coffee before.

For the first month it works perfectly. Every time we sample a new coffee, we add it to the log and flip the appropriate bit in our filter from 0 to 1, as shown in [Figure 13-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c13.xhtml#figure13-1). We use the coffee name as the input to our hash function, allowing us to check future coffees by name.

![The string “House Blend” is mapped to bin 6, which has the value 1. The other 11 bins have value 0.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c13/f13001.png)

Figure 13-1: A single hash function can map a string to the index of an array.

For the first few entries, the single-hash-function Bloom filter acts similarly to a regular hash table (without collision resolution) that stores binary values for each entry. We insert a 1 into the appropriate bin for each key seen, and a value of 0 indicates we have not seen any entry hashing to that value. When we ask whether we’ve tried the House Blend variety, we receive a simple “yes” when our lookup returns a 1.

However, problems emerge as we add more and more values to our filter. After a day of coffee drinking, our binary array begins to fill up. As shown in [Figure 13-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c13.xhtml#figure13-2), even if we’re only consuming a few varieties of coffee a day, we start to fill the array with 1s.

![Four of the 12 bins in the binary array have the value 1; the rest are zero.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c13/f13002.png)

Figure 13-2: A binary array that is starting to fill up

Since Bloom filters, unlike hash tables, don’t use chaining or any other mechanism to resolve collisions, two different coffees will occasionally map to the same entry. Soon we don’t know whether we really sampled the Burnt-Bean Dark Roast or whether the corresponding 1 is due to our previous tasting of Raw Bean, Uncooked, Bitter Blast, which happens to map to the same entry. Remember from Chapter 10 that whenever we are mapping from a large key space (the set of coffee names) to a smaller key space (the entries in an array), we will see collisions. This problem grows worse and worse as we add more entries to our log.

Within a year, our initial filter is effectively useless. Our varied coffee experiences, while enjoyable, have packed the array with 1s. We now have something like the array shown in [Figure 13-3](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c13.xhtml#figure13-3). Well over half of our queries result in hash collisions and thus false positives. The filter is no longer an efficient prefilter. Rather, it almost always adds the overhead of a useless check before the real search. In our party example, this correlates to using a single attribute to describe our friend. If we provide only our friend’s hair color, the event organizer will almost always have seen someone matching that description.

![Eight of the 12 bins in the binary array have the value 1; only 4 are zero.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c13/f13003.png)

Figure 13-3: A binary array that is too full to be a useful coffee tracker

The simplest fix to our mounting collisions would be to increase the space of hash values. We could make our binary array larger. Instead of using 100 indicator values, we could try 1,000. This reduces the likelihood of collisions, but does not remove it entirely. A single collision will still result in a false positive, and, as we saw in Chapter 10, some collisions are unavoidable. Our hashing approach isn’t doomed yet, though. We can do even better by adopting a strategy that also reduces the _impact_ of each collision.

### The Bloom Filter

A Bloom filter solves the problem of collisions by taking the idea of hash functions to the extreme. Instead of using a single hash function for each key, it employs _k_ independent hash functions, all mapping the key onto the same range of hash values. For example, as shown in [Figure 13-4](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c13.xhtml#figure13-4), we might use three hash functions for our coffee list. The hash function _f1_ maps this key to index 2; the second function, _f2_, maps it to index 6; and the third, _f3_, maps it to index 9.

![The string “House Blend” is mapped to bin 2 by the first hash function, bin 6 by the second hash function, and bin 9 by the third hash function.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c13/f13004.png)

Figure 13-4: Inserting the string HOUSE BLEND into a Bloom filter using three hash functions

Formally, we define the Bloom filter’s operations as follows:

1. Insert key For each of our _k_ hash functions, map the key to an index and set the value at that index to 1.
2. Lookup key For each of our _k_ hash functions, map the key to an index and check whether the value at that index is 1. Return true if and only if all _k_ array values are 1.

At first glance, all we have done is make matters worse. Instead of filling up one bin per sample, we’re filling three. Our array is practically guaranteed to fill up faster and see collisions earlier. If we add a second entry, as shown in [Figure 13-5](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c13.xhtml#figure13-5), we add more 1s to our array.

![The string “Morning shock” is mapped to bin 5 by the first hash function, bin 2 by the second hash function, and bin 8 by the third hash function.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c13/f13005.png)

Figure 13-5: Inserting the string MORNING SHOCK into a Bloom filter using three hash functions

On the contrary, we’ve actually improved things. The power of the Bloom filter enables us to look up entries efficiently. Instead of succumbing to a single collision, producing a false positive if we see a see a single 1, we require _all_ the bins for our target string to contain a 1, as shown in [Figure 13-6](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c13.xhtml#figure13-6).

![The string “Pure Caffeine” is mapped to bin 9 by the first hash function, bin 8 by the second hash function, and bin 5 by the third hash function.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c13/f13006.png)

Figure 13-6: Looking up the string PURE CAFFEINE in a Bloom filter with three hash functions

If we see a single 0, we know that this entry has not been inserted into the array. In [Figure 13-7](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c13.xhtml#figure13-7), we can see we’ve never sampled the Caffeine +10 roast. In the ballroom, the event organizer only needs to know that our friend is wearing a bowler hat to categorically answer that they’ve seen no one of that description. They’ve seen someone of the correct height and someone with the correct hair color, but no one wearing that hat. We can safely avoid a full search.

![The string “Caffeine +10” is mapped to bin 9 by the first hash function, bin 8 by the second hash function, and bin 4 by the third hash function. The first two functions map to bins containing 1, but the final hash function maps to a bin containing 0.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c13/f13007.png)

Figure 13-7: Looking up the string CAFFEINE +10 in a Bloom filter with three hash functions

To register a false positive, _each_ of our hash values must collide with a previous entry. If we balance the size of our array with the number of hash functions, we can reduce the probability for a false positive.

We can visualize a Bloom filter’s treatment of collisions via a conversation with a knowledgeable barista. Imagine that, on a recent trip, a friend hands us an incredible cup of coffee. After basking in the joy of the rich flavors and potent caffeine content, we ask our friend about this new discovery. To our dismay, our soon to be former friend shrugs, points in a general direction, and claims that they purchased it at a shop “that way.” They can’t remember the name of the barista, the shop, or even the coffee brand. Unfortunately, we don't have time to find the shop and interrogate the owner ourselves. We need to get on our flight home. Holding back tears at the lost knowledge, we jot a few attributes on a piece of paper and resolve to track down the mystery coffee.

Upon returning home, we visit the most knowledgeable barista we know and show them our notes. We have managed to capture five key traits about our coffee, such as “primary smell is chocolate.” This clearly isn’t enough information to identify the coffee uniquely, but we can ask our barista if they know of anything matching this description. Before paging through their own 10,000-page coffee log to look for a coffee matching all five attributes, the barista considers them independently and rules out a match. Despite our arguments that the flavors worked surprisingly well, our expert has never heard of any coffee that includes an “extra-sweet bubblegum flavor.” They give us a funny look and mumble something about having standards.

In this case, our five attributes are effectively hash functions, projecting the complex coffee experience into a lower-dimensional space—an array of descriptor words. The barista uses each of these attributes individually to check whether they know of any coffee that could match, using the index of their very own coffee log. An entry in the index means at least one match. The per-attribute test might run into false positives, but that’s okay. In the worst case, we waste some time looking through old coffee log entries before determining that nothing perfectly matches. At least we know we will never see a false negative. If even one of the characteristics is unique, the barista can confidently say no, and we can leave confident in the knowledge that no coffee in our barista’s extensive catalog could be our mystery blend.

We can use just the Bloom filter step to make fast, in-the-moment decisions without subsequently searching a larger data structure. Imagine that our trusted barista maintains a secret list of coffees to avoid, 500 coffees around the world that are so vomit-inducingly terrible they will make us stop drinking coffee for a full month. For various liability reasons, they don’t publish the list. But if we ask our barista friend about a specific coffee on the list, they will give a polite warning that maybe we’d prefer decaf instead. Before sampling any new coffee, we’d do well to check it isn’t on their list.

Of course, we won’t be able to call the barista each time we get the opportunity to try a new coffee, so we need a quick way of making the decision. Using five informative attributes, including primary smell and viscosity, the barista constructs a Bloom filter for those coffees, marking 1 for each of the five attributes for each coffee on the list. When we have an opportunity to sample a new coffee, we check the five attributes against the list. If any of them map to 0, we can safely drink the new coffee. It is guaranteed to not be on the barista’s list. However, if all the attributes are 1, maybe we should order something else. As a bonus, the barista never has to distribute their secret list.

We can apply Bloom filters to computer science applications in a similar fashion. Consider the problem of checking a password against a list of known weak passwords. We could systematically search the full list every time someone proposes a new password. Alternatively, we could create a Bloom filter to quickly check if we should reject the password. Occasionally we might reject a reasonable password, but we can guarantee we’ll never let a bad one through.

### Bloom Filter Code

In its simplest form, a Bloom filter can be stored as just an array of binary values. To make the code clearer, we wrap the Bloom filter in a simple composite data structure containing the parameters, such as the size and the number of hash functions:

```
BloomFilter {
    Integer: size
    Integer: k
    Array of bits: bins
    Array of hash functions: h
}
```

Given this wrapper, the code for both the insertion and lookup functions can be implemented with a single `WHILE` loop:

```
BloomFilterInsertKey(BloomFilter: filter, Type: key):
    Integer: i = 0
    WHILE i < filter.k:
        Integer: index = filter.h[i](key)
        filter.bins[index] = 1
        i = i + 1

BloomFilterLookup(BloomFilter: filter, Type: key):
    Integer: i = 0
    WHILE i < filter.k:
        Integer: index = filter.h[i](key)
        IF filter.bins[index] == 0:
            return False
        i = i + 1
    return True
```

In this code, `filter.h[i](key)` denotes the Bloom filter’s `i`th hash function applied to `key`. Both functions use a loop to iterate over the _k_ hash functions, computing `key`’s hash value and accessing the corresponding bin in the Bloom filter’s array. In the case of insertion, the code sets the value of the bin to `1`. In the case of lookup, the code checks whether the bin contains a `0` and returns `False` if so.

In the worst case, the functions cost scales linearly with _k_ as we need to iterate over each hash function for each operation. The structure of lookup provides an additional potential advantage. The lookup can terminate immediately after finding the first 0, skipping any further hash functions. Importantly, the runtime of both insertion and lookup is independent of both the size of the Bloom filter (number of bins) and the number of items inserted.

## Tuning Bloom Filter Parameters

There are multiple parameters that impact the Bloom filter’s false positive rate, including the size of the array and the number of hash functions used. By tuning these parameters to the problem at hand, we can often keep the false positive rate very low while minimizing the amount of memory used. We can do this empirically with real data, through simulation, or with a variety of mathematical approximations of the false positive rate.

One common and simple approximation is:

_FalsePositiveRate_ = (1 – (1 – 1/_m_)_nk_)_k_

where _n_ is the number of items inserted into the Bloom filter, _m_ is the size of the array, and _k_ is the number of hash functions used. This approximation uses simplified assumptions, but it gives a good view into how the various parameters interact:

- Increasing the size of the array (_m_) always decreases the false positive rate, because there are more bins to store information.
- Increasing the number of items inserted (_n_) always increases the false positive rate, because we set more of those bins to 1.
- Increasing the number of hash functions (_k_) can increase or decrease the false positive rate depending on the other parameters. If we use too many hash functions, we are going to fill up large quantities of the array with each insertion. If we use too few, a small number of collisions could produce a false positive.

[Table 13-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c13.xhtml#table13-1) provides insight into how the Bloom filter sizes (_m_) and number of hash functions (_k_) impact the false positive rate for a fixed number of items inserted (_n_ = 100).

Table 13-1: Example False Positive Rates for Different Parameterizations (_m_, _k_) with _n_ = 100

|**_m_**|**_k_ = 1**|**_k_ = 3**|**_k_ = 5**|
|---|---|---|---|
|200|0.3942|0.4704|0.6535|
|400|0.2214|0.1473|0.1855|
|600|0.1536|0.0610|0.0579|
|800|0.1176|0.0306|0.0217|
|1000|0.0952|0.0174|0.0094|

Ultimately the optimal parameter setting will depend on the specific problem. We need to choose a tradeoff among false positive rate, computational cost, and memory cost that best suits our application.

## Bloom Filters vs. Hash Tables

At this point, the skeptical reader may again fling their hands in the air in protest: “Why not just use a hash table? Whenever a key occurs, we can add it along with some trivial data, such as a Boolean value of True, to the hash table. Then we can search this table for exact match keys. Sure, we might have to do some chaining due to collisions, but we’d get exact answers. Why do you always make things so complicated?”

It’s a fair point. Hash tables do allow us to answer the same question that Bloom filters answer, and more exactly. But, as our skeptical reader noted, they do so at the cost of additional space and potentially runtime. In order for a hash table to fully resolve collisions, we need to store enough information to deterministically say that we have seen this exact key before, and that means storing the key itself. Add to that the overhead of pointers in a chained hash table, as shown in [Figure 13-8](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c13.xhtml#figure13-8), and we could be using significantly more memory.

![A linked list for a hash table bucket consists of nodes with data for the key, the value, and a pointer to the next node.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c13/f13008.png)

Figure 13-8: A hash table with chaining requires memory for each key and at least one pointer.

In contrast, the Bloom filter doesn’t bother storing the keys or pointers to subsequent nodes. It only keeps around a single binary value for each bucket. We can store _m_ bins using exactly _m_ bits. This extreme space efficiency proves valuable for multiple reasons. First, it allows us to vastly increase the number of bins (the size of our filter) while keeping the memory manageable. We can store 32 individual bins for the price of a single 32-bit integer.

Second, and more importantly for computationally sensitive applications, it often allows us to keep the Bloom filter in memory, or even in the memory’s cache, for fast access. Consider the tradeoffs we explored in the previous chapter of structuring B-tree nodes to reduce the number of retrievals from slower memory. Bloom filters work to maximize the amount of the data structure that directly helps with filtering with the goal of keeping the whole data structure in very fast memory.

## Why This Matters

Bloom filters can be powerful tools when we need to hyper-optimize the tradeoff between memory usage and accuracy. They combine the mathematical mappings introduced in Chapter 10 with the focus on fitting data into a compact form that can be stored in local memory that we saw in Chapter 12. Like caches, they provide an intermediate step to an expensive lookup that can help reduce the cost on average.

More important, however, is that Bloom filters provide a glimpse into a new type of data structure—one that can occasionally return a false positive. The accuracy of a lookup operation is not guaranteed but rather probabilistically depends on the data. If we’re lucky, we could insert a large number of elements before seeing any false positives. If we’re not, though, we could see collisions early on. This type of data structure provides a different approach to thinking about how we organize data and the relevant tradeoffs. If we are willing to accept some (hopefully small) number of errors, how far can we push the efficiency?

In the next chapter, we consider a data structure that relies on a different type of randomness. Instead of providing a probabilistically correct answer, skip lists use randomness to avoid bad worst-case behavior and provide efficient operations on average.