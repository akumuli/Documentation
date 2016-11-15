Query language reference
------------------------

To retrieve information from Akumuli you should send HTTP POST query. Post data should contain JSON query.
This section describes JSON query format. Data returned using chunked transfer encoding (because query result
in Akumuli can be huge or infinite). Depending on the query result can be returned in RESP or in CSV format.

### Series names

Each series name consist of metric name and set of tags. Unique combination of metric and tags is used to distinguish
between different time-series. Metric can be thought as unit of measure of the time-series, e.g. `cpu`, `mem`, `voltage`, etc. Tags, on the other hand, can be viewed as unique identifier of the object that is monitored, e.g. if you're measuring performance of the machines in the network you can use tag `host=$particular_machine_host` to distinguish between machines. 
If you're measuring properties of some object, you will have set of series with the same set of tags and different metrics.

### Metadata query

Metadata query can be used to retrieve information about series. Only series names can be retrieved at the
moment.

```json
{
    "select": "meta:names"
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
    "select": "meta:names",
    "output": {"format": "csv"}
}
```

This will return list of all series names in CSV format:

```
cpu host=Foo
cpu host=Bar
cpu host=Buz
```

This list can be filtered by specifying metric and `where` statement.
Note that "where" can't be used without metric.

```json
{
    "select": "meta:names:cpu",
    "where": { "host": ["Foo", "Bar"] }
}
```

This will return list of all series that matches search predicate (Buz is filtered out):

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

### Select query

Time-series query consist of several components:

```json
{
    "select": "...",
     "range": "...",
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

#### Select field

Each query can filter results using `select` field. This field is mandatory and can't be omitted.

```json
{
    "select": "cpu",
    "range": {
        "from": "20160102T123000.000000",
        "to":   "20160102T123010.000000"
    }
}
```

#### Where field

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

This query will retrieve only series with `cpu` metric that have `region` tag which value is set to `europe`
or `us-east`.

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
    "select": "pressure_kPa",
    "group-by": [ "valve_num" ]
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


##### Output field

You can change query results formatting method using `output` field. Example:

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


##### Limit and offset fields.

You can use `limit` and `offset` query fields to limit number of returned tuples and to skip some tuples at
the begining of the query output. This fields works the same as LIMIT and OFFSET clauses in SQL.

#### Map field.

**Note: this functionality is disabled now!**

Map field can be used to transform time-series data. This field should contain a list of dictionaries.
Each dictionary can describe single operation. For example:

```json
{
    "select": "pressure_kPa",
    "map": [ { "name": "paa" } ],
    "group-by": [ "valve_num" ]
}
```

In this sample we're using `paa` transformation on data. 
All transforms can be divided into two groups: samplers that requires regular time-series as an input and samplers that doesn't require anything. 
Samplers can be pipelined. In this case each data sample will be passed through all
samplers of the pipeline. Example:

```json
{
    "select": "pressure_kPa",
    "map": [ { "name": "paa" },
             { "name": "sax", "alphabet_size": 8, "window_width": 10 }
           ],
    "group-by": { "tag": "valve_num" }
}
```

In this case we're using PAA transformation followed by SAX transformation. SAX transformation has two extra
parameters `alphabet_size` and `window_width`. SAX transformation works with regular time-series and PAA transforms
irregular time-series into regular.

##### Piecewise Aggregate Approximation of time series.

PAA is a dimentionality reduction technique. It transforms irregullar time-series into the regullar one. To 
do this data should be divided into equaly sized bins. After that all data points in each bin are aggregated
producing single value. PAA transformation can be viewed as aproximation using box functions.

[[images/PAA.png|alt=PAA diagram]]

Different functions can be used with PAA transform. By default data points in each bin aggregated using `mean`
function but you can use `min-paa`, `max-paa`, `first-paa`, `last-paa` or `median-paa`. 
In this cases query will look like this:

```json
{
    ...
    "map": [ { "name": "min-paa" } ],
}
```

In `first-paa` and `last-paa` variants data point with smallers or largest timestamp will be used as a 
resulting value. In `min-paa` and `max-paa` variants data point with min or max value will be used. With
`median-paa` median will be computed using all data point values.

If original time-series has a gap, PAA transformed time-series will have a gap too (this is subject to change,
in future versions this behavior will be configurable).


##### SAX transformation.

Symbolic Aggregate approXimation is a method of converting real valued time-series into symbolic time-series.
It requires two extra parameters: `alphabet_size` and `window_width`. First parameter is an alphabet size used
to encode time-series. It shluld be in range [2, 20]. Second parameter `window_width` defines width of the
sliding window used by SAX algorithm. 

```json
{
    ...
    "map": [ { "name": "sax", "alphabet_size": 4, "window_width": 4 } ],
}
```

SAX transformation doesn't require `group-by:time` field but if your time-series data is irregular you 
should use it in conjunction PAA transform.

```json
{
    ...
    "map": [ { "name": "paa" },
             { "name": "sax", "alphabet_size": 4, "window_width": 4 }
           ],
    "group-by": { "tag": "valve_num" }
}
```

Output will be a bit unusual:

```
+pressure_kPa valve_num=0
+20160118T171000.000000000
+aacd
+pressure_kPa valve_num=1
+20160118T171000.000000000
+abaa
+pressure_kPa valve_num=0
+20160118T171001.000000000
+acda
+pressure_kPa valve_num=1
+20160118T171001.000000000
+cdab
...
```

SAX has some sort of dimensionality reduction so some time-stamps may be missing.

##### Frequent items and heavy hitters.

##### Anomaly detection.
