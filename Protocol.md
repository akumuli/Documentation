Protocol specification
======================
Akumuli protocol is based on [redis protocol](http://redis.io/topics/protocol).

### Writing
To write data to akumuli you should specify id (integer or string), timestamp (integer or string) and value (integer, string or bulk string).

Id should be a string:
- Integer id:
  + _":112233\r\n"
- String id should starts with parameter name and can contain list of key-value pairs separated by spaces:
  + _"+balancers.host1.cpuload\r\n"_
  + _"+network.loadavg host=postgres\r\n"_ parameter name is "network.loadavg" and key "host" is set to "postgres"

Id should be followed by timestamp:
- Integer timestamp (nanoseconds since epoch):
  + _":1418224205000000000\r\n"_
- ISO 8601 encoded UTC date-time (nanosecond precision or lower):
  + _"+20141210T074343.999999999\r\n"_

Timestamp should be immediately followed by the value:
- Numeric value encoded by string.
  + _"+24.3\r\n"_
- Numeric value encoded by int.
  + _":24\r\n"_


##### Examples
Full message can look like this (\r\n is replaced with real newlines):
 - String timestamp, integer value and string id with two keys:
```
+balancers.memusage host=machine1 unit=Gb
+20141210T074343.999999999
:31
```
 - Integer timestamp, string value and string id with one key:
```
+balancers.cpuload host=machine1
:1418224205000000000
+22.0
```