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
    "sample": [ ... ]
     "range": { ... }
    "metric":   ...
     "where": { ... }
  "group-by": { ... }
    "output": { ... }
}
```

#### Range field

Range field is used to set time bounds. 

```json
{
    ...
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
    ...
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
"to" field. We can reverce it by swapping "from" and "to" fields. If timestamp in "to" field is less then
timestamp in "from" field Akumuli will return results in backward direction.

##### TBD: continuous queries

#### Metric field

Each query can filter results using "metric" field. If "metric" field is set only series with corresponding
metric name will be included.

```json
{
    ...
    "metric": "cpu",
    "range": {
        "from": "20160102T123000.000000",
        "to":   "20160102T123010.000000"
    }
}
```

Note that this field is optional (unlike "range" field that is mandatory).

#### Where field
