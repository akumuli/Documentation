# Hardware Requirements

## Memory

Every individual time-series Akumuli is stored on disk using a memory resident component. This memory resident component is composed using IOVec blocks. Each IOVec structure contains up to 4KB of memory. The number of IOVec blocks needed to handle individual time-series depends on its disk-resident size. Akumuli stores time-series in LSM-tree like structure every level of which allocates a single IOVec block. The number of extents depend on disk space used by time seires. This memory/disk space dependency looks like this:

| Memory | Disk |
| :--- | :--- |
| 4KB | 0 |
| 8KB | 128KB |
| 12KB | 4MB |
| 16KB | 128MB |
| 20KB | 4GB |
| 24KB | 128GB |
| 28KB | 4TB |
| 32KB | 128TB |
| 36KB | 4PB |
| 40KB | 128PB |

The individual data element can occupy from 0.1 to 9 bytes, depending on randomness of the data. For many monitoring workloads that number can be in 1-2 byte range \(integer values\). This mean that time-series with hundreds of millions values will eat up less than 16KB of RAM. 

IOVec blocks are allocated with 1KB step. So for the 1-element time-series only 1KB will be allocated. When series won't fit into 1KB the second 1KB chunk will be allocated and so on, until IOVec block won't be filled. This means that the numbers in the table above are worst case numbers. On average IOVec blocks are half full on every level. For instance, this means that for the series that occupies from 128MB to 4GB on disk Akumuli may need to use 5-20KB for the memory-resident component. Averaged for high number of individual time-series that will give us 11KB of RAM per/series in this specific scenario.

The expected memory requirements **per-series** have to be multiplied by data-set cardinality. For instance, if we have 1-million time-series and all individual time-series are below 4GB on disk we can expect Akumuli to use 20GB of RAM in worst case. It will actually use just above 10GB because of partial IOVec allocation. This is a terrific result since it will allow the database to store 10E15 data-points. 

This won't work as good if you have a lot of small time-seires. For instance, if you have 10-million series and each one of those is small and fits 128KB around 40GB of RAM will be needed.

## Disk

Akumuli is designed for SSD and NVMe drives. It writes data sequentially and frees it in large blocks to avoid write amplification. All reads and writes are page alighned for the same purpose. Akumuli don't read anything from disk to write new data so queries can't deplete the read bandwidth and affect write speed. The database will work on HDD but it will work slower, especially on read side.

## CPU

Ingestion depend on number of available CPU's. In best case every CPU available for ingestion gives around 1M write/sec \(if dictionary mode is used\). With OpenTSDB format the per-CPU write effeciency will get lower. CPU's can be provisioned for ingestion in configuration. 

