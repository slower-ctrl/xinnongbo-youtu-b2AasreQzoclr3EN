
# 磁盘管理


Linux哲学思想：Linux中一切皆文件


所以对于硬件设备来说，在Linux中也是以文件的形式存在，设备文件



```
[root@kylin-xu ~]# ll /dev/sda*
brw-rw---- 1 root disk 8, 0 Nov 20 04:11 /dev/sda
brw-rw---- 1 root disk 8, 1 Nov 20 04:11 /dev/sda1
brw-rw---- 1 root disk 8, 2 Nov 20 04:11 /dev/sda2
				 主设备号  次设备号
[root@kylin-xu ~]# ll /dev/zero 
crw-rw-rw- 1 root root 1, 5 Nov 20 04:11 /dev/zero

```


> 主设备号：标识设备类型 major number
> 
> 
> 次设备号：标识同一类型下的不同设备 minor number


## 【1】、磁盘外部结构


磁盘分类:


固态硬盘: 内部是主板和U盘类似


机械硬盘: 盘片 主轴 传动手臂 做机械运动 类似 DVD


Nvme硬盘


PCI\-E接口


大小分类:


3\.5英寸 : 台式机


2\.5英寸: 服务器 笔记本


接口类型:


IDE接口 \# 淘汰


SCSI接口 \# 淘汰


SATA接口 \# 台式机 笔记本


SAS接口 \#企业服务器标配


固态磁盘价格高,存储少。有寿命。


机械磁盘价格低,存储大。老不死。


固态速度比机械磁盘速度快


磁盘存储大小和转速:


企业标配SAS接口: 300G 600G 900G 转速 每分钟转多少圈 5400转 7200转 10K 15K


转速越快性能越好


存储越大转速越慢 1T 转速最高10K 2T 4T 8T 20T


**注意：速度不是由单纯的接口类型决定，支持Nvme协议硬盘速度是最快的**


## 【2】、磁盘阵列Raid



```
作用:
获得更大的容量		 # 将多块磁盘逻辑的组合成一块磁盘
获得更高的性能	     # 写入服务器 写两块磁盘比写一块磁盘速度快
获得更好的安全性    # 可以同时将数据写入两块盘 一块盘做备份

```

**面试\+笔试题**




| RAID级别 | 硬盘数量 | 可用容量 | 安全性 | 性能 | 使用场景 |
| --- | --- | --- | --- | --- | --- |
| 0 | 至少一块 | 磁盘数量总和 | 不安全 | 读写最快 | 要求速度不要安全，例如数据库从库 |
| 1 | 只能2块 | 磁盘数量的一半 | 可以坏一块 | 写慢，读凑合 | 要求安全,速度一般的场景，例如系统盘 |
| 5 | 至少3块 | n\-1块 | 坏1块 | 0和1的折中 | 业务流量较稳定的场景 |
| 10 | 至少4块，2的倍数即可 | 一半 | 坏1半 | 读写速度快 | 高并发场景 |


## 【3】、磁盘分区


windows磁盘默认的是MBR格式


MBR格式最多支持4个主分区 C D E F


MBR格式支持3个主分支\+1个扩展分区


Linux磁盘表示方法



```
sda   # 表示第一块磁盘
    sda1 # 表示第一块磁盘的第一个分区
    sda2 # 表示第一块磁盘的第二个分区
sdb   # 表示第二块磁盘
    sdb1 # 表示第2块磁盘的第1个分区
    sdb5 # 表示第2块磁盘的第1个逻辑分区

```

### 1、Linux中磁盘分区方式



```
1.系统分区
第一种分区: 标准分区   300G磁盘
/boot   200M    # 存放系统内核的位置 引导程序所在的位置
/		剩余空间 # 存放系统

第二种分区: swap分区
/boot  200M
swap   2G       # 当内存空间不够用时，临时使用磁盘空间充当内存来使用 速度慢 解决OOM问题 内存溢出。
			    # linux内存如果达到最大限制,则自动杀死占用最高内存的进程来让系统正常运行
				# swap 对用户的服务器需要增加物理内存
		        # 比较着急，或者公司内部测试服务器 自己使用的。
/     剩余空间

第三种分区: 比较少
/boot  200M
swap   2G
/      50G    # 系统
/data  1.8T   # 数据分区

```

