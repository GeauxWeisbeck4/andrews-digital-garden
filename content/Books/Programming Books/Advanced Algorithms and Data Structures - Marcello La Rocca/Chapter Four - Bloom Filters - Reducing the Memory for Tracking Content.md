---
id: 01J9QFNG8JDWNH9YX1N2BP1Z9N
modified: 2024-10-08T21:53:10-04:00
---
# 4 Bloom filters: Reducing the memory for tracking content

This chapter covers

- Describing and analyzing Bloom filters
- Keeping track of large documents using little memory
- Showing why dictionaries are an imperfect solution
- Improving the memory print by using Bloom filters
- Recognizing use cases where Bloom filters improve performance
- Using metrics to tune the quality of Bloom filters’ solutions

Starting with this chapter we’ll be reviewing less common data structures that solve, as strange as it might seem, common problems. _Bloom filters_ are one of the most prominent examples; they are widely used in most industries, but not as widely known as you would expect for such a cornerstone.

In section 4.1 we introduce the problem that will be our North Star in this chapter: we need to keep track of large entities with the smallest memory print possible.

In section 4.2 we continue our narration by discussing a few increasingly complex solutions, showing their strengths and weaknesses; the latter, in particular, ought to be considered chances for improvement and fertile ground for algorithms designers.

As part of this discussion, we introduce the _dictionary_, an abstract data type that we discuss in depth in section 4.3, while section 4.4 switches to concrete data structures that implement dictionaries: hash tables, binary search trees, and Bloom filters.

You have probably guessed that we are particularly interested in the latter, because it is the topic of this chapter. We describe in section 4.5 the principles governing how a Bloom filter works; then we delve into each of its methods in section 4.6, showing the pseudo-code for the crucial parts.

Section 4.7 closes the first part of the chapter with a discussion of some typical use cases for Bloom filters: from distributed databases and file systems to rooters, this technology is pervasive.

In order to reach this goal, we keep a practical approach with an emphasis on enabling readers to recognize opportunities to use Bloom filters and giving them the instruments to do so.

Starting with section 4.8, the focus shifts to theory, providing a background for the interested reader who wants to understand not just how, but why, Bloom filters work. To facilitate understanding of these sections, readers can go through appendix F (as well as appendices B and C, if needed) to review randomized algorithms, big-O notation, and basic data structures prior to starting with section 4.8.

The next few sections will then closely examine the performance of our data structure, including running time and memory print (section 4.9), as well as the accuracy of the algorithm (section 4.10).

Finally, section 4.11 describes some of the most advanced variants of Bloom filters used to provide new features or lower the false-positive rate.

## 4.1 The dictionary problem: Keeping track of things

Let’s go over a hypothetical scenario: you work for a company that is large enough to maintain its own email service. This is a legacy service, providing basic features only. After the last reorganization, the new CTO[1](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034617) decides that you need to reinvent the email service to be aggressive on the market segment, and she puts your new manager in charge of the product redesign.

They want a brand-new, modern client with features like a contacts list and cool tricks. For instance, when you add a new recipient to an email, your application should check whether it’s already in the contacts list, and if it’s not, a popup (like the one shown in figure 4.1) should appear asking you if you’d like to add the new recipient.

And, needless to say, you are the lucky one who gets to be responsible for implementing this feature.

Of course, the resources allocated for the project are scarce, so you are only going to do a refactoring of the client, while on the server side you’ll have to make do with legacy code and legacy services running on proprietary machines.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F221.png)

Figure 4.1 Our email application prompting the user to add a new contact after an unsuccessful search

Calling the database every time you need to check an email address against your contacts list is out of the question: you are stuck with this legacy machine that can’t scale up, and you didn’t get funds for refactoring and scaling it out.[2](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034632) Your DB could not support more than a few calls per second, and your management’s projection is in the hundreds of emails written per second (they are very optimistic, and a bit reckless, but let’s not worry about that for now!). Your first instinct would be to ask for a remote distributed cache, like Memcached, Cassandra, or Redis. A roundtrip to the cache server alone would take in the range of a hundred milliseconds, best case scenario, which would be good. But neither spinning up a new server for the cache, nor buying it as a cloud service is feasible for you, budget-wise.

In the end, you decide you only have one way to solve this: asynchronously get your contacts list when you log in (or, more lazily, the first time you click on Compose during the current browser session), save the contacts list in your web page’s session storage space, and check the local copy of that data every time you look for existing contacts.

So far for the application design, figure 4.2 shows a possible architecture for our minimum viable product. Now you just need an efficient way to browse a contacts list and check if an email belongs to it.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F222.png)

Figure 4.2 A possible architecture for the “save new contact (with suggestion)” feature. On the client side, the web app receives the contacts list from the web server, and with those data creates a dictionary on the session storage. Whenever the user adds a recipient on an email, the web app checks the dictionary. If the contact is not in the list, a popup is shown to the user, who can then decide to save it. In that scenario, to save the new contact another HTTP call is made to the web server, and at the same time the dictionary is updated with the new value (although without going through the server).

Looking through a list for a certain entry is a common problem in computer science, known as the _dictionary problem_.

## 4.2 Alternatives to implementing a dictionary

The name shouldn’t be surprising; it’s exactly like when you need to look up a word in a dictionary (and by dictionary I mean one of those old gigantic paper books that have been almost entirely replaced by online dictionaries and search engines) or even in a phone book (which had no better luck after the third computer revolution).

To recap, our contacts web app needs to

- Download the list of contacts from a server
    
- Create a local copy for fast lookup/storage
    
- Allow looking up for a contact
    
- Provide the option to add a new contact if lookup is unsuccessful
    
- Sync with the server when a new contact is added (or an existing one modified)
    

What we really need is a data structure that is specialized in these kinds of operations; we need it to support fast insertion, and at the same time it needs to provide a way to look up an entry by value.

To be clear, when we use a plain array, we don’t have an efficient array’s method that tells us the index of an element `X`, nor an efficient (as in _sublinear_) method to tell us if an element is in the array or not. The only way we have to tell if an element is in the array is by going through all the array’s elements, although in a sorted array we could use binary search to speed up the search.

For example, we could store the strings `[“the”, “lazy”, “fox”]` in an array, and to search for `“lazy”`, we would skim through the whole array, element by element.

An associative array, instead, by definition has a native method that efficiently accesses the stored entries with a lookup by value. Usually this structure allows storing (key, value) pairs. For instance, we would have a list like `<(“the”, article)`, `(“lazy”, adjective)`, `(“fox”, noun)>`. We could search for `“lazy”` and the associative array would return `adjective`.

Another difference with regular arrays would be that the order of insertion in an associative array doesn’t matter; it’s not even well defined. That’s the price you pay to speed up lookup by value.

To really understand how efficiently we can solve this problem, we need to get into the details of implementations. Using the dictionary abstraction, however, allows us to discuss how to solve a problem (for instance, finding whether an email belongs to a list of contacts) without having to deal with the details of the data structure’s representation, hence leaving us free to focus on the task itself.

## 4.3 Describing the data structure API: Associative array

An associative array (also referred to as _dictionary_,[3](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034656) _symbol table_, or just _map_), is composed of a collection of (key, value) pairs, such that

- Each possible key appears at most once in the collection.
    
- Each value can be retrieved directly through the corresponding key.
    

The easiest way to grasp the essence of associative arrays is to think about regular arrays as a special case: the keys are just the set of indices between `0` and the size of the array minus `1`, and we can always retrieve a value by providing its index, so that the (plain) array `[“the”, “lazy”, “fox”]` can be interpreted as a dictionary storing the associations `(0, “the”), (1, “lazy”)` and `(2, “fox”)`.

Associative arrays generalize this concept, allowing keys to be drawn from virtually any possible domain.

|Abstract data structure: Associative array (aka dictionary)|   |
|---|---|
|API|class Dictionary {<br>  insert(key, value)<br>  remove(key) → value<br>  contains(key) → value<br>}|
|Contract with client|A dictionary permanently stores all the pairs added by the client(s). If a pair `(K,V)` was added to the dictionary (and not removed afterward), then `contains(K)` will return `V`.|

With this API defined, we can sketch a solution for our initial problem.

When users log in to their email, our client receives a list of contacts from the server and stores them in a _dictionary_ that we can keep in memory (having so many contacts that it won’t fit the browser’s session storage would be an exceptional situation even for an Instagram rock star!). If the user adds a new contact to our address book, we perform a call to `insert` on the dictionary; likewise, if users remove an existing contact, we just keep the dictionary in sync by calling `remove`. Whenever a user writes an email and inserts a recipient, we first check the dictionary, and only if the contact is not in the address book do we show a popup asking users if they would like to save the new contact.

This way, we never do an HTTP call to our server (and in turn to the DB) to check whether a contact is on our address book, and we only read from the DB once on startup (or the first time during a session that we write an email).

## 4.4 Concrete data structures

So far, so good with the theory, but of course, implementing associative arrays to be used on real systems is a completely different thing.

In theory, if the domain (the set of possible keys) is small enough, we can still use arrays by defining a total ordering on the keys and using their position in the order as the index for a real array. For instance, if our domain is made of the words `{“a”, “terrible”, “choice”}`, we could sort keys lexicographically, and then we would store the values in a plain string array; for instance, `{“article”, “noun”, “adjective”}`. If we need to represent a dictionary that only contains a value for key `“choice”`, we could do that by setting the values corresponding to missing keys to `null`: `{null, “noun”, null}`.

Usually, however, this is not the case, and the set of possible values for the keys is large enough to make it impractical to use an array with an element for any possible key’s value; it would just require too much memory, which would remain largely unused.

To overcome this issue with the memory, we present two naïve implementations, and three of the most widely used alternatives.

