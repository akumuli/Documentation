Akumuli can compress data using in-house compression algorithm. It uses Delta-RLE for timestamps and offsets, base-128 variable width encoding for parameter ids and RLE encoding for element lengths. From user perspective this means that you pay only for things that you use. 
* If you write only data-elements of the same size - only fiew bytes will be wasted to store element sizes and offsets.
* If you write elements periodically and doesn't use full timestamp precission - only fiew bytes will be used to represent timestams. For example: you write element every ms, at first delta algorithm will transform increasing timestamp sequence to sequence of 1ms values, after that RLE encoding very effectively compress this sequence.
* If you use only small parameter ids - this data will be compressed more effectevly.

Akumuli doens't compress payload. It's only compress timestamps, parameter ids, sizes and offsets! This will be targeted in future releases.

Compression can't be disabled. You can control only one compression parameter - `compression_threshold`. This parameter sets minimum number of elements thats can be compressed (this is only actual for slow insert rates).