### 2、磁盘分区类型


MBR：4 primary parted or 3 primary parted \+ 1 extended parted \+ many logical parted


GPT： 128 primary parted


### 3、磁盘分区步骤



```
1.MBR格式  小于2T磁盘使用fdisk分区
2.GPT分区  大于2T磁盘使用parted分区

第一步：插入一块20G SCSI类型硬盘

```


```
第二部：重启 或者 通过命令行重新扫描SCSI总线来添加硬盘
echo '- - -' > /sys/class/scsi_host/host0/scan 
echo '- - -' > /sys/class/scsi_host/host1/scan 
echo '- - -' > /sys/class/scsi_host/host2/scan 
# 由于我们并不知道哪一个总线是，就都扫一遍就行

# 再Ubuntu系统中/sys/class/scsi_hosts/有许多总线，我们有两种方式
root@xu-ubuntu:~# for i in `ls /sys/class/scsi_host/` ; do echo '- - -' > /sys/class/scsi_host/${i}/scan; done
# 找出SPI总线对应的 host，只扫描该 host
root@xu-ubuntu:~# grep mptspi /sys/class/scsi_host/host*/proc_name
/sys/class/scsi_host/host31/proc_name:mptspi
root@xu-ubuntu:~# echo '- - -' > /sys/class/scsi_host/host31/scan

```


```
第三部：磁盘分区  # fdisk创建分区
[root@kylin-xu ~]# fdisk  /dev/sdb
Command (m for help): m			  # 查看菜单

Help:
   d   delete a partition		  # 删除一个分区  
   l   list known partition types # 显示分区类型
   n   add a new partition		  # 创建新的分区
   p   print the partition table  # 输出分好的分区表
   m   print this menu		      # 打印菜单
   w   write table to disk and exit #保存并且推出
   q   quit without saving changes	# 退出不保存
Command (m for help): n   # 创建分区
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p  # 指定类型，默认是primary
Partition number (1-4, default 1): 
First sector (2048-41943039, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-41943039, default 41943039): +5G

Created a new partition 1 of type 'Linux' and of size 5 GiB.
Command (m for help): n   # 第二个分区
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (2-4, default 2): 
First sector (10487808-41943039, default 10487808): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (10487808-41943039, default 41943039): +10G

Created a new partition 2 of type 'Linux' and of size 10 GiB.

Command (m for help): d   # 删除分区 
Partition number (1-4, default 4): 

Command (m for help): n  # 创建一个扩展分区 
Partition type
   p   primary (3 primary, 0 extended, 1 free)
   e   extended (container for logical partitions)
Select (default e): 

Using default response e.
Selected partition 4
First sector (33556480-41943039, default 33556480): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (33556480-41943039, default 41943039): 

Created a new partition 4 of type 'Extended' and of size 4 GiB.

Command (m for help): p   # 查看创建的分区类型
Disk /dev/sdb: 20 GiB, 21474836480 bytes, 41943040 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x5fbfe031

Device     Boot    Start      End  Sectors Size Id Type
/dev/sdb1           2048 10487807 10485760   5G 83 Linux
/dev/sdb2       10487808 31459327 20971520  10G 83 Linux
/dev/sdb3       31459328 33556479  2097152   1G 83 Linux
/dev/sdb4       33556480 41943039  8386560   4G  5 Extended


Command (m for help): n  # 创建逻辑分区
All primary partitions are in use.
Adding logical partition 6
First sector (35657728-41943039, default 35657728): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (35657728-41943039, default 41943039): 

Created a new partition 6 of type 'Linux' and of size 3 GiB.


Command (m for help): w  # 保存分区
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

[root@kylin-xu ~]# lsblk 
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   60G  0 disk 
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   59G  0 part 
  ├─klas-root   253:0    0 38.3G  0 lvm  /
  ├─klas-swap   253:1    0    2G  0 lvm  [SWAP]
  └─klas-backup 253:2    0 18.7G  0 lvm  /backup
sdb               8:16   0   20G  0 disk 
├─sdb1            8:17   0    5G  0 part 
├─sdb2            8:18   0   10G  0 part 
├─sdb3            8:19   0    1G  0 part 
├─sdb4            8:20   0    1K  0 part 
├─sdb5            8:21   0    1G  0 part 
└─sdb6            8:22   0    3G  0 part 
sr0              11:0    1  4.3G  0 rom 

# 如果我们使用分区完成后，使用lsblk查看不到我们分区的内容，这是由于lsblk是查看内存中的信息，又可能我们分区完成后，内存没有更新，这是我们需要手动刷新下
[root@kylin-xu ~]# partprobe

```

