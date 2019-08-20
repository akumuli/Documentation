# Writing data

Akumuli protocol is based on [redis protocol](http://redis.io/topics/protocol) \(aka RESP or REdis Serialization Protocol\). The protocol is designed to be simple to implement, human readable, and fast to parse. Apart from RESP, Akumuli supports [OpenTSDB telnet-style API](http://opentsdb.net/docs/build/html/api_telnet/put.html).

## Serialization

Akumuli borrows RESP serialization format but not the protocol. Serialization format uses five data types:

* String
* Integer
* Error message
* Array
* Bulk string

### String

Simple string starts with **+** character followed by characters of the string and terminated by CRLF \(carriage return and new line characters\), or just by new line. The length of the string is limited by 1 kilobyte. The string can't contain a new-line or carriage return characters.

```text
+proc.net.bytes\r\n
```

The strings are used to represent time stamps, floating-point values, and series names.

### Integer

Integer value starts with colon character, followed by the series of digits, and terminated by the CRLF sequence.

```text
:314159\r\n
```

Integer values in Akumuli are limited by 84 digits. They are used to represent values and timestamps.

### Error

Error starts with minus character, followed by the series of characters that contains error message, terminated by CRLF.

```text
-RESP Error:  declared object size is too large
```

This data type is used to return information about errors to the client.

### Array

Array is a compound data structure that can be composed from arbitrary number of values of different types.

* Array starts with asterisk, followed by the number of elements in the array, terminated by the CRLF sequence.
* A RESP encoded value for every element.

```text
*2\r\n
:1\r\n
+second\r\n
```

### Bulk string

Bulk strings are used to represent arbitrary large \(up to 1MB\) binary objects. 

* The value starts with **$** character, followed by the length of the string, terminated by the CRLF sequence \(or a single new line character\).
* The actual bulk string data, terminated by the CRLF. The length of the string should match the value after **$** character.

```text
$11\r\n
Hello world\r\n
```

## Writing measurements by one

This is the simpliest possible way to write data to Akumuli instance. You should send each data point individually. Each data-point contains the following:

* Series name
* Timestamp
* Value

### Series name

Series name have the following format: `<metric-name> <tags>`, where `<tags>` is a list of key-value pairs separated by space. You should specify both metric name and tags \(at least one key-value pair\), otherwise it's not a valid series name. It should be encoded using the [RESP string](writing-data.md#string).

Examples:

```text
+cpu_user host=hostname region=NW\r\n
```

Here, "cpu\_user" is a metric name and "host" and "region" are tags.

```text
+!network.tcp.packet_loss host=postgres\r\n
```

Event name is "!network.tcp.packet\_loss" and tag "host" is set to "postgres". You can find more information about series names in the [Data Model](data-model.md) section.

### Timestamp

Timestamp should be in UTC time \(Akumuli can't work with local time\). To encode the timestamp you can use RESP [string ](writing-data.md#string)or [integer](writing-data.md#integer).

If the **string** is used, Akumuli will try to interpret its value as ISO 8601 encoded UTC date-time \(with nanosecond precision or lower\).

```text
+20141210T074343.999999999\r\n
```

Note: only basic ISO8601 format is supported. All timestamps are expected to be UTC timestamps. Timezones are not supported.

If the **integer** was used the value will be interpreted as a 64-bit timestamp with nanosecond precision. The beginning of the epoch is Jan-1 1970 \(Unix epoch\).

```text
:1418224205000000000\r\n
```

Note: event timestamps are stored with microsecon precision.

### Metric Value

Value can be encoded using the RESP [string ](writing-data.md#string)or [integer](writing-data.md#integer). If the **string** is used, it will be interpreted as a string representation of the floating point value. Scientific format is supported.

```text
+3.14159\r\n
```

Note that the string representation of the floating point value may loose precision. You may use something like this to print floating point numbers without precision loss:

```text
printf("%.17g", float64);
```

If the **integer** is used, it will be interpreted as is. Note that Akumuli uses double precision floating point numbers, defined by IEEE 754 standard. Thus, only 54-bit integers can be represented precisely.

### Event value

Event value can be encoded using RESP [string](writing-data.md#string). 

```text
+1m ipv4 listen drops
```

Note that the value will be interpreted as an event value only if the message contains event name that starts with '!' symbol.

### Composing the message

Each individual message should be started with series name, followed by the timestamp and the value.

Full message can look like this \(\r\n is replaced with real newlines\):

* Metric with string timestamp, integer value and string id with two keys:

  ```text
  +balancers.memusage host=machine1 region=NW
  +20141210T074343.999999999
  :31
  ```

* Metric with integer timestamp, string value and string id with one key:

  ```text
  +balancers.cpuload host=machine1 region=NW
  :1418224205000000000
  +22.0
  ```

* Event with string timestamp, string value and string id with two keys:

  ```text
  +!net.tcp host=machine1 region=NW
  +20141210T074343.999999999
  +1m ipv4 listen drops
  ```

**Important:** even the last line in the stream must be terminated with CRLF.

You can concatenate multiple messages together and send them via TCP connection. Akumuli reads the incoming stream of data, parses it and writes individual data points.

## Error messages

Akumuli doesn't send anything back in response to your writes if everything is OK. But if anything goes wrong it will send back an error message. Client can read data from socket \(but not obliged\) asynchronously to receive error messages. Error messages are RESP-encoded and will start with '-'.

## Workflow

To send data to Akumuli the client should open a TCP connection and start sending the data. Optionally, client may start to read data from socket in parallel. Akumui will send anything back only in case of error. I case of error, Akumuli will send error message and close the connection. 

### Late write

At the moment, Akumuli can only write ordered data. The data-points from each individual time-series should be ordered by timestamp \(it's OK to write data-points from different series out of order if individual series are ordered by time-stamp\).

## Writing measurements in bulk

Bulk transfer can reduce the transfer size. It allows to minimize redundancy in transferred data by grouping together values with the same timestamp and origin. 

```text
+mem.usage host=machine1 region=NW\r\n
+20180102T000200\r\n
+87.4\r\n
+cpu.user host=machine1 region=NW\r\n
+20180102T000200\r\n
+22.1\r\n
```

In this example, _"mem.usage host=machine1 region=NW"_ and _"cpu.user host=machine1 region=NW"_  originate from the same host. They share the set of tags and timestamp \(because the collector generated both measurements at the same time, with hundred of other measurements like this\). The only difference between two measurements are the metric name \(cpu.user vs mem.usage\) and value. In situation like this, it's logical to send the values in bulk, without duplicating the tags and timestamp for each value \(especially, when we have dozens of values, instead of two\). Using the bulk format this example will look like this:

```text
+mem.usage|cpu.user host=machine1 region=NW\r\n
+20180102T000200\r\n
*2\r\n
+87.4\r\n
+22.1\r\n
```

To write data to Akumuli in bulk format you should specify a compound series name, timestamp \(integer or string\), and [an array](writing-data.md#array) of values \(each value can be integer or string\).

### Compound series name

Compound series name format contains pipe delimited list of metric and event names followed by the set of tags:

```text
<name1>|<name2>|...|<name3> <tags>
```

List of metric/event names inside the compound series name shouldn't have any spaces. On the storage side this will be converted to the list of series names: `<name1> <tags>`, `<name2> <tags>`, etc.

```text
+cpu.real|!cpu.real|cpu.user|cpu.sys|!net.tcp host=machine1 region=NW
```

This encoding is more compact and saves a lot of network bandwidth.

### Timestamp

Series name should be followed by the timestamp, the same way as [described previously](writing-data.md#timestamp).

### Array of values

The list of values is represented using the [RESP array](writing-data.md#array). RESP array starts with **\*** symbol followed by the number of elements in the array. This number should match the number of metrics in the compound series name! This number should be followed by values \(number of values should match the number of metric names and all values should have the same order, e.g. if cpu.real goes first in the compound series name it's numeric value should go first in the array\).

Example:

```text
+cpu.real|cpu.user|cpu.sys host=machine1 region=NW
+20141210T074343
*3
+3.12
+8.11
+12.6
```

This will produce three writes:

* Series name: cpu.real host=machine1 region=NW, TS: 20141210T074343, Value: 3.12
* Series name: cpu.user host=machine1 region=NW, TS: 20141210T074343, Value: 8.11
* Series name: cpu.sys host=machine1 region=NW, TS: 20141210T074343, Value: 12.6

## Dictionary mode

In many situations client knows what series it will be sending in advance. This can help to cut down the redundancy in transferred data by providing the series name dictionary. Consider this example:

```text
+mem.usage host=machine1 region=NW\r\n
+20180102T000200\r\n
+87.4\r\n
+mem.usage host=machine1 region=NW\r\n
+20180102T000201\r\n
+87.5\r\n
+mem.usage host=machine1 region=NW\r\n
+20180102T000202\r\n
+88.1\r\n
```

Here, the lines that change are the timestamps and values. The series name is always the same. The client needs to send it every time, and Akumuli have to parse it for every data-point. To minimize this expenses, client can send the pre-computed dictionary for series names. This dictionary maps series names to user-defined integer ids. This ids then can be used instead of series names in protocol \(this also works with bulk protocol\).

### Sending the dictionary

The dictionary can only be sent in the beginning of the TCP session. The dictionary is represented using the [RESP array](writing-data.md#array). Every key-value pair is represented using two consecutive elements of the array. The series name is represented using the [RESP string](writing-data.md#string). Series name should be followed by the unique id. Id should be represented using the [RESP integer](writing-data.md#integer). 

```text
*4\r\n
+cpu.user host=machine1 region=NW\r\n
:1\r\n
+mem.usage host=machine1 region=NW\r\n
:2\r\n
```

This dictionary can later be used in the transmission. To use the series name from the dictionary one should use it's id, in place of series name \(or compound series name\). The id should be encoded using [RESP integer](writing-data.md#integer).

```text
:1\r\n
+20180102T000200\r\n
+11.2\r\n
:2\r\n
+20180102T000200\r\n
+43.99\r\n
```

Here, the first data point corresponds to **cpu.user** series, and the second one to **mem.usage**.

You can send the dictionary using a single array or several consecutive ones. The only requirement is that the ids should be unique for the TCP-session, and that the dictionary should be sent first. The following data-points can use or ignore the dictionary.

## OpenTSDB telnet-style API

Akumuli have limited support for OpenTSDB telnet-style API. Only `put` command supported at the moment. The data can be inserted in this form: `put <metric-name> <timestamp> <value> <list-of-tags>`. Example:

```text
put cpu.user 1483228800 10.005344383927394 OS=Ubuntu_14.04 arch=x64 host=host_0 instance-type=m3.large rack=86 region=eu-central-1 team=NJ
put cpu.sys 1483228800 9.9992693762580025 OS=Ubuntu_14.04 arch=x64 host=host_0 instance-type=m3.large rack=86 region=eu-central-1 team=NJ
put cpu.real 1483228800 10.002083289000792 OS=Ubuntu_14.04 arch=x64 host=host_0 instance-type=m3.large rack=86 region=eu-central-1 team=NJ
put idle 1483228800 9.9815370970857522 OS=Ubuntu_14.04 arch=x64 host=host_0 instance-type=m3.large rack=86 region=eu-central-1 team=NJ
put mem.commit 1483228800 9 OS=Ubuntu_14.04 arch=x64 host=host_0 instance-type=m3.large rack=86 region=eu-central-1 team=NJ
put mem.virt 1483228800 10 OS=Ubuntu_14.04 arch=x64 host=host_0 instance-type=m3.large rack=86 region=eu-central-1 team=NJ
```

### Timestamp

OpenTSDB uses timestamps with 1-second precision. It's a regular Unix timestamp \(number of seconds since epoch\). Akumuli can also understand timestamps in this format and all tools that can write data to OpenTSDB will work. But in addition, passing ISO8601-formatted date time or nanosecond precision timestamp will also work.

OpenTSDB endpoint can be disabled in configuration \(it's enabled by default in configuration generated by `akumulid --init` command\). Note that OpenTSDB endpoint in Akumuli is a bit slower than native TCP endpoint due to protocol verbosity.

### Differences with OpenTSDB

When you're using OpenTSDB you should went to great length in your database schema design. OpenTSDB stores all series with the same metric name together. That's why you should be careful not introducing to many series with the same metric name. With Akumuli this is not needed, since it stores each series individually.

Akumuli implements 'put' command. Other commands, like 'histogram' or 'rollup' are ignored. The 'version' command returns a string that indicates that Akumuli endpoint is used.

