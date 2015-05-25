### Short term goals

* Storage engine with good compression and write throughput (completed)
* Tags and metrics, it should be possible to organize data using set of metrics and tags (completed)
* Query language. It should be possible to use tags and metrics to query data. Various aggregation functions should be available (sum, join, etc). (work in process)
* Server application. Data ingestion through special protocol over TCP. Querying throw basic HTTP (chunked transfer encoding to return large amounts of data). (work in process)

### Long term goals

* Data mining support
    * Special tasks
        * Query by content: kNN, full sequence matching, subsequence matching;
        * Clustering: full sequence clustering, subsequence clustering;
        * Classification;
        * Segmentation (summaryzation);
        * Prediction;
        * Anomaly detection;
        * Motif discovery;