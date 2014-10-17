### Database management functions

### Writing
```cpp
aku_Status aku_write(aku_Database* db, 
                     aku_ParamId param_id, 
                     aku_TimeStamp timestamp, 
                     aku_MemRange value);
```
Add single value to database.
* `db` - pointer to database instance
* `param_id` - parameter id
* `timestamp` - timestamp
* `value` - memory range with value

Returns status code. AKU_SUCCESS - everything OK. AKU_EOVERFLOW - page rotation is triggered, value is not writen to the storage. User must call function again.
This function must be called from writer thread.

### Reading
```cpp
AKU_EXPORT int aku_cursor_read_columns 	( 	
        aku_Cursor*     pcursor,
        aku_TimeStamp*  timestamps,
        aku_ParamId*    params,
        aku_PData*      pointers,
        uint32_t*       lengths,
        size_t          arrays_size)
```
Read data from storage in column-wise manner.


Parameters:

* `pcursor` - pointer to cursor
* `timestamps` - output buffer for storing timestamps
* `params` - output buffer for storing paramids
* `pointers` - output buffer for storing pointers to data
* `lengths` - output buffer for storing lengths of the data items
* `array_size` - specifies size of the all output buffers (it must be the same for all buffers)

**Note**

every output parmeter can be null if we doesn't interested in it's value 


### Monitoring