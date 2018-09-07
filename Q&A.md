# 命令集合
## 删除pool
1.找出要删除的pool名称 ```ceph osd pool ls ``` \
2.关闭mds ```systemctl  stop cep-mds@mds1``` \
3.无效化 ```ceph mds fail 0``` \
4.```ceph fs rm cephfs --yes-i-really-mean-it``` \
5.```ceph osd pool delete metadata metadata --yes-i-really-really-mean-it``` \
## cephfs挂载
```建议使用ceph-fuse方式来挂载，操作系统自带的fuse版本会比较低，稳定性会差点```\
1.内核挂载 ```mount -t ceph 192.168.159.131:6789:/  /mnt/cephfs -o name=admin,secret=AQDuGjFabSMHAxAAEbYLDjpa3EQUaSGB/EtkXg== ```\
2.ceph-fuse 挂载（本地需要有 /etc/ceph/ceph.client.admin.keyring ） ```ceph-fuse -m  xxxx,yyyy,yyy:6789 /mnt/cephfs```  \

## 清理
 ```
  ps aux|grep ceph |awk '{print $2}'|xargs kill -9
  ps -ef|grep ceph
 #确保此时所有ceph进程都已经关闭！！！如果没有关闭，多执行几次。
 umount /var/lib/ceph/osd/*
 rm -rf /var/lib/ceph/osd/*
 rm -rf /var/lib/ceph/mon/*
 rm -rf /var/lib/ceph/mds/*
 rm -rf /var/lib/ceph/bootstrap-mds/*
 rm -rf /var/lib/ceph/bootstrap-osd/*
 rm -rf /var/lib/ceph/bootstrap-rgw/*
 rm -rf /var/lib/ceph/tmp/*
 rm -rf /etc/ceph/*
 rm -rf /var/run/ceph/* 
```

## 创建mon失败
```
sudo -u ceph ceph-mon --mkfs -i mon1 --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring  
/var/lib/ceph/mon/ceph-mon1' already exists and is not empty: monitor may already exist
解决：
rm -fr /var/lib/ceph/mon/ceph-node1/
 ```
 ##常用命令
 * 1.查看osd 属于哪台机器 <br>
  ```ceph osd find 0```
  * 删除osd
   ceph osd out osd.2
  ceph osd crush remove osd.2
   ceph auth del osd.2
   ceph osd rm 2
  systemctl stop ceph-osd@0
  
  ceph osd create
  ceph osd crush add osd.2 1.0 host=mon1
  
  
  ceph osd pool get rbd size
ceph osd pool set <poolname> size 2
ceph osd pool set <poolname> min_size 1
ceph osd pool set <poolname> max_size 10
 
 Sep  6 21:30:56 ceph-create-keys: admin_socket: exception getting command descriptions: [Errno 2] No such file or directory
Sep  6 21:30:56 ceph-create-keys: INFO:ceph-create-keys:ceph-mon admin socket not ready yet.

关闭selinux&firewalld

*sed -i 's/SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
setenforce 0
systemctl stop firewalld 
systemctl disable firewalld


 ### 查看某个文件落在哪个PG和OSD
 ceph osd  map  rbd 1.txt
