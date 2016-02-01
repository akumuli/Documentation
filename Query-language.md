Query language reference
------------------------

To retreive information from Akumuli you should send HTTP POST query. Post data should contain JSON query.
This section describes JSON query format. Data returned using chunked transfer encoding (because query result
in Akumuli can be huge or infinite). Depending on the query result can be returned in RESP or in CSV format.

### Metadata query

Metadata query can be used to retreive information about series. Only series names can be retreived at the
moment.

```json
{
    "select": "names"
}
```

This will return list of all series names. For example:

```
+cpu host=Foo
+cpu host=Bar
+cpu host=Buz
```

By default results returned in RESP format. This can be altered using "output" field.

```json
{
    "select": "names",
    "output": {"format": "csv"}
}
```

This will return list of all series names in CSV format:

```
cpu host=Foo
cpu host=Bar
cpu host=Buz
```

This list can be filtered using "where" and "metric" fields.
Note that both "where" and "metric" should be used together.

```json
{
    "select": "names",
    "metric": "cpu",
    "where": { "host": ["Foo", "Bar"] }
}
```

This will return list of all series that matches search predicate (Buz is filtered out):

```
+cpu host=Foo
+cpu host=Bar
```

### Scan query

Time-series query consist of serveral components:

```json
{
    "sample": "...",
     "range": "...",
    "metric": "...",
     "where": "...",
  "group-by": "...",
    "output": "...",
     "limit": "...",
    "offset": "..."
}
```

#### Range field

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
less then timestamp in "from" field Akumuli will return results in backward direction.

##### TBD: continuous queries

#### Metric field

Each query can filter results using "metric" field. If "metric" field is set only series with corresponding
metric name will be included.

```json
{
    "metric": "cpu",
    "range": {
        "from": "20160102T123000.000000",
        "to":   "20160102T123010.000000"
    }
}
```

Note that this field is optional (unlike "range" field that is mandatory).

#### Where field

Query results can be further filtered using "where" field.

```json
{
    "where": {
        "region": [ "europe", "us-east" ]
    },
    "metric": "cpu",
    "range": {
        "from": "20160102T123000.000000",
        "to":   "20160102T123010.000000" }
}
```

This query will retreive only series from `cpu` metric that have `region` tag which value is set to `europe`
or `us-east`. Note that "metric" field is mandatory if you're using "where" field.

#### Group-by field

Suppose that you're have the time-series of the valve pressure. Pressure in each valve is measured by
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

Each combination of sensor and valve produces it's own time-series. If you want to group data only by valve
you can use "group-by" field.

```json
{
    "group-by": { "tag": "valve_num" }
}
```

As result series that shares the same `valve_num` tag value will be joined together. Series that
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

Note that series name is changed now. It contains only those tags that's listed in `group-by` field. You can
use several tags in group-by field using the following syntax: `{ "tag": [ "foo", "bar" ] }` (in this case
resulting series names will have both tags `foo` and `bar`).

Group by field can be used to group values by time using the following syntax:

```json
{
    "group-by": { "time": "10ms" }
}
```

In this case all results will be splitted into 10ms bins and only one value for each bin and series will be
added to output (this require corresponding sampling method in "sample" field, more on this latter). We can
add this field to our valve example:

```json
{
    "sample": [ { "name": "paa" } ],
    "group-by": { "tag": "valve_num", "time": "1s" }
}
```

and this will produce the following output:

```
+pressure_kPa valve_num=0
+20160118T171000.000000000
+204.05
+pressure_kPa valve_num=1
+20160118T171000.000000000
+208.1
+pressure_kPa valve_num=0
+20160118T171001.000000000
+204.06
+pressure_kPa valve_num=1
+20160118T171001.000000000
+208.2
...
```

in this case values from both sensors from the same valve are groupped by valve and by one second time
interval and averaged.


##### Output field

You can change query results formatting method using `output` field. Example:

```json
{
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


##### Limit and offset fields.

You can use `limit` and `offset` query fields to limit number of returned tuples and to skip some tuples at
the begining of the query output. This fields works the same as LIMIT and OFFSET clauses in SQL.

#### Sample field.

Sample field can be used to transform time-series data. This field should contain a list of dictionaries.
Each dictionary can describe single operation. For example:

```json
{
    "sample": [ { "name": "paa" } ],
    "group-by": { "tag": "valve_num", "time": "1s" }
}
```

In this sample we're using `paa` transformation on data. This sampler requires `group-by:time` field.
All samplers can be diveded into two groups: samplers that requires `group-by:time` field and samplers that
doesn't requires anything. Samplers can be pipelined. In this case each data sample will be passed through all
samplers of the pipeline. Example:

```json
{
    "sample": [ { "name": "paa" },
                { "name": "sax", "alphabet_size": 8, "window_width": 10 }
              ],
    "group-by": { "tag": "valve_num", "time": "1s" }
}
```

In this case we're using PAA transformation followed by SAX transformation. SAX transformation has two extra
parameters `alphabet_size` and `window_width`.

##### Piecewise Aggregate Approximation of time series.

PAA is a dimentionality reduction technique. It transforms irregullar time-series into the regullar one. To 
do this data should be divided into equaly sized bins. After that all data points in each bin are aggregated
producing single value. PAA transformation can be viewed as aproximation using box functions.

[[images/PAA.png|alt=PAA diagram]]

Different functions can be used with PAA transform. By default data points in each bin aggregated using `mean`
function but you can use `paa-min`, `paa-max`, `paa-first`, `paa-last` or `paa-median`. 
In this cases query will look like this:

```json
{
    "sample": [ { "name": "paa-min" } ]
}
```

In `paa-first` and `paa-last` variants data point with smallers or largest timestamp will be used as a 
resulting value. In `paa-min` and `paa-max` variants data point with min or max value will be used. With
`paa-median` median will be computed using all data point values.

If original time-series has a gap, PAA transformed time-series will have a gap too (this is subject to change,
in future versions this behavior will be configurable).

##### Random sampling.

Only reservoir sampling supported at the moment. It requires only one argument.

```json
{
    "sample": [ { "name": "reservoir", "size": 1000 } ]
}
```

This query will return 1000 or less random data points. This sampler is sensible to `group-by:time` field.
If results is grouped by time, `reservoir` will return results on each time interval, for example:

```json
{
    "sample": [ { "name": "reservoir", "size": 1000 } ],
    "group-by": { "time": 10s }
}
```

This query will return 1000 values (or less) every 10 seconds.

##### SAX transformation.

Symbolic Aggregate approXimation is a method of converting real valued time-series into symbolic time-series.
TBD

```json
{
    "sample": [ { "name": "paa" },
                { "name": "sax", "alphabet_size": 8, "window_width": 10 }
              ],
    "group-by": { "tag": "valve_num", "time": "1s" }
}
```

##### Frequent items and heavy hitters.

##### Anomaly detection.
