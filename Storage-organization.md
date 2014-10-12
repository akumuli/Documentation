Akumui stores data in large binary files with fixed size. This files is called volumes, each volume is 4Gb in size. When you create database, you need to specify number of volumes that will be used to store data. Volumes organaized in ring buffer.
```
    +----+     +----+     +----+
 +->|vol1|---->|vol2|---->|vol3|---+
 |  +----+     +----+     +----+   |
 +---------------------------------+
```
When you starts write to the database your writes goes to the first volume (vol1), when first volume is full, your writes goes to second volume and then to third. When third volume is full (and it's the last empty volume we have) - first volume will be cleared and used to write new data.

### Temporary volumes
Sometimes you can see extra volume with 'tmp' extension. It's created when tail volume need to be recycled but somebody still reading data from it. In this case akumuli can't stop client or wait unitl read will be completed, it just copies existing tail volume and creates a new one. After all cursors that reads this 'tmp' volume finishes it will be deleted.