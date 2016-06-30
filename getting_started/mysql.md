# Mysql安装配置
1. 下载

	```bash
	wget 
	```
2. 安装

	```bash
	tar -xvf MySQL-5.6.29-1.el7.x86_64.rpm-bundle.tar
	rpm -ivh MySQL-devel-5.6.29-1.el7.x86_64.rpm
	rpm -ivh MySQL-server-5.6.29-1.el7.x86_64.rpm
	rpm -ivh MySQL-client-5.6.29-1.el7.x86_64.rpm
	```
3. 配置
 创建```/etc/my.cnf```文件，内容如下：
 
	 ```bash
	[client]
	port = 3306
	socket = /tmp/mysql.sock
	
	[mysql]
	default-character-set = utf8mb4
	
	[mysqld]
	gtid-mode=on
	enforce-gtid-consistency=true
	log_slave_updates=1 #上面3句配置启用了GTIDS,master和slave都要进行配置
	
	rpl_semi_sync_master_enabled=1
	rpl_semi_sync_master_timeout=1000 #启用半同步复制
	# Remove leading # and set to the amount of RAM for the most important data
	# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
	# innodb_buffer_pool_size = 128M
	
	# Remove leading # to turn on a very important data integrity option: logging
	# changes to the binary log between backups.
	 log_bin=mysql-bin
	 #binlog_format = "STATEMENT"
	 #binlog_format = "ROW"
	 binlog_format = "MIXED"
	 binlog-do-db=fansz
	# These are commonly set, remove the # and set as required.
	 basedir = /usr
	 datadir = /home/mysql/data
	 pid-file = /home/mysql/data/mysql.pid
	 user = root
	 bind-address = 0.0.0.0
	 port = 3306
	 server_id = 1
	 socket = /tmp/mysql.sock
	
	 max_connections=1000
	 character-set-client-handshake = FALSE
	 character-set-server = utf8mb4
	 collation_server=utf8mb4_unicode_ci
	 init_connect = 'SET NAMES utf8mb4'
	# Remove leading # to set options mainly useful for reporting servers.
	# The server defaults are faster for transactions and fast SELECTs.
	# Adjust sizes as needed, experiment to find the optimal values.
	# join_buffer_size = 128M
	# sort_buffer_size = 2M
	# read_rnd_buffer_size = 2M
	
	sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
	 ```
	 
	 4. 启用半同步复制
	master:
	
	  ```sql
    install plugin rpl_semi_sync_master SONAME 'semisync_master.so';
    SET GLOBAL rpl_semi_sync_master_enabled=1; 
    SET GLOBAL rpl_semi_sync_master_timeout=1000; 
   ```
slave:

     ```sql
     install plugin rpl_semi_sync_slave SONAME 'semisync_slave.so';
     set global rpl_semi_sync_slave_enabled=1;
     STOP SLAVE; 
     CHANGE MASTER TO master_heartbeat_period= 1000; 
     START SLAVE;
   ```
5. 数据库文件修复
 	  
 	 ``` bash
	 mysql_install_db --defaults-file=/etc/my.cnf  --user=root --basedir=/usr --datadir=/data/mysql
	 ```
 
   修改密码
   
	 ``` bash
   /usr/bin/mysqladmin -u root password new-password
   /usr/bin/mysqladmin -u root -h localhost password new-password
   /usr/bin/mysql_secure_installation
	 ```
	 
   创建数据库
   
   ``` bash
   create database fansz;
   grant all privileges on fansz.* to fansz_dev@"%" identified by 'fansz_dev';
   flush privileges;
   ``` 

# 主从复制

1.配置文件

    [mysqld]
    server-id=1
    log-bin=mysql-bin
    binlog_format = "MIXED"
    binlog-do-db=disc_user_center
    binlog-do-db=ods_push_info
    binlog-ignore-db=mysql 
    log_slave_updates=1
    sync_binlog=1 
    innodb_flush_log_at_trx_commit=1
    
    [mysqld]
    server-id=2
    log-bin=mysql-bin
    binlog_format = "MIXED"
    #replicate-do-db=disc_user_center
    #replicate-do-db=ods_push_info
    replicate-wild-do-table=disc_user_center.info_cfg
 	 replicate-wild-do-table=ods_push_info.%
    replicate-ignore-db=mysql
    relay-log=mysqld-relay-bin
	

2.创建复制用户
		
	CREATE USER 'repl'@'%.mydomain.com' IDENTIFIED BY 'slavepass';
	GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%.mydomain.com';

3.常用命令
    
     SHOW SLAVE HOSTS;
     SHOW MASTER STATUS;
     SHOW SLAVE STATUS\G;
     STOP SLAVE;
	   START SLAVE;  
	      
4.数据同步

    * Master只读
    FLUSH TABLES WITH READ LOCK;
    SET GLOBAL read_only = ON;
    
    * 导出数据
    mysqldump --all-databases --master-data=1 > dbdump.db
    或
    mysqldump --databases disc_user_center ods_push_info >dbdump.db
    mysqladmin shutdown
    tar cf /tmp/db.tar ./data
    --master-data=1不用再执行change master语句
    * 导入数据
    mysql < fulldb.dump
    tar xvf dbdump.tar）；
    
    *解锁Master
     SET GLOBAL read_only = OFF;
	  UNLOCK TABLES;

    ＊ 启用主从复制 
    mysqladmin start-slave
    change master to master_host='10.36.40.26', master_user='f_test', master_password='f_test_2015',master_log_pos=195,master_log_file='mysql-bin.000005';
    show master status可以查询到master_log_file和master_log_pos参数；
   
    binlog转换:  
    mysqlbinlog mysqld-relay-bin.000004 --base64-output=decode-rows -v > decoded.log
 

   
   