**非交互式创建分区**



```
[root@kylin-xu ~]# echo -e 'p\nn\n\n+1G\np\n' | fdisk /dev/sdb
# \n 表示回车

```


```
# parted创建分区
(parted) mklabel gpt 		# 将磁盘类型修改为gpt格式
(parted) print
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdc: 4295GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

(parted) mkpart             	# 创建分区                                              
Partition name?  []? primary    # 分区的名称  自定义叫啥都行                         
File system type?  [ext2]? xfs                                            
Start? 0                                                                  
End? 500G                                                                 
Warning: The resulting partition is not properly aligned for best performance: 34s % 2048s != 0s
Ignore/Cancel? I  
(parted) print   
Disk Flags: 

Number  Start   End    Size   File system  Name     Flags
 1      17.4kB  500GB  500GB  xfs          primary

(parted) mkpart 
Partition name?  []? parimary                                             
File system type?  [ext2]? xfs                                            
Start? 500G                                                               
End? 1000G
(parted) print                                                            
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdc: 4295GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size   File system  Name      Flags
 1      17.4kB  500GB   500GB               primary
 2      500GB   1000GB  500GB  xfs          parimary

(parted) mkpart primary xfs 1000G  4000G    	# 非交互式分区                             
(parted) print                                                            
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdc: 4295GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:
(parted) quit  # 退出parted分区


```


> parted重要的子命令
> 
> 
> help \[COMMAND] \# 打印菜单帮助
> mklabel,mktable LABEL\-TYPE \# 创建分区表类型
> mkpart PART\-TYPE \[FS\-TYPE] START END \# 创建分区
> print \[devices\|free\|list,all\|NUMBER] \# 查看分区表
> quit \# 退出
> rm NUMBER \# 删除分区



```
第四步：格式化分区，在格式化分区之后就会出现UUID
[root@kylin-xu ~]# mkfs.xfs  /dev/sdb1
meta-data=/dev/sdb1              isize=512    agcount=4, agsize=327680 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=1310720, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0

```


```
第五步：创建挂载点，挂载分区
[root@kylin-xu ~]# mkdir  /mnt/xfs
[root@kylin-xu ~]# mount /dev/sdb1 /mnt/xfs
[root@kylin-xu ~]# df -Th | grep /mnt/xfs
/dev/sdb1               xfs       5.0G   68M  5.0G   2% /mnt/xfs

```


```
第六步：写入/etc/fstab，开机自动挂载
vim /etc/fstab
/dev/sdb1 /mnt/xfs xfs defaults 0 0


卸载分区
[root@kylin-xu xfs]# touch 123
[root@kylin-xu xfs]# cd
[root@kylin-xu ~]# umount /mnt/xfs 
[root@kylin-xu ~]# df -Th | grep /mnt/xfs
[root@kylin-xu ~]# ll /mnt/xfs
total 0
# 我们将分区卸载之后，我们创建的内容也会消失，
# 我们重新挂载上之后就又会出现
[root@kylin-xu ~]# mount /dev/sdb1 /mnt/xfs
[root@kylin-xu ~]# ll /mnt/xfs
total 0
-rw-r--r-- 1 root root 0 Nov 20 04:08 123

```

**我们在表示logical parted时尽量不要使用路径去表示，要使用UUID**


### 4、Ubuntu中进行磁盘分区



