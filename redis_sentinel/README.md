**Thực hiện trên cả 3 servers**

1. Khai báo rule firewalld
```
firewall-cmd --permanent --new-zone=redis
firewall-cmd --permanent --zone=redis --add-port=6379/tcp
firewall-cmd --permanent --zone=redis --add-port=26379/tcp
firewall-cmd --permanent --zone=redis --add-port=16379/tcp
firewall-cmd --reload

firewall-cmd --permanent --zone=redis --add-source=client_server_private_IP
```

2. Optimize
```
optimize_redis.sh
echo "redis hard nofile 100000" > /etc/security/limits.d/redis.conf
echo "redis soft nofile 100000" >> /etc/security/limits.d/redis.conf
echo "redis hard nproc 100000" >> /etc/security/limits.d/redis.conf
echo "redis soft nproc 100000" >> /etc/security/limits.d/redis.conf
sysctl vm.overcommit_memory = 1
echo vm.overcommit_memory = 1 >> /etc/sysctl.conf
echo 1 > /proc/sys/vm/swappinesss
echo 'vm.swappiness = 1' >> /etc/sysctl.conf
sysctl -w fs.file-max = 1000000
echo never > /sys/kernel/mm/transparent_hugepage/enabled

```
3. Cài đặt Redis

**Redis Installation**
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

**Now lets create required directories**
```
mkdir -p /etc/redis /var/log/redis /var/lib/redis/redis /var/lib/redis/sentinel
cp /opt/redis-6.2.14/redis.conf /etc/redis/redis.conf
```

**Also add less-privileged user to run Redis daemon**
```
useradd redis -s /usr/sbin/nologin
passwd -l redis

```

**Change config redis.conf**
```
sed -i 's/bind 127.0.0.1 -::1/bind 0.0.0.0/'g /etc/redis/redis.conf
echo requirepass 5Kx98DsA#2024 >> /etc/redis/redis.conf && echo masterauth 5Kx98DsA#2024 >> /etc/redis/redis.conf

sed -i 's/^\(logfile.*\)$/# \1/' /etc/redis/redis.conf
sed -i 's/^\(dir.*\)$/# \1/' /etc/redis/redis.conf
echo logfile "/var/log/redis/redis.log" >> /etc/redis/redis.conf && echo dir "/var/lib/redis/redis" >> /etc/redis/redis.conf

sed -i 's/^\(daemonize.*\)$/#\1/' /etc/redis/redis.conf
echo daemonize yes >> /etc/redis/redis.conf && echo supervised systemd >> /etc/redis/redis.conf
```
Note: Chỉ thực hiện een 2 server slave
```
echo replicaof 10.54.132.33 6379 >> /etc/redis/redis.conf # IP redis master
```

**Create service restart**
```
cat << EOF > /etc/redis/sentinel.conf
port 26379
daemonize yes				
logfile "/var/log/redis/sentinel.log"
dir "/var/lib/redis"
sentinel monitor mymaster 10.54.132.33 6379 2 # IP set master slave
sentinel down-after-milliseconds mymaster 30000
sentinel failover-timeout mymaster 180000
sentinel auth-pass mymaster 5Kx98DsA#2024
sentinel parallel-syncs mymaster 1
EOF


cat << EOF > /etc/systemd/system/redis-sentinel.service
[Unit]
Description=Advanced key-value store
After=network.target
Documentation=http://redis.io/documentation, man:redis-sentinel(1)

[Service]
Type=forking
ExecStart=/usr/local/bin/redis-sentinel /etc/redis/sentinel.conf
ExecStop=/bin/kill -s TERM $MAINPID
#PIDFile=/run/redis-sentinel.pid
TimeoutStopSec=0
Restart=always
User=redis
Group=redis
RuntimeDirectory=sentinel
RuntimeDirectoryMode=2755

UMask=007
PrivateTmp=yes
LimitNOFILE=65535
PrivateDevices=yes
ProtectHome=yes
ReadOnlyDirectories=/
ReadWriteDirectories=-/var/lib/redis/sentinel
ReadWriteDirectories=-/var/log/redis
ReadWriteDirectories=-/run/sentinel

NoNewPrivileges=true
CapabilityBoundingSet=CAP_SETGID CAP_SETUID CAP_SYS_RESOURCE
RestrictAddressFamilies=AF_INET AF_INET6 AF_UNIX


# redis-sentinel can write to its own config file when in cluster mode so we
# permit writing there by default. If you are not using this feature, it is
# recommended that you replace the following lines with "ProtectSystem=full".
ProtectSystem=true
ReadWriteDirectories=-/etc/redis

[Install]
WantedBy=multi-user.target
Alias=sentinel.service
EOF


cat << EOF > /etc/systemd/system/redis.service
[Unit]
Description=Redis persistent key-value database
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/redis-server /etc/redis/redis.conf --supervised systemd
ExecStop=/usr/libexec/redis-shutdown
Type=forking
User=redis
LimitNOFILE=65536
Group=redis
RuntimeDirectory=redis
RuntimeDirectoryMode=0755

[Install]
WantedBy=multi-user.target
EOF
```

**Chown owner**
```
chown -R redis. /etc/redis && chown -R redis. /var/log/redis && chown -R redis. /var/lib/redis && chown -R redis. /opt/redis-6.2.14
```

**Start service**
```
systemctl daemon-reload

systemctl enable redis && systemctl enable redis-sentinel && systemctl restart redis && systemctl restart redis-sentinel && systemctl status redis && systemctl status redis-sentinel
```

4. Verify sentinel cluster
```
redis-cli -p 26379 info Sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=10.255.76.31:6379,slaves=2,sentinels=3

redis-cli -p 26379 SENTINEL get-master-addr-by-name mymaster
1) "10.255.76.31"
2) "6379"
```

