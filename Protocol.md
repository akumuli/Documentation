Protocol draft
==============
Akumuli protocol is based on [redis protocol](http://redis.io/topics/protocol).

### Writing
To write data to akumuli you should specify timestamp (string or integer), value (string or bulk string) and id (integer or string).

Timestamp:
- Integer timestamp:
  + _":1418224205\r\n"_
- ISO 8601 encoded UTC date-time:
  + _"+2014-12-10T07:43:43Z\r\n"_

Timestamp should be immediately followed by the value:
- Bulk string that contain blob. Bulk string encoded the same way as in redis protocol:
  + _"$64\r\n...data...\r\n"_ (64 - size of the byte array)
- Numeric value encoded by string.
  + _"+24.3\r\n"_
- Numeric value encoded by int.
  + _":24\r\n"_

Timestamp and value should be followed by id:
- Integer id:
  + _":112233\r\n"
- String id should starts with parameter name and can contain list of key-value pairs separated by spaces:
  + _"+balancers.host1.cpuload\r\n"_
  + _"+network.loadavg host=postgres\r\n"_ parameter name is "network.loadavg" and key "host" is set to "postgres"

All keys should be predefined in database schema.

##### Examples
Full message can look like this (\r\n is replaced with real newlines):
 - String timestamp, integer value and string id with two keys:
```
+2014-12-10T07:43:43Z
:31
+balancers.memusage host=machine1 unit=Gb
```
 - Integer timestamp, string value and string id with one key:
```
:1418224205
+22.0
+balancers.cpuload host=machine1
```
 - Blob example:
```
+2014-12-10T07:43:43Z
$9
500 error
+balancers.events host=machine1
```
- Integer timestamp, value and id:
```
:1418224205
:31
:12345
```