```
# 添加磁盘
root@xu-ubuntu:~# for i in `ls /sys/class/scsi_host/` ; do echo '- - -' > /sys/class/scsi_host/${i}/scan; done
root@xu-ubuntu:~# fdisk  /dev/sdb -l
Disk /dev/sdb: 20 GiB, 21474836480 bytes, 41943040 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


```

在Ubuntu中磁盘分区使用的是gpt分区



```
root@xu-ubuntu:~# gdisk /dev/sdb
GPT fdisk (gdisk) version 1.0.8

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries in memory.

Command (? for help): p
Disk /dev/sdb: 41943040 sectors, 20.0 GiB
Model: VMware Virtual S
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): C4537E56-7D8E-4F18-916F-4C63377E3032
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 41943006
Partitions will be aligned on 2048-sector boundaries
Total free space is 41942973 sectors (20.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name

Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-41943006, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-41943006, default = 41943006) or {+-}size{KMGTP}: +4G
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 
Changed type of partition to 'Linux filesystem'

Command (? for help): p
Disk /dev/sdb: 41943040 sectors, 20.0 GiB
Model: VMware Virtual S
Sector size (logical/physical): 512/512 bytes
Disk identifier (GUID): C4537E56-7D8E-4F18-916F-4C63377E3032
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 41943006
Partitions will be aligned on 2048-sector boundaries
Total free space is 33554365 sectors (16.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         8390655   4.0 GiB     8300  Linux filesystem
Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully.


```


```
# 格式化
mkfs

```

## 【4】、文件系统


### 1、什么是文件系统


文件系统是操作系统用于明确存储设备或分区上的文件的方法和数据结构；即在存储设备上组织文件的方法。


操作系统中负责管理和存储文件信息的软件结构称为文件管理系统，简称文件系统。从系统角度来看，文件系统是对文件存储设备的空间进行组织和分配，负责文件存储并对存入的文件进行保护和检索的系统。具体地说，它负责为用户建立文件，存入、读出、修改、转储文件，控制文件的存取，安全控制，日志，压缩，加密等。


查看当前内核支持的文件系统：



```
[root@kylin-xu ~]# ls /lib/modules/`uname -r`/kernel/fs
root@xu-ubuntu:~#  ls /lib/modules/`uname -r`/kernel/fs

```

系统支持的文件系统并不全是可用的，查看当前系统可用的文件系统：



```
 cat /proc/filesystems

```

当前系统支持的文件系统和当前系统可用的文件系统是两回事，modules 中的文件系统在编译时选择了才是可用的，而可用的文件系统包含了默认支持的文件系统，如果需要使用某个文件系统，而该文件系统又不在proc 中，则需要重新编译内核；


### 2、文件系统类型


**Linux** **常用文件系统**




| 文件系统 | 备注 |
| --- | --- |
| ext2 | Extended file system 适用于那些分区容量不是太大，更新也不频繁的情况，例如/boot 分 区 |
| ext3 | ext2 的改进版本，其支持日志功能，能够帮助系统从非正常关机导致的异常中恢复 |
| ext4 | ext 文件系统的最新版。有很多新的特性，包括纳秒级时间戳、巨型文件 (16TB)、最大1EB的文件系统，以及速度的提升 |
| xfs | SGI，支持最大8EB的文件系统 |
| swap | 交换分区专用的文件系统 |
| iso9660 | 光盘文件系统 |


**Windows** **常用文件系统**




| 文件系统 | 备注 |
| --- | --- |
| FAT32 | 最多只能支持16TB的文件系统和4GB的文件 |
| NTFS | 最多只能支持16EB的文件系统和16EB的文件 |
| extFAT |  |


**Unix**\***常用文件系统**




| 文件系统 | 备注 |
| --- | --- |
| FFS（fast） |  |
| UFS（unix） | UFS是UNIX文件系统的简称，几乎是大部分UNIX类操作系统默认的基于磁盘的文件系统 |
| JF32 |  |


### 3、创建文件系统



