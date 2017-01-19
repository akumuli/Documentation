### Naming conventions

Akumuli uses `aku_` prefix for all API functions and types in `libakumuli.h` and `AKU_` prefix for all definitions in `akumuli_def.h`. All other code doesn't use any prefixes for types and functions. 

- All type names should start with capital letter and use camel case (e.g. DataFormatter).
- All variable names should use lower case letters and underscore (e.g. local_var_name).
- All method and function names should use lower case letters and underscore (e.g. void format_data(...)).
- Enumerations should contain only capital letters and underscores (e.g. enum { BUFFER_SIZE = 0x1000 }).
- Constants and definitions should use only uppercase letters and underscores (e.g. #define AKU_LOG_ERROR 2).

### Coding conventions

- C-style cast shouldn't be used anywhere.
- The implicit type cast shouldn't be used anywhere.
- Constructors with single argument should be marked as `explicit`.
- Exceptions use should be limited to cases when the error should propagate through several stack frames.
- In most cases, error codes should be used (aku_Status).
- In the case of unrecoverable error `panic` should be used instead of error code or exception.
- If several values should be returned from the function std::tuple should be preferred to out parameters.

### Error handling

Akumuli uses error codes for error handling. Every API operation that can fail returns value of type `aku_Status` (alias for `int`). All possible error codes are listed in `akumuli_def.h` header. `AKU_SUCCESS` is a success error code and it's equal to zero, this allows you to write code like this:

```cpp
if (!aku_open_database(...)) {
    // error handling goes here
    ...
}
```


All error codes that represent errors starts with AKU_E prefix for example, `AKU_EBUSY`, `AKU_EOVERFLOW` etc. There is a special error code - `AKU_EGENERAL` that represents any error usually, if this error is unexpected but is not severe enough to trigger panic or if there are several errors that can't be mapped to Akumuli error codes. Rule of thumb here - you don't need to handle `AKU_EGENERAL` in a special way.

Every error code can be converted to a string using `aku_error_message` Akumuli API call:
```cpp
auto status = aku_open_database(...);
if (status != AKU_SUCCESS) {
    cerr << "can't open database " << aku_error_message(status) << endl;
}
```
This function doesn't allocate memory.

Internally, error codes should be preferred to exceptions unless we're dealing with some complex error handling scenario. For example, the protocol parser in `akumulid` code returns error codes in the case when there is not enough data and we need to wait for more to come. But when parser meets ill-formed input it throws an exception. This exception is handled on the lower level of the stack, in the TCP server. TCP server can send the response to the client and shutdown the connection.

If libakumuli is faced with unrecoverable error - it throws an exception, the only purpose of this exception is to generate a core dump. If this behavior is not what you need - you can implement your own panic handler. E.g. if you want to catch all exceptions or don't want any exceptions to cross library boundary - use `std::terminate` instead.

Panic handler is a callback function with signature:
```cpp
void (const char* message);
```
This function can be set when the library is initialized using `aku_initialize` function. The panic handler must throw an exception if the panic handler didn't throw an exception then Akumuli will throw its own exception! Panic handler can't be used to suppress exception - it can be used to replace library internal exception type with your own. Akumuli doesn't catch anything inside, so your exception will pass through its code unwinding the stack.

The panic should be triggered only in the cases when it is impossible to guarantee correct behavior. 

Example:
```cpp
void panic_handler(const char* m) {
    throw MyError(m);
}

...

aku_initialize(&panic_handler);
```
