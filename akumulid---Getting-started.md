###Installation

TODO: build from source
TODO: install using binary distribution packages

##First steps

###Configuration

You should create configuration file first. This can be done using command:
```
> akumulid --init
OK configuration file created at: "/home/username/.akumulid"
```
Now you can edit configuration file `~/.akumulid`. This configuration file contains default settings and comments. Two main configuration parameters are `path` and `nvolumes`. First should contain path to directory when database files should be stored. By default akumuli stores files in `~/.akumuli` directory. You can change this to whatever you like (I'm using `path=/tmp` to run tests most often). Second parameter `nvolumes` should contain number of volumes that akumuli can use to store data. Each volume's size is 4Gb so choose this value wisely.

###Database creation

Now we can create database itself! Run this command:
```
> akumuli --create
OK database created, path: /home/username/.akumuli
```
You can check that database files is actually created by running `~/.akumuli`. This directory shouldn't be empty. (NOTE: you can delete all this files by running the following command: `akumulid --delete`)

###Configuring akumuli

Let's return to configuration file (`~/.akumulid`). You can read parameter's descriptions in configuration file. Most of this parameters should be leaved as is. The most important one is `window_width`. This parameter controls time interval for which akumuli will store values in memory. Setting it to some large value (tens of seconds or minutes) if high ingestion rate is expected is not a good idea. If high write speed is desirable this value should be small.

Another important parameter is a `compression_threshold`. Generally speaking this parameter defines how many values should be compressed and stored together. Larger value means that compressed chunks on disk will be larger. For compression algorithm to be efficient this parameter should be larger than expected dataset cardinality. So for example if you have 10K different, periodically updated metrics, you will get better compression ratio when `compression_threshold` will be equal 100000 or more. Smaller `compression_threshold` means faster random reads.

###Running server

To run `akumulid` as a server - just run it without parameters:
```
> akumulid
OK UDP  server started, port: 8383
OK TCP  server started, port: 8282
OK HTTP server started, port: 8181
```
Now you can write data through TCP or UDP and read data using HTTP.