```
root@xu-ubuntu:~# mkfs.ext4 /dev/sdb1
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 1048576 4k blocks and 262144 inodes   # 创建block和inode
Filesystem UUID: 83d410e3-5dcb-415a-8ef0-54c3d73e4b34   # 设置UUID
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

root@xu-ubuntu:~# mkfs.xfs  /dev/sdb2
meta-data=/dev/sdb2              isize=512    agcount=4, agsize=327680 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0 inobtcount=0
data     =                       bsize=4096   blocks=1310720, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0


root@xu-ubuntu:~# lsblk  -f /dev/sdb
NAME   FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sdb                                                                           
├─sdb1 ext4   1.0         83d410e3-5dcb-415a-8ef0-54c3d73e4b34                
├─sdb2 xfs                af832cf3-01c7-48eb-b4bc-a76605c7d5f6                
└─sdb3

```

## 【5】、磁盘挂载



```
 mount [-lhV]
 mount -a [options]
 mount [options] [--source] <source> | [--target] 
 mount [options] <source> 
 mount   []

```

**device：指明要挂载的设备**


* 设备文件：例如:/dev/sda5
* 卷标：\-L 'LABEL', 例如 \-L 'MYDATA'
* UUID： \-U 'UUID'：例如 \-U '0c50523c\-43f1\-45e7\-85c0\-a126711d406e'
* 伪文件系统名称：proc, sysfs, devtmpfs, configfs


**mountpoint：挂载点目录必须事先存在，建议使用空目录**


**挂载规则**


* 一个挂载点同一时间只能挂载一个设备
* 一个挂载点同一时间挂载了多个设备，只能看到最后一个设备的数据，其它设备上的数据将被隐藏
* 一个设备可以同时挂载到多个挂载点
* 通常挂载点一般是已存在空的目录



```
root@xu-ubuntu:~# mount --source /dev/sdb1 -o ro /dir1
# --source 指定挂载源，-o 指定挂载选项  ro 表示只读

root@xu-ubuntu:~# df -lh | grep  dir1
/dev/sdb1                          3.9G   24K  3.7G   1% /dir1
root@xu-ubuntu:~# touch /dir1/lll
touch: cannot touch '/dir1/lll': Read-only file system
root@xu-ubuntu:~# mount  | grep sdb1
/dev/sdb1 on /dir1 type ext4 (ro,relatime)

```

## 【6】、持久化挂载


/etc/fstab


每行定义一个要挂载的文件系统,，其中包括共 6 项


* 要挂载的设备或伪文件系统设备文件(LABEL\=label \| UUID\=uuid \| /dev/sda1\)
* 挂载点：必须是事先存在的目录
* 文件系统类型：ext4，xfs，iso9660，nfs，none
* 挂载选项：defaults ，acl，bind，ro，rw 等
* 转储频率：0 不做备份; 1 每天转储; 2 每隔一天转储
* fsck检查的文件系统的顺序：0 不自检 ; 1 首先自检，一般只有rootfs才用；2 非rootfs使用 0



```
/dev/sdb1    /mnt/xfs     xfs    defaults   0   0

```

## 【7】、磁盘常用工具


### 1、df


**文件系统查看工具**



```
-a：显示所有
-h：以方便的形式显示数据
-T：显示文件系统类型
-t：按照文件系统类型过滤
root@xu-ubuntu:~# df -t ext4 
-l：只显示本机的文件系统
--total：最后一行汇总输出
root@xu-ubuntu:~# df -lTh --total 
Filesystem                        Type   Size  Used Avail Use% Mounted on
tmpfs                             tmpfs  388M  1.6M  386M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv ext4    24G  8.9G   14G  40% /
tmpfs                             tmpfs  1.9G     0  1.9G   0% /dev/shm
tmpfs                             tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/sda2                         ext4   2.0G  242M  1.6G  14% /boot
tmpfs                             tmpfs  388M  4.0K  388M   1% /run/user/1000
/dev/sdb1                         ext4   3.9G   24K  3.7G   1% /dir1
total                             -       32G  9.2G   22G  31% -

```

### 2、du


**目录统计工具**



