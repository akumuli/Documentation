Query language reference
------------------------

The API endpoint is `http://<host>:<http_port>/api/query`, e.g. `http://localhost:8181/api/query`.

To retrieve information from Akumuli you should send HTTP POST query. Post data should contain JSON query.
This section describes JSON query format. Data returned using chunked transfer encoding (because query result
in Akumuli can be huge or infinite). Depending on the query result can be returned in RESP or in CSV format.

There are five types of queries: metadata query, select, aggregate, group-aggregate, and join.

### Series Names

Series name is a combination of metric name and tags. Metric can be thought as unit of measure of the time-series, e.g. `cpu_user`, `mem_commit_mb`, `voltage`, etc. Tags, on the other hand, can be viewed as a unique identifier of the object that is monitored, e.g. if you're measuring the performance of the machines in the network you can use tag `host=$particular_machine_host` to distinguish between machines. 
If you're measuring different properties of some object, you will have set of series with the same set of tags and different metrics, e.g. `cpu_user host=192.168.10.22`, `cpu_system host=192.168.10.22`, and `mem_commit_mb host=192.168.10.22`, all correspond to the same host.

Metric names and tag names and value can't contain spaces, ':', '=', and '|' symbols.

### Metadata Query

Metadata query can be used to retrieve information about series. Only series names can be retrieved at the
moment.

```json
{
    "select": "meta:names"
}
```

This will return the list of all series names. For example:

```
+cpu host=Foo
+cpu host=Bar
+cpu host=Buz
```

By default results returned in RESP format. This can be altered using "output" field (this field can be used with other types of queries).

```json
{
    "select": "meta:names",
    "output": {"format": "csv"}
}
```

This will return the list of all series names in CSV format:

```
cpu host=Foo
cpu host=Bar
cpu host=Buz
```

This list can be filtered by specifying metric and `where` statement.
Note that "where" can't be used without a metric.

```json
{
    "select": "meta:names:cpu",
    "where": { "host": ["Foo", "Bar"] }
}
```

This will return the list of all series that matches search predicate (Buz is filtered out):

```
+cpu host=Foo
+cpu host=Bar
```

Note that you can filter by metric without using `where` statement:

```json
{
    "select": "meta:names:mem"
}
```

This query will return names of all series with `mem` metric.

### Select Query

`Select` query can be used to retrieve results by metric name, `select` field is mandatory and can't be omitted.

```json
{
    "select": "cpu",
    "range": {
        "from": "20160102T123000.000000",
        "to":   "20160102T123010.000000"
    }
}
```

Select query consist of several components:

```json
{
           "select": "...",
            "range": "...",
            "where": "...",
         "group-by": "...",
         "order-by": "...",
           "output": "...",
            "limit": "...",
           "offset": "...",
           "filter": "..."
}
```

#### Range Field

Range field is used to set time bounds. 

```json
{
    "range": {
        "from": "20160102T123000.000000",
        "to":   "20160102T123010.000000"
    }
}
```

This query will scan data from timestamp 2016-01-02 12:30:00 to timestamp 2016-01-02 12:30:10. Timestamps
should be encoded using ISO 8601 format (only basic format is supported at the moment). Alternatively you
can use raw timestamps. Note that timestamps is 64-bit value that contains number of nanoseconds since
Epoch. Example:

```json
{
    "range": {
        "from": "1453137644600000000",
        "to":   "1453137644799999999"
    }
}
```

Query results will look like this:

```
+test tag=Foo
+20160118T173724.646397000
+999996
+test tag=Foo
+20160118T173724.647397000
+999997
+test tag=Foo
+20160118T173724.648397000
+999998
+test tag=Foo
+20160118T173724.649397000
+999999
```

Note that timestamps are increasing. This is because timestamp in "from" field is greater than timestamp in
"to" field. We can reverse output order by swapping "from" and "to" fields. If timestamp in "to" field is 
less then timestamp in "from" field Akumuli will return results in backward direction. The "from" boundary is inclusive and "to" boundary is exclusive.

#### Where Field

Query results can be further filtered using "where" field.

```json
{
    "select": "cpu",
    "where": {
        "region": [ "europe", "us-east" ]
    },
    "range": {
        "from": "20160102T123000.000000",
        "to":   "20160102T123010.000000" }
}
```