### 4.4.1 Unsorted array: Fast insertion, slow search

Even if you have never seen one of those paper dinosaurs (a dictionary), you ought to at least be familiar with print books, like this one (unless, of course, you bought the e-book version!).

Suppose you need to look for a specific word in this book—maybe a name—like Bloom, and note all the places where it occurs. One option you have is to go through the book starting from the first page, word by word, until you encounter it. If you need to find all the occurrences of the word Bloom, you will have to go through the book from cover to cover.

A book taken as a collection of words, in the order they are printed, is like an unsorted array. Table 4.1 summarizes the performance of the main operations needed using this approach with unsorted arrays.

Table 4.1 Using unsorted arrays as dictionaries

|Operation|Running time|Extra memory|
|---|---|---|
|`Creating the structure`|O(1)|No|
|`Looking up an entry`|O(n)|No|
|`Insert a new entry`|O(1) [4](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1041837)|No|
|`Remove an entry`|O(1)|No|

Unsorted arrays have the advantage of no extra work needed on creation, and adding a new entry is pretty easy, provided you have enough capacity.

### 4.4.2 Sorted arrays and binary search: Slow insertion, fast(-ish) search

That’s not exactly practical, as you can imagine. If, after going through the whole book, you need to search a second word, like “filter,” you would have to start all over and do a second pass through the hundreds of thousands of words in the book. That’s why most books have what’s called an index, usually toward the end of the book; there you can find an alphabetically ordered list of the most uncommon words (and names) used in the book. Common words don’t make it onto this list because they are used too frequently (articles like “the” and “a” are probably used on every single page) to be worth being listed, since the value of finding places where they are used would be minimal. Conversely, the rarer a word is in English (and names are the perfect example here), the greater importance it has when it’s used in your text.[5](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034684)

So, you can check the index and look for the name Bloom. Being in lexicographical order, you could go through it from the start until you catch the word you are looking for; it shouldn’t be too long with Bloom. You wouldn’t be that lucky with terms like hashing or, even worse, tree, which will be toward the end of the index.

That’s why, naturally, we do lookups in sorted lists by subconsciously using binary search:[6](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034718) with a phone book, you open it at a random page around the middle (or closer to the beginning or end, if you have an idea where the name you are looking for might be), then look at the first letter of the first word on that page, and jump either before or after the current page depending on what you are looking for. For example, if you are still looking for Bloom, and you open a phone book to a page where the first surname is Kurtz, then you know you can discard every page after that, and look only at the pages before. You randomly open another page (to the left of the one with Kurtz) and the last name on that page is Barrow; then you know Bloom will be on a page after the one with Barrow and before the one with Kurtz.

Going back to our problem with the contact list, one approach could be sorting your contacts and searching them using binary search.

As you can see from table 4.2, in terms of running time, both the initial cost (to sort the list) and the cost for adding a new entry are pretty high. Also, linear extra memory might be needed if we need to make a copy of the original list, to preserve the original order.

Table 4.2 Using sorted arrays as dictionaries

|Operation|Running time|Extra memory|
|---|---|---|
|`Creating the structure`|O(n*log(n))|O(n)|
|`Looking up an entry`|O(log(n))|No|
|`Add a new entry`|O(n)|No|
|`Remove an entry`|O(n)|No|

### 4.4.3 Hash table: Constant-time on average, unless you need ordering

We introduced hash tables and hashing in appendix C. The main take away for hash tables is that they are used to implement associative arrays where the possible values to store come from a very large set (for instance, all possible strings or all integers), but normally we only need to store a limited number of them. If that’s the case, then we use a hashing function to map the set of possible values (the _domain_, or source set) to a smaller set of `M` elements (the _codomain_, or target set), the indices of a plain array where we store the values associated to each key. (As we explained in appendix C, we get to decide how big `M` is depending on some considerations on the expected performance.) Typically, the set of values in the domain is referred to as _keys_, and the values in the codomain are indices from `0` to `M-1`.

Since the target set of a hashing function is typically smaller than the source set, there will be collisions: at least two values will be mapped to the same index. Hash tables, as we have seen in chapter 2, use a few strategies to resolve conflicts, such as _chaining_ or _open addressing_.

The other important thing to keep in mind is that we distinguish hash maps and hash sets. The former allows us to associate a value[7](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034732) to a key; the latter only to record the presence or absence of a key in a set. Hash sets implement a special case of _dictionary_, the `Set`. With respect to our definition of _dictionary_ as an abstract data structure, given at the beginning of this section, a _set_ is a specialization of a dictionary, where the `value` type is set to `Boolean`; the second parameter to `insert` becomes redundant, as the value associated with a key in the hash set will be implicitly assumed to be `true`.

|Abstract data structure: Set|   |
|---|---|
|API|class Set {<br>  insert(key)<br>  remove(key)<br>  contains(key) → true/ false<br>}|
|Contract with client|A set maintains a set of keys. If a key K was added to the set (and not removed afterward), then `contains(K)` will return `true`; otherwise it will return `false`.|

As we explain in appendix C, all operations in a hash table (and hash set) can be performed in amortized `O(1)` time.

### 4.4.4 Binary search tree: Every operation is logarithmic

Binary search trees (_BSTs_) are another old acquaintance of ours; we met them in chapters 2 and 3, and they are also discussed in appendix C.

A BST is a special kind of binary tree that can store keys on which it defined a total ordering: this means that for each pair of keys, it must be possible to compare them and decide which one is smaller, or if they are equal. A total ordering benefits from _reflexive_, _symmetric__,_ and _transitive_ properties.

Ordering relations

Given a set `S` on which we defined an ordering relation ≤, this relation is a total ordering if, for any three keys `x, y, z`, the following properties hold:

Reflexive: `x` ≤ `x`

Symmetric: if `x` ≤ `y`, then `y` ≤ `x`

Transitive: if `x` ≤ `y` and `y` ≤ `z` then `x` ≤ `z`

BSTs use these properties to make sure that the position of a key in the tree can be determined just by looking at a single path from the root to a leaf.

When we insert a new key, in fact, we compare it to the tree’s root. If it’s smaller, we take a left turn, traversing the root’s left subtree; otherwise, we go on the right subtree. At the next step, we repeat the comparison with the subtree’s root, and so on until we get to a leaf, and that’s exactly the position where we need to insert the key.

If you remember what we saw in chapter 2 regarding heaps (or if you had a chance to refresh your memory, in appendix C), all operations in a BST take time proportional to the height of the tree (the longest path from root to a leaf). In particular for balanced BSTs, all operations take `O(ln(n))` time, where `n` is the number of keys added to the tree.

Of course, compared to the `O(1)` amortized running time of hash tables, even balanced BSTs don’t seem like a good choice for implementing an associative array. The catch is that while their performance on the core methods is slightly slower, BSTs allow a substantial improvement for methods like finding the predecessor and successor `o` key, and finding minimum and maximum: they all run in `O(ln(n))` asymptotic time for BSTs, while the same operations on a hash table all require `O(n)` time.

Moreover, BSTs can return all keys (or values) stored, sorted by key, in linear time, while for hash tables you need to sort the set of keys after retrieving it, so it takes `O(M + n*ln(n))` comparisons.

Now that we have described the basic data structures most commonly used to implement dictionaries, it feels like a good time to recap what we have described so far. Table 4.3 gathers the running times of the main operations on the possible implementations of dictionaries we mentioned.

Table 4.3 Running time of operations on dictionaries for different implementations

|Operation|Unsorted arrays|Sorted arrays|BST|Hash table|
|---|---|---|---|---|
|`Create the DS`|O(1)|O(n*log(n))|O(n*log(n))|O(n)|
|`Look-up an entry`|O(n)|O(log(n))|O(log(n))|O(n/M) [8](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1042436)|
|`Add a new entry`|O(1) [8](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1042436)|O(n)|O(log(n))|O(n/M) [8](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1042436)|
|`Remove an entry`|O(1)|O(n)|O(log(n))|O(n/M) [8](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1042436)|
|`Sorted List`|O(n*log(n)|O(n)|O(n)|O(M+n*log(n))|
|`Min/Max`|O(n)|O(1)|O(1) [9](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1042466)|O(M+n)|
|`Prev/Next`|O(n)|O(1)|O(log(n))|O(M+n)|

Looking at table 4.3, it’s evident that if we don’t have to worry about any operation that involves the order of the element, or the order they were inserted, then the amortized time of hash tables is the best. If `n~=M` (and there are approximately as many buckets as elements), then a hash table can perform insertion, removal, and look-up in amortized constant time.

### 4.4.5 Bloom filter: As fast as hash tables, but saves memory (with a catch)

We haven’t officially met this data structure yet in the book, but there is a good chance you have already heard of _Bloom filters_. They are a data structure named after Burton Howard Bloom, who invented them in the 1970s.

There are three notable differences between hash tables and Bloom filters:

- Basic Bloom filters don’t store data; they just answer the question, is a datum in the set? In other words, they implement a hash set’s API, not a _hash table_’s API.
    
- Bloom filters require less memory in comparison to hash tables; this is the main reason for their use.
    
- While a negative answer is 100% accurate, there might be false positives. We will explain this in detail in a few sections. For now, just keep in mind that sometimes a Bloom filter might answer that a value was added to it when it was not.
    
- It is not possible to delete a value from a Bloom filter.[10](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034748)
    

There is a tradeoff between the accuracy of a Bloom filter and the memory it uses. The less memory, the more false positives it returns. Luckily, there is an exact formula that, given the number of values we need to store, can output the amount of memory needed to keep the rate of false positives within a certain threshold. We’ll go into the details for this formula in the advanced sections toward the end of the chapter.

## 4.5 Under the hood: How do Bloom filters work?

Let’s now delve into the details of Bloom filter implementation. A Bloom filter is made of two elements:

- An array of `m` elements
    
- A set of `k` hash functions
    

The array is (conceptually) an array of bits, each of which is initially set to `0`; each hash function outputs an index between `0` and `m-1`.

It is crucial to clarify as soon as possible that there is not a 1-to-1 correspondence between the elements of the array and the keys we add to the Bloom filter. Rather, we will use `k` bits (and so `k` array’s elements) to store each entry for the Bloom filter; `k` here is typically much smaller than `m`.

Note that `k` is a constant that we pick when we create the data structure, so each and every entry we add is stored using the same amount of memory, exactly `k` bits. With string values, this is pretty amazing, because it means we can add arbitrarily long strings to our filter using a constant amount of memory, just `k` bits.

When we insert a new element key into the filter, we compute `k` indices for the array, given by the values `h0(key)` up to `h(k-1)(key)`, and set those bits to `1`.

When we look up an entry, we also need to compute the `k` hashes for it as described for `insert`, but this time we check the `k` bits at the indices returned by the hash functions and return `true` if and only if all bits at those positions are set to `1`.

Figure 4.3 shows both operations in action.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F223.png)

