
## 单副本情况,故障域OSD
* ceph -s
```
[root@hostname ~]# ceph -s
    cluster 08068b52-f0ab-4a50-8dad-cc3512082031
     health HEALTH_OK
     monmap e1: 1 mons at {mon1=221.230.143.150:6789/0}
            election epoch 4, quorum 0 mon1
      fsmap e8: 1/1/1 up {0=mds1=up:active}
     osdmap e95: 5 osds: 5 up, 5 in
            flags sortbitwise,require_jewel_osds
      pgmap v52100: 576 pgs, 3 pools, 13351 bytes data, 22 objects
            5311 MB used, 18608 GB / 18613 GB avail
                 576 active+clean
```
* ceph osd tree
```
[root@hostname ~]# ceph -sceph osd tree
ID WEIGHT  TYPE NAME         UP/DOWN REWEIGHT PRIMARY-AFFINITY 
-2       0 host mon1                                           
-1 4.00000 root default                                        
-3 4.00000     host hostname                                   
 0 1.00000         osd.0          up  1.00000          1.00000 
 1 1.00000         osd.1          up  1.00000          1.00000 
 2 1.00000         osd.2          up  1.00000          1.00000 
 3 1.00000         osd.3          up  1.00000          1.00000 
 4 1.00000         osd.4          up  1.00000          1.00000 
 ```
 * 创建三个文件1.txt 2.txt 3.txt
 * 查看1.txt 落到哪个OSD 
 ```
[root@hostname cephfs]# ceph osd map rbd 1.txt
osdmap e95 pool 'rbd' (0) object '1.txt' -> pg 0.e0e3a40b (0.b) -> up ([4], p4) acting ([4], p4)
```
* 停掉osd4
 systemctl stop ceph-osd@4
 ```
 [root@hostname cephfs]# ceph -s
    cluster 08068b52-f0ab-4a50-8dad-cc3512082031
     health HEALTH_ERR
            206 pgs are stuck inactive for more than 300 seconds
            161 pgs degraded
            47 pgs peering
            11 pgs stale
            206 pgs stuck inactive
            206 pgs stuck unclean
            161 pgs undersized
            recovery 5/46 objects degraded (10.870%)
            1/5 in osds are down
     monmap e1: 1 mons at {mon1=221.230.143.150:6789/0}
            election epoch 4, quorum 0 mon1
      fsmap e8: 1/1/1 up {0=mds1=up:active}
     osdmap e98: 5 osds: 4 up, 5 in; 208 remapped pgs
            flags sortbitwise,require_jewel_osds
      pgmap v52122: 576 pgs, 3 pools, 23083 bytes data, 23 objects
            5312 MB used, 18608 GB / 18613 GB avail
            5/46 objects degraded (10.870%)
                 357 active+clean
                 161 undersized+degraded+peered
                  47 peering
                  11 stale+active+clean
```
```
[root@hostname ~]# ceph osd tree
ID WEIGHT  TYPE NAME         UP/DOWN REWEIGHT PRIMARY-AFFINITY 
-2       0 host mon1                                           
-1 5.00000 root default                                        
-3 5.00000     host hostname                                   
 0 1.00000         osd.0          up  1.00000          1.00000 
 1 1.00000         osd.1          up  1.00000          1.00000 
 2 1.00000         osd.2          up  1.00000          1.00000 
 3 1.00000         osd.3          up  1.00000          1.00000 
 4 1.00000         osd.4        down  1.00000          1.00000 
```
[root@hostname cephfs]# ceph osd map rbd 1.txt
osdmap e98 pool 'rbd' (0) object '1.txt' -> pg 0.e0e3a40b (0.b) -> up ([], p-1) acting ([], p-1)
```
文件的读取和写入会卡主
```
[root@hostname cephfs]# touch 4.txt
c^H^H
```
* 过几分钟后集群会自动修复，我们看下修复后的状态
```
[root@hostname ~]# ceph -s
    cluster 08068b52-f0ab-4a50-8dad-cc3512082031
     health HEALTH_ERR
            208 pgs are stuck inactive for more than 300 seconds
            208 pgs degraded
            11 pgs stale
            208 pgs stuck inactive
            208 pgs stuck unclean
            208 pgs undersized
            8 requests are blocked > 32 sec```
            
            `recovery 11/46 objects degraded (23.913%)`
            ```
            1/5 in osds are down
     monmap e1: 1 mons at {mon1=221.230.143.150:6789/0}
            election epoch 4, quorum 0 mon1
      fsmap e8: 1/1/1 up {0=mds1=up:active}
     osdmap e98: 5 osds: 4 up, 5 in; 208 remapped pgs
            flags sortbitwise,require_jewel_osds
      pgmap v52162: 576 pgs, 3 pools, 23083 bytes data, 23 objects
            5312 MB used, 18608 GB / 18613 GB avail
            11/46 objects degraded (23.913%)
                 357 active+clean
                 208 undersized+degraded+peered
                  11 stale+active+clean
```
