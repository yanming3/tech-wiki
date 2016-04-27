# Mysql安装配置
1. 下载

	```shell
	wget 
	```
2. 安装

	```shell
	tar -xvf MySQL-5.6.29-1.el7.x86_64.rpm-bundle.tar
	rpm -ivh MySQL-devel-5.6.29-1.el7.x86_64.rpm
	rpm -ivh MySQL-server-5.6.29-1.el7.x86_64.rpm
	rpm -ivh MySQL-client-5.6.29-1.el7.x86_64.rpm
	```
3. 配置
 创建```/etc/my.cnf```文件，内容如下：
 
	 ```shell
	[client]
	port = 3306
	socket = /tmp/mysql.sock
	default-character-set = utf8mb4
	
	[mysql]
	default-character-set = utf8mb4
	
	[mysqld]
	
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
	
4. Tips
 	  数据库文件修复
 	  
	``` shell
	 mysql_install_db --defaults-file=/etc/my.cnf  --user=root --basedir=/usr --datadir=/data/mysql
	```
 
   修改密码
   
	 ``` shell
   /usr/bin/mysqladmin -u root password new-password
   /usr/bin/mysqladmin -u root -h localhost password new-password
   /usr/bin/mysql_secure_installation
	 ```
	 
   创建数据库
   
   ``` shell
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
    sync_binlog=1 
    innodb_flush_log_at_trx_commit=1
    
    [mysqld]
    server-id=2
    log-bin=mysql-bin
    binlog_format = "MIXED"
    replicate-do-db=disc_user_center
    replicate-do-db=ods_push_info
    replicate-ignore-db=mysql
    relay-log=mysqld-relay-bin
	

2.创建复制用户
		
	CREATE USER 'repl'@'%.mydomain.com' IDENTIFIED BY 'slavepass';
	GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%.mydomain.com';

3.常用命令
    
     SHOW SLAVE HOSTS;
     SHOW MASTER STATUS;
     SHOW SLAVE STATUS;
     STOP SLAVE;
	   START SLAVE;  
	      
4.数据同步

    * Master只读
    FLUSH TABLES WITH READ LOCK;
    SET GLOBAL read_only = ON;
    
    * 导出数据
    mysqldump --all-databases --master-data > dbdump.db
    或
    mysqldump --databases disc_user_center ods_push_info >dbdump.db
    mysqladmin shutdown
    tar cf /tmp/db.tar ./data
    
    * 导入数据
    mysql < fulldb.dump
    tar xvf dbdump.tar）；
    
    *解锁Master
     SET GLOBAL read_only = OFF;
	  UNLOCK TABLES;

    ＊ 启用主从复制 
    mysqladmin start-slave
    change master to master_host='10.36.40.41', master_user='repl', master_password='Creditease4152',master_log_pos=1029,master_log_file='mysql-bin.000009';
    show master status可以查询到master_log_file和master_log_pos参数；
   
   

   
   


