# pgSQL的HA部署



[TOC]



## Ubuntu20.04

### 安装



```shell
# 安装etcd3.4
# 下载https://github.com/etcd-io/etcd/releases/download/v3.4.25/etcd-v3.4.25-linux-amd64.tar.gz
tar zxf etcd-v3.4.25-linux-amd64.tar.gz
cd etcd-v3.4.25-linux-amd64
sudo cp etcd etcdctl /usr/bin/
sudo chmod 755 /usr/bin/etcd /usr/bin/etcdctl


# 1和2上安装
# pgsql 12, patroni 4.1.3
sudo apt install -y  postgresql postgresql-client postgresql-contrib python3 python3-pip keepalived
sudo pip3 install patroni[all] python-etcd psycopg2-binary -U -i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn

# 仲裁节点安装
sudo apt install -y python3-pip keepalived
sudo pip3 install patroni[all] python-etcd psycopg2-binary -U -i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn


```



```shell
# 1和2上执行
sudo systemctl stop postgresql
sudo systemctl disable postgresql
sudo systemctl stop etcd
sudo systemctl disable etcd
sudo systemctl stop keepalived
sudo systemctl disable keepalived

# 仲裁节点执行
sudo systemctl stop etcd
sudo systemctl disable etcd
sudo systemctl stop keepalived
sudo systemctl disable keepalived
```



### etcd集群部署



```shell
# hadoop1
ETCD_NAME=etcd1
ETCD_DATA_DIR=/var/lib/etcd
ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380
ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379
ETCD_INITIAL_ADVERTISE_PEER_URLS=http://192.168.120.131:2380
ETCD_ADVERTISE_CLIENT_URLS=http://192.168.120.131:2379
ETCD_INITIAL_CLUSTER=etcd1=http://192.168.120.131:2380,etcd2=http://192.168.120.132:2380,etcd3=http://192.168.120.133:2380
ETCD_INITIAL_CLUSTER_STATE=new
ETCD_INITIAL_CLUSTER_TOKEN=pg-cluster

# hadoop2
ETCD_NAME=etcd2
ETCD_DATA_DIR=/var/lib/etcd
ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380
ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379
ETCD_INITIAL_ADVERTISE_PEER_URLS=http://192.168.120.132:2380
ETCD_ADVERTISE_CLIENT_URLS=http://192.168.120.132:2379
ETCD_INITIAL_CLUSTER=etcd1=http://192.168.120.131:2380,etcd2=http://192.168.120.132:2380,etcd3=http://192.168.120.133:2380
ETCD_INITIAL_CLUSTER_STATE=new
ETCD_INITIAL_CLUSTER_TOKEN=pg-cluster

# hadoop3
ETCD_NAME=etcd3
ETCD_DATA_DIR=/var/lib/etcd
ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380
ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379
ETCD_INITIAL_ADVERTISE_PEER_URLS=http://192.168.120.133:2380
ETCD_ADVERTISE_CLIENT_URLS=http://192.168.120.133:2379
ETCD_INITIAL_CLUSTER=etcd1=http://192.168.120.131:2380,etcd2=http://192.168.120.132:2380,etcd3=http://192.168.120.133:2380
ETCD_INITIAL_CLUSTER_STATE=new
ETCD_INITIAL_CLUSTER_TOKEN=pg-cluster

```



```shell
# 在三台设备中启动
systemctl enable --now etcd

# 查看etcd状态
etcdctl member list

# 正常的集群显示如下
3a37fae5a64a862c: name=etcd3 peerURLs=http://192.168.120.133:2380 clientURLs=http://192.168.120.133:2379 isLeader=false
63f1e781fb90a5b3: name=etcd2 peerURLs=http://192.168.120.132:2380 clientURLs=http://192.168.120.132:2379 isLeader=false
e90bd99ffe7a66ba: name=etcd1 peerURLs=http://192.168.120.131:2380 clientURLs=http://192.168.120.131:2379 isLeader=true
```



#### 节点1中配置 Patroni 

Patroni配置

```shell
sudo mkdir -p /etc/patroni
sudo vim /etc/patroni/patroni.yml
scope: pg-ha
name: hadoop1
namespace: /db/

restapi:
  listen: 0.0.0.0:8008
  connect_address: 192.168.120.131:8008

etcd3:
  hosts: 192.168.120.131:2379,192.168.120.132:2379,192.168.120.133:2379

bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
  initdb:
    - encoding: UTF8
    - data-checksums
  pg_hba:
    - host replication repl 192.168.120.0/24 md5
    - host all all 192.168.120.0/24 md5
  users:
    admin:
      password: admin123
      options:
        - createrole
        - createdb

postgresql:
  listen: 0.0.0.0:5432
  connect_address: 192.168.120.131:5432
  data_dir: /var/lib/postgresql/12/main
  config_dir: /etc/postgresql/12/main
  bin_dir: /usr/lib/postgresql/12/bin
  authentication:
    superuser:
      username: postgres
      password: Pg@123456
    replication:
      username: repl
      password: Pg@123456
  parameters:
    wal_level: replica
    max_wal_senders: 10
    wal_keep_size: 1GB
    hot_standby: "on"
    lc_messages: en_US.utf8
    
    
sudo sh -c 'cat >> /usr/lib/tmpfiles.d/postgresql.conf << EOF
d /var/run/postgresql/12-main.pg_stat_tmp 0700 postgres postgres -
EOF'
```





