---
description: Akumuli query language reference.
---

# Query Language

Akumuli uses REST interface for data retrieval. You should use [**api/query**](api-endpoints.md#read-query) endpoint for data queries and [**api/search**](api-endpoints.md#search) endpoint for metadata queries.

## Data Model

All data is split between the different metrics. You can think about the metric as a namespace for series names. Each series name starts with metric name:

```text
cpu.user host=pg-balancer OS=Trusty arch=x86_amd64 region=NE
```

Here, **cpu.user** is a metric name, and **host=pg-balancer** is a tag-value pair. Akumuli uses [Boolean model \(BIR\)](https://en.wikipedia.org/wiki/Standard_Boolean_model) for series name searching. For every query you need to specify a metric name and optionally, a set of tag-value pairs. Series that have this metric name and contains this tag-value pairs will be added to result set.

Every series name is linked to the list of data points \(the actual time-series data\). Every data-point contains a timestamp \(64-bit, nanosecond precision\) and a value \(double precision floating point\). You need to specify a search range for that data inside the query.

## Query Object

To retrieve the data you should create a query object that describes what data you need and what shape it should have. 

## Query Types

Query object can be of one of the following types:

* Select query
* Aggregate query
* Group-aggregate query
* Join query

### Select Query

This is a simplest possible query type. It can return raw time-series data without aggregation. Select query can have a post-processing steps \(e.g. rate or sliding window computation\).

Select query can return more than one series but they should have the same metric name.

| Field | Required | Commentary |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| [select](query-language.md#select-field) | Yes | Metric name |
| [range](query-language.md#range-field) | Yes | Time range |
| [where](query-language.md#where-field) | No | Tag filter |
| group-by | No | Series transformation |
| order-by | No | Order of the data-points in the result set |
| filter | No | Value based filtering |
| limit | No | Limit on output size |
| offset | No | Offset of the query output |

## Query Fields

The query object is JSON encoded. It can have a pre-defined set of fields. Some of this fields are mandatory and other are optional.

### Range Field

Range field denotes the time interval that query should fetch.

| Field | Format | Description |
| --- | --- |
| "range" | { "from": "20180530T123000", "to": "20180530T130000" } | Field should contain a dictionary with two keys, "from" and "to". Both values are timestamps. |

Both timestamps should be encoded using the basic ISO8601 format. The same format is used for data ingestion with RESP protocol. Both "range.from" and "range.to" fields are mandatory for data queries. If "range.from" is less than "range.to" the time-series data points will be returned in accending order \(from old to new\). If "range.from" is greater than "range.to" then the time-series data points will be returned in descending order \(from recent to old\).

### Select Field

Select field is used to tell Akumuli what metric should be fetched.

| Field | Format | Description |
| --- | --- |
| "select" | "metric.name" | Metric name |

This field defines the query type. If this field is used the query will be a simple [select query](query-language.md#select-query). You can provide only one "select" field. This field can have only one metric name. The query will fetch all series with this metric name. This series can be further filtered by using "where" field.

### Where Field

Where field is used to limit number of series returned by the query.

| Field | Format | Description |
| --- | --- | --- |
| "where" | { "tag-name": "tag-value" } | Include only series names which has tag "tag-name" set to "tag-value". |
| "where" | { "tag-name": \[ "value1", "value2" \] } | Include only series names which has tag "tag-name" set to one of the values "value1" or "value2". |

You can specify many tags in one where field. This data in conjunction with metric name \(or names\) will form be used to [search series](query-language.md#data-model) inside the index.

### Group-by Field

Group-by field is used to merge several series together. If `group-by` field was used to specify a tag name, all series which names has this tag with the same value will collapse into one. All data points from that series will be joined together. The resulting time-seires will contain all data-points from the original series. The series name will contain only those tags that was specified in `group-by` field.

| Field | Format | Description |
| --- | --- |
| "group-by" | \[ "tag1", "tag2", ..., "tagN" \] | The list of tags that resulting series name should have. |

Suppose that you need to store the valve pressure measurements. Pressure in each valve is measured by two separate sensors so you're end up with this schema: `pressure_kPa valve_num=XXX sensor_num=YYY`. Here we have `pressure_kPa` metric with two tags: `valve_num` and `sensor_num`. If you query this series you will get the following results:

```text

+pressure_kPa valve_num=0 sensor_num=0+20160118T171000.000000000+204.0+pressure_kPa valve_num=0 sensor_num=1+20160118T171000.000000000+204.1+pressure_kPa valve_num=1 sensor_num=0+20160118T171000.000000000+208.0+pressure_kPa valve_num=1 sensor_num=1+20160118T171000.000000000+208.2...
```

 Each combination of sensor and valve produces its own time-series. If you want to group data only by valve you can use "group-by" field. If you add a `"group-by": [ "valve_num" ]` field to the query the result will look like this:

```text

+pressure_kPa valve_num=0+20160118T171000.000000000+204.0+pressure_kPa valve_num=0+20160118T171000.000000000+204.1+pressure_kPa valve_num=1+20160118T171000.000000000+208.0+pressure_kPa valve_num=1+20160118T171000.000000000+208.2...
```

### Order-by Field

This field can be used to control the order of the data-points in the query output.

| Field | Format | Description |
| --- | --- | --- |
| "order-by" | "series" | Sort output by series name |
| "order-by" | "time" | Sort output by timestamp |

This field takes single string. It can be "series" or "time". If `order-by` is "series" the results will be ordered by series name first and then by timestamp. If `order-by` is "time" then data points will be ordered by timestamp first and then by series name.

### Output Field

This field can be used to control format of the output.

| Field | Format | Description |
| --- | --- | --- |
| "output" | { "format": "csv", "timestamp": "raw" } | Set output format to "csv" and timestamp format to "raw". |
| "output" | { "format": "resp", "timestamp": "iso" } | Set output format to "resp" and timestamp format to "iso". |

The field is a dictionary with two possible values. The first one is `output.format` . It can be set to "resp" or "csv". The first value is used by default. The output will be formatted using [RESP serialization](writing-data.md#serialization) format. The same that is used to send data to Akumuli. The second value changes the output format to CSV. This is how the output of the query will look with `output.format` set to "csv":

```text

test tag=Foo, 20160118T173724.646397000, 999996test tag=Foo, 20160118T173724.647397000, 999997test tag=Foo, 20160118T173724.648397000, 999998test tag=Foo, 20160118T173724.649397000, 999999
```

The second field is `output.timestamp`.  It controls formatting of the timestamps in the output of the query. If it's set to "raw" Akumuli will format timestamps as 64-bit integers.

```text

test tag=Foo, 1453127844646397000, 999996test tag=Foo, 1453127844647397000, 999997test tag=Foo, 1453127844648397000, 999998test tag=Foo, 1453127844649397000, 999999
```

If it's set to "iso" timestamps will be formatted according to ISO8601 standard.

### Filter Field

Filter field can be used to filter data-points by value.

| Field | Format | Description |
| --- | --- | --- |
| "format" | { "gt": 10, "lt": 100 } | Filter out all values less or equal to 10 and greater or equal to 100. |
| "format" | { "ge": 0, "le": 1 } | Filter out all negative values and all values greater then one. |

This field should contain a dictionary with the predicates. The possible predicates are "gt" \(greater than\), "ge" \(greater or equal\), "lt" \(less than\), and "le" \(less or equal\). It is possible to combine two predicates if you want to read values that fit some range, for instance `"filter: {"gt": 0, "lt": 10 }` will select all values between 0 and 10, but not 0 and 10. You can use only predicate if needed.

The use of filter field can speed up query execution if the number of returned values is small. In this case the query engine won't read all the data from disk but only those pages that have the data the query needs.

### Limit and Offset Fields

You can use `limit` and `offset` query fields to limit the number of returned tuples and to skip some tuples at the beginning of the query output. This fields works the same as LIMIT and OFFSET clauses in SQL.

Don't use this fields if you need to read all the data in chunks. Akumuli executes queries lazily. To read data in chunks, you can issue a normal query \(without limit and offset\) and read the first chunk \(without disconnecting from the server afterwards\). When you done with the first chunk you can read the next one, and so on. The query will be executed as far as you read data through the TCP connection. When you'll stop reading to process the data the query execution on the server will pause. It will resume when you'll continue reading.