Figure 4.3 An example of a Bloom filter. (A) Initially the filter is an array of zeroes. (B) To store an item `Xi`, it is hashed `k` times (in the example, `k=3`), with each hash yielding the index of a bit; then these `k` bits are all set to `1`. Notice how the two triplets of indices generated by `X1` and `X2` partially overlap (they both point to the sixth element). For a more detailed example of how insertion works, see figure 4.5. (C) Likewise, to check if an element `Yi` is in the set, it’s hashed k times, obtaining just as many indices; then we read the corresponding bits, and if and only if all of them are set to `1`, we return `true`. In the bottom drawing, element `Y1` appears to be in the set (but we can’t exclude that the filter is returning a false positive), while element `Y2` can’t be in the set because one of the indices generated from its hashing holds a `0`. Check figure 4.4 for further insight into how lookup works.

Ideally, we would need `k` different independent hash functions, so that no two indices are duplicated for the same value. It is not easy to design a large number of independent hash functions, but we can get good approximations. There are a few solutions commonly used:

- Use a parametric hash function `H(i)`. This meta-function, which is a generator of hash functions, takes in as input an initial value `i` and outputs a hash function `Hi=H(i)`. During the Bloom filter’s initialization, we can create `k` of this functions, `H0` to `Hk-1`, by calling the generator `H` on `k` different (and usually random) values.
    
- Use a single hash `H` function but initialize a list `L` of `k` random (and unique) values. For each entry `key` that is inserted/searched, create `k` values by adding or appending `L[i]` to `key`, and then hash them using `H`. (Remember that well-designed hash functions will produce very different results for small changes in the input.)
    
- Use double or triple hashing.[11](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034762)
    

While the latter won’t guarantee independence among the hash functions generated, it has been proven[12](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034781) that we can relax this constraint with a minimal increase in the false positive rate. To keep things simple, in our implementation we use double hashing with two independent hash functions: Murmur hashing[13](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034797) and Fowler-Noll-Vo[14](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034811) (fnv1) hashing.

The general format of our `i-th` hash function, for `i` between `0` and `k-1`, will be

Hi(key) = murmurhash(key) + i * fnv1(key) + i * i

## 4.6 Implementation

Enough with the theory; now it’s time to once again get our hands dirty! As usual, in the next sections we’ll show pseudo-code snippets and comment the key sections. Trivial methods will be omitted. On the book’s repo on GitHub,[15](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034825) you can also find an implementation with the full code along with unit tests.

### 4.6.1 Using a Bloom filter

Back to our contacts application: How would we use a Bloom filter to make it faster? Well, as mentioned, we need to use it as a dictionary, so we are going to create a new Bloom filter when our email application starts, retrieve all the contacts from the server, and add them to the Bloom filter. Listing 4.1 summarizes this initialization process.

Listing 4.1 Starting up an email application

**function** initBloomFilter(server, minSize)     ❶ 
  contactsList ← server.loadContacts()        ❷
  size ← **max**(2 * |contactsList|, minSize)     ❸
  bloomFilter ← **new** BloomFilter(size)         ❹
  **for** contact **in** contactsList **do**              ❺
    bloomFilter.insert(contact)               ❻
  **return** bloomFilter

❶ Method `initBloomFilter` takes an interface to a server (a façade object) and the minimum size that should be used to initialize the Bloom filter; it returns the newly created Bloom filter.

❷ On startup, it optionally loads the list of contacts from a server that takes care of durable storage.

❸ The size of the Bloom filter should be at least twice the current contact list’s, but at least equal to `minSize`, a minimum value that can be passed as an argument.

❹ Creates an empty Bloom filter with the right size

❺ Cycles through the list of contacts

❻ For each contact, adds it to the Bloom filter

Once we set up our directory application, we have two operations that we are mainly interested in: checking if a contact is on the list and adding a new contact to the directory.

For the former operation, shown in listing 4.2, we can check the Bloom filter, and if it says the contact has never been added, then we have our answer; the contact is not in the system. If, however, the Bloom filter returns `true`, then it might be a false positive, so we need to contact the server to double-check.

Listing 4.2 Checking an email

**function** checkContact(bloomFilter, server, contact)   ❶
  **if** bloomFilter.contains(contact) **then**               ❷
    **return** server.contains(contact)                   ❸
  **else**
    **return false**                                      ❹

❶ Method `checkContact` verifies if an email contact is stored in the application. It takes a Bloom filter, a server façade, and the contact to check. It returns `true` if `contact` is already in our contacts book.

❷ Checks the Bloom filter for the contact passed to the method

❸ If the Bloom filter returned `true`, we need to check if the server actually stores the contact, because it could be a false positive.

❹ Otherwise, since Bloom filters don’t have false negatives (but only false positives), we can return `false`.

For adding new contacts, we always have to sync to our permanent storage,[16](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034841) as shown in listing 4.3. Since this likely implies a remote connection through a network, there is a non-negligible probability that the call to the server fails; therefore, we need to handle possible failures and make sure the remote call succeeds before also updating the Bloom filter.

To be thorough, in real implementations we should also synchronize the access to the server and Bloom filter, using a locking mechanism (see chapter 7), and using a try-catch around the whole operation, rolling back (or retrying) if the call on the Bloom filter fails.

Listing 4.3 Adding a new contact

**function** addContact(bloomFilter, server, contact)    ❶
  **if** server.storeContact(contact) **then**               ❷
    bloomFilter.insert(contact)                      ❸
    **return true**                                      ❹
  **else** 
    **return false**                                     ❺

❶ Method `addContact` adds a new contact to the system; it takes a Bloom filter and a server object, in addition to the new contact to add. It returns `true` if and only if the operation succeeds.

❷ Try to add the contact to the server, and if it succeeds . . .

❸ . . . then add it to the Bloom filter as well...

❹ . . . and return `true`.

❺ Otherwise, the insertion failed, so return `false`.

### 4.6.2 Reading and writing bits

Now, let’s move to the implementation of a Bloom filter, starting, as usual, with the helper methods that will give us the basic blocks to build the implementation of our API.

In particular, we need

- Some way to read and write bits at any location in our filter’s buffer
    
- A mapping between a key in input and the bits’ indices in the buffer
    
- A set of deterministically generated hash functions that will be used to transform keys into a list of indices
    

If we are using Bloom filters to save memory, it wouldn’t make sense to store bits inefficiently. We will need to pack bits into the smallest integer type available in the programming language we choose; therefore, both reading and writing a bit forces us to map the index of the bit to be accessed into a couple of integers.

In modern programming languages, in fact, you can typically speed up these operations by using fixed-size numerical arrays of primitive types and vector algebra. The price to pay is that when we get a request to access the `i`-th bit in the filter, we need to extract two coordinates from index `i`: the array element that is storing the `i`-th bit and the offset of the bit we need to extract with respect to that element.

Listing 4.4 shows what this means and how it can be computed.

Listing 4.4 `findBitCoordinates`

**function** findBitCoordinates(index)           ❶
  byteIndex ← **floor**(index / BITS_PER_INT)    ❷
  bitOffset ← index **mod** BITS_PER_INT         ❸
  **return** (byteIndex, bitOffset)              ❹

❶ Function `findBitCoordinates` is a utility method that, given the index of a bit in a bit array, returns the index of the array and the offset of the bit with respect to the array’s element at that index.

❷ Given the index of the bit to retrieve, we extract the byte index; that is, which element of the array buffer contains the bit to extract. `BITS_PER_INT` is a (system) constant whose value is the number of bits used to store an `int` in the programming language used (for most languages it’s 32).

❸ Extract the bit offset inside the `buffer`’s byte. In other words, to extract the local index of the bit by performing a modulo operation, we need the rest of the division performed at line #2.

❹ Returns the byte index and bit offset as a couple of values. Note that some programming languages allow native structures for tuples; in other languages, you can make do by returning an array with two elements.

Once we have those two indices, we can easily read or write any bit; it just becomes a matter of bitwise arithmetic. For instance, listing 4.5 shows the `readBit` method taking care of the reading part.

Listing 4.5 Method `readBit`

**function** readBit(bitsArray, index)                         ❶
  (element, bit) ← findBitCoordinates(bitsArray, index)    ❷
  **return** (bitsArray[element] & (1 << bit)) >> bit          ❸

❶ Method `readBit` extracts the index-th bit from the bits array passed as first argument. It returns the value of the bit, so either 0 or 1.

❷ Retrieves the element index and offset for the bit in the bits array

