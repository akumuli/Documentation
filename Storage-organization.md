Akumui stores data in large binary files with fixed size. These files are called volumes, each volume is 4Gb in size. When you create a database, you need to specify the number of volumes that will be used to store data. Volumes are organaized in a ring buffer.
```
    +----+     +----+     +----+
 +->|vol1|---->|vol2|---->|vol3|---+
 |  +----+     +----+     +----+   |
 +---------------------------------+
```
When you start writing to the database, your writes go to the first volume (vol1), when first volume is full, your writes go to second volume and then to the third one. When the third volume is full (and it's the last empty volume we have) - first volume will be cleared and used to write new data.

### Temporary volumes
Sometimes you can see extra an volume with a 'tmp' extension. It's created when tail volume needs to be recycled but somebody is still reading data from it. In this case akumuli can't stop the client or wait unitl read is completed, it simply copies existing tail volume and creates a new one. After all cursors reading this 'tmp' volume have finished, it will be deleted.

### Metadata storage
All parameters, volumes and series names is stored in sqlite3 database. Database file has extension ".akumuli", it can be opened and queried using sqlite3 command line tool.

### API
New storage can be created using akumuli API using function aku_create_database.
```cpp
apr_status_t aku_create_database( const char*  file_name
                                , const char*  metadata_path
                                , const char*  volumes_path
                                , int32_t      num_volumes
                                // optional args
                                , const uint32_t *compression_threshold
                                , const uint64_t *window_size
                                , const uint32_t *max_cache_size
                                , aku_printf_t logger
                                );
```
Function creates a new storage with specified number of volumes and other parameters.
* `file_name` - metadata file name (without path).
* `metadata_path` - metadata file path.
* `volumes_path` - path to volumes directory, all volumes will be created in this directory.
* `num_volumes` - number of volumes to create, each volume will be 4Gb size and minimal number of volumes is two.
* `compression_threshold` - optional compression threshold. This parameter determines how storage needs to compress data if insert rate is small. Large `compression_threshold` value means that any data element can be held in memory longer if insert rate is too slow. Recommended value is `window_size/2`. This parameter can be null, in which case some viable default will be used.
* `window_size` - sliding window depth. Large `window_size` means that more values will be held in memory, smaller `window_size` means that more data elements will be discarded as late writes.
* `max_cache_size` - upper limit on memory cache size in bytes.
* `logger` - logger callback that will be used by this API call.

Function returns a value, that is an APR error code and can be decoded using `libapr`.
