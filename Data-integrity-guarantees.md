Data integrity aspect is taken seriously in Akumuli. It can tolerate any failure with small data loss, even if failure occurs during `msync` syscall. Akumuli's simple storage format and periodic checkpoints make this possible.

### What can be lost?
If failure occurs, most recent writes can be lost up to two `window_size` depths. So if your `window_size` is 100 seconds, up to 200 seconds can be lost.