❸ Some bitwise algebra to return the value. First retrieve the buffer element, then put it in `AND` with a mask that extracts a single bit (at the right position); finally, it shifts the extracted value back so that the result will either be a 0 or a 1. We could save a left shift by preparing a constant array of `BITS_PER_INT` masks, and use `bit` as an index to decide which mask should be applied.

Listing 4.6 shows the writing counterpart, method `writeBit`. You might be surprised to see that we don’t pass the value to write, but since (this version of) Bloom filter doesn’t support deleting elements, we can only write a 1; we never write zeros.

Listing 4.6 Method `writeBit`

**function** writeBit(bitsArray, index)                        ❶
  (element, bit) ← findBitCoordinates(bitsArray, index)    ❷
  bitsArray[element] ← bitsArray[element] | (1 << bit)     ❸
  **return** bitsArray

❶ Method `writeBit` takes the bits arrays and the index of the bit where a 1 should be written; it returns the bits array after modifying it.

❷ Retrieves the element index and offset for the bit in the bits array

❸ Some more bitwise algebra to store the value. Put the current buffer’s byte in `OR` with a mask having a single 1 in the position where we need to write, and then store the result back on the buffer. If the buffer already had a 1 at the index-th bit, then it won’t change; otherwise, only that bit will be updated. We are assuming, here, that we only write ones, never zeroes (because our version of Bloom filter does not support the delete operation).

Let’s go through an example for `readBit` and `writeBit`. Suppose we have this buffer: `B=[157, 25, 44, 204]`, with `BITS_PER_INT=8`.

We call `readBit(B, 19)`; then we have: `element==2, bit==3`.

Therefore

- `bitsArray[element]              (evaluates to 44)`
    
- `(1 << bit)                      (8)`
    
- `bitsArray[element] & (1 << bit) (8)`
    

And the returned value will be `1`.

If, instead, we call `writeBit(B, 15)`, then we have: `(element==1, bit==7)`.

Therefore

- `bitsArray[element]              (evaluates to 25)`
    
- `(1 << bit)                      (128)`
    
- `bitsArray[element] | (1 << bit) (153)`
    

And the buffer will be updated to `B=[157, 153, 44, 204]`.

### 4.6.3 Find where a key is stored

To generate all the indices for the bits used to store a key, we go through a two-step process, described in listing 4.7.

Keep in mind that our ultimate goal is to transform a string into `k` positions, between `0` and `m - 1`.

First, we use two hash functions on strings very different from each other: _murmur hashing_ and _fnv1 hashing_. The chances that, for a given string, both of them produce the same result are beyond slim.

Then, for each of the `k` bits we have to store, we retrieve the corresponding hash function in our pool. For each position `i` between `0` and `k-1` we have generated (on initialization) a _double hashing_ function `hi`. The `i`-th bit will therefore be returned by `hi(hM, hF)`, where hM is the result of murmur hashing on the input key and hF the result of fnv1 hashing.

Although the highest level of randomization would be obtained with a random seed for each run, we need to have a way to force a deterministic behavior both for tests and to recreate a Bloom filter that can make sense of a given buffer, in case the filter is serialized, or to support quick restart over failure. Therefore, we should also leave the option to pass the seed to the Bloom filter’s constructor.

Listing 4.7 Method `key2Positions`

**function** key2Positions(hashFunctions, seed, key)   ❶
  hM ← murmurHash32(key, seed)                     ❷
  hF ← fnv1Hash32(key)                             ❸
  **return** hashFunctions.map(h => h(hM, hF))         ❹

❶ Method `key2Positions` takes an array of hash functions as input, together with a seed to initialize these functions and the key that will be hashed. It returns the set of bit indices that needs to be updated in the Bloom filter to read/write key.

❷ Applies murmur hashing to the key, with a given seed

❸ Applies fnv1 hashing to the key

❹ We use a functional programming notation here. We create a lambda function taking a hash function `h` as input and applying `h` to the two values generated by murmur and fnv1 hashing. Then we map this lambda function to every element of the `hashFunctions` array. This operation transforms an array of hash functions (taking two integers as arguments and producing an integer as result) into an array of integers.

### 4.6.4 Generating hash functions

In listing 4.7 we described how, in the `key2Positions` method, we pass an array of hash functions and use it to transform a key into a list of indices: the positions in the filter’s bits array where we store the key. Now it’s time to see in listing 4.8 how we initialize these `k` hash functions needed to map each key (already transformed into a string) into a set of `k` indices, pointing to the bits that will hold the information about a key (stored vs not stored).

The set of functions will be created by using double hashing to combine the two arguments in `k` different ways. With respect to linear or quadratic hashing, double hashing will increase the number of possible hashing functions that we can obtain, from `O(k)` to `O(k2)`. Still, this is far from the ideal `O(k!)` guaranteed by uniform hashing, but in practice it is close enough to have good performance (meaning a low collision rate).

Listing 4.8 Method `initHashFunctions`

**function** initHashFunctions(numHashFunctions, numBits)      ❶
  **return range**(0, numHashFunctions).map(i **=>** ((h1, h2) 
  ➥ => (h1 + i * h2 + i * i) **mod** numBits))                ❷

❶ Method `initHashFunctions` takes the number of desired functions and the number of bits held by the Bloom filter and creates and returns a list of double hashing functions, taking two values and returning their hash.

❷ We again use functional notation, applying a lambda function to an array. For convenience, we map from integers `0` to `numHashes-1` (included) into a list of equally many double hashing functions.

### 4.6.5 Constructor

Let’s move on to the public API, which will mirror the API for `set` that we defined in section 4.3.3. Let’s start from the constructor method.

As it happens most of the times, the task for our constructor is mostly boilerplate code to set up all the internal state of a Bloom filter. In this case, however, there is also some non-trivial math to work out in order to compute the resources to allocate in order for the container to live up to the accuracy required by the client.

Listing 4.9 describes the code for a possible constructor. Notice, in particular, at lines #5 and #8 how we compute respectively the number of bits and the number of hash functions needed to have a ratio of false positives within the tolerance specified by `maxTolerance` (and consequently, at line #9, the number of array elements needed to store the filter). Here we assume that we use an array whose elements are _integers_, and `BITS_PER_INT` is a system variable that gives us the size, in bits, of integers. Clearly, for those languages supporting multiple numerical types, we can also choose to have arrays of _bytes_, when available.

Listing 4.9 Bloom filter’s constructor

**function** BloomFilter(maxSize, maxTolerance=0.01, seed=random())        ❶
  **this**.size ← 0                                                        ❷
  **this**.maxSize ← maxSize                                               ❸
  **this**.seed ← seed                                                     ❸
  **this**.numBits ← -**ceil**(maxS * ln(maxTolerance) / **ln**(2) / **ln**(2))        ❹
  **if** numBits > MAX_SIZE **then**                                           ❺
    **throw new**  Error(“Overflow”)                                       ❻
  **this**.numHashFunctions ← -**ceil**(**ln**(maxTolerance) / **ln**(2))              ❼
  numElements ← **ceil**(numBits / BITS_PER_INT)                           ❽
  **this**.bitsArray ← 0 (∀ i ϵ {0, ..., numElements-1})                   ❾
  **this**.hashFunctions ← initHashFunctions(numHashFunctions, maxSize)    ❿

❶ Signature of the constructor method. Argument `maxTolerance` has a default value of 0.01; `seed` is by default initialized to a random integer. Not all programming languages provide an explicit syntax for default values in function signatures, but there are workarounds to cope with those that don’t.

❷ Initially no element is stored in the filter, so the size is initialized to 0.

❸ We store in class variables the (local) arguments for the constructor.

❹ Computes the optimal number of bits needed: `m = - n * ln(p) / ln(2)2; ceil(x)` is the standard ceiling function returning the smallest integer larger or equal to `x`

❺ Checks that the size will fit in memory without issues

❻ We throw an error that can be handled by the client.

❼ Computes the optimal number of hash functions needed. Equivalently, as we’ll see, this can be written in terms of size of the array versus the maximum size of the Bloom filter as: `k = m/n * ln(2)`.

❽ The number of elements for the (integer) buffer is computed by dividing the number of total bits needed by the number of bits per int. Notice how we use the ceiling function here.

❾ Creates the buffer that will store the filter’s bits, all initialized to 0

❿ Creates and stores the hash functions that will be used to get the bit indices for a key

On creation, we need to provide only the maximum number of elements that the filter will be expected to contain. If at any time we realize that we stored in the filter more than `maxSize` elements, the good news is that we won’t run out of space, but the bad news is that we can no longer guarantee the expected precision.

Speaking of which, we can pass an optional second parameter to set the expected accuracy. By default, the threshold for the probability of a false positive (`maxTolerance`) is set to 1%, but we can aim for better accuracy by passing a smaller value, or settle for worse accuracy as a tradeoff for a lower amount of memory needed by passing a higher value.

The last optional parameter is needed, as explained in the previous section, to force a deterministic behavior for the filter. When omitted by the caller, a random value is generated for the seed.

After validating the arguments received (omitted in listing 4.9), we can start setting a few base fields. Then comes the trickiest part; given the number of elements and the expected precision, compute how large our buffer needs to be. We use the formula described in section 4.10, but we need to verify that the size of the buffer is still safe to be held in memory.

Once we have the size of the buffer, we can compute the optimal number of hashes needed to keep the rate of false positives as low as possible.

### 4.6.6 Checking a key

We can now start composing the helper methods presented so far to build the Bloom filter’s API methods. One note of caution: we assume keys are going to be strings, but they can also be serializable objects. If that’s the case, you will need a consistent serialization function that turns equivalent objects (for instance, two sets containing the same elements) into the same string; otherwise, no matter how good your implementation of the Bloom filter (or any other dictionary you might use) is, your application won’t work properly.

NOTE Data massaging and preprocessing are often as important, or more important, than the actual algorithms run on it.

