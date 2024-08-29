


Thực hiện trên cả 3 servers

Cài đặt Redis
```
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


useradd redis
mkdir /etc/redis; mkdir /var/log/redis; mkdir -p /var/lib/redis/redis; mkdir -p /var/lib/redis/sentinel
cp /opt/redis-6.2.14/redis.conf /etc/redis/redis.conf
```
