# API Endpoints

{% api-method method="get" host="http://<host>:8181" path="/api/stats" %}
{% api-method-summary %}
Statistics
{% endapi-method-summary %}

{% api-method-description %}
This endpoint allows you to get storage statistics of the running Akumuli instance. It's also can be used as a test endpoint to test service availability.  
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}
Storage statistics successfully retrieved.
{% endapi-method-response-example-description %}

```javascript
{    "volume_0":    {        "free_space": "0",        "file_name": "\/root\/.akumuli\/db_0.vol"    },    "volume_1":    {        "free_space": "0",        "file_name": "\/root\/.akumuli\/db_1.vol"    },    "volume_2":    {        "free_space": "0",        "file_name": "\/root\/.akumuli\/db_2.vol"    },    "volume_3":    {        "free_space": "2027974656",        "file_name": "\/root\/.akumuli\/db_3.vol"    }}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="post" host="http://<host>:8181" path="/api/query" %}
{% api-method-summary %}
Read query
{% endapi-method-summary %}

{% api-method-description %}
This endpoint allows you to retrieve time-series data from the database. The client should provide valid query. The response will use chunked transfer encoding to return the results. The results are encoded using the RESP protocol.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-body-parameters %}
{% api-method-parameter name="Query Body" type="object" required=true %}
JSON encoded query
{% endapi-method-parameter %}
{% endapi-method-body-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```
+RESP encoded data
+just like this
```
{% endapi-method-response-example %}

{% api-method-response-example httpCode=400 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```
-RESP encoded error message
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="post" host="http://<host>:8181" path="/api/search" %}
{% api-method-summary %}
Search
{% endapi-method-summary %}

{% api-method-description %}
This API endpoint can be used to retreive the metadata like series names and tag values.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="Query Body" type="string" required=true %}
JSON encoded query
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```
+RESP encoded output
```
{% endapi-method-response-example %}

{% api-method-response-example httpCode=400 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```
-RESP encoded error message
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="post" host="http://<host>:8181" path="/api/suggest" %}
{% api-method-summary %}
Suggest
{% endapi-method-summary %}

{% api-method-description %}
This endpoint can be used to retrieve metric names, tag names, and tag values. It powers autocomplete function of the **akumuli-datasource** for Grafana. 
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="Query Body" type="string" required=false %}
JSON encoded query
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```
+RESP encoded list or results
```
{% endapi-method-response-example %}

{% api-method-response-example httpCode=400 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```
-RESP encoded error message
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="get" host="http://<host>:8181" path="/api/function-names" %}
{% api-method-summary %}
List functions
{% endapi-method-summary %}

{% api-method-description %}
This endpoint can be used to retrieve the list of functions that can be used in queries.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}
List of functions
{% endapi-method-response-example-description %}

```

absaccumulatecmacusumdiffdivideewmaewma-errorfrequent-itemsheavy-hittersmultiplyratescalesmasma-errorsumtop
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}



