---
title: influxdb 基本操作
tags: influxdb time-series database
---

[InfluxDB](https://www.influxdata.com/) is a time-series database for storing events and statistics.

# 安装
## 配置安装源

```
cat <<EOF | sudo tee /etc/yum.repos.d/influxdb.repo
[influxdb]
name = InfluxDB Repository - RHEL \$releasever
baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
EOF
```
## 安装
```
dnf install influxdb
systemctl enable influxdb
systemctl start influxdb

firewall-cmd --permanent --add-port=8086/tcp
firewall-cmd --reload
```
## 数据库登录 创建admin账号

```
influx --precision rfc3339

CREATE USER "admin" WITH PASSWORD "password" WITH ALL PRIVILEGES
```

## 配置文件修改 启用身份验证

```
vim /etc/influxdb/influxdb.conf
[http]
auth-enabled = true
```
## 重启

```
systemctl restart influxdb
```
## 重新连接数据库

```
influx --precision rfc3339 -username username -password password
```

## 创建数据库

```
CREATE DABATASE "DATABASE1"
CREATE DABATASE "DATABASE2"
```
## 提权

```
GRANT READ ON "DATABASE1" TO "username"
GRANT READ ON "DATABASE2" TO "username"
```

[更多操作](https://docs.influxdata.com/influxdb/)

# 数据库操作
- CREATE DATABASE
- DROP DATABASE
- DROP SERIES
- DELETE
- DROP MEASUREMENT
- DROP SHARD

## CREATE DATABASE
```
CAEATE DATABASE <database_name> [WITH [DURATION <duration>] [REPLICATION <n>] [SHARD DURATION <duration>] [NAME <retention-policy-name>]]
```
example
```
CREATE DATABASE "moon"
```
## Delete a database with DROP DATABASE
The **DROP DATABASE** query deletes all of the data, measurements, series, continuous queries, and retention policies from the specified database.
```
DROP DATABASE <database_name>
```
## Drop series from the index with DROP SERIES
The **DROP SERIES** query deletes all points from a **series** in a database, and it drops the series from the index.
```
DROP SERIES FROM <measurement_name[,measurement_name]> WHERE <tag_key>='<tag_value>'
```
drop all series from a single measurement:
```
DROP SERIES FROM "snmpd"
```
Drop series with a specific tag pair from a single measurement:
```
DROP SERIES FROM "snmpd" WHERE "location" = 'santa_monica'
```
Drop all points in the series that have a specific tag pair from all measurements in the database:
```
DROP SERIES WHERE "location" = 'santa_monica'
```
## Delete series with DELETE
The **DELETE** query deletes all points from a series in a database. Unlike **DROP SERIES**, it does not drop the series from the index and it supports time intervals in the **WHERE** clause.
```
DELETE FROM <measurement_name> WHERE [<tag_key>='<tag_value>'] | [<time interval>]
```
Delete all data associated with the measurement **h2o_feet**:
```
DELETE FROM "h2o_feet"
```
Delete all data associated with the measurement **h2o)quality** and where the tag **randtag** equals 3:
```
DELETE FROM "h2o_quality" WHERE "randtag" = '3'
```
## Delete measurements with DROP MEASUREMENT
The **DROP MEASUREMENT** query deletes all data and series from the specified **measurement** and deletes the measurement from the index.
```
DROP MEASUREMENT <measurement_name>
```

示例：
```
# influx --precision rfc3399 -username username -password password
Connected to http://localhost:8086 version x.x.x
InfluxDB shell version: x.x.x
> show databases
> use database_name
> show measurements
> show series
> select * from measurement_name limit 10
> drop measurement measurement_name
> drop series from "measurement_name" where 'tag_key'='tag_value'
```