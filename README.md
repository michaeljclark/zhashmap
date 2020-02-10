# zhashmap

Fast open addressing hash table in C++.

## Overview

This hashmap is an open-addressing hashtable similar to
`google/dense_hash_map`, however, it uses tombstone bitmaps
to eliminate necessity for empty or deleted key sentinels.
This means the hash map could potentially be made into a
drop-in replacement for `std::map` or `std::unordered_map`.
The code supports enough of the C++ unordered associative
container requirements such that it can be substituted for
many typical use-cases.

These small maps are useful for histograms and other frequently
accessed data structures where performance is critical.

## Benchmarks

The following benchmarks were performed on an Intel Core i9-7980XE
using GCC 9.2. Note this benchmark is somewhat contrived, but is
intended to measure minimum latency for small histogram hashtables.

#### update 10M mod 16383 records

|container                               |  spread|       count| time_ns|
|:-------------------------------------- |  -----:|       ----:| ------:|
|`zedland::hashmap::operator[]`          |   16383|    10000000|     1.5|
|`google::dense_hash_map::operator[]`    |   16383|    10000000|     2.8|
|`absl::flat_hash_map::operator[]`       |   16383|    10000000|     4.0|
|`std::unordered_map::operator[]`        |   16383|    10000000|     6.4|
|`std::map::operator[]`                  |   16383|    10000000|    55.7|

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