```
-s：只显示外层目录
-h：以方便阅读的格式显示
--total：汇总统计到的所有数据
root@xu-ubuntu:~# du /etc/ -sh
6.7M    /etc/
root@xu-ubuntu:~# du /etc/  /var/log/ -sh --total 
6.7M    /etc/
36M     /var/log/
42M     total

```

### 3、dd



```
if=file				#从所命名文件读取而不是从标准输入
of=file			    #写到所命名的文件而不是到标准输出
bs=size				#指定块大小（既是是ibs也是obs）
count=n				#复制几个bs

```


```
# 备份MBR
root@xu-ubuntu:~# dd if=/dev/sdb of=./sdb.img bs=512 count=1

# 备份整个磁盘
dd if=/dev/sdx of=/path/to/image

```

## 【8】、磁盘相关案例


**企业案例1：java环境内存不够用，大量占用swap**



```
swap磁盘分区: 大小是内存的1-1.5倍 如果内存大于8G，则最多swap给8G即可。
什么情况下使用swap: 
       测试服务器 
       内部服务器 
       自己用的虚拟机 
       流量波动的业务。
线上服务器，业务服务器禁止使用swap，增加内存的方式解决OOM（内存溢出）问题。

```

如何创建swap


* 方法1: 安装操作系统的时候
* 方法2: 安装完成操作系统



```
# 生成一个1G的文件
[root@kylin-xu ~]# dd if=/dev/zero  of=/1g bs=1G count=1
1+0 records in
1+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 6.15619 s, 174 MB/s

# 格式化为swap
[root@kylin-xu ~]# mkswap  /1g 
mkswap: /1g: insecure permissions 0644, 0600 suggested.
Setting up swapspace version 1, size = 1024 MiB (1073737728 bytes)
no label, UUID=3cbda684-b958-476e-b08c-1b94834d0657
[root@kylin-xu ~]# blkid /1g
/1g: UUID="3cbda684-b958-476e-b08c-1b94834d0657" TYPE="swap"

# 挂载使用swap 直接增加到原有的swap空间上
[root@kylin-xu ~]# swapon /1g
swapon: /1g: insecure permissions 0644, 0600  suggested.
[root@kylin-xu ~]# swapon -s  # 查看分区的组成
Filename                                Type            Size    Used    Priority
/dev/dm-1                               partition       2138108 13324   -2
/1g                                     file            1048572 0       -3
[root@kylin-xu ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:          1.9Gi       178Mi       1.1Gi        10Mi       697Mi       1.6Gi
Swap:         3.0Gi        13Mi       3.0Gi

# 写入/etc/fstab

```

**企业案例2\.无法写入文件到磁盘**



```
第一种情况：磁盘空间满了
# 需要查找系统中的大文件，找出磁盘中大于1G的文件。
# 注意mount 会隐藏原来目录下的文件
find / -size +1G -type f
dd if=/dev/zero of=/opt/2g bs=1M count=2000
[root@kylin-xu ~]# find / -type f -size  +1G
/proc/kcore
/opt/2g

第二种情况：inode号满了,找出文件中大量的小文件
[root@kylin-xu ~]# find / -type d -size +1M 

```

**企业案例3\.解决磁盘不够用的问题**


需求: 目前磁盘不够用



```
1.业务会持续输出日志内容到 /var/log/nginx.log文件中 10G
[root@kylin-xu ~]# dd if=/dev/zero of=/var/log/10g bs=1M count=10000
2.日志的路径不能变,解决磁盘不够用的问题。

```


