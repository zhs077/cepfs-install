# 命令集合
## 删除pool
1.找出要删除的pool名称 ```ceph osd pool ls ``` \
2.关闭mds ```systemctl  stop cep-mds@mds1``` \
3.无效化 ```ceph mds fail 0``` \
4.```ceph fs rm cephfs --yes-i-really-mean-it``` \
5.```ceph osd pool delete metadata metadata --yes-i-really-really-mean-it``` \
## cephfs挂载
1.内核挂载 mount -t ceph 192.168.159.131:6789:/  /mnt/cephfs -o name=admin,secret=AQDuGjFabSMHAxAAEbYLDjpa3EQUaSGB/EtkXg== \
2.ceph-fuse 挂载（本地需要有 /etc/ceph/ceph.client.admin.keyring ） ceph-fuse -m  xxxx,yyyy,yyy:6789 /mnt/cephfs  \
