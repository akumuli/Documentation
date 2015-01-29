Protocol draft
==============
Akumuli protocol is based on [redis protocol](http://redis.io/topics/protocol).

### Writing
To write data to akumuli you should specify id (integer or string), timestamp (string or integer) and value (string or bulk string).


Id can be an integer or string:
- Integer id:
  + _":112233\r\n"
- String id should starts with parameter name and can contain list of key-value pairs separated by spaces:
  + _"+balancers.host1.cpuload\r\n"_
  + _"+network.loadavg host=postgres\r\n"_ parameter name is "network.loadavg" and key "host" is set to "postgres"

Id should be followed by timestamp:
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


All keys should be predefined in database schema.

##### Examples
Full message can look like this (\r\n is replaced with real newlines):
 - String timestamp, integer value and string id with two keys:
```
+balancers.memusage host=machine1 unit=Gb
+2014-12-10T07:43:43Z
:31
```
 - Integer timestamp, string value and string id with one key:
```
+balancers.cpuload host=machine1
:1418224205
+22.0
```
 - Blob example:
```
+balancers.events host=machine1
+2014-12-10T07:43:43Z
$9
500 error
```
- Integer timestamp, value (33) and id (12345):
```
:12345
:1418224205
:33
```