```
解决方案:
1.增加大的磁盘
增加4T的磁盘
2.挂载磁盘到/data目录
[root@kylin-xu ~]# mount /dev/sdc3 /data
3.将10g文件移动到/data目录
[root@kylin-xu ~]# mv /var/log/10g  /data/
[root@kylin-xu ~]# ll /data/
total 9.8G
-rw-r--r-- 1 root root 9.8G Nov 20 17:11 10g
[root@kylin-xu ~]# df -Th
Filesystem              Type      Size  Used Avail Use% Mounted on
devtmpfs                devtmpfs  963M     0  963M   0% /dev
tmpfs                   tmpfs     979M     0  979M   0% /dev/shm
tmpfs                   tmpfs     979M   22M  957M   3% /run
tmpfs                   tmpfs     979M     0  979M   0% /sys/fs/cgroup
/dev/mapper/klas-root   xfs        39G  6.7G   32G  18% /
tmpfs                   tmpfs     979M     0  979M   0% /tmp
/dev/sdb1               xfs       5.0G   68M  5.0G   2% /mnt/xfs
/dev/mapper/klas-backup xfs        19G  171M   19G   1% /backup
/dev/sda1               xfs      1014M  169M  846M  17% /boot
tmpfs                   tmpfs     196M     0  196M   0% /run/user/0
/dev/sdc3               xfs       2.8T   30G  2.7T   2% /data
4.软链接到/var/log/10g
[root@kylin-xu ~]# ln -s /data/10g  /var/log/10g
[root@kylin-xu ~]# ll /var/log/10g 
lrwxrwxrwx 1 root root 9 Nov 20 17:15 /var/log/10g -> /data/10g


```

**企业案例4\.删除文件**



```
磁盘上有个一39G的文件,删除后发现磁盘没有释放。
文件如果被进程所占用,会出现删除文件磁盘没有释放的问题。
查看当前的文件被哪个进程所占用:
[root@oldboyedu ~]# lsof |grep 10g
tail      2873                          root    3r      REG               8,32 41838247936        132 /data/10 (deleted)

杀死进程或者重启或者重新加载
[root@oldboyedu ~]# kill -9 2873

```

## 【9】、逻辑卷


### 1、什么是逻辑卷


LVM: Logical Volume Manager 可以允许对卷进行方便操作的抽象层，包括重新设定文件系统的大小，允许在多个物理设备间重新组织文件系统


LVM可以弹性的更改LVM的容量


**实现过程**


* 将设备指定为物理卷
* 用一个或者多个物理卷来创建一个卷组，物理卷是用固定大小的物理区域（Physical Extent，PE）来定义的
* 在物理卷上创建的逻辑卷， 是由物理区域（PE）组成
* 可以在逻辑卷上创建文件系统并挂载


### 2、创建逻辑卷



```
# 创建pvs
root@xu-ubuntu:~# pvcreate /dev/sdb{1..3}
  Physical volume "/dev/sdb1" successfully created.
  Physical volume "/dev/sdb2" successfully created.
  Physical volume "/dev/sdb3" successfully created.
root@xu-ubuntu:~# pvs
  PV         VG        Fmt  Attr PSize   PFree  
  /dev/sda3  ubuntu-vg lvm2 a--  <48.00g  24.00g
  /dev/sdb1            lvm2 ---    4.00g   4.00g
  /dev/sdb2            lvm2 ---    5.00g   5.00g
  /dev/sdb3            lvm2 ---  <11.00g <11.00g
  
# 创建vg
root@xu-ubuntu:~# vgcreate vg1 /dev/sdb{1..2}
  Volume group "vg1" successfully created
root@xu-ubuntu:~# vgs
  VG        #PV #LV #SN Attr   VSize   VFree 
  ubuntu-vg   1   1   0 wz--n- <48.00g 24.00g
  vg1         2   0   0 wz--n-   8.99g  8.99g


# 创建lvm -L：指定逻辑卷大小  -n：指定逻辑卷名字   -l：按照PE数量进行划分
root@xu-ubuntu:~# lvcreate -L 5G -n lv1 vg1
  Logical volume "lv1" created.
root@xu-ubuntu:~# lvs
  LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  ubuntu-lv ubuntu-vg -wi-ao---- <24.00g                                                    
  lv1       vg1       -wi-a-----   5.00g  
  
root@xu-ubuntu:~# lvcreate -l 500 -n lv2 vg1
  Logical volume "lv2" created.
root@xu-ubuntu:~# lvs
  LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  ubuntu-lv ubuntu-vg -wi-ao---- <24.00g                                                    
  lv1       vg1       -wi-a-----   5.00g                                                    
  lv2       vg1       -wi-a-----   1.95g                                                    


```

我们后续如果想使用lvm，也需要进行格式化和挂载才可以



