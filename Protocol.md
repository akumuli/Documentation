Protocol specification
======================
Akumuli protocol is based on [redis protocol](http://redis.io/topics/protocol). Please read the Redis protocol description before proceding with Akumuli ingestion protocol.

### Writing measurements by one
To write data to Akumuli you should specify a series name, timestamp (integer or string), and value (integer or string).

Series name have the following format: <metric-name> <tags>, where <tags> is a list of key-value pairs separated by space. You should specify both metric name and tags (at least one key-value pair) otherwise, it's not a valid series name.

Examples:
- _"+cpu_user host=hostname region=NW\r\n"_ "cpu_user" is a metric name and "host" and "region" are tags
- _"+network.loadavg host=postgres\r\n"_ metric name is "network.loadavg" and tag "host" is set to "postgres"

Series name should be followed by timestamp:
- Integer timestamp (nanoseconds since epoch):
  + _":1418224205000000000\r\n"_ 
- ISO 8601 encoded UTC date-time (nanosecond precision or lower):
  + _"+20141210T074343.999999999\r\n"_ (fractional part can go down to nanoseconds)

Note: only basic ISO8601 format is supported. All timestamps are expected to be UTC timestamps. Timezones are not supported.

The timestamp should be immediately followed by the value:
- Numeric value encoded by a string.
  + _"+24.3\r\n"_
- Numeric value encoded by int.
  + _":24\r\n"_


##### Examples
Full message can look like this (\r\n is replaced with real newlines):
 - String timestamp, integer value and string id with two keys:
```
+balancers.memusage host=machine1 region=NW
+20141210T074343.999999999
:31
```
 - Integer timestamp, string value and string id with one key:
```
+balancers.cpuload host=machine1 region=NW
:1418224205000000000
+22.0
```