This query will retrieve only those series that have `cpu` metric and `region` tag which value is set to `europe`
or `us-east`.

#### Group-by Field

Suppose that you need to store the valve pressure mesurements. Pressure in each valve is measured by
two separate sensors so you're end up with this schema: `pressure_kPa valve_num=XXX sensor_num=YYY`. Here we
have `pressure_kPa` metric with two tags: `valve_num` and `sensor_num`. If you query this series you will 
get the following results:

```
+pressure_kPa valve_num=0 sensor_num=0
+20160118T171000.000000000
+204.0
+pressure_kPa valve_num=0 sensor_num=1
+20160118T171000.000000000
+204.1
+pressure_kPa valve_num=1 sensor_num=0
+20160118T171000.000000000
+208.0
+pressure_kPa valve_num=1 sensor_num=1
+20160118T171000.000000000
+208.2
...
```

Each combination of sensor and valve produces its own time-series. If you want to group data only by valve
you can use "group-by" field.

```json
{
    "select": "pressure_kPa",
    "group-by": [ "valve_num" ]
}
```

As result series that shares the same `valve_num` tag value will be merged together. Series that
don't have "valve_num" tag will be excluded from search results. Output will look like this:

```
+pressure_kPa valve_num=0
+20160118T171000.000000000
+204.0
+pressure_kPa valve_num=0
+20160118T171000.000000000
+204.1
+pressure_kPa valve_num=1
+20160118T171000.000000000
+208.0
+pressure_kPa valve_num=1
+20160118T171000.000000000
+208.2
...
```

Note that series name is changed now. It contains only those tags that was listed in `group-by` field. You can
use several tags in group-by field using the following syntax: `"group-by": [ "foo", "bar" ]` (in this case
resulting series names will have both tags `foo` and `bar`).


##### Order-by Field

This field can be used to control output ordering. 

```json
{
    "select": "cpu",
  "order-by": "series"
}
```

This field takes single string. It can be "series" or "time". If `order-by` is "series" the results will be ordered by series name first and then by timestamp. If `order-by` is "time" then data points will be ordered by timestamp first and then by series name.

##### Output Field

You can change query results formatting using `output` field. Example:

```json
{
    "select": "test",
    "output": { "format": "csv" },
}
```

This query will return CSV formatted output. `Format` field can take two values: `csv` or `resp`.

```
test tag=Foo, 20160118T173724.646397000, 999996
test tag=Foo, 20160118T173724.647397000, 999997
test tag=Foo, 20160118T173724.648397000, 999998
test tag=Foo, 20160118T173724.649397000, 999999
```

You can change timestamp representation using `timestamp` field:

```json
{
    "select": "test",
    "output": { "format": "csv", "timestamp": "raw" }
}
```

This query will return CSV formatted output with timestamps formatted as integers:

```
test tag=Foo, 1453127844646397000, 999996
test tag=Foo, 1453127844647397000, 999997
test tag=Foo, 1453127844648397000, 999998
test tag=Foo, 1453127844649397000, 999999
```

`Timestamp` field can take only two values: `iso` or `raw`.


##### Filter field

This field can be used to filter the output.

```json
{
    "select": "test",
    "filter": { "gt": 100 },
    ...
}
```

This query will return values greater than 100. The possible predicates are "gt" (greater than), "ge" (greater or equal),
"lt" (less than), and "le" (less or equal). It is possible to combine two predicates if you want to read values that
fit some range, for instance `"filter: {"gt": 0, "lt": 10 }` will select all values between 0 and 10, but not 0 and 10.

##### Limit and Offset Fields.

You can use `limit` and `offset` query fields to limit the number of returned tuples and to skip some tuples at
the beginning of the query output. This fields works the same as LIMIT and OFFSET clauses in SQL.

### Aggregate Query

Aggregate query consist of several components:

```json
{
        "aggregate": "...",
            "range": "...",
            "where": "...",
           "output": "...",
            "limit": "...",
           "offset": "..."
}
```

##### Aggregate Field

The `aggregate` field is used to tell Akumuli what metric should be aggregated and what aggregation function should be used.

```json
{
    "aggregate": { "cpu": "max" }
}
```