With all the helper functions we already defined, checking the Bloom filter for a key becomes a piece of cake. We just need to retrieve the positions where the bits for the key would be stored, and check that those bits are all 1. To review the whole process, you can take a look at figure 4.4, and find the pseudo-code for the method in listing 4.10.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F224.png)

Figure 4.4 Checking if an entry is in a Bloom filter, step by step. (A) We start with the email we would like to check, `“duffy@acme.com”`. (B) The key (our email) is processed through our set of hash functions. In our example, we assume `k=3`, so we’ll have three different hash functions, `H0`, `H1`, and `H2`. (C) Each hash function produces an index for our binary array. In this case, the three indices might be, for instance, `<9, 6, 2>`. (D) We access the elements of the binary array at those indices. (E) The first element, at index `2`, is `0`. The other bits are `0` and `1` respectively. (F) Since not all the bits we checked were equal to `1`, it means that `“duffy@acme.com”` is not stored in the Bloom filter, and we will return `false`. The same workflow is followed for `“roger@acme.com”`, on the right. Because all three bits checked are set to `1`, we return `true`: this means that `“roger@acme.com”` might have been stored in the Bloom filter, with a certain degree of confidence.

Listing 4.10 Method `contains`

**function** contains(key, positions=**null**)                               ❶
  **if** positions == **null then**                                          ❷
    positions ← key2Positions(**this**.hashFunctions, **this**.seed, key)    ❸
  **return** positions.all((i) **=>** readBit(**this**.bitsArray, i) **!=** 0)       ❹

❶ Function `contains` takes a key and returns `true` if and only if all bits corresponding to the key are set to 1. It is possible to pass explicitly the array of positions. Some operations on the filter require multiple accesses to a location, and this allows saving some computations. In languages allowing private methods and overloading, passing the second parameter should only be allowed for internal methods.

❷ Checks if the array of positions has been passed. If so, avoids computing it again.

❸ Retrieves the bit indices corresponding to current key

❹ Returns `true` if and only if all bits read are not 0. This line also uses a functional notation, with the `all` method that is similar to `map`, except it takes a predicate (a particular lambda function that returns a `Boolean`), applies it to every element in the list, and its result is `true` only if the predicate is `true` for all elements.

You might have noticed that in `contains` we check that the value returned by `readBit` isn’t `0`. While it would technically suffice to check that the bit we read is equal to `1`, this would force us to use an extra right-shift bitwise operation. If our bit is stored in the `i`-th element of the array as its `j`-th bit (from the right), in fact, we should in theory shift the result of our bitwise extraction process `j` bits to the right, or compare it with a mask made of a single `1` shifted `j` positions to the left. This way we don’t need to, and we can trim off a few milliseconds (per operation) from our implementation.

Also notice that the method takes an optional second parameter. The reason will become clear in the next section.

### 4.6.7 Storing a key

Storing a key is pretty similar to checking it, only we need a little extra effort to keep track of the number of elements added to the filter and to use `write` instead of `read`. Figure 4.5 shows a step-by-step example, putting together all the little pieces we have coded so far.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F1.png)

Figure 4.5 Adding a new entry to a Bloom filter, step by step. (A) We start with the email we would like to store, `“duffy@acme.com”`. (B) The key (our email) is processed through our set of hash functions: in our example we assume `k=3`, so we’ll have three different hash functions, `H0`, `H1`, and `H2`. (C) Each hash function produces an index for our binary array. In this case the three indices might be, for instance, `<9, 6, 2>`. Notice how these indices are the same as in figure 4.4: hash functions are deterministic, though their results might seem random. (D) We access the elements of the binary array at those indices. The first element, at index `2`, is `0`. The other bits are `0` and `1` respectively. (E) We flip the bits that were still set to `0:` (in this example, the ones at indices `2` and `9`). Now all the bits to which `“duffy@acme.com”` hashes to are set to `1`, so any future lookup will return `true`.

Note that in this implementation for `insert`, shown in listing 4.11, when we compute the size of the filter, we keep track of the number of unique elements added to the filter, rather than the total number of times the `add` method is called.

Listing 4.11 Method `insert`

**function** insert(key)                                     ❶
  positions ← key2Positions(key)                         ❷
  **if not** contains(key, positions) **then**                   ❸
    **this**.size ← **this**.size + 1      
    positions.map((i) => writeBit(**this**.bitsArray, i));   ❹

❶ Function `insert` takes a key and stores it in the Bloom filter

❷ Transforms the string representation of the key to add into a sequence of `k` bit indices

❸ Before incrementing the size of the filter and storing 1 in each of the bits corresponding to the bits, we check that the key is not already contained by the filter. This is more than just optimization: `size` is crucial to estimating the filter’s false positive ratio, so we need to accurately count the elements actually stored. Notice how we pass the `positions` array as a second argument to `contains`. As mentioned in the previous section, this allows us to avoid computing it again in `contains`, and to perform this expensive operation only once for each call to `insert`.

❹ For each index, we need to write a 1 in our buffer (here, using functional notation, we can leverage `map`, just ignoring its result, or use a more appropriate functional operator like `reduce` or `forEach`).

This is because the precision of the filter would not be altered by adding the same key twice, thrice, or an infinite number of times. To all extents, it would count as if the key was added once.

There is a twist, though. If we add a new unique key `x` for which all bits’ indices clash with locations already set to `1`, it will be treated as a duplicate key and the data structure’s size won’t be incremented. Again, this makes perfect sense if you consider that in that situation and before actually adding the new colliding key, a call to `contains(x``)` would have returned a false positive anyway.

In listing 4.11 you can see the reason why, when we wrote method `contains`, we added the possibility to pass it an array with the pre-computed bit indices for the current key. Inside `insert`, we perform a read closely followed by a write. The operation to compute all the indices for the bits of a key can be expensive, so to avoid repeating this operation twice at a short distance, we need a way to pass its result to `contains`. At the same time, we don’t want to add this option to our API, since this is internal magic that clients don’t need to know about. If your programming language supports polymorphism and private methods, restricting the optional parameter to a private version of `contains` (internally called by the public one) would be wise.

Another possible approach to save this duplicated effort could be having `writeBit` check whether the bit overridden was already set to `1` and return a Boolean to state if the bit’s value has changed. Then `insert` could check to see if at least one bit was flipped. This alternative implementation will be provided in our repo.[17](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034855) It’s up to you to decide which one you consider the cleanest.

Either way, striving to count unique keys added to the filter is going to be costly. Will the overhead be justified? That depends: if you don’t expect many duplicate keys added to your filter, then it’s probably not worth it. But just know that you need it to have an accurate estimate of the current probability to get a false positive.

### 4.6.8 Estimating accuracy

In fact, our last task is to provide a method to estimate the probability of a false positive based on the current status of the filter; that is, on the number of elements currently stored in the filter, compared to its maximum capacity.

As we will see in section 4.10, this probability is roughly[18](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034869)

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ1.png)

Listing 4.12 briefly illustrates the pseudo-code that computes this method.

Listing 4.12 Method `falsePositiveProbability`

**function** falsePositiveProbability()
    **return pow**((1 - pow(E, **this**.numHashes * **this**.size / **this**.numBits)), 
    ➥ **this**.numHashes)

This concludes the current section about how to implement a Bloom filter. Before we delve into the theoretical part explaining the mathematical basis of this data structure, let’s first review some of the many applications for Bloom filters.

## 4.7 Applications

I’d like to let you think about this consideration: it is pretty likely that some software you are using right now is leveraging Bloom filters. In fact, if you are reading the digital version of this book, then it’s 100% sure that its download leveraged Bloom filters, because it’s common for internet nodes to use them for their routing tables.[19](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034887)

### 4.7.1 Cache

With _caching_ we refer to the process of storing some information retrieved on a fast storage system `A`, in order to have it ready in case we need to read it again in the near future. The data might have been previously retrieved from a slow(er) storage system `B` or be the result of a CPU-intensive computation.

For web applications, scalability is a major concern, probably the one aspect that is most debated in design reviews for any product that aspires to become viral.

One of my favorite books tackling design and scalability, _Scalability Rules_ (by Martin L. Abbot and Michael T. Fisher, Addison-Wesley Professional, 2016), has a chapter titled “Use Caching Aggressively,” where it is shown how caching at all levels of a web app is fundamental to allowing its scaling.

Cache is often the only thing saving databases from literally catching fire—or at least crashing. But even your laptop has several levels of caching, from fast L1-cache inside its CPU to in-memory cache used to process big files. Your very operating system caches memory pages in and out of RAM, swapping them to disk, in order to have a larger virtual address space and give you the impression that it has more memory available than is actually installed on your machine.

In other words, caches are one of the foundations of modern IT systems. Of course, since fast storage is costlier, there is only a limited amount of it, and the majority of the data will have to be left out of the cache.

The algorithm used to decide what data stays in the cache determines the behavior of the cache and the rate of cache hit (when the data searched for is already in the cache) and miss. Some of the most used algorithms are _LRU_ (Least Recently Used), _MRU_ (Most Recently Used), and _LFU_ (Least Frequently Used). Chapter 7 gets into the details of these algorithms, but for now it is enough to note that they, as well as many other cache replacement policies, all suffer from “one-hit wonders.” In other words, they struggle with objects, memory locations, or web pages requested just once, and never again (in the average lifetime of the cache). This is particularly common for routers and _Content Delivery Networks_ (CDNs), where an average of 75% of the requests for a node are one-hit wonders.

Using a dictionary to keep track of requests allows us to only store an object in cache when it’s requested for the second time, filtering out one-hit wonders and improving the cache _hit ratio_. Bloom filters allow performing such lookups using amortized constant time operations and with limited space, at the cost of accepting some painless false positives. For this application, the only result of false positives, however, would be a tiny reduction of the cache performance gain we get by using the Bloom filter in the first place (so, no harm done).

