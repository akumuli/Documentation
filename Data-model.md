Akumuli stores data as a three element tuples. 
* First element is parameter id. This is a 64-bit value that represents value origin. Akumuli doesn't makes any asumptions about this value format. The only limitation is - you can't use values max(uint64_t) and max(uint64_t)-1 are reserved by the system.
* Second element is time-stamp. This timestamp is 64-bit and doesn't need to be real timestamp. This can be any increasing sequence number. Akumuli doesn't assume it to represent time in some format.
* Last parameter is payload, it consist of length value (size - uint32_t) and data itself.

You can write new data elements only if timestamp is within sliding window. Depth of the sliding window is set in configuration. If you're trying to write old data - you will get `AKU_ELATE_WRITE` error.

```cpp
aku_Status status = aku_add_sample(db, paramId, timeStamp, data);
if (status == AKU_ELATE_WRITE) {
    // value must be discarded because it's too old
}
```

All data in akumuli is immutable. You can't change any existing tuple element or delete it.