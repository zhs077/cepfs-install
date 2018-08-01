# 版本jewel,没有采用ceph-deploy 部署，操作系统centos7.2
## 前置步骤
### 1. 安装包部署 ### <br>
2. 防火墙关闭，时钟同步 <br>
3. 创建一个ceph虚拟用户 <br>
  ```echo "ceph:x:167:167:Ceph daemons:/var/lib/ceph:/sbin/nologin" >> /etc/passwd```<br>
  ```echo "ceph:x:167:" >>  /etc/group``` <br>
4. 生成一个uuid <br>
  ```uuidgen``` <br>
  4834e3bd-3b59-4536-9465-36f2bd13f68a <br>
5. ceph.conf <br>
 ## 部署MON节点(3台机器）
1.为监控节点创建管理密钥 <br>
  ```ceph-authtool --create-keyring /tmp/ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *'``` <br>
  creating /tmp/ceph.mon.keyring <br>
  <br>
2.为ceph amin用户创建管理集群的密钥并赋予访问权限\
  ```sudo ceph-authtool --create-keyring /etc/ceph/ceph.client.admin.keyring --gen-key -n client.admin --set-uid=0 --cap mon 'allow *' --cap osd 'allow *' --cap mds 'allow *' --cap mgr 'allow *'``` <br>
  creating /etc/ceph/ceph.client.admin.keyring <br><br>
3.生成一个引导-osd密钥环，生成一个client.bootstrap-osd用户并将用户添加到密钥环中 <br>
  ```sudo ceph-authtool --create-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring --gen-key -n client.bootstrap-osd --cap mon 'profile bootstrap-osd'```<br>
  creating /var/lib/ceph/bootstrap-osd/ceph.keyring <br><br>
4.将生成的密钥添加到ceph.mon.keyring <br> 
  ```sudo ceph-authtool /tmp/ceph.mon.keyring --import-keyring /etc/ceph/ceph.client.admin.keyring``` <br>
  importing contents of /etc/ceph/ceph.client.admin.keyring into /tmp/ceph.mon.keyring <br>
  ```sudo ceph-authtool /tmp/ceph.mon.keyring --import-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring``` <br>
  importing contents of /var/lib/ceph/bootstrap-osd/ceph.keyring into /tmp/ceph.mon.keyring <br><br>
5.使用主机名、主机IP地址(ES)和FSID生成monmap。把它保存成/tmp/monmap
  ```monmaptool --create --add node1 192.168.1.10 --fsid bdfb36e0-23ed-4e2f-8bc6-b98d9fa9136c /tmp/monmap``` <br>
 
  monmaptool: monmap file /tmp/monmap <br>
  monmaptool: set fsid to bdfb36e0-23ed-4e2f-8bc6-b98d9fa9136c <br>
  monmaptool: writing epoch 0 to /tmp/monmap (1 monitors) <br><br>
6.创建一个默认的数据目录 <br>
  ```sudo -u ceph mkdir /var/lib/ceph/mon/ceph-node1``` <br>
7. 改ceph.mon.keyring属主和属组为ceph <br>
  ```chown ceph.ceph /tmp/ceph.mon.keyring``` <br><br>
8.为了防止重新被安装创建一个空的done文件 <br>
   ```sudo touch /var/lib/ceph/mon/ceph-node1/done``` <br><br>
9.启动mon
  ```systemctl start ceph-mon@node1``` <br><br>
10.查看运行状态
  ```systemctl status ceph-mon@node1``` <br><br>
11. 设置mon开机自动启动 <br>
  ```systemctl enable ceph-mon@node1``` <br><br>
12.第一个算节点部署成功 
16.  /etc/ceph/* 文件拷贝到所有机器 <br>
    /var/lib/ceph/bootstrap-osd/ceph.keyring 拷贝到所有机器（理论上拷贝到osd机器就可以） <br>
    /tmp/ceph.mon.keyring 拷贝到所有机器（理论上是mon机器就可以） <br>
    /var/lib/ceph/bootstrap-mds/ceph.keyring 拷贝到所有机器（理论上拷贝到mds机器就可以） <br><br>
## 新增mon2,mon3节点 <br>
### 1.在mon2上创建一个默认的数据目录
 ```sudo -u ceph mkdir /var/lib/ceph/mon/ceph-mon2``` <br><br>
### 2.在node2上修改ceph.mon.keyring属主和属组为ceph
 ```chown ceph.ceph /tmp/ceph.mon.keyring``` <br><br>
 ### 3.获取密钥和monmap信息(从mon1机器拷贝过来的秘钥)
 


  
参考
https://yq.aliyun.com/articles/604372
