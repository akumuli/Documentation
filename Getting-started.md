## Beginning
All Akumuli functions starts with `aku` prefix (macros starts with the same uppercase prefix).
You need to call `aku_initialize` first, before you call any other function.

## Creating new database
```cpp
apr_status_t result = aku_create_database("test", "/tmp", "/tmp", 8, 
                                          nullptr, nullptr, nullptr, nullptr);

if (result != APR_SUCCESS) { exit(1); }
```
This function will create new database instance on disk. First argument of the call is database name, second - path to directory, third - path to volumes directory. In this case database will be called `test` and all data will be placed in `/tmp` directory (it must be created beforehand). Fourth parameter is more interesting, this is database size. In this example size is eight, this means that eight volumes will be created. Each volume is 4Gb and resulting database size will be 32Gb. After call in tmp directory will be created 8 files with "volume" extension, each is 4Gb is size. Disk space allocated beforehand, during call to `aku_create_database`.

Last four parameter can be used to specify optional configuration parameters, we don't need this at this moment.

This call returns APR status, it can be compared to `APR_SUCCESS` constant and examined with `libapr` function `apr_strerror` to get human readable error message.