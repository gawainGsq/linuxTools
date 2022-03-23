```
#控制台安装
virt-install --name=vm_1 --vcpus=2 --ram=4096  --disk path=/home/kvm/images/kylinv10.img,format=qcow2,size=120,bus=virtio --cdrom /home/kvm/images/Kylin-Server-10-SP1-Release-Build04-20200711-arm64.iso --network bridge=br1,model=virtio --force  --autostart

#克隆
virt-clone -o kylinv10temp -n testdocker -f /opt/kvm/images/testdocker.img 
#控制台
virsh console --domain testdocker
#列出虚拟机
virsh list --all
#重启虚拟机
virsh shutdown --domain node25 
virsh start --domain node25 
#编辑配置文件，可修改cpu内存大小，修改完重启配置
virsh edit master24

virsh create /etc/libvirt/qemu/master24.xml 
#查看虚拟机信息
virsh dominfo  --domain master24 
#查看网桥，是否关联网口，无关联可以添加关联
brctl show
brctl addif brx vnety 

#临时添加磁盘。创建卷 kylinv10.img  raw和qcow2（精简）格式
 virsh vol-create-as --pool storage_pool --name testdisk.img --capacity 20G --allocation 1G --format raw
 virsh vol-info /opt/kvm/images/testdisk.img
 virsh attach-disk testdocker /opt/kvm/images/testdisk.img vdb
#解除磁盘 
 virsh detach-disk testdocker /opt/kvm/images/testdisk.img

  #永久添加磁盘
  #virsh shutdown docker
  #virsh edit docker

​    <disk type='file' device='disk'>
​      <driver name='qemu' type='raw' cache='none'/>
​      <source file='/opt/kvm/images/testdisk.img'/>
​      <target dev='vdb' bus='virtio'/>
​    </disk>
​    
  #virsh create /etc/libvirt/qemu/docker.xml           
  umount /tmp
  mount -t tmpfs -o size=1500M /tmp
#快照管理        
1.创建自定义名称快照：
virsh snapshot-create-as 虚拟机名称 快照名称
2.查看虚拟机相关的快照
virsh snapshot-list 虚拟机名
3.删除快照
virsh snapshot-delete 虚拟机名称 快照名称
4.启动快照
virsh snapshot-revert 虚拟机名称 快照名称
#外部快照
virsh shutdown vm_kylinv10_4C8G
virsh snapshot-create-as vm_kylinv10_4C8G snap1-raw "snap1 description" --disk-only --atomic
cd /home/kvm/images/
qemu-img info kylinv10_4C8G.snap1-raw
virsh snapshot-list vm_kylinv10_4C8G
virsh domblklist vm_kylinv10_4C8G --details
virsh snapshot-info vm_kylinv10_4C8G  --current
virsh edit  vm_kylinv10_4C8G

 #删除虚拟机
 virsh list --all
 virsh destroy vm_docker
 virsh undefine vm_docker
 virsh undefine vm_docker --nvram
        
```



```
#克隆虚拟机
virt-clone -o kylin10_8c16g_temp -n new_vm1 -f /opt/kvm/images/new_vm1.img 
#删除虚拟机
virsh  undefine kylin10_8c16g_temp --nvra
```



**docker与kvm冲突，导致虚拟机ping不通网关**

```
#停止docker service
systemctl stop docker
#卸载docker
yum remove -y docker 
#关闭docker0网卡(如果没有ifconfig，则需要先安装(yum install net-tools))
ifconfig docker0 down 
#删除docker0网卡(如果没有brctl，则需要先安装(yum install bridge-utils))
brctl delbr docker0

#清除防火墙规则，慎重
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT
iptables -t nat -F
iptables -t mangle -F
iptables -F
iptables -X
```

​              