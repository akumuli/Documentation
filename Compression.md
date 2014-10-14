Akumuli can compress data using in-house compression algorithm. It uses Delta-RLE for timestamps and offsets, base-128 variable width encoding for parameter ids and RLE encoding for element lengths. From user perspective this means that you pay only for things that you use. 
* If you write only data-elements of the same size - only a few bytes will be wasted to store element sizes and offsets.
* If you write elements periodically and don't use full timestamp precision - only fiew bytes will be used to represent timestamps. For example: you write one element every millisecond; first, delta algorithm will transform increasing timestamp sequence into sequence of 1ms values, after that, RLE encoding very effectively compresses this sequence.
* If you only use small parameter ids - this data will be compressed more effeciently.

Akumuli doens't compress payload. It only compresses timestamps, parameter ids, sizes and offsets! This will be addressed in future releases.

Compression can't be disabled. You can only control one compression parameter - `compression_threshold`. This parameter sets the minimum number of elements that can be compressed (this only makes sense for slow insert rates).
