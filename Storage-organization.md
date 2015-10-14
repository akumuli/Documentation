Akumui stores data in fixed sized binary files. These files are called volumes, each volume is 4Gb in size. You can specify the number of volumes that Akumuli must use to store all the data. Volumes are organized in a ring buffer.
```
    +----+     +----+     +----+
 +->|vol1|---->|vol2|---->|vol3|---+
 |  +----+     +----+     +----+   |
 +---------------------------------+
```
When you start writing to the database, your writes are going to the first volume (vol1), when the first volume will be full, your writes will end up in the second volume and so on. When the last volume will be full - first volume will be recycled.

### Temporary volumes
Sometimes you will see a volume with a '*.tmp' extension. This volume is created when tail volume needs to be recycled but that volume is used at the moment. In this case Akumuli can't stop the client or wait until read operation will be completed. In this case extra disk space will be used to store new data. When the temporary volume will be freed - it will be deleted by the storage engine.

### Metadata storage
All parameters, volumes and series names are stored in sqlite3 database. Database file has ".akumuli" extension, it can be opened and queried using sqlite3 command line tool.

### Compression
Akumuli can compress data using specialized compression algorithm. It uses DeltaRLE + LEB128 for timestamps and ids. For values algorithm described in this paper is used: http://users.ices.utexas.edu/~burtscher/papers/dcc06.pdf

Compression can't be disabled.