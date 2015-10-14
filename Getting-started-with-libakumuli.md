Building
--------

Akumuli uses CMake build system. CMake 2.8 or higher is required. Dependencies:
* Boost libraries version 1.52 or higher.
* Apache Portable Runtime (libapr).

All dependencies can be installed in ubuntu 14.04 using prerequisites.sh script.
When all dependencies where installed, akumuli can be built using these commands:
```
> cd build_dir
> cmake path_to_sources -G "Unix Makefiles"
> make
```

Working with library
--------------------
Akumuli headers are located in `include` directory. Header "akumuli.h" contains all function definitions, "akumuli_def.h" - macrodefinitions, "version.h" - version definitions and "config.h" - configuration structure definition.

All Akumuli functions start with `aku` prefix (macros starts with the same uppercase prefix).
You need to call `aku_initialize` first, before you call any other function.

### Creating new database
```cpp
apr_status_t result = aku_create_database("test", "/tmp", "/tmp", 8, 
                                          nullptr, nullptr, nullptr, nullptr);

if (result != APR_SUCCESS) { exit(1); }
```
This function will create new database instance on disk. First argument of the call is database name, second - path to directory, third - path to volumes directory. In this case database will be called `test` and all data will be placed in `/tmp` directory (it must be created beforehand). Fourth parameter is more interesting, this is database size. In this example size is eight, this means that eight volumes will be created. Each volume is 4Gb and resulting database size will be 32Gb. After call in tmp directory 8 files with "volume" extension will be created, each is 4Gb in size. Disk space will be reserved beforehand, during call to `aku_create_database`.

Last four parameters can be used to specify optional configuration parameters, we don't need it at this moment.

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
Function `aku_open_database` can open database that already exists. Structure `params` must contain some useful parameters, we interested in second one - 1000, this is a window size. After call to aku_open_database we can check its state with `aku_open_status` this function returns status code for the open operation. Variable `db` will always contain pointer to database instance, no matter what it is, even if file doesn't exist. In this case we can check for error using `aku_open_error` function.

### Writing data
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
  if (status == AKU_EBUSY) {
    status = aku_write(db, 42, i, memr);
    if (status != AKU_SUCCESS) {
      exit(1);
    }
  }
}
```
This code will write one million values to the database. First parameter to `aku_write` function is previously opened database instance, second parameter is a parameter id (sequence id), third parameter `i` is a timestamp, and last `memr` is memory range that points to payload - useful data that we can send to database.

If we get `AKU_EBUSY` error - we need to rewrite the data, but only ones! This happens when we try to write the data while merging or syncing.

### Reading
Let's build a query and run it.
```cpp
aku_ParamId params[] = {42};
aku_TimeStamp begin = 1000000ul;
aku_TimeStamp end = 0ul;
aku_SelectQuery* query = aku_make_select_query( begin
                                              , end
                                              , 1
                                              , params);
aku_Cursor* cursor = aku_select(db, query);
```
We introduce three new variables. The first one - `params` is a list of parameter ids that we want to read, in this case we need to send only one parameter id. Second and third parameters define time range. Note that `begin` variable is larger that `end`. This is because we want to read the data in reverse direction, from larger timestamps to lower.

This code doesn't read data from disk, it just creates cursor object. We can read data from disk using this cursor. Let's try to actually read data!
```cpp
const int NUM_ELEMENTS = 0x100;
while(!aku_cursor_is_done(cursor)) {
    aku_Status err = AKU_SUCCESS;
    if (aku_cursor_is_error(cursor, &err)) {
        std::cout << aku_error_message(err) << std::endl;
        return false;
    }
    aku_TimeStamp timestamps[NUM_ELEMENTS];
    aku_PData pointers[NUM_ELEMENTS];
    uint32_t lengths[NUM_ELEMENTS];
    int n_entries = aku_cursor_read_columns(cursor, timestamps, 
                                            nullptr, pointers, 
                                            lengths, NUM_ELEMENTS);
    for (int i = 0; i < n_entries; i++) {
        uint64_t value = *static_cast<const uint64_t*>(pointers[i]);
        assert(timestamp[i] == value);
    }
}
```
In first line we check that there is some data to read using `aku_cursor_is_done` function. After that we checking cursor for errors using `aku_cursor_is_error` function. If it returns true, than we get an error that can be converted to human readable form using `aku_error_message` function.

After that we can read data to three arrays - `timestamps`, `pointers` and `lengths`. Note that there is no array for parameter ids! We pass `nullptr` to the function instead of it because we don't need it. All parameter ids will be the same anyway. After call to `aku_cursor_read_columns` this arrays will be filled with data elements ordered by timestamp (in reverse order). We can compare timestamp to stored value to see that everything is OK.

After that we need to close cursor and database.
```cpp
aku_close_cursor(cursor);
aku_close_database(db);
```
And finally we can remove the database completely.
```cpp
aku_remove_database("/tmp/test.akumuli", nullptr);
```