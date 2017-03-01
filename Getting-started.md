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
> akumulid --create
OK database created, path: /home/username/.akumuli
```
You can check that database files is actually created by running `~/.akumuli`. This directory shouldn't be empty. (NOTE: you can delete all this files by running the following command: `akumulid --delete`)

###Configuring akumuli

Let's return to configuration file (`~/.akumulid`). You can read parameter's descriptions in configuration file. The most important parameters are:

* `path` - tells Akumuli where database volumes should be stored (default value is ~/.akumuli)
* `nvolumes` - number of volumes that should be created (this parameter is only used when you run `akumulid --create` command)
* `volume_size` - size of the individual volume (this parameter is only used when you run `akumulid --create` command)
* `HTTP.port` - port used by HTTP server
* `TCP.port` - port used by TCP server
* `TCP.pool_size` - number of threads that should be used to process data (should be less then number of CPUs)
* `UDP.port` - port used by UDP server
* `UDP.pool_size` - number of threads that should be used to process data (should be less then the number of CPUs)
* Log4cpp configuration


###Running server

To run `akumulid` as a server - just run it without parameters:
```
> akumulid
OK UDP  server started, port: 8383
OK TCP  server started, port: 8282
OK HTTP server started, port: 8181
```
Now you can write data through TCP or UDP and read data using HTTP.