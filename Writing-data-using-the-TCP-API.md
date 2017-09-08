Protocol specification
======================
Akumuli protocol is based on [redis protocol](http://redis.io/topics/protocol). Please read the Redis protocol description before proceding with Akumuli ingestion protocol.

### Writing measurements by one
To write data to Akumuli you should specify a series name, timestamp (integer or string), and value (integer or string).

Series name have the following format: `<metric-name> <tags>`, where `<tags>` is a list of key-value pairs separated by space. You should specify both metric name and tags (at least one key-value pair), otherwise it's not a valid series name.

Examples:
- _"+cpu_user host=hostname region=NW\r\n"_ "cpu_user" is a metric name and "host" and "region" are tags
- _"+network.loadavg host=postgres\r\n"_ metric name is "network.loadavg" and tag "host" is set to "postgres"

Series name should be followed by the timestamp:
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

**Important:** even the last line in the stream must be finished with \r\n. 

### Error messages

Akumuli doesn't send anything back in response to your writes if everything is OK. But if anything goes wrong it will send error message to client. Client can read data from socket (but not obliged) asynchronously to receive error messages. Error messages are RESP-encoded and will start with '-'.

### Writing measurements in bulk

Akumuli assumes that measurements with the same set of tags came from the same object. These measurements will differ only by metric names. E.g. _"mem.usage host=machine1 region=NW"_ and _"cpu.user host=machine1 region=NW"_ will be considered originating from the same host. That host is described by the set of tags - _"host=machine1 region=NW"_ and metric name can be seen as a column name. Usually, it is preferable to write these metrics together and Akumuli has special message format for this case.

To write data to Akumuli in bulk format you should specify a compound series name, timestamp (integer or string), and an array of values (each value can be integer or string).

Compound series name format: `<metric1>|<metric2>|...|<metricN> <tags>`. Instead of the single metric name, we're writing the list of metric names separated by `|` symbol (without spaces!). On the storage side this will be converted to the list of series names: `<metric1> <tags>`, `<metric2> <tags>`, etc.

Example:
- _"+cpu.real|cpu.user|cpu.sys host=machine1 region=NW"_

Series name should be followed by the timestamp:
- Integer timestamp (nanoseconds since epoch):
  + _":1418224205000000000\r\n"_ 
- ISO 8601 encoded UTC date-time (nanosecond precision or lower):
  + _"+20141210T074343.999999999\r\n"_ (fractional part can go down to nanoseconds)

The timestamp should be immediately followed by the array of values. RESP array starts with `*` symbol followed by the number of elements in the array. This number should match the number of metrics in the compound series name! This number should be followed by values (number of values should match the number of metric names and all values should have the same order, e.g. if cpu.real goes first in the compound series name it's numeric value should go first in the array). 

Example:

```
+cpu.real|cpu.user|cpu.sys host=machine1 region=NW
+20141210T074343
*3
+3.12
+8.11
+12.6
```

This will produce three writes:
- Series name: cpu.real host=machine1 region=NW, TS: 20141210T074343, Value: 3.12
- Series name: cpu.user host=machine1 region=NW, TS: 20141210T074343, Value: 8.11
- Series name: cpu.sys host=machine1 region=NW, TS: 20141210T074343, Value: 12.6

### OpenTSDB telnet-style API

Akumuli have limited support for OpenTSDB telnet-style API. Only `put` command supported at the moment (v0.7.14).
The data can be inserted in this form: `put <metric-name> <timestamp> <value> <list-of-tags>`. Example:

```
put cpu.user 1483228800 10.005344383927394 OS=Ubuntu_14.04 arch=x64 host=host_0 instance-type=m3.large rack=86 region=eu-central-1 team=NJ
put cpu.sys 1483228800 9.9992693762580025 OS=Ubuntu_14.04 arch=x64 host=host_0 instance-type=m3.large rack=86 region=eu-central-1 team=NJ
put cpu.real 1483228800 10.002083289000792 OS=Ubuntu_14.04 arch=x64 host=host_0 instance-type=m3.large rack=86 region=eu-central-1 team=NJ
put idle 1483228800 9.9815370970857522 OS=Ubuntu_14.04 arch=x64 host=host_0 instance-type=m3.large rack=86 region=eu-central-1 team=NJ
put mem.commit 1483228800 9 OS=Ubuntu_14.04 arch=x64 host=host_0 instance-type=m3.large rack=86 region=eu-central-1 team=NJ
put mem.virt 1483228800 10 OS=Ubuntu_14.04 arch=x64 host=host_0 instance-type=m3.large rack=86 region=eu-central-1 team=NJ
```

The timestamp is a regullar Unix timestamp (number of seconds since epoch), but passing ISO8601-formatted date time or nanosecond precision timestamp will also work.

OpenTSDB endpoint should be enabled in configuration (it's enabled in default configuration generated by `akumulid --init` command). Note that OpenTSDB endpoint in Akumuli is a bit slower than native TCP endpoint.