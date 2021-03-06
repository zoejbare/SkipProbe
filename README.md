# SkipProbe

SkipProbe is a hashing algorithm that uses a variation on Linear Probing, solving the performance issues that Linear Probing sees when collisions happen around keys that are clustered together.

While many hash algorithms are built on an array of linked lists, Linear Probing is built on a flat array, with each "bucket" in the array holding one and only one item. When a collision is detected, the bucket index is incremented until an empty bucket is found and the item is placed in a position that doesn't actually match its hash. If the keys are highly clustered, the nearest empty bucket could be some distance away. When performing a lookup in such a highly-clustered hash table, the lookup starts at the bucket matching the hash and must check every bucket adjacent to it until it either finds the key it's searching for, or finds an empty bucket. And because items can't be moved, if the bucket that matches a key is already filled with a key that doesn't match the bucket, the key must still go in a non-matching bucket - and likewise, any time a key is removed, a linear search must be made through the bucket list moving every item that matches the hash into another bucket to ensure they can still be found when that hash is calculated on a search.

To solve these problems, SkipProbe encodes into each key a "skip pointer", which points at the next item containing a key that matches the bucket's hash. When doing a lookup, the entire array need not be scanned, because these pointers allow the scan to jump around the array finding only items that actually match the hash value.

Likewise, when a new item is inserted and the cell it finds is already occupied, it can skip a lot of searching by jumping to the end of the skip chain before probing linearly. It then alternates skips and probes - skipping to the end of the chain, probing forward one, and then repeating if the new bucket is occupied until it finds an empty bucket. It then encodes the result of that calculation onto the previous item in the chain for its bucket to ensure that lookups don't have to repeat the calculation.

An additional optimization that SkipProbe allows is that it ensures that if an item exists with a matching hash for any given bucket, the item in that bucket will match the bucket's hash, ensuring that lookups only ever have to consider items that actually match the lookup hash. If an insert is going to occur and the desired insert position is occupied by a value with a mis-matched hash, that value is moved to the end of its own skip chain in order to free up the position for the matching element.

As a result, lookups on a SkipProbe have the same big-O complexities (best, worst, and average) as lookups on a traditional array-of-linked-lists hash table, but benefit from ensuring all of the elements in the list are part of one contiguous array, resulting in much better cache coherency and, as a result, better performance.