### 4.7.2 Routers

Modern routers have limited space and, with the volume of packets they process per second, they need extremely fast algorithms. They are thus the perfect recipient for Bloom filters, for all those operations that can cope with a small rate of errors.

Besides caching, routers often employ Bloom filters to keep track of forbidden IPs and to maintain statistics that will be used to reveal DoS (Denial of Service) attacks.[20](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034903)

### 4.7.3 Crawler

Crawlers are automated software agents scanning a network (or even the entire web) and looking for content, parsing, and indexing anything they find.

When a crawler finds links in a page or document, it is usually programmed to follow them and recursively crawl the link’s destination. There are some exceptions: for instance, most file types will be ignored by crawlers, as will links created using `<a>` tags with an attribute `rel="nofollow"`.

Tip It is actually recommended that you mark in this way any anchor with a link to an action having side effects. Otherwise, search engines’ crawlers, even if they respect this policy, will cause unpredictable behavior.

What can happen is that if you write your own crawler and you are not careful, it might end up in an endless loop between two or more pages with mutual links (or chain of links) to each other.

To avoid such loops, crawlers need to keep track of pages they already visited. Bloom filters, again, are the best way to do so, because they can store URLs in a compact way and perform checking and saving of the URLs in constant time.

The price you pay here for false positives is a bit higher than for the previous examples, because the immediate result will be that the crawler will never visit a URL that caused a false positive.

To overcome this issue, it is possible to keep a complete list of the URLs visited in a proper dictionary (or another kind of collection) stored on disk, and if and only if the Bloom filter returns `true`, double-check the answer in the dictionary. This approach doesn’t allow any space saving, but it provides some savings on the execution time if there is a high percentage of one-hit wonders among the URLs.

### 4.7.4 IO fetcher

Another area where Bloom filter-based caching helps is reducing the unnecessary fetching/storage of expensive IO resources. The mechanism is the same as with crawling: the operation is only performed when we have a “miss,” while “hits” usually trigger a more in-depth comparison (for instance, on a hit, retrieving from disk just the first few lines or the first block of a document, and comparing them).

### 4.7.5 Spell checker

Simpler versions of spell checkers used to employ Bloom filters as dictionaries. For every word of the text examined, a lookup on a Bloom filter would validate the word as correct or mark it as a spelling error. Of course, the false positive occurrences would cause some spelling error to go undetected, but the odds of this happening could be controlled in advance. Today, however, spell checkers mostly take advantage of tries: these data structures provide good performance on text searches without the false positives.

### 4.7.6 Distributed databases and file systems

Cassandra uses Bloom filters for index scans to determine whether an SSTable has data for a particular row.

Likewise, Apache HBase uses Bloom filters as an efficient mechanism to test whether a StoreFile contains a specific row or row-col cell. This in turn boosts the overall read speed, by filtering out unnecessary disk reads of HFile blocks that don’t contain a particular row or row-column.

We are at the end of our excursus on practical ways to use Bloom filters. It’s worth mentioning that other applications of Bloom filters include rate limiters, blacklists, synchronization speedup, and estimating the size of joins in DBs.

## 4.8 Why Bloom filters work[21](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034919)

So far, we have asked you to take for granted that Bloom filters do work as we described. Now it’s time to look more closely and explain why a Bloom filter actually works. Although this section is not strictly needed to implement or use Bloom filters, reading it might help you understand this data structure in more depth.

As already mentioned, Bloom filters are a tradeoff between memory and accuracy. If you are going to create an instance of a Bloom filter with a storage capacity of 8 bits and then try to store 1 million objects in it, chances are that you won’t get great performance. On the contrary, considering a Bloom filter with an 8-bit buffer, its whole buffer would be set to `1` after approximately 10-20 hashes. At that point, all calls to `contains` will just return `true`, and you will not be able to understand whether or not an object was actually stored in the container. Figure 4.6 shows an example of such a situation.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F2.png)

Figure 4.6 Bloom filter saturation. In the example, `m=10`, so the bloom filter has 10 bits, and `k=4`, so each element is stored with 4 bits. We can saturate the filter by adding three elements, setting all bits to `1`. Then even if we look up `Y1` and `Y2`, which were not added to the filter, we still get a false positive. Although the example is an extremization (with `k` unrealistically too large for the value of `m`), it illustrates the mechanism that leads to saturation or to degraded precision.

If, instead, we allocate sufficient space and choose our hash functions well, then the indices generated for each key won’t clash, and for any two different keys the overlap between the lists of indices generated will be minimal, if any.

But how much space is sufficient? Internally a Bloom filter translates each key into a sequence of `k` indices chosen out of `m` possible alternatives:[22](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034937) the trick is that we store keys efficiently by flipping these `k` bits in the filter’s buffer, a bit-array.

If you have brushed up on your school algebra recently, you might have guessed that we can only represent `mk` different sequences of `k` elements drawn from `m` values; we need, however, to make two considerations:

1. We actually can’t even use all these sequences. We would like all indices associated to a key to be different (otherwise, we would actually store less than `k` bits per key), so we strive for all lists of `k` indices to be duplicate-free.
    
2. We are not really interested in the order of these `k` indices. It’s completely irrelevant if we first write the bit at index `0` and then the bit at index `3`, or vice versa. We can thus consider sets instead of sequences.
    

All considered, we only (at least in theory) allow a fraction of all the possible sets of exactly `k` (distinct) indices drawn from the range `0..m-1`, and the number of all these possible (valid) sets is given by

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ2.png)

The binomial coefficient[23](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034953) in this formula expresses the number of ways we can extract `k` (unique) elements from a set of size `m`, and hence it tells us how many sets with exactly `k` distinct indices our `k` hash functions can return.

If we want each key to map to a different set of `k` indices, then given `k` and `n`, the number of keys to store, we can compute a lower bound for `m`, the size of the buffer, by using the formula above.

Another way to look at the question is that given a sequence of `m` bits (the buffer), we can only represent `2m` different values, so at a given time the Bloom filter can only be in one of `2m` possible states; however, this only gives us a loose (although easier to compute) bound on `n`, because we would not take into consideration that we will store `k` bits per key (`2m` becomes an exact bound only for `k==1`).

We’ll see in section 4.10 how to choose the number of hash functions and the size of the array in order to optimize the ratio of false positives for a Bloom filter capable of storing a given number of keys.

### 4.8.1 Why there are no false negatives . . .

In the simplest version of Bloom filters, deletion is not allowed. This means that if a bit is flipped when a key is stored, it will never ever be set to `0` again.

At the same time, the output of the hash functions is deterministic and constant in time.

Tip Remember to pay attention and abide by these properties in your implementation if you need to serialize Bloom filters and deserialize them later.

So, if a lookup finds out that one of the bits associated with a key `X` is set to `0`, then we know for sure that `X` was never added to the filter; otherwise, all the bits to which `X` is hashed would be `1`.

### 4.8.2 . . . But there are false positives

The reverse, though, doesn’t hold up, unfortunately! An example will help you understand why. Suppose we have a simple Bloom filter with 4 bits and 2 hash functions. Initially our buffer is empty:

B = [0, 0, 0, 0]

First, we insert the value `1` into the Bloom filter. Assume we had chosen our hash functions such that `h0(1) = 0` and `h1(1) = 2`, so the key `1` maps to indices `{0,2}`. After updating it, our buffer now looks like this:

B = [1, 0, 1, 0]

Now we insert the value `2`, and it turns out that `h0(2) = 1` and `h1(2) = 2`. Therefore, we now have:

B = [1, 1, 1, 0]

Finally, suppose that `h0(3) = 1` and `h1(3) = 0`. If we perform a lookup for the value `3` after those two insertions, both bits at indices `1` and `0` will have been set to `1` even if we never stored `3` in our instance of the filter! Therefore, `3` would give us a false positive.

On the other hand, if we had a different mapping from the hash functions, for instance, `h0(3) = 3` and `h1(3) = 0`, since the fourth bit hadn’t been set yet `(B[0]==0)`, a lookup would have returned `false`.

Of course, this is a simplistic example intentionally crafted to prove a point: false positives are possible, and they are more likely to happen if we don’t choose the parameters of our Bloom filter carefully. In section 4.10, we’ll learn how to tune these parameters, based on the number of elements we anticipate we will store and on the precision we need.

### 4.8.3 Bloom filters as randomized algorithms

Appendix F introduces the taxonomy of randomized algorithms, and in particular the dichotomy between _Las Vegas_ and _Monte Carlo_ algorithms.

If you don’t have a clear idea of the difference between these two classes or what randomized algorithms are, this would be a good time to check appendix F.

Once those definitions are clear to you, it should not be hard to make an educated guess: Which category do our Bloom filters belong to?

As you might have already figured out, Bloom filters are an example of a Monte Carlo data structure. Method `contains`, the algorithm that checks if a key is stored in a Bloom filter, in particular, is a _false-biased_ algorithm. In fact, it might return `true` for some keys never added to the filter, but it always correctly returns `true` when a key was previously added, so there are no false-negatives (that is, every time it answers `false`, we are sure the answer is correct).

NOTE Bloom filters are also a tradeoff between memory and accuracy. The deterministic version of a Bloom filter is a _hash set_.

## 4.9 Performance analysis

Before starting on Bloom filter analysis, I suggest a deep dive into metrics for classification algorithms in appendix F.

We have seen how and why Bloom filters work; now let’s see how efficient they are. First, we’ll examine the running time for the most important operations provided by a Bloom filter. Next, in section 4.10, we will tackle the challenge of predicting the precision of method `contains`, given a Bloom filter with a certain structure (in particular, its size and the number of hash functions used will matter).