#### 节点2中配置 Patroni 

Patroni配置

```shell
sudo mkdir -p /etc/patroni
sudo vim /etc/patroni/patroni.yml
scope: pg-ha
name: hadoop2
namespace: /db/

restapi:
  listen: 0.0.0.0:8008
  connect_address: 192.168.120.132:8008

etcd3:
  hosts: 192.168.120.131:2379,192.168.120.132:2379,192.168.120.133:2379

postgresql:
  listen: 0.0.0.0:5432
  connect_address: 192.168.120.132:5432
  pg_hba:
    - host replication repl 192.168.120.131/32 md5
    - host all all 0.0.0.0/0 md5
  data_dir: /var/lib/postgresql/12/main
  config_dir: /etc/postgresql/12/main
  bin_dir: /usr/lib/postgresql/12/bin
  use_pg_rewind: true
  authentication:
    superuser:
      username: postgres
      password: Pg@123456
    replication:
      username: repl
      password: Pg@123456
  parameters:
    wal_level: replica
    max_wal_senders: 10
    wal_keep_size: 1GB
    hot_standby: "on"
    lc_messages: en_US.utf8
    
    
sudo sh -c 'cat >> /usr/lib/tmpfiles.d/postgresql.conf << EOF
d /var/run/postgresql/12-main.pg_stat_tmp 0700 postgres postgres -
EOF'

```







#### Patroni 配置服务启动

​	在hadoop1和hadoop2中都配置

```shell
sudo vim /etc/systemd/system/patroni.service

[Unit]
Description=Patroni
After=network.target

[Service]
User=postgres
Group=postgres
ExecStart=/usr/local/bin/patroni /etc/patroni/patroni.yml
Restart=always

[Install]
WantedBy=multi-user.target

# 启动服务
sudo systemctl daemon-reload
sudo systemctl enable --now patroni
sudo systemctl enable --now keepalived


# 验证
patronictl -c /etc/patroni/patroni.yml list
# 显示如下为正常
+ Cluster: pg-ha (7643451515583566297) -----------+----+-------------+-----+------------+-----+
| Member  | Host            | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
+---------+-----------------+---------+-----------+----+-------------+-----+------------+-----+
| hadoop1 | 192.168.120.131 | Replica | streaming | 10 |   0/90001F8 |   0 |  0/90001F8 |   0 |
| hadoop2 | 192.168.120.132 | Leader  | running   | 10 |             |     |            |     |
+---------+-----------------+---------+-----------+----+-------------+-----+------------+-----+

# 以下为修复的操作
journalctl -u patroni -f


sudo rm -rf /var/lib/postgresql/12/main/

sudo mkdir -p /var/lib/postgresql/12/main
sudo chown -R postgres:postgres /var/lib/postgresql/12
sudo chmod 700 /var/lib/postgresql/12/main


sudo systemctl stop patroni
sudo systemctl reset-failed patroni
sudo systemctl start patroni


sudo mkdir -p /var/lib/postgresql/12/main
sudo chown -R postgres:postgres /var/lib/postgresql/12
sudo chmod 700 /var/lib/postgresql/12/main

sudo -u postgres /usr/lib/postgresql/12/bin/initdb -D /var/lib/postgresql/12/main


```



### keepalived配置

#### hadoop1配置

```shell
# hadoop1
sudo vim /etc/keepalived/keepalived.conf

global_defs {
    router_id PG_HA_1
}

vrrp_script check_patroni {
    script "/etc/keepalived/check_patroni.sh"
    interval 2
    weight -20
    fall 2
    rise 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 51
    priority 110
    advert_int 1
    unicast_src_ip 192.168.120.131
    unicast_peer {
        192.168.120.132
    }
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.120.200/24
    }
    track_script {
        check_patroni
    }
}

sudo vim check_patroni.sh

#!/bin/bash
status=$(/usr/local/bin/patronictl -c /etc/patroni/patroni.yml list | grep $(hostname) | awk '{print $6}')
if [ "$status" = "Leader" ]; then
    exit 0
else
    exit 1
fi

sudo chmod +x /etc/keepalived/check_patroni.sh

sudo systemctl restart keepalived
sudo systemctl enable keepalived
```



