# 版本jewel,没有采用ceph-deploy 部署，操作系统centos7.2
## 前置步骤
### 1.安装包部署 
### 2.防火墙关闭，时钟同步 
### 3.创建一个ceph虚拟用户 
  ```echo "ceph:x:167:167:Ceph daemons:/var/lib/ceph:/sbin/nologin" >> /etc/passwd```<br>
  ```echo "ceph:x:167:" >>  /etc/group```
### 4.机器规划，多少台mon,osd, mds 
### 5. /etc/hosts 添加以下mon信息
  ```xx.xxx.143.xxx mon1```<br>
  ```xxx.230.xxx.xxx mon2```<br>
  ```xxx.230.xxx.xxx mon3```<br>
### 5. 生成一个uuid <br>
  ```uuidgen``` <br>
  4834e3bd-3b59-4536-9465-36f2bd13f68a <br>
### 6.ceph.conf 
```
[global]
fsid = 08068b52-f0ab-4a50-8dad-cc3512082031
mon_initial_members = mon1
mon_host =xxxxxx

auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
public_network = xxxx.0/24
mon_clock_drift_allowed = 2
osd journal size = 1024
#设置副本数
osd pool default size = 1
#设置最小副本数
osd pool default min size = 1
osd pool default pg num = 64
osd pool default pgp num = 64
osd crush chooseleaf type = 1
osd_mkfs_type = xfs
max mds = 5
mds max file size = 100000000000000
mds cache size = 1000000
#设置osd节点down后900s，把此osd节点逐出ceph集群，把之前映射到此节点的数据映射到其他节点。
mon osd down out interval = 900
```
 ## 部署MON节点(3台机器）
### 1.为监控节点创建管理密钥 <br>
  ```ceph-authtool --create-keyring /tmp/ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *'``` <br>
  creating /tmp/ceph.mon.keyring <br>
### 2.为ceph amin用户创建管理集群的密钥并赋予访问权限\
  ```sudo ceph-authtool --create-keyring /etc/ceph/ceph.client.admin.keyring --gen-key -n client.admin --set-uid=0 --cap mon 'allow *' --cap osd 'allow *' --cap mds 'allow *' --cap mgr 'allow *'``` <br>
  creating /etc/ceph/ceph.client.admin.keyring <br>
### 3.生成一个引导-osd密钥环，生成一个client.bootstrap-osd用户并将用户添加到密钥环中 <br>
  ```sudo ceph-authtool --create-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring --gen-key -n client.bootstrap-osd --cap mon 'profile bootstrap-osd'```<br>
  creating /var/lib/ceph/bootstrap-osd/ceph.keyring <br>
### 4.将生成的密钥添加到ceph.mon.keyring <br> 
  ```sudo ceph-authtool /tmp/ceph.mon.keyring --import-keyring /etc/ceph/ceph.client.admin.keyring``` <br>
  importing contents of /etc/ceph/ceph.client.admin.keyring into /tmp/ceph.mon.keyring <br>
  ```sudo ceph-authtool /tmp/ceph.mon.keyring --import-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring``` <br>
  importing contents of /var/lib/ceph/bootstrap-osd/ceph.keyring into /tmp/ceph.mon.keyring <br><br>
### 5.使用主机名、主机IP地址(ES)和FSID生成monmap。把它保存成/tmp/monmap
  ```monmaptool --create --add mon1 192.168.1.10 --fsid bdfb36e0-23ed-4e2f-8bc6-b98d9fa9136c /tmp/monmap --clobber ``` <br>
	monmaptool: monmap file /tmp/monmap <br>
	monmaptool: set fsid to bdfb36e0-23ed-4e2f-8bc6-b98d9fa9136c <br>
    monmaptool: writing epoch 0 to /tmp/monmap (1 monitors) <br>
### 6.创建一个默认的数据目录 <br>
  ```sudo -u ceph mkdir /var/lib/ceph/mon/ceph-mon1``` <br>
### 7. 改ceph.mon.keyring属主和属组为ceph <br>
  ```chown ceph.ceph /tmp/ceph.mon.keyring``` <br>
### 8. 初始化mon
   ```sudo -u ceph ceph-mon --mkfs -i mon1 --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring``` <br>
   	ceph-mon: set fsid to bdfb36e0-23ed-4e2f-8bc6-b98d9fa9136c <br>
	ceph-mon: created monfs at /var/lib/ceph/mon/ceph-node1 for mon.node1 <br>
### 8.为了防止重新被安装创建一个空的done文件 <br>
   ```sudo touch /var/lib/ceph/mon/ceph-mon1/done``` <br>
### 9.启动mon
  ```systemctl start ceph-mon@mon1``` <br>
### 10.查看运行状态
  ```systemctl status ceph-mon@mon1``` <br>
