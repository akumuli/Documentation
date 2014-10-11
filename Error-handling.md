Akumuli uses error codes for error handling. Every operation that can fail returns value of type `aku_Status` (alias for int). All possible error codes are listed in akumuli_def.h header. AKU_SUCCESS is success error code and it's equal to zero, this allows you to write code like this:
```cpp
if (!aku_open_database(...)) {
    // error handling goes here
    ...
}
```
All error codes that represents error starts with AKU_E prefix, for example AKU_EBUSY, AKU_EOVERFLOW etc. There is special error code - AKU_EGENERAL that represents any error, usually if this error is unexpected but doesn't severe enough to trigger panic or if there is several errors that can't be mapped to akumuli error codes. Rule of thumb here - you didn't need to handle AKU_EGENERAL in a special way.

Akumuli doesn't uses exceptions for anything instead panic. If akumuli is faced with unrecoverable error - it throws exception, the only puprose of this exception - generate core dump. If this behavior is not desired - you can implement your own panic handler.

Panic handler is a callback function with signature:
```cpp
void (const char* message);
```
This function can be set when library is initialized with aku_initialize function. Panic handler must throw exception, if panic handler didn't throw exception - akumuli will throw it's own exception anyway! Panic handler can't be used to suppress exception - it can be used to replace library internal exception type with your own. Akumuli doesn't catch anything inside and your exception will pass throw it's code unwinding the stack.

After panic is triggered - akumuli is not guaranted to work correctly. Panic is triggered only in a case of unexpected error. Usually this is logic error inside a library code. All possible I/O, usage and configuration errors is handled by library using error codes. There is no way you can use akumuli after panic is triggered. Period.

Example:
```cpp
void panic_handler(const char* m) {
    LOG4CXX_ERROR(s_logger, "Panic is triggered, " << m);
}

...

aku_initialize(&panic_handler);
```

Panic handlers is optional, if you don't set it - default panic handler will be used and akumuli will throw it's own internal exception.