
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
