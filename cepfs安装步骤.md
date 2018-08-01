# 版本jewel,没有采用ceph-deploy 部署，操作系统centos7.2 
## 前置步骤
1. 安装包部署 \
2. 防火墙关闭，时钟同步\
3.创建一个ceph虚拟用户\
  ```echo "ceph:x:167:167:Ceph daemons:/var/lib/ceph:/sbin/nologin" >> /etc/passwd```\
  ```echo "ceph:x:167:" >>  /etc/group```\

3. ceph.conf \
4.生成一个uuid\
  ```uuidgen```\
  4834e3bd-3b59-4536-9465-36f2bd13f68a\
   ##部署MON节点(3台机器）
1.为监控节点创建管理密钥 \
  ```ceph-authtool --create-keyring /tmp/ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *'```\
  creating /tmp/ceph.mon.keyring\
6.为ceph amin用户创建管理集群的密钥并赋予访问权限\
  ```sudo ceph-authtool --create-keyring /etc/ceph/ceph.client.admin.keyring --gen-key -n client.admin --set-uid=0 --cap mon 'allow *' --cap osd 'allow *' --cap mds 'allow *' --cap mgr 'allow *'```\
  creating /etc/ceph/ceph.client.admin.keyring\
7.生成一个引导-osd密钥环，生成一个client.bootstrap-osd用户并将用户添加到密钥环中\
  ```sudo ceph-authtool --create-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring --gen-key -n client.bootstrap-osd --cap mon 'profile bootstrap-osd'```\
  creating /var/lib/ceph/bootstrap-osd/ceph.keyring
8.将生成的密钥添加到ceph.mon.keyring
  sudo ceph-authtool /tmp/ceph.mon.keyring --import-keyring /etc/ceph/ceph.client.admin.keyring
  importing contents of /etc/ceph/ceph.client.admin.keyring into /tmp/ceph.mon.keyring
  sudo ceph-authtool /tmp/ceph.mon.keyring --import-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring
  importing contents of /var/lib/ceph/bootstrap-osd/ceph.keyring into /tmp/ceph.mon.keyring
9.使用主机名、主机IP地址(ES)和FSID生成monmap。把它保存成/tmp/monmap
  monmaptool --create --add node1 192.168.1.10 --fsid bdfb36e0-23ed-4e2f-8bc6-b98d9fa9136c /tmp/monmap
 
  monmaptool: monmap file /tmp/monmap
  monmaptool: set fsid to bdfb36e0-23ed-4e2f-8bc6-b98d9fa9136c
  monmaptool: writing epoch 0 to /tmp/monmap (1 monitors)
10.创建一个默认的数据目录
  sudo -u ceph mkdir /var/lib/ceph/mon/ceph-node1
11. 改ceph.mon.keyring属主和属组为ceph
  chown ceph.ceph /tmp/ceph.mon.keyring
12.为了防止重新被安装创建一个空的done文件
   sudo touch /var/lib/ceph/mon/ceph-node1/done
13.启动mon
  systemctl start ceph-mon@node1
14.查看运行状态
  systemctl status ceph-mon@node1
15. 设置mon开机自动启动
  systemctl enable ceph-mon@node1
  
16.  /etc/ceph/* 文件拷贝到所有机器
    /var/lib/ceph/bootstrap-osd/ceph.keyring 拷贝到所有机器（理论上拷贝到osd机器就可以）
    /tmp/ceph.mon.keyring 拷贝到所有机器（理论上是mon机器就可以）
    /var/lib/ceph/bootstrap-mds/ceph.keyring 拷贝到所有机器（理论上拷贝到mds机器就可以）
.新增mon2,mon3节点
17。 在node2上创建一个默认的数据目录
 sudo -u ceph mkdir /var/lib/ceph/mon/ceph-node2
 在node2上修改ceph.mon.keyring属主和属组为ceph


  
参考
https://yq.aliyun.com/articles/604372
