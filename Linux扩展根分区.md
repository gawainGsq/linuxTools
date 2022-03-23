### Linux扩展根分区

```
#默认已有centos这个lvm，先添加硬盘，分区
fdisk /dev/sdb 
pvcreate /dev/sdb1
partprobe /dev/sdb
vgextend centos  /dev/sdb1 
lvextend -l +100%FREE /dev/mapper/centos-root
xfs_growfs /dev/mapper/centos-root
```



```
#减少根目录101G到vg里
lvreduce -L -101G  /dev/mapper/centos-root
```



```
#扩展根目录
umount /home/
vgs
lvremove /dev/mapper/centos-home
lvextend -l +100%FREE /dev/mapper/centos-root
xfs_growfs /dev/mapper/centos-root
df -h
vi /etc/fstab 
#删除home
mount -a
```



```
#公有云添加硬盘挂载目录
fdisk /dev/vdb
mkfs.ext4 /dev/vdb1 
mount /dev/vdb1 /opt/
mkdir /var/lib/docker -p
partprobe /dev/vdb
mkfs.ext4 /dev/vdb2 
mount /dev/vdb2 /var/lib/docker/
mount -a #测试
vim /etc/fstab
/dev/vdb1 /opt ext4 defaults 0 0
/dev/vdb2 /var/lib/docker ext4 defaults 0 0
```

