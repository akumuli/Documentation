Akumuli uses error codes for error handling. Every operation that can fail returns value of type `aku_Status` (alias for `int`). All possible error codes are listed in `akumuli_def.h` header. `AKU_SUCCESS` is a success error code and it's equal to zero, this allows you to write code like this:
```cpp
if (!aku_open_database(...)) {
    // error handling goes here
    ...
}
```
All error codes that represent errors start with AKU_E prefix, for example `AKU_EBUSY`, `AKU_EOVERFLOW` etc. There is a special error code - `AKU_EGENERAL` that represents any error, usually if this error is unexpected but is not severe enough to trigger panic or if there are several errors that can't be mapped to akumuli error codes. Rule of thumb here - you don't need to handle `AKU_EGENERAL` in a special way.

Every error code can be converted to a string using `aku_error_message` akumuli API call:
```cpp
auto status = aku_open_database(...);
if (status != AKU_SUCCESS) {
    cerr << "can't open database " << aku_error_message(status) << endl;
}
```
This function doesn't allocate memory.

Akumuli doesn't use exceptions for anything other than panic. If akumuli is faced with unrecoverable error - it throws an exception, the only purpose of this exception is generating a core dump. If this behavior is not what you need - you can implement your own panic handler.

Panic handler is a callback function with signature:
```cpp
void (const char* message);
```
This function can be set when library is initialized with `aku_initialize` function. Panic handler must throw exception, if panic handler didn't throw an exception - akumuli will throw it's own exception anyway! Panic handler can't be used to suppress exception - it can be used to replace library internal exception type with your own. Akumuli doesn't catch anything inside, so your exception will pass through its code unwinding the stack.

After panic has been triggered - akumuli is not guaranted to work correctly. Panic is triggered only in case of an unexpected error. Usually this is logic error inside a library code. All possible I/O, usage and configuration errors are handled by library using error codes. There is no way you can use akumuli after panic is triggered. Period.

Example:
```cpp
void panic_handler(const char* m) {
    throw MyError(m);
}

...

aku_initialize(&panic_handler);
```

Panic handlers are optional, if you don't set it - default panic handler will be used, and akumuli will throw its own internal exception.
