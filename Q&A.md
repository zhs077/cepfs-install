# 命令集合
## 删除pool
1.找出要删除的pool名称 ```ceph osd pool ls ```
2.关闭mds ```systemctl  stop cep-mds@mds1```
3.无效化 ```ceph mds fail 0```
4.```ceph fs rm cephfs --yes-i-really-mean-it```
5.```ceph osd pool delete metadata metadata --yes-i-really-really-mean-it```