### 4.9.1 Running time

We have already hinted at the fact that Bloom filters can store and look up a key in constant time. Technically, this is only true for constant-length inputs; here we will examine the most generic case, when keys stored are strings of arbitrary length.[24](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034975)

But let’s start from the beginning: the very construction of a Bloom filter. Afterwards we’ll examine in detail `insert` and `contains`.

### 4.9.2 Constructor

Construction of a Bloom filter is pretty easy; we just have to initialize an array of bits, with all its elements set to `0`, and generate the set of `k` hash functions. We have seen that the implementation also involves some computation, but it’s safe to mark that part as constant time.

Creating and initializing the array requires, obviously, `O(m)` time, while generating each of the hash functions typically requires constant time; hence `O(k)` time overall is needed to generate the whole set.

The whole construction can be ultimately finished in `O(m+k)` steps.

### 4.9.3 Storing an element

For each key to store, we need to produce `k` hash values, and flip a bit in each of the elements of the array indexed by those results.

We’ll make the following assumptions:

- Storing a single bit requires constant time (possibly including the time needed for bitwise operations in case we use compressed buffers to save space).
    
- Hashing a key `X` requires `T(X)` time.
    
- The number of bits used to store a key doesn’t depend on the key’s size, or on the number of elements already added to the container.
    

Given these assumptions, the running time for `insert(X)` is `O(k*T(|X|))`. For each of the `k` hash functions we need, in fact, to generate a hash value from the key and store a single bit. If keys are numbers—for instance, integers or doubles—then `|X|=1` and `T(|X|) = O(1)`, meaning that we can typically generate a hash value in constant time.

If, however, our keys are of variable length—for example, strings—then computing each hash value requires linear time in the length of the string. In this case, `T(|X|) = O(|X|)`, and the running time will depend on the length of the keys we add.

Now, suppose we know that the longest key will have at most `z` characters, where `z` is a constant. Remembering our assumption about the length of keys being independent of anything else, we can then still argue that running time for `insert(X``)` is `O(k*(1+z)) = O(k)`. Thus, this is a constant time operation, no matter how many elements have been already added to the container.

### 4.9.4 Looking up an element

The same considerations hold true for the lookup of a key: we need to transform keys into a set of indices (done in time `O(z*k)`), and then check each bit at those indices (`O(k)` time for all of them). So, lookup is also a constant time operation, under the assumption that keys’ lengths are bounded by a constant.

## 4.10 Estimating Bloom filter precision[25](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1034991)

Before we start, we need to fix some notations and make a few more assumptions:

- `m` is the number of bits in our array.
    
- `k` is the number of hash functions we use to map a key to `k` different positions in the array.
    
- Each of the `k` hash functions we use is independent from the others.
    
- The pool of hash functions from which we extract our `k` functions is a universal hashing set.
    

If these assumptions hold, then it can be proven that a good approximation for the false positive probability after `n` insertions is given by

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ3.png)

where `e` is Euler’s number, the base of natural logarithms.

Now we have a formula to estimate the probability of a false positive! That’s nice per se, but what’s even better is that it allows us to tune `m` and `k`, our Bloom filter parameters. This way we can decide the size of the Bloom filter’s buffer and the number of hash functions needed to have optimal precision.

There are three variables in the formula for `p(n, m, k)`:

- `m`, the number of bits in the buffer
    
- `n`, the number of elements that will be stored in the container
    
- `k`, the number of hash functions
    

Of those three, `k` is the one that seems less meaningful to us. Or, from another point of view, it is the one less coupled with our problem. In fact, `n` is probably going to be a variable that can we can estimate, but we can’t fully control. We might need to store as many elements as we get requests for, but most of the time we can anticipate the volume of requests and make estimates pessimistic, to be on the safe side.

We might be limited in the choice of `m` as well, since we could have memory constraints; maybe we can’t use more than `m` bits.

For `k`, instead, we have no constraint on it, and we can tune it to get optimal precision, meaning, as we explained in appendix F, minimal probability of false positives.

Luckily finding the optimal value for `k`, given `m` and `n`, isn’t even that hard—it’s just a matter of finding the minimum for the function:

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ4.png)

Note that `n` and `m` are constants in the formula.

We will go into the details of finding the minimum for `f(k)` in the next section; for now (or if you are more interested in the result), just know that the optimal value is

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ5.png)

Now that we have a formula for `k*`, we can substitute it in our previous formula, the one for `p(n, m, k)`, and after a some algebraic manipulation, we obtain an expression for the optimal value for `m` (let’s call it `m*`):

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ6.png)

This means that if we know in advance the total number `n` of unique elements that we will insert in the container, and we set the probability `p` of a false positive to the largest value that is acceptable, then we can compute the optimal size of the buffer (and the number of bits per key we need to use) in order to guarantee the desired precision.

Two aspects are significant when looking at the formulas we derived:

- The size of the Bloom filter’s buffer is proportional to the number of elements being inserted.
    
- At the same time, the required number of hash functions only depends on the target false positive probability `p` (you can see this by substituting `m*` into the formula for `k*`, but don’t worry, we’ll discuss the math in the next section).
    

### 4.10.1 Explanation of the false-positive ratio formula

In this section we’ll explain in more detail how the formulas to estimate a Bloom filter’s precision are derived; first, let’s see how we obtain the estimate for the false probability ratio.

After a single bit has been stored in a Bloom filter with a capacity of `m` bits, the probability that a specific bit is set to `1` is `1/m`; then the probability that the same bit is set to `0` after all `k` bits used to store an element have been flipped (assuming the hash functions will always output `k` different values for the same input[26](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1035005)) is therefore

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ7.png)

If we consider the events of flipping any specific bit to `1` as independent events, then after inserting `n` elements, for each individual bit in the buffer the probability that the bit is still 0 is given by

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ8.png)

To have a false positive, all the `k` bits corresponding to an element `V` must have been set to `1` independently, and the probability that all of those `k` bits are `1` is given by

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ9.png)

Which is, lo and behold, the probability formula we gave at the beginning of this section.

At this stage, as we mentioned in the last section, we can consider `n` and `m` as constants. It makes sense because in many cases we know how many elements we need to add to the Bloom filter (`n`) and how many bits of storage we can afford (`m`); what we would like to do is trade performance for accuracy by tuning `k`, the number of (universal) hash functions we use.

This is equivalent to finding the global minimum of function `f`, defined as

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ10.png)

If you know some calculus, you probably have already guessed that we need to compute the derivatives of `f` with respect to `k`. (If you don’t know calculus, don’t worry; you can skip the next few lines and resume on the next page, where we will be using the result of this computation.)

To make our job easier, we can rewrite `f` by applying natural logarithm and exponentiation,[27](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1035019) so that we get

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ11.png)

This function is minimal when its exponent is minimal, so we can define function `g`

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ12.png)

and compute the derivatives of `g` instead, which is easier to work with.

The first order derivative of `g(k)` is

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ13.png)

and it becomes equal to `0` for `k=ln(2)*m/n`.

To make sure this is a minimum for the function, we still need to compute the second order derivative and check that it returns a negative value when computed on the _zero_ of `g’`:

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ14.png)

We’ll skip this step for the sake of space, but you can double check that it’s indeed true.

It’s worth noting that

- The formula for `k` gives us a single, exact value for the optimal choice of the number of hash functions.
    
- `k` must obviously be an integer, so the result needs to be rounded.
    
- Larger values for `k` mean worst performance for insert and lookup (because more hash functions need to be computed for each element), so a compromise with slightly smaller values of `k` can be preferred.
    

If we use the best value for `k` as computed previously, this means that the rate of false positives `f` becomes

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ15.png)

By replacing the value `k` in the formula for `p(n,m,k)`, we can get a new formula that links the number of storage bits to the (maximum) number of elements that can be stored, independently of `k` (the value for `k` can be computed later) to guarantee a false-positive probability smaller than a certain value `p`:

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ16.png)

Taking the base 2 logarithm of both sides

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ17.png)

and then solving for `m` finally gives us

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ18.png)

This means that if we know in advance the total number `n` of unique elements that we will insert in the Bloom filter, and we set the false positive probability `p` to the largest value that is acceptable, then we can compute the optimal size of the buffer that guarantees the desired precision. We will also have to set `k` accordingly, using the formula we derived for it:

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ19.png)

For instance, if we would like to have a precision of 90%, and so at most 10% false positives, we can plug in the numbers and get

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781617295485/files/OEBPS/Images/ch04_F_EQ20.png)

## 4.11 Improved variants

Bloom filters have been around for almost 50 years, so it’s natural that many variations and improvements have been proposed in the meantime. Let’s examine some of them, with a focus on those that improve accuracy.

### 4.11.1 Bloomier filter

As we hinted at, Bloom filters corresponds to faster, lighter versions of _HashSets_, since they can only store whether a key is present/absent.

The leaner counterpart of _HashTables_ has been introduced only recently: _Bloomier filters_ [28](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1035051) allow associating values to keys. When the key/value pair has actually been stored in the Bloomier filter, then the value returned is always correct. There are still false positives; that is, keys that were not actually stored in the data structure but for which a value is returned.

### 4.11.2 Combining Bloom filters

By storing the same key in two or more different Bloom filters, possibly with different buffer sizes, but most importantly with a different set of hash functions for each one, we can arbitrarily lower the probability of false positives.

Of course, this is not for free, so the space needed grows proportionally, and the time needed to store or check for a key in theory would double as well.

At least for the running time, though, there is a silver lining: each of the component filters can actually be queried in parallel on a multicore hardware! So, besides maintaining the `O(k)` constant time bound, even actual implementations could possibly be as fast as regular Bloom filters (in other words, the constant factor remains approximately the same).

