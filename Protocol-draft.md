Protocol draft
==============
Akumuli protocol is based on [redis protocol](http://redis.io/topics/protocol).

### Writing
To write data to akumuli we must specify id (string or numeric), timestamp and value (byte array or double).
Command starts from identifier. Ids encoded very much like redis's strings and integers:
- String id starts with '+' symbol followed by id-string and ends with CRLF, example: "+balancer.cpu\r\n".
- Integer id starts with ':' symbol followed by numeric id and ends with CRLF, example: ":1735\r\n".

Id must be followed by the payload. Payload can be encoded in different ways.
- Bulk string that contain binary representation. Bulk string encoded the same way as in redis protocol.
  + _"+balancer.cpu\r\n$64\r\n...data...\r\n"_ (id - "balancer.cpu", 64 - size of the byte array)
  + _":1735\r\n$127\r\n...data...\r\n"_ (id - 1735, 127 - size of the byte array)
- Integer timestamp followed by the value encoded by string.
  + _"+balancer.mem\r\n:1418224205\r\n+24.3\r\n"_ (id - "balancer.mem", timestamp - 1418224205, value - 24.3)
  + _":1733\r\n:1418224205\r\n+24.3\r\n"_ (id - 1733, timestamp - 1418224205, value - 24.3)
- Integer timestamp followed by the value encoded by int.
  + _"+balancer.mem\r\n:1418224205\r\n:24\r\n"_ (value - 24)
  + _":1733\r\n:1418224205\r\n:24\r\n"_ (value -24)
- String timestamp followed by rather int or string value. Timestamp must be ISO 8601 encoded UTC date-time.
  + _"+balancer.mem\r\n+2014-12-10T07:43:43Z\r\n:24\r\n"_ (timestamp - 2014-12-10T07:43:43Z, value - 24)
  + _"+balancer.mem\r\n+2014-12-10T07:43:43Z\r\n+24.3\r\n"_ (timestamp - 2014-12-10T07:43:43Z, value - 24.3)
- Array of interleaved timestamps and values. Size fo the array should be even and each individual timestamp or value can be encoded rather by integer or string.
  + _":1734\r\n*4\r\n:1418224205\r\n+233.23\r\n:1418222534\r\n:234\r\n"_ (id - 1734, 4 - size of the array, array - [ts - 1418224205, val - 233.23, ts - 1418222534, val - 234])
- String or integer timestamp followed by bulk string, content of the bulk string interpreted as a blob.
  + _"+balancer.mem\r\n+2014-12-10T07:43:43Z\r\n$22\r\n...22.bytes...\r\n"_

##### Bulk data format.
Little endian byte order should be used for all binary data. All data should be aligned by one byte boundary (no align).
Bulk data format created to send efficiently large amounts of numeric data. Bulk data frame starts from unsigned 32-bit integer that stores number of values in the frame. This number then followed by the compressed array of timestams (delta-rle encoding should be used, values should be delta encoded first, then run-length encoded and finally, base-128 coding should be used to store all resulting integers). All timestamps should be sorted in accending order.
Array of timestamps should be followed by array of parameter ids. Base-128 variable length encoding should be used to represent each id.
Ids should be followed by compressed array of values. Compression algorithm: TBD.
