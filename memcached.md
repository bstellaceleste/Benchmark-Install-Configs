> Memcached is an in-memory key-value store for small chunks of arbitrary data (strings, objects) from results of database calls, API calls, or page rendering.
> 
> You need a load injector to fill the db and be able to benchmark

#### Installation

1. **Memcached**
- From apt
`sudo apt update && sudo apt install memcached`

- From sources
```
wget http://memcached.org/latest
mv latest memcached-1.x.x.tar.gz
tar -zxvf memcached-1.x.x.tar.gz
cd memcached-1.x.x
./configure && make && sudo make install
```
If installed from source, the daemon will automatically be launched.

To start memcached manually:
`memcached -d -m 1024 -u stella -p 11211 -l 127.0.0.1`

Check the installation was successful
`pidof memcached` or `sudo netstat -ano | grep 11211`

2. **Libmemcached**
   
Libmemcached will install memaslap, which is a popular load generation and benchmarking tool for memcached servers (or servers abiding by memcached protocol). 

It supports configurable workload such as requests rate, the number of concurrent connections, key size and value size distribution, get/set proportion, and so on. Memaslap also displays key statistics such as latency. [https://medium.com/swlh/the-complete-guide-to-benchmark-the-performance-of-memcached-on-ubuntu-16-04-71edeaf6e740#:~:text=Memaslap%20is%20a%20popular%20load,set%20proportion%2C%20and%20so%20on]

- download the [source code](https://launchpad.net/libmemcached/+download) 
- install dependencies: `sudo apt-get install libevent-dev`
- configure: `./configure --enable-memaslap`
- edit the Makefile: (the Makefile is generated after configuration)
```
# Edit Makefile by adding LDFLAGS "-L/lib64 -lpthread"
# https://bugs.launchpad.net/libmemcached/+bug/1562677
### diff of Makefile
# -LDFLAGS =
# +LDFLAGS = -L/lib64 -lpthread 
###
```
- install: `sudo make install`


#### Run memaslap
memaslap will be installed with the libmemcached
`memaslap -s 127.0.0.1:11211 -t 20s`

If issuing *libmemcached.so.11: cannot open shared object file: No such file or directory* simply do `ldconfig`

#### YCSB
This is another load injector that could be used instead of memaslap

1. install java and maven: `sudo apt update && sudo apt install default-jre && sudo apt install maven`

2. clone the git: `git clone http://github.com/brianfrankcooper/YCSB.git`, `cd YCSB`

3. compile: `mvn -pl site.ycsb:memcached-binding -am clean package`

4. load the date for the memcached db: `./bin/ycsb load memcached -s -P workloads/workloada > outputLoad.txt`

5. then run using the previously generated date: `./bin/ycsb run memcached -s -P workloads/workloada > outputRun.txt`

> If you encounter a `site.ycsb.DBException: java.lang.NullPointerException`, you must explitly indicate the host as follows:
> `./bin/ycsb load memcached -s -P workloads/workloada -p "memcached.hosts=127.0.0.1" > outputLoad.txt`
   
