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

Now we can open this database and do something useful with it.
```cpp
aku_FineTuneParams = {
  0, 1000, 0x1000000, nullptr
};
aku_Database* db = aku_open_database("/tmp/test.akumuli", params);
// check error!
aku_Status status = aku_open_status(db);
if (status != AKU_SUCCESS) {
  aku_close_database(db);
  exit(1);
}
```
Function `aku_open_database` can open database that already exists. Structure `params` must contain some useful parameters, we interested in second one - 1000, this is window size. After call to aku_open_database we can check it's state with `aku_open_status` this function return status code for the open operation. Variable `db` will always contain pointer to database instance, no matter what, even if file doesn't exists. In this case we can check for error using `aku_open_error` function.

## Writing data
Let's write some data to our new database!
```cpp
for(uint64_t i = 0; i < 1000000; i++) {
  aku_MemRange memr;
  memr.address = (void*)&1;
  memr.length = sizeof(1);
  aku_Status status = aku_write(db, 42, i, memr);
  if (status != AKU_SUCCESS && status != AKU_EBUSY) {
    exit(1);
  }
}
```
This code will write one million values to the database. First parameter to `aku_write` function is previously opened database instance, second parameter is a parameter id (sequence id), third parameter `i` is a timestamp, and last `memr` is memory range that points to payload - useful data that we can send to database.

## Reading