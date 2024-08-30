


Thực hiện trên cả 3 servers

Redis Installation
```
yum update -y
yum groupinstall -y 'Development Tools'
yum install -y wget
wget https://download.redis.io/releases/redis-6.2.14.tar.gz
tar zxvf redis-6.2.14.tar.gz -C /opt/
cd /opt/redis-6.2.14/
make distclean
make install

Hint: It's a good idea to run 'make test' ;)

    INSTALL redis-server
    INSTALL redis-benchmark
    INSTALL redis-cli
make[1]: Leaving directory '/opt/redis-6.2.14/src'
```

Now lets create required directories
```
mkdir -p /etc/redis /var/log/redis /var/lib/redis/redis /var/lib/redis/sentinel
cp /opt/redis-6.2.14/redis.conf /etc/redis/redis.conf
```

Also add less-privileged user to run Redis daemon
```
useradd redis -M -g daemon
passwd -l redis

chown -R redis:daemon /opt/redis-6.2.14
chown -R redis:daemon /var/log/redis
chown -R redis:daemon /var/lib/redis
```

Note: Repeat these steps on all hosts.


