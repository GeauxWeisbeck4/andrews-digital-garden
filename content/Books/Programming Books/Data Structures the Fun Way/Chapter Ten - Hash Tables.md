---
id: 01J9QF8JJ5E4TYANMTRMRHTEKC
modified: 2024-10-08T21:46:06-04:00
---
This chapter introduces _hash tables_, dynamic data structures hyper-optimized for insertions and lookups. Hash tables use mathematical functions to point us toward the data’s location. They are particularly useful in pure storage cases where the goal is to find and retrieve information quickly.

This is the type of tradeoff we might want to make for our coffee pantry. Forget trying to sort coffees by expiration date or tastiness—we are true coffee aficionados who effortlessly remember the smallest details of every bean in our pantry. For any given attribute (or combination of attributes), we can instantly remember the coffee’s name. By the time we walk to the coffee pantry, we’ve already decided which coffee we want. Storing our coffee selection in sorted order along these other dimensions will just slow us down. What we need is efficient retrieval: given just the name of the coffee we want to drink, we want to find those beans with minimal effort. Hash tables enable exactly this type of rapid retrieval by name.

Arrays provide a compact structure for storing individual pieces of data and a mechanism for efficient retrieval—but only when we know the item’s index. With an index, we can look up any element in constant time. As we saw in Chapter 2, without an index, the process for looking up items in an array becomes more complicated. If we only have the item’s value, then we need to search through the array to find its correct location. We can find items efficiently with binary search only at the cost of keeping the array in sorted order, which makes for inefficient insertions and deletions.

After the past chapters of exploring data structures and algorithms to efficiently search for target values, imagine we could build a magical function that mapped our target value directly to its index (with a few caveats, of course). This is the core idea behind hash tables. Hash tables use mathematical functions to compute a value’s index in our data structure, allowing us to map directly from value to an array bin. The downside is that no mapping is perfect. We will see how different values can map to the same location, causing collisions. We will then examine two approaches to resolving collisions.

As with all data structures, hash tables are not a perfect solution for every problem—we’ll examine both the benefits and the tradeoffs, including the use of memory and worst-case performance. In doing so, we’ll examine a new way to organize data by using mathematical mappings.

## Storage and Search with Keys

