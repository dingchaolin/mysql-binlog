# mysql-binlog

## 连接mysql
- mysql -h 127.0.0.1 -uroot -proot

## 查看binlog是否开启
- show variables like 'log_bin';

## 查看 my.cnf文件路径
- mysql --help --verbose | grep my.cnf

## 系统的目录
- /usr/local/etc

## 配置文件信息
```
# Default Homebrew MySQL server config
[mysqld]
# Only allow connections from localhost
bind-address = 127.0.0.1
# log_bin
log-bin = mysql-bin #开启binlog
binlog-format = ROW #选择row模式
server_id = 1 #配置mysql replication需要定义，不能喝canal的slaveId重复
```

## 重启
- brew services restart mysql

## 查看binlog文件
- show binary logs;

## 查看所有的binlog日志
- show binlog events;   

## 查看某个文件的日志
- show binlog events in 'mysql-bin.000002';

# binlog接收流程
- 1. client <-> DB 握手协议。
- 2. client发送show master status，此命令中带回当前binlog存储在哪个文件
以及罪行的偏移量
- 3. 发送show global variables like 'binlog_checksum'命令，这是由于binlog
在发送回来的时候需要，在最后获取event内容的时候，会增加4个额外字节做校验用。
mysql5.6.5以后的版本中binlog_checksum=crc32，而低版本都是binlog_checksum=none,
如果不想校验，可以使用set命令设置set binlog_checksum=none.
- 4. 最终到发送dump命令的阶段。

# mysql-binlog-dump 命令

### 结构
- packetLength 3bytes
- sequenceid 防串包
- command=18，binlog_dump
- binlogPosition, 4bytes
- 0 1byte
- 0 1byte
- binlogFileName  binlogFileName.length bytes

# mysql-binlog-event
- 一旦发送了binlogDump命令，master就会在数据库有变化的源源不断的推送binlog event
到client

# mysql-binlog-event 类型
- Statement:每一条会修改数据库的sql都会记录在binlog中
- Row:不记录sql语句的上下文相关信息，仅保存哪条记录被修改
- Mixedlevel: 以上两种level的混合

# select @@GLOBAL.binlog_checksum as checksum
- 查看校验加密方式

# set @master_binlog_checksum=@@global.binlog_checksum
- 做了这个设置之后，从库与主库就能通信了

# SHOW BINARY LOGS
- 查看binlig日志的所有存放文件
```
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |       177 |
| mysql-bin.000002 |       177 |
| mysql-bin.000003 |       504 |
| mysql-bin.000004 |       154 |
| mysql-bin.000005 |       154 |
+------------------+-----------+
```

# SELECT COLUMN_NAME, COLLATION_NAME, CHARACTER_SET_NAME, COLUMN_COMMENT, COLUMN_TYPE FROM columns WHERE table_schema='doc' AND table_name='test'
- table_schema 数据库名
- table_name 表名


# ddl
- ALTER TABLE `doc`.`test` CHANGE COLUMN `name` `name1` varchar(255) DEFAULT NULL

# find
- SELECT COLUMN_NAME, COLLATION_NAME, CHARACTER_SET_NAME, COLUMN_COMMENT, COLUMN_TYPE FROM columns WHERE table_schema='zentao' AND table_name='zt_cron'