This query will return max values of all series with `cpu` metric. At the moment you can use only one aggregation function per metric and only one metric name. You can use the following aggregation functions:

- `count` - total number of data points in the series (or in time range)
- `max` - largest value in the series (or in time range)
- `min` - smallest value in the series (or in time range)
- `mean` - mean value of the series (or in time range)
- `sum` - sum of all data points in the series (or in time range)
- `min_timestamp` - time when smallest value was registered
- `max_timestamp` - time when largest value was registered

You can use `where`, `group-by`, `range`, `output`, `limit`, and `offset` fields the same way as in `select` query. If `range` field is used, aggregate function will be calculated with respect to the specified range.

### Join Query

Join query consist of several components:

```json
{
             "join": "...",
            "range": "...",
         "order-by": "...",
            "where": "...",
           "output": "...",
            "limit": "...",
           "offset": "..."
}
```

##### Join Field

Join field takes string value with the following format:

```json
{
    "join": ["cpu", "mem", "iops"],
}
```

Here `cpu`, `mem`, and `iops` is different metric names. Query processor will find series names with the same set of tags in this metrics and join them. E.g. if we have three series - "cpu host=host1", "mem host=host1", and "iops host=host1" - all three series will be joined together producing single series "cpu|mem|iops host=host1". The output will contain records in [bulk format](https://github.com/akumuli/Akumuli/wiki/Protocol#writing-measurements-in-bulk):

```
+cpu|mem|iops host=host1\r\n
+20161231T235500\r\n
*3\r\n
+10.5\r\n
+4870\r\n
+148\r\n
```

You can use `range`, `where`, `order-by`, `limit`, `offset`, and `output` fields the same way as in `select` query.

##### Join with filter

You can filter by any column using the `filter` clause, for example:

```json
{
    "join": ["cpu", "mem", "iops"],
    "filter": {
        "cpu": { "gt": 200 },
        "mem": { "lt": 100000000 }
    },
    ...
}
```

This query will work the same way as previous one but return only those value that match the filter.

### Group-Aggregate Query

Group-aggregate query consist of these components:

```json
{
  "group-aggregate": "...",
            "range": "...",
         "order-by": "...",
         "group-by": "...",
            "where": "...",
           "output": "...",
            "limit": "...",
           "offset": "..."
}
```

In a nutshell, this query divides all data points into equally sized bins based on timestamp. Then it uses aggregation function(s) to produce single value (or several values if several aggregation functions have been used). This query can be used to resample time-series.

##### Group-Aggregate Field

Group-aggregate field has the following format:

```json
{
  "group-aggregate": {
           "metric": "cpu",
             "step": "1m",
             "func": [ "min", "max" ]
  },
  ...
}
```

- `metric` should contain valid metric name;
- `step` should contain time interval (e.g. 10s, 1min, 2h);
- `func` should contain the list of aggregation functions (it is possible to apply several aggregation functions at a time);

The output of this query will look like this:

```
+cpu:min|cpu:max host=host1\r\n
+20170101T221015.001\r\n
*2\r\n
+0.05\r\n
+99.7\r\n
```

As you can see, the new metric name will be created by concatenating original metric name with function name using ':' as a separator, and if you're using several aggregation functions several metric names will be concatenated using '|' as a separator (as in `join` query and bulk-load format).

You can use `range`, `where`, `group-by`, `order-by`, `limit`, `offset`, and `output` fields the same way as in `select` query.

##### Filter 

Filter field works the same way as in join query but instead of metric names you should use aggregation function names.

```json
{
  "group-aggregate": {
           "metric": "cpu",
             "step": "1m",
             "func": [ "min", "max" ]
  },
  "filter": {
    "max": { "gt": 100 }
  }
  ...
}
```

This query will rows rows which have value greater than 100 in the second column. You can combine several filters the same
way as in join query. The returned values should match all filters. 

### Error handling

Query parsing errors are reported using the [RESP protocol](https://redis.io/topics/protocol). The only line in the response will be started with '-' followed by the error message.

Some errors can be reported using the HTTP error codes (e.g. when the wrong API endpoint is used). The query parsing errors are reported using the error messages and the query processing errors usually reported using the HTTP error codes.