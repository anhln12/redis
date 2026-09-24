Triển khai mô hình Redis Active-Passive

```
             Application
                  |
             Redis VIP
                  |
          +-------+-------+
          |               |
      Redis-01        Redis-02
      MASTER           REPLICA
       WRITE             READ
```

Khi Redis-01 chết, Redis-02 được PROMOTE lên Master

1. Dọn Redis package cũ

```
systemctl stop redis-server 2>/dev/null || true
systemctl disable redis-server 2>/dev/null || true

apt purge -y redis-server redis-tools redis-sentinel 2>/dev/null || true
apt autoremove -y

rm -f /etc/systemd/system/redis-server.service
systemctl daemon-reload
```

2. Xóa binary Redis
```
rm -f /usr/bin/redis-server
rm -f /usr/bin/redis-cli
rm -f /usr/bin/redis-benchmark
rm -f /usr/bin/redis-check-aof
rm -f /usr/bin/redis-check-rdb
rm -f /usr/bin/redis-sentinel
```

Refresh shell
```
hash -r
```

3. Cài dependency
```
apt update

apt install -y \
    build-essential \
    pkg-config \
    tcl \
    libsystemd-dev \
    libssl-dev \
    wget
```

4. Download Redis 7.4.1
```
cd /usr/local/src
rm -rf redis-7.4.1 redis-7.4.1.tar.gz
wget https://download.redis.io/releases/redis-7.4.1.tar.gz
tar -xzf redis-7.4.1.tar.gz
cd redis-7.4.1
```

5. Build
```
make -j"$(nproc)"
./src/redis-server --version
make install
```

6. Kiểm tra binary
```
/usr/local/bin/redis-server --version
/usr/local/bin/redis-cli --version
```

Đặt ```/usr/local/bin``` lên đầu PATH:
```
export PATH=/usr/local/bin:$PATH
hash -r
```

7. Tạo user và thư mục Redis
```
id redis >/dev/null 2>&1 || \
useradd --system --group --no-create-home --shell /usr/sbin/nologin redis

mkdir -p /etc/redis
mkdir -p /var/lib/redis
mkdir -p /var/log/redis
mkdir -p /run/redis

chown redis:redis /var/lib/redis
chown redis:redis /var/log/redis
chown redis:redis /run/redis
chown redis:redis /etc/redis
```

8. Tạo config Redis 7.4.1
```
cat > /etc/redis/redis.conf <<'EOF'
bind 127.0.0.1
protected-mode yes
port 6379

daemonize no
supervised systemd

pidfile /run/redis/redis-server.pid

dir /var/lib/redis
dbfilename dump.rdb

loglevel notice
logfile /var/log/redis/redis-server.log

appendonly no

replica-read-only yes
EOF
```

9. Tạo systemd service
```
cat > /etc/systemd/system/redis-server.service <<'EOF'
[Unit]
Description=Redis In-Memory Data Store
After=network.target
Wants=network.target

[Service]
Type=notify
User=redis
Group=redis

ExecStart=/usr/local/bin/redis-server /etc/redis/redis.conf
ExecStop=/usr/local/bin/redis-cli -p 6379 shutdown

Restart=always
RestartSec=3

LimitNOFILE=65535

PrivateTmp=true
ProtectSystem=full
ProtectHome=true
ReadWritePaths=/var/lib/redis /var/log/redis /run/redis

[Install]
WantedBy=multi-user.target
EOF
```

10. Start Redis 7.4.1
```
systemctl daemon-reload
systemctl enable redis-server
systemctl start redis-server

systemctl status redis-server --no-pager
redis-cli ping
```

11. Cấu hình Repica

* Trên master
- Kiểm tra ip master
- Config cho Master nhận được kết nối replica
```
redis-cli CONFIG GET bind
```
Nếu kết quả:
```
bind
127.0.0.1
```
Thì cần update lại config ```/etc/redis/redis.con```
```
bind 127.0.0.1 => bind 127.0.0.1 10.10.10.100
```

Restart lại redis
```
systemctl restart redis-server
```

Kiêm tra lại
```
ss -lntp | grep 6379
```
- Mở firewall trên Master
```
ufw allow from 10.10.10.101 to any port 6379 proto tcp
```

* Trên slave
- Update config bổ sung thông tin server master ```/etc/redis/redis.conf```
```
replicaof 10.10.10.100 6379
replica-read-only yes
masterauth PASSWORD # Nếu có password
```
- Restart Redis
```
systemctl restart redis-server
```
- Verify
```
redis-cli INFO replication

Output
role:slave
master_host:10.10.10.100
master_port:6379
master_link_status:up
```
- Kiểm tra trên Master
```
redis-cli INFO replication

output
role:master
connected_slaves:1
slave0:ip=10.10.10.101,port=6379,state=online,...
```
- Test đồng bộ dữ liệu
```
Trên master:
redis-cli SET replication_test "OK_20260924"

Trên slave:
edis-cli GET replication_tes
```

Chú ý: Khi config xong, Redis sẽ tự Full sync data từ Master sang Salve, bạn không cần dump/import thủ công