```
# 格式化和挂载
root@xu-ubuntu:~# mkfs.ext4 /dev/vg1/lv1
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 1310720 4k blocks and 327680 inodes
Filesystem UUID: ff6b294f-b30c-4ac2-94e9-cc57b086d207
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 

root@xu-ubuntu:~# mount /dev/vg1/lv1 /lv1


```

### 2、对lvm进行扩容



```
# ext系列文件系统
# 在扩容lvm之前我们得看下vg是否还有空间允许扩容，如果vg还有空间，我们直接扩容lvm即可。如果vg没有空间，我们得先扩容vg，扩容vg需要添加pv，如果有空闲的pv直接扩容即可，没有空闲的pv，先使用pvcreate创建出pv
root@xu-ubuntu:~# vgextend vg1  /dev/sdb3
  Volume group "vg1" successfully extended
root@xu-ubuntu:~# lvextend -L 7G /dev/vg1/lv1
  Size of logical volume vg1/lv1 changed from 5.00 GiB (1280 extents) to 7.00 GiB (1792 extents).
  Logical volume vg1/lv1 successfully resized.
root@xu-ubuntu:~# df -Th   # 扩容完成后我们还需要告知文件系统，才能生效
Filesystem                        Type   Size  Used Avail Use% Mounted on
tmpfs                             tmpfs  388M  1.7M  386M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv ext4    24G  8.9G   14G  40% /
tmpfs                             tmpfs  1.9G     0  1.9G   0% /dev/shm
tmpfs                             tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/sda2                         ext4   2.0G  242M  1.6G  14% /boot
tmpfs                             tmpfs  388M  4.0K  388M   1% /run/user/1000
/dev/mapper/vg1-lv1               ext4   4.9G   24K  4.6G   1% /lv1
root@xu-ubuntu:~# resize2fs /dev/vg1/lv1    # 使用resize2fs告知ext系列文件系统
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/vg1/lv1 is mounted on /lv1; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/vg1/lv1 is now 1835008 (4k) blocks long.

root@xu-ubuntu:~# resize2fs /dev/vg1/lv1 | grep lv1
resize2fs 1.46.5 (30-Dec-2021)
The filesystem is already 1835008 (4k) blocks long.  Nothing to do!

root@xu-ubuntu:~# df -Th | grep  lv1
/dev/mapper/vg1-lv1               ext4   6.9G   24K  6.5G   1% /lv1000000000000000000

```


```
# xfs类型文件系统
# 其他过程都一样，只是在告知文件系统时的命令不同 xfs文件系统使用的是 xfs_growfs
root@xu-ubuntu:~# lvextend -L 5G /dev/vg1/lv2
  Size of logical volume vg1/lv2 changed from 1.95 GiB (500 extents) to 5.00 GiB (1280 extents).
  Logical volume vg1/lv2 successfully resized.
root@xu-ubuntu:~# df -Th
Filesystem                        Type   Size  Used Avail Use% Mounted on
tmpfs                             tmpfs  388M  1.7M  386M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv ext4    24G  8.9G   14G  40% /
tmpfs                             tmpfs  1.9G     0  1.9G   0% /dev/shm
tmpfs                             tmpfs  5.0M     0  5.0M   0% /run/lock
/dev/sda2                         ext4   2.0G  242M  1.6G  14% /boot
tmpfs                             tmpfs  388M  4.0K  388M   1% /run/user/1000
/dev/mapper/vg1-lv1               ext4   6.9G   24K  6.5G   1% /lv1
/dev/mapper/vg1-lv2               xfs    2.0G   47M  1.9G   3% /lv2
root@xu-ubuntu:~# xfs_growfs /dev/vg1/lv2
meta-data=/dev/mapper/vg1-lv2    isize=512    agcount=4, agsize=128000 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0 inobtcount=0
data     =                       bsize=4096   blocks=512000, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 512000 to 1310720
root@xu-ubuntu:~# df -Th | grep lv2
/dev/mapper/vg1-lv2               xfs    5.0G   69M  5.0G   2% /lv2

```

 本博客参考[MeoMiao 萌喵加速](https://biqumo.org)。转载请注明出处！