#### hadoop2配置

```shell
# hadoop2
sudo vim /etc/keepalived/keepalived.conf

global_defs {
    router_id PG_HA_2
}

vrrp_script check_patroni {
    script "/etc/keepalived/check_patroni.sh"
    interval 2
    weight -20
    fall 2
    rise 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 51
    priority 100
    advert_int 1
    unicast_src_ip 192.168.120.132
    unicast_peer {
        192.168.120.131
    }
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.120.200/24
    }
    track_script {
        check_patroni
    }
}

sudo vim check_patroni.sh

#!/bin/bash
status=$(/usr/local/bin/patroni -c /etc/patroni/patroni.yml list | grep $(hostname) | awk '{print $6}')
if [ "$status" = "Leader" ]; then
    exit 0
else
    exit 1
fi

sudo chmod +x /etc/keepalived/check_patroni.sh

sudo systemctl restart keepalived
sudo systemctl enable keepalived

```

#### keepalive验证



```shell

# 查看vip在哪个设备上生效
ip addr show ens33
systemctl status keepalived.service

# 检查是否和主库在同一个设备上
```



## postgres基本操作







```shell
# 进入postgres
psql -h 127.0.0.1 -U postgres -d postgres

# 查看所有数据库
\l
# 连接/切换数据库（例如切换到 testdb）
\c testdb
# 查看当前库下的所有表
\dt
# 查看表结构（例如查看 user 表）
\d user
# 查看所有用户（角色）
\du
# 退出 psql
\q
# 清空屏幕
\clear
# 查看命令帮助
\?



# 创建一张测试表
CREATE TABLE test(id int, name text);
# 插入一条数据
INSERT INTO test VALUES (1, '测试数据');
# 显示表中所有数据
SELECT * FROM test;
```



#### 测试

```sql
-- 访问数据库
psql -h 192.168.120.200 -U postgres
-- 进入测试数据库
\c testdb
-- 建表，写数据
CREATE TABLE device_data (
    id          SERIAL PRIMARY KEY,                -- 自增主键ID
    create_time TIMESTAMPTZ NOT NULL DEFAULT NOW(),-- 带时区时间戳，自动当前时间
    temperature NUMERIC(12, 4) NOT NULL,           -- 温度，总共12位，小数点后4位
    voltage     NUMERIC(12, 4) NOT NULL,           -- 电压，小数点后4位
    current     NUMERIC(12, 4) NOT NULL            -- 电流，小数点后4位
);

INSERT INTO device_data (temperature, voltage, current)
VALUES (24.1234, 221.5678, 2.1020);

SELECT * FROM device_data;

INSERT INTO device_data (temperature, voltage, current)
SELECT
    random() * 40 + 10,     -- 温度 10~50 度，随机小数
    random() * 10 + 210,    -- 电压 210~220V
    random() * 5 + 0.1      -- 电流 0.1~5.1A
FROM generate_series(1, 1000000);  -- 100万行

SELECT count(*) FROM device_data;
SELECT *  FROM device_data  ORDER BY id DESC  LIMIT 10;

-- 给表增加一列 temperature2，数据类型和 temperature 一致
ALTER TABLE device_data ADD COLUMN temperature2 NUMERIC(12,4);
INSERT INTO device_data (temperature, voltage, current, temperature2)
VALUES (25.1234, 220.5678, 1.1020, 26.4321);

-- 给表增加一列 temperature3，数据类型和 temperature 一致，并给默认值
ALTER TABLE device_data ADD COLUMN temperature3 NUMERIC(12,4) DEFAULT 0.0000;

-- 修改列名
ALTER TABLE device_data RENAME COLUMN temperature2 TO temperature_in;

-- 删除 temperature 这一列
ALTER TABLE device_data DROP COLUMN temperature;


INSERT INTO device_data (voltage, current, temperature_in, temperature3)
SELECT
  220 + (random() * 5),        -- 电压 220左右
  1.5 + (random() * 2),        -- 电流 1.5~3.5A
  25 + (random() * 10),        -- 温度1
  25 + (random() * 10)         -- 温度3
FROM generate_series(1, 2000000); -- 正好 2000000 行


INSERT INTO device_data (voltage, current, temperature_in, temperature_in2, temperature4)
SELECT
  220 + (random() * 5),        -- 电压 220左右
  1.5 + (random() * 2),        -- 电流 1.5~3.5A
  25 + (random() * 10),        -- 温度1
  25 + (random() * 10)         -- 温度3
  25 + (random() * 10)         -- 温度3
FROM generate_series(1, 10); -- 正好 2000000 行



```



