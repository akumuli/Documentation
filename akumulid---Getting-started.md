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
