
### Utility functions

```cpp
void aku_initialize(aku_panic_handler_t optional_panic_handler=0);
```
> This function must be called before any other library function.
>* `optional_panic_handler` function to alternative panic handler

```cpp
const char* aku_error_message(int error_code);
```
> Convert error code to error message. Function returns pointer to statically allocated string there is no need to free it.

```cpp
void aku_console_logger(int tag, const char* message);
```
>Default logger that is used if no logging function is specified. Exported for testing reasons, no need to use it explicitly.

```cpp
void aku_destroy(void* any);
```
>Destroy any object created with `aku_make_XXX` function

### Database management functions

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
>Creates storage for new database on the hard drive
>* _file_name_ database file name
>* _metadata_path_ path to metadata file
>* _volumes_path_ path to volumes
>* _num_volumes_ number of volumes to create

>_Returns_ APR errorcode or APR_SUCCESS

```cpp
aku_Database* aku_open_database(const char        *path, 
                                aku_FineTuneParams parameters);
```
>Open recenlty create storage.
>* _path_ path to storage metadata file
>* _parameters_ open parameters

>_Returns_ pointer to new db instance, null if db doesnt exists.

```cpp
AKU_EXPORT void aku_close_database(aku_Database* db);
```
>Close database. Free resources.

### Writing

```cpp
aku_Status aku_write(aku_Database* db, 
                     aku_ParamId param_id, 
                     aku_TimeStamp timestamp, 
                     aku_MemRange value);
```
>Add single value to database.
>* _db_ pointer to database instance
>* _param_id_ parameter id
>* _timestamp_ timestamp
>* _value_ memory range with value

>_Returns_ status code. AKU_SUCCESS - everything OK. AKU_EOVERFLOW - page rotation is triggered, value is not writen to the storage. User must call function again.
>This function must be called from writer thread.

### Queries

```cpp
aku_SelectQuery* aku_make_select_query(aku_TimeStamp begin,
                                       aku_TimeStamp end, 
                                       uint32_t      n_params, 
                                       aku_ParamId*  params);
```
>Create select query with single parameter-id

```cpp
aku_Cursor* aku_select(aku_Database* db, aku_SelectQuery* query);
```
>brief Execute query
>* _query_ data structure representing search query
>_Rreturns_ new cursor.

```cpp
void aku_close_cursor(aku_Cursor* pcursor);
```
>Close cursor
>* _pcursor_ pointer to cursor.

```cpp
int aku_cursor_read_columns ( 	
        aku_Cursor*     pcursor,
        aku_TimeStamp*  timestamps,
        aku_ParamId*    params,
        aku_PData*      pointers,
        uint32_t*       lengths,
        size_t          arrays_size)
```
>Read data from storage in column-wise manner.
>* _pcursor_ - pointer to cursor
>* _timestamps_ - output buffer for storing timestamps
>* _params_ - output buffer for storing paramids
>* _pointers_ - output buffer for storing pointers to data
>* _lengths_ - output buffer for storing lengths of the data items
>* _array_size_ - specifies size of the all output buffers (it must be the same for all buffers)

>every output parmeter can be null if we doesn't interested in it's value 

```cpp
AKU_EXPORT bool aku_cursor_is_done(aku_Cursor* pcursor);
```
>Check cursor state.

```cpp
bool aku_cursor_is_error(aku_Cursor* pcursor, int* out_error_code_or_null);
```
>Check cursor error state.

### Stats and counters

```cpp
void aku_global_search_stats(aku_SearchStats* rcv_stats, bool reset=false);
```
>Get search counters.
>* _rcv_stats_ pointer to `aku_SearchStats` structure that will be filled with data.
>* _reset_ reset all counter if true

```cpp
AKU_EXPORT void aku_global_storage_stats(aku_Database *db, aku_StorageStats* rcv_stats);
```
>Get storage stats.
>* _db_ database instance.
>* _rcv_stats_ pointer to destination
