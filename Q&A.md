# 命令集合
## 删除pool
1.找出要删除的pool名称 ```ceph osd pool ls ``` \
2.关闭mds ```systemctl  stop cep-mds@mds1``` \
3.无效化 ```ceph mds fail 0``` \
4.```ceph fs rm cephfs --yes-i-really-mean-it``` \
5.```ceph osd pool delete metadata metadata --yes-i-really-really-mean-it``` \
## cephfs挂载

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
 ## 常用命令
  * 1.查看osd 属于哪台机器 <br>
  ```ceph osd find 0```
  * 2.删除osd
  ```
  ceph osd out osd.2
  ceph osd crush remove osd.2
  ceph auth del osd.2
  ceph osd rm 2
  systemctl stop ceph-osd@0
  ```
  
  * 3.设置副本数
 ```
  ceph osd pool get rbd size
  ceph osd pool set <poolname> size 2
  ceph osd pool set <poolname> min_size 1
  ceph osd pool set <poolname> max_size 10
  ```
 * 4.查看某个文件落在哪个PG和OSD
 ```
 ceph osd  map  rbd 1.txt
 osdmap e33 pool 'rbd' (0) object '2.txt' -> pg 0.5a885de9 (0.29) -> up ([3,1], p3) acting ([3,1], p3)
 
 ```
 ## 遇到的问题
 ### 创建mon失败
```
 sudo -u ceph ceph-mon --mkfs -i mon1 --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring  
 /var/lib/ceph/mon/ceph-mon1' already exists and is not empty: monitor may already exist
  解决：
 rm -fr /var/lib/ceph/mon/ceph-node1/
 ```
 ### 启动mon报错
 ```
 Sep  6 21:30:56 ceph-create-keys: admin_socket: exception getting command descriptions: [Errno 2] No such file or directory
 Sep  6 21:30:56 ceph-create-keys: INFO:ceph-create-keys:ceph-mon admin socket not ready yet.
```
 * 解决办法：关闭selinux&firewalld
 ```
 sed -i 's/SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
 setenforce 0
 systemctl stop firewalld 
 systemctl disable firewalld
 ```
 ### 磁盘故障恢复方法
 当一个 OSD 被 out 后，部分 PG 限于 active+remapped 状态是经常出现的。解决办法是先运行 ceph osd in {osd-num} 将集群状态恢复到初始状态，然后运行  ceph osd crush reweight osd.{osd-num} 0 来将这个 osd 的 crush weight 修改为 0，然后集群会开始数据迁移。对小集群来说，reweight 命令甚至更好些。
等集群迁移完毕后，在将这个osd删除掉

 ### 挂载cephfs
  ```
  mount -vw -t ceph mon1:6789:/ /mnt/cephfs -o name=admin,secret=AQBampNbrmnXBxAAYXtTNP0u7qzh1JpFvY887g==
  parsing options: rw,name=admin,secret=AQBampNbrmnXBxAAYXtTNP0u7qzh1JpFvY887g==
  mount error 5 = Input/output error
  ```
  * 原因是系统内核版本较低，某些特性不支持，http://cephnotes.ksperis.com/blog/2014/01/21/feature-set-mismatch-error-on-ceph-kernel-client/
  ```
  ceph osd crush show
  {
    "choose_local_tries": 0,
    "choose_local_fallback_tries": 0,
    "choose_total_tries": 50,
    "chooseleaf_descend_once": 1,
    "chooseleaf_vary_r": 0,
    "chooseleaf_stable": 0,
    "straw_calc_version": 1,
    "allowed_bucket_algs": 54,
    "profile": "unknown",
    "optimal_tunables": 0,
    "legacy_tunables": 0,
    "minimum_required_version": "bobtail",
    "require_feature_tunables": 1,
    "require_feature_tunables2": 1,
    "has_v2_rules": 0,
    "require_feature_tunables3": 0,
    "has_v3_rules": 0,
    "has_v4_buckets": 0,
    "require_feature_tunables5": 0,
    "has_v5_rules": 0
}
  ```
  * 解决办法
 `` ceph osd crush tunables hammer ```（暴力方式，关闭所有特性）
 关闭 chooseleaf_vary_r  chooseleaf_stable 特性
 ```
 ceph osd getcrushmap -o /tmp/crush
 crushtool -i /tmp/crush --set-chooseleaf-vary-r 0 --set-chooseleaf-stable 0  -o /tmp/crush.new
 ceph osd setcrushmap -i /tmp/crush.new
 ```


## 网络带宽测试方法
 ```
 nc -v -l -n 17480 > /dev/null
 time dd if=/dev/zero | nc -v  221.230.143.150 17480
```
随机读性能
fio -filename=//mnt/cephfs/dlw1  -direct=1 -iodepth 1 -thread -rw=randread -ioengine=psync -bs=16k -size=2G -numjobs=10 -runtime=60 -group_reporting -name=mytest
