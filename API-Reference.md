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

### Monitoring