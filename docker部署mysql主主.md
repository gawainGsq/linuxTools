### **docker部署mysql主主**

```
docker pull mysql:5.7.37
mkdir -pv /data/mysql/{master_1/{data,conf},master_2/{data,conf},master_3/{data,conf}}
touch /data/mysql/master_1/conf/my.cnf
touch /data/mysql/master_2/conf/my.cnf
touch /data/mysql/master_3/conf/my.cnf
tree mysql

#MASTER主机部署两台一主一从

docker run \
--name mysql-master1 \
-v /data/mysql/master_1/data:/var/lib/mysql \
-v /data/mysql/master_1/conf:/etc/mysql \
-p 3307:3306 \
-e MYSQL_ROOT_PASSWORD='123$abc' \
-e TZ=Asia/Shanghai \
-d mysql:5.7.37 \
--character-set-server=utf8mb4 \
--collation-server=utf8mb4_unicode_ci

docker run \
--name mysql-master2 \
-v /data/mysql/master_2/data:/var/lib/mysql \
-v /data/mysql/master_2/conf:/etc/mysql \
-p 3307:3306 \
-e MYSQL_ROOT_PASSWORD='123$abc' \
-e TZ=Asia/Shanghai \
-d mysql:5.7.37 \
--character-set-server=utf8mb4 \
--collation-server=utf8mb4_unicode_ci

#修改master1 my.cnf
[mysqld]
log_bin=mysql-bin
server-id=1
sync-binlog=1
relay_log=mysql-relay-bin
binlog-ignore-db=performance_schema

#修改master2 my.cnf
[mysqld]
log_bin=mysql-bin
server-id=2
sync-binlog=1
relay_log=mysql-relay-bin
binlog-ignore-db=performance_schema

#重启mysql 容器

#接下来就是配置容器内的mysql了，原理就是创建一个复制账户供从节点从主节点复制获取log_bin到本地进行刷盘复制，复杂原理就不提了，logbin->relaylog->磁盘的过程自行查阅吧

#Master主机 192.168.32.130:3306
grant replication slave on *.* to "slaver"@"%" identified by "123456";
grant all privileges on *.* to "slaver"@"%" identified by "123456";
flush privileges;

#MasterTwo主机 192.168.32.133:3306
grant replication slave on *.* to "slaver"@"%" identified by "123456"; #创建复制账户
grant all privileges on *.* to "slaver"@"%" identified by "123456"  #赋值权限
flush privileges  #刷新权限

#Slave主机 192.168.32.130:3307
change master to master_host="10.10.5.41",master_port=3307,master_user="slaver",master_password="123456",master_log_file="mysql-bin.000001",master_log_pos=154; 
change master to master_host="10.10.5.31",master_port=3307,master_user="slaver",master_password="123456",master_log_file="mysql-bin.000001",master_log_pos=154; 

start slave;

#到这双主一从的环境应该已经运行成功了，自己去测试一下看看效果就可以了
```

