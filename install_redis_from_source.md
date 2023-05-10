# Install redis 3.0 from source on Ubuntu 14.04

In Centos 7/ Rhel 7
```
yum install make gcc wget
```
In Ubuntu
```
sudo apt-get update
sudo apt-get install install make gcc wget
```

Download Redis 3 stable Source package
```
wget http://download.redis.io/releases/redis-3.0.2.tar.gz
```

Untar downloaded Redis Tar ball
```
tar -xvzf redis-3.0.2.tar.gz
```
Compiling of Redis from source
```
cd redis-3.0.2
cd deps
make hiredis lua jemalloc linenoise
cd ..
make
make install
```
Install init script
```
cd utils
./install_server.sh

Welcome to the redis service installer
This script will help you easily set up a running redis server

Please select the redis port for this instance: [6379] 
Selecting default: 6379
Please select the redis config file name [/etc/redis/6379.conf] 
Selected default - /etc/redis/6379.conf
Please select the redis log file name [/var/log/redis_6379.log] 
Selected default - /var/log/redis_6379.log
Please select the data directory for this instance [/var/lib/redis/6379] 
Selected default - /var/lib/redis/6379
Please select the redis executable path [/usr/local/bin/redis-server] 
Selected config:
Port           : 6379
Config file    : /etc/redis/6379.conf
Log file       : /var/log/redis_6379.log
Data dir       : /var/lib/redis/6379
Executable     : /usr/local/bin/redis-server
Cli Executable : /usr/local/bin/redis-cli
Is this ok? Then press ENTER to go on or Ctrl-C to abort.
Copied /tmp/6379.conf => /etc/init.d/redis_6379
Installing service...
Successfully added to chkconfig!
Successfully added to runlevels 345!
Starting Redis server...
Installation successful!
```
Redis start/stop/restart/status

In CentOS 7 / RHEL 7
```
## To check status of Redis Server
systemctl stop redis_6379

## To start Redis Server
systemctl start redis_6379

## To stop Redis Server
systemctl stop redis_6379

## To restart the Redis Server
systemctl restart redis_6379
```
In Ubuntu
```
## To check the status Redis Server
service redis_6379 status

## To start Redis Server
service redis_6379 start

## To stop Redis Server
service redis_6379 stop

## To restart Redis Server
service redis_6379 restart
```
Login into Redis Server
```
redis-cli
```
Check listening ports by redis
```
ss -tanp|grep 6379
```