Before we delve into the mechanics of hash tables, let’s consider an idealized indexing scheme for efficient retrieval of integer values—maintaining an individual array bin for each _possible_ value and indexing that bin with the value itself. This structure is shown in [Figure 10-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c10.xhtml#figure10-1). To insert the value 9, we simply place it in the bin at index 9. Under this arrangement, we can insert or retrieve items in constant time.

![An array with ten bins numbered 0 to 9. Bins 1, 5, and 7 have arrows to data outside the array. The remaining bins have slashes to indicate they are empty.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c10/f10001.png)

Figure 10-1: A large array with a bin for every potential entry

The obvious downside of our idealized data structure is the absurd cost of maintaining an array of every possible key. Consider the case of storing all possible 16-digit credit card numbers. We’d need several quadrillion bins, 1016 to be exact. That is a lot of memory. Worse, it’s unlikely that we would even use this many bins. If we are writing a program to track the corporate credit cards for a 1,000-person company, we need only a tiny fraction of the available bins—one bin out of every 1013 we’ve allocated. The rest are wasted. They sit empty, hoping that someday they’ll have data to store. Similarly, we wouldn’t want to reserve a spot in the library for every possible book, a room in a hotel for every possible patron, or a spot in our coffee pantry for every known coffee. That’d be absurd (except maybe for the coffee).

But, as a thought experiment, let’s consider how this idealized data structure could work for other types of data. We immediately run into the question of what value to use for more complex data types, such as strings or even composite data types. Suppose we’re looking to create a simple database of coffee records. Chapter 3 showed how to use an array of pointers to store such dynamically sized data, as in the following code:

```
CoffeeRecord {
    String: Name
    String: Brand
    Integer: Rating
    FloatingPoint: Cost_Per_Pound
    Boolean: Is_Dark_Roast
    String: Other_Notes
    Image: Barcode
}
```

We could still place all our items in a single massive array with one bin for every _possible_ entry. In this case, the bin contains not just a single value but a pointer to a more complex data structure, as in the array of pointers in [Figure 10-2](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c10.xhtml#figure10-2).

![An array for ten bins, numbered 0 through 9. Each bin contains an arrow pointing to data outside the array.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c10/f10002.png)

Figure 10-2: An array of pointers

However, this still leaves the question of how to do the actual lookup. If we want to find the rating we gave for “Jeremy’s Gourmet High-Caffeine Experience: Medium Roast,” one of the coffees introduced in Chapter 6, we cannot use the entire composite data structure as the value. We do not have all this information on hand. Even if we did have the full information on hand, it is not clear how we would use a composite data structure or even a string as an index.

Computer programs often use keys to identify records. A _key_ is a single value stored with or derived from the data itself that can be used to identify a record. In the case of an RSVP list, the key might be the string containing the invitee’s name; in the case of our coffee records, the key could be the name or barcode of the coffee. In many data structures, from sorted arrays to tries, we use the key to organize the data. For the numerical examples earlier in this book, the key is just the value itself. Each search of a sorted array or binary search tree for a specific numeric value corresponded to retrieving a record by looking for a matching numeric key. Similarly, the tries introduced in Chapter 6 use a string for the key.

This does not solve the indexing problem though. Unless we have an integer key, we still cannot index the array bin. Arrays do not have a bin with index “Jeremy’s Gourmet High-Caffeine Experience: Medium Roast.” We can search over our data structure, looking for a record with a matching key. Linear scan and binary search both work this way, using a target value as the key. However, we’ve lost the magic of our idealized data structure. We are back to searching for a matching key.

In some cases, we might be able to find a natural numeric key for our records. In the coffee example, we could list every coffee we’ve ever tasted in order of when we first tasted it and use the corresponding date as a key. If “Jeremy’s Gourmet High-Caffeine Experience: Medium Roast” was first sampled on January 1, 2020 (and we magically remember that), we could retrieve the record with a binary search. Alternatively, we could use the coffee’s barcode or its page number in the _Compendium of World Coffees, Brands, and Manufacturers_.

More generally, we want a function that generates an index from our key. In the next section, we introduce hash functions, which solve exactly this problem.

## Hash Tables

Hash tables use mathematical mappings to compress the key space. They squish large key spaces into a small domain by using a _hash function_ to map the raw key into the location in the table (also called the _hash value_). We denote the hash function that maps key _k_ into a table with _b_ bins as a function _f_(_k_) with range [_0_, _b_ − 1]_._ This mapping solves both problems from the previous section. We no longer require an infinite number of bins. We just need _b_ of them. As we will see, functions can also map non-integers onto a numerical range, solving the problem of non-integer keys.

A simple example hash function for integer keys is to use the division method to compute the hash value from the numeric key. We divide the key by the number of bins and take the remainder:

_f_(_k_) = _k_ % _b_

where _%_ is the modulo operation. Every possible (integer) key is mapped to a single bin within the correct range [_0_, _b_ − 1]_._ For example, for a 20-bin hash table, this function would produce the mappings shown in [Table 10-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c10.xhtml#table10-1).

Table 10-1: Example Mappings for the Division Method of Hashing with 20 Bins

|**_k_**|**_f_(_k_)**|
|---|---|
|5|5|
|20|0|
|21|1|
|34|14|
|41|1|

Consider the problem of mapping the space of all credit card numbers into 100 bins. The division method compresses the key space from 16 digits to 2 digits using the last 2 digits of the card’s number. Of course, this simplistic mapping might not produce the best results for some key distributions. If we have many credit cards ending in 10, they will all map to the same bin. However, it solves one of our core problems: with a single (and efficient) mathematical computation, we’ve compressed a large range of keys to a limited number of indices.

The skeptical reader might balk at the description above: “We can’t store two different items in the same element of the array. You told us so in Chapter 1. And hash functions can clearly map two different values to the same bin. Just look at [Table 10-1](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c10.xhtml#table10-1). Both 21 and 41 map to bin 1.” This is the aforementioned caveat. Unfortunately, hash functions are not truly magical. As we will see in the next section, this complexity is where the rest of the hash table’s structure comes in—to handle collisions. For now, we can note that the hash function partitions our keys into disjoint sets and we only need to worry about collisions within a set.

Hash functions are not limited to numbers. We can similarly define a hash function that maps a coffee’s name to a bin. This allows us to directly access the coffee record for any entry in two steps, as shown in [Figure 10-3](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c10.xhtml#figure10-3). First, we use the coffee’s name to compute its hash value. The key “House Blend” is mapped to a value of 6; we will describe a simple method for hashing strings later the chapter. Second, we look up the hash value in our table by using the hash value as an index into our array. We could even use this scheme to map our extensive real-world coffee collection to a fixed number of shelves in our coffee pantry.

![The left side shows a list of coffee names, such as “House Blend” and “Morning Shock.” The Middle column holds hash functions  that mapping the string to a numeric value. House Blend maps to 6. Arrows point from the hash functions to the corresponding bin in the array in the right column.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c10/f10003.png)

Figure 10-3: Hash functions mapping strings to indices within an array

One real-world example of hash tables is the registration tables we see throughout our lives, whether at the first day of summer camp, the morning of a race, or the beginning of an academic conference. The items to be stored (registration packets) are partitioned into unique bins based on their key (the name). People can find their correct bin by applying a hash function that is usually as simple as a sign mapping a range of the alphabet to a given line. Names starting with A−D go to line 1, names with E−G go to line 2, and so forth.

### Collisions

Even the best hash functions in the world won’t provide perfect one-to-one mappings of keys to bins. To do that, we’d need to return to our ginormous array and its excessive use of memory. Any mathematical function that maps a large set of keys to a much smaller set of values will encounter occasional _collisions_—instances where two keys map to the same hash value. Imagine applying this approach to mapping license plates to 10 parking spaces by taking the first number on the license plate. We don’t need our coworker’s license plate to exactly match our own in order to start a fight over a parking space. Imagine you go to register your car, with the plate “Binary Search Trees Are #1,” only to find that your coworker has already claimed the spot with the plate “100,000 Data Structures and Counting.” Both plates may happen to hash to 1, so they would be assigned the same spot.

We have lines at a conference’s registration table because of collisions. Consider a conference registration that hashes into eight lines based on the first letter of your last name. Everyone with a name starting with A−D goes to table 1, everyone with a name starting with E−G goes to table 2, and so forth. If we have more than a handful of attendees, we’re almost guaranteed to see a collision. If there were no collisions, everyone would have their own place to check in. Instead, attendees with surnames A−D wait in the same line because their keys (last names) collide.

We can alleviate some of the collisions by increasing the size of our hash table or by choosing a better hash function. However, as long as our key space is larger than the number of bins, it’s not possible to eliminate collisions altogether. We need a way to gracefully handle two pieces of data fighting to sit in the same place. If this were a kindergarten class, we might be able to employ such strategies as “Ann was sitting there first,” or “You need to learn to share.” None of these approaches work in data structure context. We can’t ignore new keys or overwrite the older data. The point of a data structure is to store all the requisite data. In the next two sections, we consider chaining and linear probing, two common approaches for handling collisions in a hash table.

### Chaining

Chaining is an approach for handling collisions within a hash table by employing additional structure _within_ the bins. Instead of storing a fixed piece of data (or a pointer to a single piece of data) in each bin, we can store the pointer to the head of a linked list:

```
HashTable {
    Integer: size
    Array of ListNodes: bins
}
```

where

```
ListNode {
    Type: key
    Type: value
    ListNode: next
}
```

These lists are like our line of conference attendees. Each person in line is a unique individual, but maps to the same registration table.

As shown in [Figure 10-4](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c10.xhtml#figure10-4), each bin’s list contains all the data that mapped to that bin. This allows us to store multiple elements in each bin. Each item in our linked list corresponds to one of the elements inserted into the bin.

![An array with 5 bins is arranged vertically on the left. Each bin has an arrow pointing from it, indicating a list. The arrow from the first bin points to the start of a linked list with three nodes.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c10/f10004.png)

Figure 10-4: A hash table using a linked list to store entries within the same bin

The code for inserting a new item into a hash table with chaining is relatively simple:

```
HashTableInsert(HashTable: ht, Type: key, Type: value):
  ❶ Integer: hash_value = HashFunction(key, ht.size)

    # If the bin is empty, create a new linked list.
  ❷ IF ht.bins[hash_value] == null:
        ht.bins[hash_value] = ListNode(key, value)
        return

    # Check if the key already exists in the table.
  ❸ ListNode: current = ht.bins[hash_value]
    WHILE current.key != key AND current.next != null:
        current = current.next
    IF current.key == key:
      ❹ current.value = value
    ELSE:
      ❺ current.next = ListNode(key, value)
    return
```

We start by computing the hash value for the `key` ❶ and checking the corresponding bin. If the bin is empty (the pointer is `null`), we create a new linked list node holding the `key` and `value` inserted ❷. Otherwise, we need to scan through the bin’s linked list and check each element for a matching key ❸. The `WHILE` loop checks that we have neither found the correct key (`current.key != key`) nor run off the end of the list (`current.next != null`). If the list already contains a matching key, we update the value associated with the key ❹. Otherwise, we append the new key and its corresponding value to the end of the list ❺.

Search follows a similar approach. However, the logic is simpler because we no longer need to insert new nodes:

```
HashTableLookup(HashTable: ht, Type: key):
  ❶ Integer: hash_value = HashFunction(key, ht.size)
  ❷ IF ht.bins[hash_value] == null:
        return null

    ListNode: current = ht.bins[hash_value]
  ❸ WHILE current.key != key AND current.next != null:
        current = current.next
    IF current.key == key:
      ❹ return current.value
  ❺ return null
```

The code for search starts by computing the hash value for the `key` ❶, checking the corresponding bin, and returning `null` if the bin is empty ❷. Otherwise, it scans over each element of the linked list using a `WHILE` loop ❸ and returns the value for the matching key ❹. If we make it to the end of the list without finding a matching key, the code returns `null` to indicate that the key is not present in the table ❺.

Finally, when removing an item, we need to find it in the list and, if it is present, splice it out. The following code both removes and returns the linked list node matching the target key:

```
HashTableRemove(HashTable: ht, Type: key):
  ❶ Integer: hash_value = HashFunction(key, ht.size)
    IF ht.bins[hash_value] == null:
        return null

    ListNode: current = ht.bins[hash_value]
    ListNode: last = null
  ❷ WHILE current.key != key AND current.next != null:
        last = current
        current = current.next
  ❸ IF current.key == key:
        IF last != null:
          ❹ last.next = current.next
        ELSE:
          ❺ ht.bins[hash_value] = current.next
        return current
    return null
```

The code again starts by computing the hash value for the `key`, checking the corresponding bin, and returning `null` if it is empty ❶. If the bin is not empty, we scan through it using a `WHILE` loop and look for a matching key ❷. In order to splice out the correct element, we need to track one additional piece of information: the final linked list node before the current node. If we find a match ❸, we need to check whether we are removing the first element in the list (`last` is `null`). If not, we can modify the `last` node’s `next` pointer to skip the node we are removing ❹. Otherwise, we need to modify the pointer from the start of the hash bin to skip the node ❺. Finally, we return `null` if we do not find a matching node.

The skeptical reader might pause here and ask, “How does this help? We still must scan through a bunch of elements of a linked list. We have lost the ability to directly map to a single entry. We’re back where we started.” However, the primary advantage to this new approach is that we are no longer scanning through a linked list of _all_ the entries. We only scan through those entries whose hash values match. Instead of searching through a giant list, we search through a single tiny list for this bin. In our coffee pantry, where our hash function maps the coffee’s name to its corresponding shelf, we might be able to cull our search from 1,000 varieties to the 20 varieties on that one shelf. Back in the computational realm, if we maintain enough bins in our hash table, we can keep the size of these lists small, perhaps with only one or two elements.

Of course, the worst-case time for a lookup can be linear in the number of elements. If we choose a terrible hash function, such as _f_(_k_) = 1, we’re basically implementing a single linked list with extra overhead. It’s vital to be careful when selecting a hash function and sizing the hash table, as we’ll discuss later.

### Linear Probing

An alternate approach to handling collisions is to make use of adjacent bins. If we are trying to insert data into a bin that already contains another key, we simply move on and check the next bin. If that neighbor is also full, we move onto the next. We keep going until we find an empty bin or a bin containing data with the same key. Linear probing extends this basic idea to the hash table’s operations. We start at the index corresponding to the key’s hash value and progress until we find what we are looking for or can conclude it is not in the hash table.

Hash tables using linear probing need a slightly different structure. Because we are not using a linked list of nodes, we use a wrapper data structure to store the combination of keys and values:

```
HashTableEntry {
    Type: key
    Type: value
}
```

We also include an additional piece of data in the hash table itself, the number of keys currently stored in the table:

```
HashTable {
    Integer: size
    Integer: num_keys
    Array of HashTableEntry: bins
}
```

This information is critical, because when the number of keys reaches the size of the hash table, there are no available bins left. Often hash tables will increase the array size when they start to get too full, although care must be taken here. Because we are using a hash function that maps keys onto the range of the current array, keys may map to different locations in a larger array. In this section, we will only consider a simplified implementation of a fixed size table for illustration purposes.

Consider a hash table with linear probing where we have inserted a few of our favorite coffees, as shown in [Figure 10-5](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c10.xhtml#figure10-5).

![The left column shows a list of three keys: Morning Shock, Liquid Alarm Clock, and Pure Caffeine. Arrows map these keys to hash functions and then to the corresponding bins in the array. Morning Shock maps to bin 1, Liquid Alarm Clock to bin 2, and Pure Caffeine to Bin 5.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c10/f10005.png)

Figure 10-5: A hash table with three entries

After we have inserted these initial three entries, we try to insert “Morning Zap” as shown in [Figure 10-6](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c10.xhtml#figure10-6). The insertion function finds another key, Morning Shock, in bin 1. It proceeds to bin 2, where it finds Liquid Alarm Clock. The insertion function finally finds an opening at bin 3.

![The left column shows a single key, Morning Zap, which maps to a hash value of 1. In the array on the right, bins 1 and 2 contain other keys. Arrows show our search through the array for an empty bin, found at bin 3.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c10/f10006.png)

Figure 10-6: Inserting the entry “Morning Zap” requires scanning through several bins in the hash table.

The code for inserting items into a fixed-size hash table is shown below. As noted previously, it is possible to increase the size of the array when the hash table is full, but this adds complexity to ensure the items are mapped correctly to new bins. For now, we will return a Boolean to indicate whether the item could be inserted.

```
HashTableInsert(HashTable: ht, Type: key, Type: value):
  ❶ Integer: index = HashFunction(key, ht.size)
  ❷ Integer: count = 0

    HashTableEntry: current = ht.bins[index]
  ❸ WHILE current != null AND current.key != key AND count != ht.size:
        index = index + 1
      ❹ IF index >= ht.size:
            index = 0
        current = ht.bins[index]
        count = count + 1

  ❺ IF count == ht.size:
        return False

  ❻ IF current == null:
        ht.bins[index] = HashTableEntry(key, value)
        ht.num_keys = ht.num_keys + 1
    ELSE:
      ❼ ht.bins[index].value = value
    return True
```

The code starts the search at the new key’s hash value ❶. The code also maintains a count of bins it has checked to prevent infinite loops if the table is full ❷. This count is not needed if we use resizing or otherwise ensure there is always at least one empty bin. The code then loops through each bin using a `WHILE` loop ❸. The loop tests three conditions: (1) whether it found an empty bin, (2) whether it found the target key, and (3) whether it has searched all the bins. The first and third conditions test whether the key is not in the table. After incrementing the index, the code checks whether the index should wrap back to the beginning of the table ❹, which allows the code to search the entire table.

After the loop completes, the code first checks whether it has examined every bin in the table without finding the key. If so, the table is full and does not contain the key ❺, so the code returns `False`. The code then checks whether it has found an empty bin or matching key. If the bin is empty ❻, a new `HashTableEntry` is created and stored. Otherwise, the value of the entry with the matching key is updated ❼. The code returns `True` to indicate that it successfully inserted the key and value.

Search follows the same pattern. We start at the index of the key’s hash value and iterate over bins from there. At each step, we check whether the current entry is empty (`null`), in which case the key is not in the table, or whether the current entry matches the target key.

```
HashTableLookup(HashTable: ht, Type: key):
  ❶ Integer: index = HashFunction(key, ht.size)
  ❷ Integer: count = 0

    HashTableEntry: current = ht.bins[index]
  ❸ WHILE current != null AND current.key != key AND count != ht.size:
        index = index + 1
      ❹ IF index >= ht.size:
            index = 0
        current = ht.bins[index]
        count = count + 1

    # Return the value if we found a match.
  ❺ IF current != null AND current.key == key:
        return current.value
  ❻ return null
```

The code starts by computing the hash value for the `key` to get the starting location of the search ❶. As with insertion, the code also maintains a count of bins it has checked to prevent infinite loops if the table is full ❷. The code then loops through each bin using a `WHILE` loop ❸. The loop tests three conditions: (1) whether it found an empty bin, (2) whether it found the target key, and (3) whether it has searched all the bins. After incrementing `index`, the code tests whether the search has run off the end of the table and, if so, wraps the search back to the start ❹. Once the loop terminates, the code checks whether it has found the matching key ❺. If so, it returns the corresponding value. Otherwise, the key is not in the table, and the code returns `null` ❻.

In contrast to search and insertion, deletion with linear probing requires more than a simple scan. If we remove an arbitrary element such as “Liquid Alarm Clock,” shown in [Figure 10-6](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c10.xhtml#figure10-6), we might break the chain of comparisons needed for other elements. If we replace “Liquid Alarm Clock” with null in [Figure 10-6](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c10.xhtml#figure10-6), we can no longer find “Morning Zap.” Different implementations use different solutions to this problem, from scanning through the table and fixing later entries to inserting dummy values into the bin.

The advantage of linear probing over chaining is that we make fuller use of the initial array bins and do not add the overhead of linked lists within the bins. The downside is that, as our table gets full, we might iterate over many entries during a search, and, unlike with chaining, these entries are not restricted to ones with matching keys.

## Hash Functions

The difference between a good and bad hash function is effectively the difference between a hash table and a linked list. We’d want our hash function to map keys uniformly throughout the space of bins, rather than pile them into a few overloaded bins. A good hash function is like a well-run conference registration. Any clumping of attendees will lead to collisions, and collisions lead to more linear scanning (more time waiting in line). Similarly, in our conference registration example, bad hash functions, such as dividing the table into two lines for names starting with A−Y and names starting with Z, leads to long waits and annoyed attendees.

In addition, any hash function must be meet a couple of key criteria. It must:

1. Be deterministic The hash function needs to map the same key to the same hash bin every time. Without this attribute, we might insert items into a hash table only to lose the ability to retrieve them.
2. Have a predefined range for a given size The hash function needs to map any key into a limited range, corresponding to the number of hash buckets. For a table with _b_ buckets, we need the function to map onto the range [0, _b_ − 1]. We’d also like to be able to vary the hash function’s range with the size of our hash table. If we need a larger table, we need a correspondingly larger range to address all the buckets. Without this ability to adjust the range, we may be stuck with a limited number of viable hash table sizes.

In our conference registration example, these criteria correspond to people being able to find their packets (deterministic for all users) and having everyone map to a line (correct range). It would be wasteful to set up a conference check-in with lines for empty parts of the range (all names starting with Zzza through Zzzb), and it would be rude to map some people’s names to no line at all (no line for names starting with K). The best hash functions are like in-person organizers, holding clipboards and directing people to the correct lines.

### Handling Non-numeric Keys

For numeric keys, we can often use a range of mathematical functions, such as the division method above. However, we may often need to handle non-numeric keys as well. Most importantly, we need to be able to compute the hash of a string containing the coffee name. The general approach to handling non-numeric keys is to use a function that first transforms the non-numeric input into a numeric value. For example, we could map each character to a numeric value (such as the ASCII value), compute the sum of values in a word, and modulo the resulting sum into the correct number of buckets. This approach, while straightforward and easy to implement, may not provide a good distribution of hash values due to how letters occur. For example, the hash function does not take into account the order of the letters, so words with the same letters, such as _act_ and _cat_, will always be mapped to the same bin.

A better approach to hashing strings is an approach often called _Horner’s method_. Instead of directly adding the letters’ values, we multiply the running sum of letters by a constant after each addition:

```
StringHash(String: key, Integer: size):
    Integer: total = 0
    FOR EACH character in key:
        total = CONST * total + CharacterToNumber(character)
    return total % size
```

where `CONST` is our multiplier, typically a prime number that is larger than the largest value for any character. Of course, by multiplying our running sum, we greatly increase the size of the value, so we need to be careful to handle overflow.

There are a wide variety of hash functions, each with their own trade-offs. A full survey of the potential hash functions and their relative merits is a topic worthy of its own book; this chapter presents just a few simple functions for the point of illustration. The key takeaway is that we can use these mathematical functions to reduce the range of our key space.

### An Example Use Case

Hash tables are particularly useful when tracking a set of items, which is why Python uses them to implement data structures such as dictionary and set. We can use them to aid in tracking metadata for the searches such as those seen in Chapter 4.

In both the depth-first and breadth-first search, we maintained a list of future options to explore. We used a queue for breadth-first search and a stack for depth-first. However, there is an additional set of information we need to track: the set of items we have already visited. In both searches, we avoid adding items to the list that we have already seen, allowing us to avoid loops or, at least, wasted effort. Hash tables provide an excellent mechanism for storing this set.

Consider the sixth step of our example breadth-first search from Chapter 4, as shown on the left side of [Figure 10-7](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c10.xhtml#figure10-7). The search has already visited the gray nodes (A, B, F, G, H, and I), and the circled node (G) is our current node. We do not want to re-add either of G’s two neighbors to our list of items to explore; we have already visited both F and I.

![Diagram of a breadth‐first search in progress on the left. A hash table on the right stores the visited nodes.](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781098156602/files/image_fi/502604c10/f10007.png)

Figure 10-7: The visited nodes of a breadth-first search tracked in a hash table

We could store the visited items in a hash table as shown on the right of [Figure 10-7](https://learning.oreilly.com/library/view/data-structures-the/9781098156602/c10.xhtml#figure10-7). This hash table uses a simple function: the division function maps the letter’s index in the alphabet into 5 bins. The letter _A_ has index 0 and maps to bin 0. The letter _G_ has index 6 and maps to bin 1. We insert a key for each item when we visit it. Before adding new items to our list of future topics, we check whether their key is in the hash table. If so, we skip the addition. Hash tables are particularly well suited to tracking this type of data because they are optimized for fast insertions and lookups.

## Why This Matters

Hash tables provide a new way of organizing data by using a mathematical function. As opposed to tree-based structures, which structure the data around the keys themselves, hash tables introduce the intermediate step of mapping the keys to a reduced range. As with all things in computer science, this mapping comes with tradeoffs. Collisions mean that we can’t always access the desired data directly. We must add another level of indirection, such as a linked list, to store items that map to the same location. As such, hash tables provide another example of how we can creatively combine data structures (in this case, an array and linked lists).

Understanding hash functions and how they map keys to bins provides a side benefit. We can use these types of mappings to partition items or spread out work, such as the lines at a conference registration or the coffees on our shelves; in the computational domain, we might use hashing to assign tasks to servers in a simple load balancer. In the next chapter, we see how hash tables can be used to create caches. Hash tables are used throughout computer science, since they provide a handy data structure with (on average) fast access time and reasonable memory tradeoffs. They are a vital tool for every computer scientist’s toolbox.