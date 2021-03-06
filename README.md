# zhashmap

High performance compact C++ hashmaps:

> ### _hashmap_
> - Fast open addressing hash table tuned for small maps such as
>   for histograms and other performance critical maps.
>
> ### _linkedhashmap_
> - Fast open addressing hash table with bidirectional link list
>   tuned for small maps that need predictable iteration order as
>   well as high performance.

These hashmaps are open-addressing hashtables similar to
`google/dense_hash_map`, but they use tombstone bitmaps to
eliminate necessity for empty or deleted key sentinels.

These containers implement most of the C++ unordered associative
container requrements thus can be substituted for `unordered_map`
and typical use cases including C++11 _for-loop_.

There is currently no equivalent to `linkedhashmap` in the STL so it
is perhaps somewhat like an `ordered_map` that uses the _pos_ iterator
hint in the first argument of _insert_ to set the position of a new
value inserted into the linked list, while _find_ uses the hashed key
index to locate _key value pairs_ in the hash table.


## Design 

These maps are designed for a small memory foorprint. There is one
structure and one heap allocation which is divided between an array
of _tuples_ composed of _(Key, Value)_ pairs and a tombstone bitmap.
_Key_ is hashed to index into the open addressing array, and
collisions are handled by skipping to the next available slot.

Deletions requires the use of tombstones. Sentinel key values occupy
key-space, and make containers incompatible with associative container
requirements, thus a 2-bit tombstone array is used. 2-bits are used
to distinguish _available_, _occupied_, and _deleted_ states.

```
    enum bitmap_state {
        available = 0, occupied = 1, deleted = 2, recycled = 3
    };
```

_linkedhashmap_ adds _(next, prev)_ indices to the array _tuple_,
_(head, tail)_ indices to the structure.

### Memory usage

The follow table shows memory usage for the default 16 slot map
_(table units are in bytes)_:

| map                         |     struct | malloc |
|:--------------------------- | ----------:| ------:|
|_hashmap<u32,u32>_           |         40 |    132 |
|_hashmap<u64,u64>_           |         40 |    260 |
|_linkedhashmap<u32,u32>_     |         48 |    260 |
|_linkedhashmap<u64,u64>_     |         48 |    388 |
|_linkedhashmap<u32,u32,i64>_ |         56 |    388 |
|_linkedhashmap<u64,u64,i64>_ |         56 |    516 |

The following table shows the structure size in bytes on _x86_64_:

| map                               | size |
|:----------------------------------| ----:|
|_sizeof(zedland::hashmap)_         |   40 |
|_sizeof(zedland::linkedhashmap)_   |   48 |
|_sizeof(absl::flat_hash_map)_      |   48 |
|_sizeof(std::unordered_map)_       |   56 |
|_sizeof(tsl::robin_map)_           |   80 |
|_sizeof(google::dense_hash_map)_   |   88 |

_linkedhashmap_ by default uses 32-bit integers for indices,
limiting it to 2^31 entries, however, it can be instantiated
with alternative integer types for indices.

### Data structure

These extracts from the source show the array data structure.

#### _hashmap_ parameters and table structure

```
template <class Key, class Value,
          class Hash = std::hash<Key>,
          class Pred = std::equal_to<Key>>
struct hashmap
{
    static const size_t default_size =    (2<<3);  /* 16 */
    static const size_t load_factor =     (2<<15); /* 0.5 */
    static const size_t load_multiplier = (2<<16); /* 1.0 */
    ...
    struct data_type {
        Key first;
        Value second;
    };	
}
```

#### _linkedhashmap_ parameters and table structure

```
template <class Key, class Value, class Offset = int32_t,
          class Hash = std::hash<Key>,
          class Pred = std::equal_to<Key>>
struct linkedhashmap
{
    static const size_t default_size =    (2<<3);  /* 16 */
    static const size_t load_factor =     (2<<15); /* 0.5 */
    static const size_t load_multiplier = (2<<16); /* 1.0 */
    ...
    struct data_type {
        Key first;
        Value second;
        Offset prev;
        Offset next;
    };
}
```

## License

This software is released under the ISC license:

```
Copyright (c) 2020 Michael Clark <michaeljclark@mac.com>

Permission to use, copy, modify, and distribute this software for any
purpose with or without fee is hereby granted, provided that the above
copyright notice and this permission notice appear in all copies.

THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
```