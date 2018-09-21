# 对象网关部署
## 创建keyring
```sudo ceph-authtool --create-keyring /etc/ceph/ceph.client.radosgw.keyring ```
   creating /etc/ceph/ceph.client.radosgw.keyring <br>
## 修改文件权限
```sudo chown ceph:ceph /etc/ceph/ceph.client.radosgw.keyring```
## 生成ceph-radosgw服务对应的用户和key
```sudo ceph-authtool /etc/ceph/ceph.client.radosgw.keyring -n client.rgw.node1 --gen-key```
## 为用户添加访问权限
```sudo ceph-authtool -n client.rgw.node1 --cap osd 'allow rwx' --cap mon 'allow rwx' /etc/ceph/ceph.client.radosgw.keyring```
## 导入keyring到集群中
```sudo ceph -k /etc/ceph/ceph.client.admin.keyring auth add client.rgw.node1 -i /etc/ceph/ceph.client.radosgw.keyring```
  added key for client.rgw.node1 <br>
## 配置ceph.conf
```
[client.rgw.node1]
host=node1
keyring=/etc/ceph/ceph.client.radosgw.keyring
log file=/var/log/radosgw/client.radosgw.gateway.log
rgw_s3_auth_use_keystone = False
rgw_frontends = civetweb port=8080
```
## 创建日志目录并修改权限
  ```
  mkdir /var/log/radosgw
  chown ceph:ceph /var/log/radosgw
  ```
## 启动rgw
```
systemctl start ceph-radosgw@rgw.node1
systemctl status ceph-radosgw@rgw.node1
systemctl enable ceph-radosgw@rgw.node1
```
## 测试
```
curl "http://mon1:8082"
```
# 部署其他的机器
以下命令在node1上执行即可
##  创建对应的client.rgw.node2、client.rgw.node3用户并进行授权
```
sudo ceph-authtool /etc/ceph/ceph.client.radosgw.keyring -n client.rgw.node2 --gen-key
sudo ceph-authtool -n client.rgw.node2 --cap osd 'allow rwx' --cap mon 'allow rwx' /etc/ceph/ceph.client.radosgw.keyring
sudo ceph -k /etc/ceph/ceph.client.admin.keyring auth add client.rgw.node2 -i /etc/ceph/ceph.client.radosgw.keyring
added key for client.rgw.node2
sudo ceph-authtool /etc/ceph/ceph.client.radosgw.keyring -n client.rgw.node3 --gen-key
sudo ceph-authtool -n client.rgw.node3 --cap osd 'allow rwx' --cap mon 'allow rwx' /etc/ceph/ceph.client.radosgw.keyring
sudo ceph -k /etc/ceph/ceph.client.admin.keyring auth add client.rgw.node3 -i /etc/ceph/ceph.client.radosgw.keyring
added key for client.rgw.node3
```
## 在ceph.conf文件中添加如下内容
```
[client.rgw.node2]
host=node2
keyring=/etc/ceph/ceph.client.radosgw.keyring
log file=/var/log/radosgw/client.radosgw.gateway.log
rgw_s3_auth_use_keystone = False
rgw_frontends = civetweb port=8080
[client.rgw.node3]
host=node3
keyring=/etc/ceph/ceph.client.radosgw.keyring
log file=/var/log/radosgw/client.radosgw.gateway.log
rgw_s3_auth_use_keystone = False
rgw_frontends = civetweb port=8080
```
##  把创建好的ceph.client.radosgw.keyring和ceph.conf传到node2和node3上
## 创建日志目录并修改权限
  ```
  mkdir /var/log/radosgw
  chown ceph:ceph /var/log/radosgw
  ```
## 启动rgw
```
systemctl start ceph-radosgw@rgw.node2
systemctl status ceph-radosgw@rgw.node
systemctl enable ceph-radosgw@rgw.node2
```

