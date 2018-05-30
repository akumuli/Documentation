---
description: Akumuli query language reference.
---

# Query Language

Akumuli uses REST interface for data retrieval. You should use [**api/query**](api-endpoints.md#read-query) endpoint for data queries and [**api/search**](api-endpoints.md#search) endpoint for metadata queries.

## Query Object

To retrieve the data you should create a query object that describes what data you need and what shape it should have. 

## Query Types

Query object can be of one of the following types:

* Select query
* Aggregate query
* Group-aggregate query
* Join query

### Select Query

This is a simpliest possible query type. It can return raw time-series data without aggregation. Select query can have a post-processing steps \(e.g. rate or sliding window computation\).

Select query can return more than one series but they should have the same metric name.

| Field | Required | Commentary |
| --- | --- | --- | --- |
| "select" | Yes | Metric name |
| "range" | Yes | Time range |
| "where" | No | Tag filter |

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

This field defines the query type. If this field is used the query will be a simple [select query](query-language.md#select-query). You can provide only one "select" field. This field can have only one metric name.

