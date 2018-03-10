# Installation

You can install Akumuli on Ubuntu 14.04 using this repository: https://packagecloud.io/Lazin/Akumuli.

Alternatively, you can use this [Docker repository](https://hub.docker.com/r/akumuli/akumuli/).

## Building from source

### Ubuntu / Debian

#### Prerequisites

##### Automatic

* Run `prerequisites.sh`. It will try to do the best thing.

##### Manual

In case automatic script didn't work:

* Boost:

  `sudo apt-get install libboost-all-dev`

* log4cxx:

  `sudo apt-get install log4cxx` or `sudo apt-get install liblog4cxx-dev` on Ubuntu 16.04

* jemalloc:

  `sudo apt-get install libjemalloc-dev`

* microhttpd:

  `sudo apt-get install libmicrohttpd-dev`

* APR:

  `sudo apt-get install libapr1-dev libaprutil1-dev libaprutil1-dbd-sqlite3`

* SQLite:

  `sudo apt-get install libsqlite3-dev`

* Cmake:

  `sudo apt-get install cmake`

#### Building

1. `cmake .`
1. `make -j4`

### Centos 7 / RHEL7 / Fedora

##### Automatic

* Run `prerequisites.sh`. It will try to do the best thing.

##### Manual

In case automatic script didn't work:

* Boost:

  `sudo yum install boost boost-devel`

* log4cxx:

  `sudo yum install log4cxx log4cxx-devel`

* jemalloc:

  `sudo yum install jemalloc-devel`

* microhttpd:

  `sudo yum install libmicrohttpd-devel`

* APR:

  `sudo yum install apr-devel apr-util-devel apr-util-sqlite`

* SQLite

  `sudo yum install sqlite sqlite-devel`

* Cmake:

  `sudo yum install cmake`


#### Building

1. `cmake .`
1. `make -j4`
1. `make`

### Centos 6 / RHEL6

#### Prequisites

* Same as for RHEL7, but we need to manually install log4cxx, as there isn't a package in the repos:
```
wget http://www.pirbot.com/mirrors/apache/logging/log4cxx/0.10.0/apache-log4cxx-0.10.0.tar.gz
tar -xzvf apache-log4cxx-0.10.0.tar.gz 
cd apache-log4cxx-0.10.0
```
* Add `#include <cstring>` to: `src/main/cpp/inputstreamreader.cpp`, `src/main/cpp/socketoutputstream.cpp` and `src/examples/cpp/console.cpp`
* Add `#include <cstdio>` to: `src/examples/cpp/console.cpp`
```
./configure --prefix=/usr --libdir=/usr/lib64
make -j4
sudo make install
```
* Go on as for RHEL7

# First steps

## Configuration

You should create configuration file first. This can be done using command:
```
> akumulid --init
OK configuration file created at: "/home/username/.akumulid"
```
Now you can edit configuration file `~/.akumulid`. This configuration file contains default settings and comments. Two main configuration parameters are `path` and `nvolumes`. First should contain path to directory when database files should be stored. By default akumuli stores files in `~/.akumuli` directory. You can change this to whatever you like (I'm using `path=/tmp` to run tests most often). Second parameter `nvolumes` should contain number of volumes that akumuli can use to store data. Each volume's size is 4Gb so choose this value wisely.

## Database creation

Now we can create database itself! Run this command:
```
> akumulid --create
OK database created, path: /home/username/.akumuli
```
You can check that database files is actually created by running `~/.akumuli`. This directory shouldn't be empty. (NOTE: you can delete all this files by running the following command: `akumulid --delete`)

## Configuring Akumuli

Let's return to configuration file (`~/.akumulid`). You can read parameter's descriptions in configuration file. The most important parameters are:

* `path` - tells Akumuli where database volumes should be stored (default value is ~/.akumuli)
* `nvolumes` - number of volumes that should be created (this parameter is only used when you run `akumulid --create` command). If `nvolumes` is set to 0 the storage will be expanded on demand without deleting old data.
* `volume_size` - size of the individual volume (this parameter is only used when you run `akumulid --create` command)
* `HTTP.port` - port used by HTTP server
* `TCP.port` - port used by TCP server
* `TCP.pool_size` - number of threads that should be used to process data (should be less then number of CPUs, if you set this value to 0 the system will try to chose optimal size on start)
* `UDP.port` - port used by UDP server
* `UDP.pool_size` - number of threads that should be used to process data (should be less then the number of CPUs)
* Log4cpp configuration


## Running the server

To run `akumulid` as a server - just run it without parameters:
```
> akumulid
OK UDP  server started, port: 8383
OK TCP  server started, port: 8282
OK HTTP server started, port: 8181
```
Now you can write data through TCP or UDP and read data using HTTP.