### 11. 设置mon开机自动启动 <br>
  ```systemctl enable ceph-mon@mon1``` <br>
### 12.第一个算节点部署成功 
### 13. /etc/ceph/* 文件拷贝到所有机器 <br>
    /var/lib/ceph/bootstrap-osd/ceph.keyring 拷贝到所有机器（理论上拷贝到osd机器就可以） 
    /tmp/ceph.mon.keyring 拷贝到所有机器（理论上是mon机器就可以） 
    /var/lib/ceph/bootstrap-mds/ceph.keyring 拷贝到所有机器（理论上拷贝到mds机器就可以） 
    /etc/ceph/* 拷贝到所有机器
## 新增mon2,mon3节点 <br>
    *** ceph.conf 需要存在mon1 mon2 ***
### 1.在mon2上创建一个默认的数据目录
 ```sudo -u ceph mkdir /var/lib/ceph/mon/ceph-mon2``` <br>
### 2.在mon2上修改ceph.mon.keyring属主和属组为ceph
 ```chown ceph.ceph /tmp/ceph.mon.keyring``` <br>
### 3.获取密钥和monmap信息(从mon1机器拷贝过来的秘钥)
 ```ceph auth get mon. -o /tmp/ceph.mon.keyring```  <br>
exported keyring for mon.<br>
 ```ceph mon getmap -o /tmp/ceph.mon.map``` <br>
	got monmap epoch 1
### 4.初始化mon
 ```sudo -u ceph ceph-mon --mkfs -i mon2 --monmap /tmp/ceph.mon.map --keyring /tmp/ceph.mon.keyring``` <br>
	ceph-mon: set fsid to 8ca723b0-c350-4807-9c2a-ad6c442616aa <br>
	ceph-mon: created monfs at /var/lib/ceph/mon/ceph-mon2 for mon.mon2 <br>
### 5.为了防止重新被安装创建一个空的done文件
 ```sudo touch /var/lib/ceph/mon/ceph-mon2/done``` <br>
### 6.将新的mon节点添加至ceph集群的mon列表
```ceph mon add mon2 192.168.1.11:6789```<br>
	adding mon.mon2 at 192.168.1.11:6789/0<br>
### 7.启动mon
  ```systemctl start ceph-mon@mon2``` <br>
### 8.查看运行状态
  ```systemctl status ceph-mon@mon2``` <br>
### 9. 设置mon开机自动启动 <br>
  ```systemctl enable ceph-mon@mon2``` <br>
## mon3节点 <br>
### 流程参考mon2，部署完毕后，执行 ceph -s 命令后可以看到有3个mon启动起来
## 部署OSD
### 1.创建osd
  ```ceph osd create```<br>
   注：0位osd的ID号，默认情况下会自动递增<br>
### 2.准备磁盘
  ```ceph-disk prepare /dev/sdb1```<br>
  通过ceph-disk命令可以自动根据ceph.conf文件中的配置信息对磁盘进行分区()<br>
### 3.对第一个分区进行格式化
  ```mkfs.xfs -f /dev/sdb1```<br>
### 4.创建osd默认的数据目录
  ```mkdir -p /var/lib/ceph/osd/ceph-0```<br>
### 5.对分区进行挂载
  ```mount /dev/sdb1 /var/lib/ceph/osd/ceph-0/ ```<br>
### 6.添加自动挂载信息,开启自动挂载
  ```echo "/dev/sdb1 /var/lib/ceph/osd/ceph-1 xfs defaults 0 0" >> /etc/fstab```<br>
### 7.初始化 OSD 数据目录
  ```ceph-osd -i 0 --mkfs --mkkey```<br>
### 8.添加key
  ```ceph auth add osd.0 osd 'allow *' mon 'allow profile osd' -i /var/lib/ceph/osd/ceph-0/keyring```<br>
### 9.把新建的osd添加到crush中
  ```ceph osd crush add osd.0 1.0 host=node1```<br>
	 add item id 0 name 'osd.0' weight 1 at location {host=node1} to crush map<br>
### 10.修改osd数据目录的属主和属组为ceph
  ```chown -R ceph:ceph /var/lib/ceph/osd/ceph-0/```<br>
### 11.启动新添加的osd
  ```systemctl start ceph-osd@0```<br>
  ```systemctl status ceph-osd@0```<br>
### 12.设置osd开机自动启动
  ```systemctl enable ceph-osd@0```<br>
### 13.查看ceph osd tree状态
  ```ceph osd tree```<br>
### 14.添加新的osd
   和添加第一个osd的方法一样，这里写了个简单的添加脚本，可以通过脚本快速进行一下添加<br>
## PG个数的设置
### 1.通过ceph -s查看状态
    *这个时候执行ceps -s 集群状态应该是EALTH_WARN ,too few PGs per OSD* 
	PG计算方式
	total PGs = ((Total_number_of_OSD * 100) / max_replication_count) / pool_count

	当前ceph集群是9个osd，3副本，1个默认的rbd pool

	所以PG计算结果为300，一般把这个值设置为与计算结果最接近的2的幂数，跟300比较接近的是256
### 2.查看当前的PG值
  ```ceph osd pool get rbd pg_num``` <br>
	pg_num: 64 <br>
  ```ceph osd pool get rbd pgp_num``` <br>
	pgp_num: 64 <br>
### 3.手动设置
  ```ceph osd pool set rbd pg_num 256``` <br>
	set pool 0 pg_num to 256 <br>
  ```ceph osd pool set rbd pgp_num 256``` <br>
	set pool 0 pgp_num to 256 <br>
### 4.再次查看状态
  ```ceph -s```  <br>
  正常集群会显示HEALTH_OK<br>
## 部署MDS
### 1.为mds元数据服务器创建一个目录
  ```mkdir -p /var/lib/ceph/mds/ceph-mds1``` <br>
### 2.确保存在 /var/lib/ceph/bootstrap-mds/ceph.keyring 文件
### 3.在root家目录里创建ceph.bootstrap-mds.keyring文件
  ```touch /root/ceph.bootstrap-mds.keyring``` <br>
### 4. 把keyring /var/lib/ceph/bootstrap-mds/ceph.keyring里的密钥导入家目录下的ceph.bootstrap-mds.keyring文件里
  ```ceph-authtool --import-keyring /var/lib/ceph/bootstrap-mds/ceph.keyring ceph.bootstrap-mds.keyring```<br>
	importing contents of /var/lib/ceph/bootstrap-mds/ceph.keyring into ceph.bootstrap-mds.keyring<br>
### 5. 在ceph auth库中创建mds.mds1用户，并赋予权限和创建密钥，密钥保存在/var/lib/ceph/mds/ceph-mds1/keyring文件里
  ```ceph --cluster ceph --name client.bootstrap-mds --keyring /var/lib/ceph/bootstrap-mds/ceph.keyring auth get-or-create mds.mds1 osd 'allow rwx' mds 'allow' mon 'allow profile mds' -o /var/lib/ceph/mds/ceph-mds1/keyring``` <br>
### 6.启动MDS
  ```systemctl start ceph-mds@mds1```<br>
  ```systemctl status ceph-mds@mds1```<br>
### 7.设置mds开机自动启动
  ```systemctl enable ceph-mds@mds1```<br>
## 部署第二个MDS
### 1.拷贝密钥文件到mds2
  ```scp ceph.bootstrap-mds.keyring mds2:/root/ceph.bootstrap-mds.keyring``` <br>  
### 2.在mds2上创建mds元数据目录
  ```mkdir -p /var/lib/ceph/mds/ceph-mds2```
### 3.在ceph auth库中创建mds.mds2用户，并赋予权限和创建密钥，密钥保存在/var/lib/ceph/mds/ceph-mds2/keyring文件里
  ```ceph --cluster ceph --name client.bootstrap-mds --keyring /var/lib/ceph/bootstrap-mds/ceph.keyring auth get-or-create mds.mds2 osd 'allow rwx' mds 'allow' mon 'allow profile mds' -o /var/lib/ceph/mds/ceph-mds2/keyring``` <br> 
### 4.启动MDS
  ```systemctl start ceph-mds@mds2```<br>
  ```systemctl status ceph-mds@mds2```<br>
### 5.设置mds开机自动启动
  ```systemctl enable ceph-mds@mds2```<br>
## 挂载文件系统
### 1.创建文件系统用到的pool
  ```ceph osd pool create cephfs_data 256```<br>
  ```ceph osd pool create cephfs_metadata 256```<br>
  ```ceph fs new cephfs cephfs_metadata cephfs_data```<br>
  备注：大小和pg大小一致<br>
### 2.内核挂载方式
  ```mount -t ceph mon1,mon2,mon3:6789:/  /mnt/cephfs -o name=admin,secret=AQDuGjFabSMHAxAAEbYLDjpa3EQUaSGB/EtkXg==```
### 3.ceph挂载方式
  ```  ceph-fuse -m xxxx,yyyy,yyy:6789 /mnt/cephfs```<br>
  备注：本地机器需要 存在/etc/ceph/ceph.client.admin.keyring <br>
  
  ### 开机自动挂载
  ```echo "mon1,mon2,mon3:6789:/     /mnt/cephfs    ceph    name=admin,secret=AQBampNbrmnXBxAAYXtTNP0u7qzh1JpFvY887g==,noatime,_netdev    0       2" >>/etc/fstab```
  
  参考
https://yq.aliyun.com/articles/604372


