Akumuli doesn't depend on any logger. Instead, it allows user to use the same logger as the host application uses. You can set your callback function that will be called for every log message. 

Callback signature: 
```cpp
void (int tag, const char* message)
```

First parameter - `tag` - is used to distinguish between multiple open databases. If message doesn't correspond to any open database, tag value will be zero. It can be used, for example, to pass messages from different databases to different log files.

If callback is not set - default mechanism will be used, all messages will be written to `stderr`.

All akumuli log messages are made equal, they don't have level or severity. But you can expect that library won't call logger callback frequently. Akumuli doesn't log any trace or debug level information, only important warnings and errors. That's why default logger sends everything to stderr instead of stdout!

Example:
```cpp
void my_logger(int tag, const char* message) {
    LOG4CXX_ERROR(my_logger, message);
}

aku_FineTuneParams params;
params.debug_mode = 0;
params.max_late_write = 10000;
params.logger = &my_logger;

auto db = aku_open_database(DB_META_FILE, params);
```