The way this structure works is the following: for `insert`, each of the components stores the key independently. For calls to `contains`, the answer is the combination of all answers: `true` is returned only if all the components return `true`.

What’s the accuracy of such an array of filters? It can be proven that the precision of a single Bloom filter using `m` bits is the same as the precision of `j` Bloom filters using `m/j` bits each.

However, when using a parallel version of the ensemble algorithm, the running time can be just a fraction of the original one, `1/j`.

### 4.11.3 Layered Bloom filter

A layered Bloom filter[29](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1035068) (_LBF_) also uses multiple Bloom filters, but organized in layers. Each layer is updated for a key only after the previous layer has already stored the same key. Layered Bloom filters are normally used to implement a counting filter: an LBF with `R` levels can count up to `R` insertions of the same key. Often deletion is also supported.

Each call to `contains` starts by checking the layers, from the closest to the deepest, and returns the index of the last layer where the key was found, or `-1` (or, equivalently, `false`) if the key is not stored in the first layer.

When storing a key, the `insert` method stores it in the first layer for which `contains` returns `false`.

Assuming that each layer has a false-positive ratio equal to `PF`, and that all layers use different sets of hash functions, then if an element has been stored `c` times on the filter

- The probability that `contains` returns `c+1`, a counter 1 unit larger than the real number of times a key was stored, is approximately `PF`.
    
- The probability of returning `c+2` is `PF2`.
    
- Similarly, for `c+d`, the probability becomes `PFd`.
    

Those, however, are approximate (optimistic) estimates, because the assumptions about the universality and independence of the hash functions are hard to guarantee. To compute the exact probabilities, we would need to take into account the depth and the number of layers.

On an LBF with `L` layers, each using `k` bits per key, the running time for both `insert` and `contains` becomes `O(L*k)`, but since both `L` and `k` are predetermined constants, it’s still equivalent to `O(1)` and independent of the number of items added to the container.

### 4.11.4 Compressed Bloom filter

When used in web caches, the main issue with Bloom filters is not with the RAM they use, but rather, since these filters need to be transferred between proxies, it’s the size of data transmitted on the network.

While at first you might think that’s a moot point, if we plan to compress the Bloom filters before transmitting them through a network, then it does actually have practical relevance. It turns out we can optimize the values of the filter’s parameters, which in turn regulate the size of the bit array, and as a result get a larger uncompressed Bloom filter that could be compressed more efficiently.

That’s the idea behind compressed Bloom filters,[30](https://learning.oreilly.com/library/view/advanced-algorithms-and/9781617295485/OEBPS/Text/ch04.htm#pgfId-1035082) where the number of hash functions `k` is chosen in such a way that the number of bits with value `1` in the bits array is kept below `m/3`, where `m` is the size of the array. As a consequence, at least `2/3` of the buffer’s bits will always be set to `0`, and we can take advantage of this fact to compress the bit array more efficiently.

Each proxy would then have to decompress the Bloom filter before looking up elements. Clearly now we have another target to optimize. We need to find a compromise between the uncompressed size of the bits array, `m`, and its compressed size, which we want to keep as small as possible.

The uncompressed size, in fact, determines the running time of lookups (through the formulas we developed in section 4.10), while the compressed size determines the transfer ratio.

Since router tables will be updated periodically, decompressing the whole filter could be a heavy overhead for nodes. A good compromise is breaking the filter into pieces and compressing each piece independently. This makes the overall compression rates slightly worse, but when updates are frequent (compared to lookups), it reduces the overhead for decompression even more, since each proxy, between two updates, won’t decompress the whole bits array, but only the chunks it needs.

### 4.11.5 Scalable Bloom filter

A scalable Bloom filter is yet another combination of several Bloom filters, working similarly to a layered Bloom filter. Only this time the different layers have increasing capacity (and hence smaller false positives ratio). This allows the container to adapt dynamically to the number of elements stored and keeps the probability of false positives low.

## Summary

- We can choose the best _abstract data structure_ (_ADT_) to perform certain operations, but we can also choose among different implementations, or concrete data structures (CDT) of an ADT, and that choice also makes a difference.
    
- Many common problems in computer science revolve around keeping track of values. It might be URLs browsed by a crawler, documents examined by an indexer, or values to store in a cache, just to name a few.
    
- Depending on the context, there are different additional constraints for the implementation of a _set_ that we might want to use.
    
- Randomized algorithms are a subset of algorithms whose execution might depend on randomization, and therefore running a randomized algorithm twice on the same input does not always return the same result.
    
- Randomized algorithms are divided into Monte Carlo and Las Vegas algorithms, depending on where the uncertainty lies.
    
- For Bloom filters, we use the _precision metric_ to estimate the ratio of false positives.
    
- If we know in advance the maximum number of items that will be stored in a Bloom filter at the same time, we have an exact formula for computing the total amount of memory needed to achieve an arbitrarily low ratio of false positives.
    

---

1. Chief Technology Officer.

2. _Scale up_ means moving your application to a more powerful, more expensive machine, while with scale out we usually refer to redesigning an application to run in a distributed architecture over several cheaper machines. While there is an upper limit to the possibility of scaling up (the most powerful machine that can be bought by your company—and price grows exponentially in the higher end), with proper design it is possible, in theory, to scale out indefinitely.

3. We used the term associative array here to avoid confusion between the d_ictionary problem_ and the _dictionary_ abstract data type. Though the two terms are connected, they are not the same thing.

4. Amortized time.

5. This reasoning is also the basis for the TF-IDF metric used in text search and text analysis. Short for _term frequency–inverse document frequency_, TF-IDF is computed as the ratio of raw occurrences of a term in a document (TF), over the logarithm of another fraction, the total number of documents in a _corpus_ divided by the number of corpus documents in which the term appears (IDF). This means that TF-IDF will be large when a term appears often in a document but rarely in the corpus, and small when, conversely, the term is used frequently in many documents.

6. Binary search takes its name from the fact that you always search the middle of a list, splitting it into two parts: one before the search location, and one after it. Depending on how the element you are looking for compares to the one in the search location, you then might recursively check the first half or the second half of your list (or neither, if you just found what you were looking for).

7. Not to be confused with the index generated by the hashing function.

8. Amortized time.

9. By storing max/min separately and amortizing the time to replace them on insert/delete.

10. At least not from their basic version, but we’ll see that some variants have been developed to also handle element removal.

11. Double hashing is a technique used to resolve hash collisions (see appendix C, sections C.2.2-C.2.3). When a collision occurs, it adds an offset to the initial position computed by using a secondary hash of the key. Triple hashing computes the offset by using a linear combination of two auxiliary hash functions.

12. Dillinger, Peter C., and Panagiotis Manolios. “Fast and accurate bitstate verification for SPIN.” International SPIN Workshop on Model Checking of Software. Springer, Berlin, Heidelberg, 2004.

13. See [https://sites.google.com/site/murmurhash/](https://sites.google.com/site/murmurhash/).

14. See [http://www.isthe.com/chongo/tech/comp/fnv/index.html](http://www.isthe.com/chongo/tech/comp/fnv/index.html).

15. See [https://github.com/mlarocca/AlgorithmsAndDataStructuresInAction#bloom-filter](https://github.com/mlarocca/AlgorithmsAndDataStructuresInAction#bloom-filter).

16. We could, of course, check our local Bloom filter for the contact, but even when it returns `true`, without checking the server we have no way to know if it’s a false positive!

17. [https://github.com/mlarocca/AlgorithmsAndDataStructuresInAction/blob/master/JavaScript/src/bloom_filter/bloom_filter.js](https://github.com/mlarocca/AlgorithmsAndDataStructuresInAction/blob/master/JavaScript/src/bloom_filter/bloom_filter.js)

18. Here `e` is Euler’s number; we’ll see it again in section 4.10.

19. Bloom filters also are or have been used for a long time in browsers for “safe browsing,” basically keeping a blacklist of malicious sites. Chromium engine, for instance, replaced it with compressed `PrefixSet`s only a few years ago.

20. “Utilizing Bloom filters for detecting flooding attacks against SIP based services.”, Geneiatakis, Dimitris, Nikos Vrakas, and Costas Lambrinoudakis, Computers & Security 28.7 (2009): 578-591.

21. This section, as well as the following ones, are theory-intensive and feature advanced concepts.

22. Assuming `m` is the size of the filter’s buffer.

23. `m` _choose_ `k` is a binomial coefficient; see [https://en.wikipedia.org/wiki/Binomial_coefficient](https://en.wikipedia.org/wiki/Binomial_coefficient).

24. Any object or value can be serialized to a string (for instance, a binary string).

25. This section includes advanced, math-intensive content.

26. In other words, as we have seen in the previous sections, we assume the hash functions are drawn from a set of universal hash functions.

27. `ex` and `ln(x)` are inverse functions, so that for `x>0`, `ln(ex) = eln(x) = x`; that holds true even if `x` is a function (as long as it’s always positive).

28. Chazelle, Bernard; Kilian, Joe; Rubinfeld, Ronitt; Tal, Ayellet (2004), “The Bloomier filter: An efficient data structure for static support lookup tables,” Proceedings of the Fifteenth Annual ACM-SIAM Symposium on Discrete Algorithms (PDF), pp. 30–39.

29. Zhiwang, Cen; Jungang, Xu; Jian, Sun (2010), “A multi-layer Bloom filter for duplicated URL detection”, Proc. 3rd International Conference on Advanced Computer Theory and Engineering (ICACTE 2010), 1, pp. V1–586–V1–591, doi:10.1109/ICACTE.2010.5578947

30. Mitzenmacher, Michael. “Compressed Bloom filters.” IEEE/ACM Transactions on Networking (TON) 10.5 (2002): 604-612.