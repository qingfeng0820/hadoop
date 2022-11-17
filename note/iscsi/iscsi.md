# iSCSI – target and initiator
* iSCSI (internet Small Computer System Interface) is an IP based storage networking protocol that’s designed for sharing block storage over the internet. 
* iSCSI follows the Server-Client model. The Server (aka Target) makes storage available for Clients (aka Initiators) to use.
## Target
* The Target makes the storage available in the form of a block device (e.g. /dev/sdb). 
### Setting up a Target
* Install target iSCSI software
  ```
  [root@iscsi-target ~]# yum install -y targetcli
  ```
* Find a block device to share
  ```
  [root@iscsi-target ~]# file /dev/sdb
  
  /dev/sdb: block special
  ```
* Create a target by targetcli command (interactive)
  ```
  [root@iscsi-target ~]# targetcli
  targetcli shell version 2.1.fb46
  Copyright 2011-2013 by Datera, Inc and others.
  For help on commands, type 'help'.
    
  />
  /> ls
  o- / ..................................................................... [...]
    o- backstores .......................................................... [...]
    | o- block .............................................. [Storage Objects: 0]
    | o- fileio ............................................. [Storage Objects: 0]
    | o- pscsi .............................................. [Storage Objects: 0]
    | o- ramdisk ............................................ [Storage Objects: 0]
    o- iscsi ........................................................ [Targets: 0]
    o- loopback ..................................................... [Targets: 0]
  />

  ```
  * you’ll find that there’s a json file whose structure mirrors the above tree. This json file gets updated in the background with changes you make in this interactive session. 
  * You can traverse this tree using the cd command, and within each section you can run the ‘help’ command to get more contextual help.
  * Use the tab+tab autocomplete technique extensively here to explore what commands are available in each section and more quickly construct your commands.
  * Steps:
    1. Register the block device: register the /dev/sdb and tag it with a name that will be used to refer to it in the targetcli
       ```
       /> backstores/block create BD1 /dev/sdb

       Created block storage object BD1 using /dev/sdb.
       /> ls
       o- / ......................................................................................... [...]
         o- backstores .............................................................................. [...]
         | o- block .................................................................. [Storage Objects: 1]
         | | o- BD1 ............................................ [/dev/sdb (2.0GiB) write-thru deactivated]
         | |   o- alua ................................................................... [ALUA Groups: 1]
         | |     o- default_tg_pt_gp ....................................... [ALUA state: Active/optimized]
         | o- fileio ................................................................. [Storage Objects: 0]
         | o- pscsi .................................................................. [Storage Objects: 0]
         | o- ramdisk ................................................................ [Storage Objects: 0]
         o- iscsi ............................................................................ [Targets: 0]
         o- loopback ......................................................................... [Targets: 0]
       />
       ```
       * another way to do the same thing
       ```
       /backstores> cd block
       /backstores/block> create BD1 /dev/sdb
       
       Created block storage object BD1 using /dev/sdb.
       /backstores/block>
       ```
    2. Create the IQN
       * Next we need to create a iSCS Qualified Name (IQN), which is a fully qualified name that will be used to refer to our new backstore block.
       * This IQN is provided to the client when the client scans the server (in discovery mode) for available devices.
       * Create a new IQN with the name ‘iqn.2018-02.net.cb.target:fqdn’. This is done in the iSCSI folder
         ```
         /backstores/block> cd /iscsi
         /iscsi> create iqn.2018-04.net.cb.target:fqdn

         Created target iqn.2018-04.net.cb.target:fqdn.
         Created TPG 1.
         Global pref auto_add_default_portal=true
         Created default portal listening on all IPs (0.0.0.0), port 3260.

         /iscsi> ls /
         o- / ......................................................................................... [...]
           o- backstores .............................................................................. [...]
           | o- block .................................................................. [Storage Objects: 1]
           | | o- BD1 ............................................ [/dev/sdb (2.0GiB) write-thru deactivated]
           | |   o- alua ................................................................... [ALUA Groups: 1]
           | |     o- default_tg_pt_gp ....................................... [ALUA state: Active/optimized]
           | o- fileio ................................................................. [Storage Objects: 0]
           | o- pscsi .................................................................. [Storage Objects: 0]
           | o- ramdisk ................................................................ [Storage Objects: 0]
           o- iscsi ............................................................................ [Targets: 1]
           | o- iqn.2018-04.net.cb.target:fqdn .................................................... [TPGs: 1]
           |   o- tpg1 ............................................................... [no-gen-acls, no-auth]
           |     o- acls .......................................................................... [ACLs: 0]
           |     o- luns .......................................................................... [LUNs: 0]
           |     o- portals .................................................................... [Portals: 1]
           |       o- 0.0.0.0:3260 ..................................................................... [OK]
           o- loopback ......................................................................... [Targets: 0]
         /iscsi>
         ```
         * IQN name conventions:
           * The ‘iqn.’ string followed by
           * YYYY-MM.    (this is usually the year the box’s domain is registered, but can be any month+year)
           * machine’s hostname written in reverse order    (Note: our box’s hostname is ‘target.cb.net’)
           * optional string     (this is to help with making the IQN unique. In our example we gave it the string ‘fqdn’)
         * Target Portal Group: something called a ‘TPG1’ has been created, TPG is something that groups various configurations together.
    3. Create a LUN (Logical Unit Number): This is done inside the luns folder.
       ```
       /iscsi> cd /iscsi/iqn.2018-02.net.cb.target:fqdn/tpg1/luns
       /iscsi/iqn.20...qdn/tpg1/luns>> create /backstores/block/BD1
       Created LUN 0.

      
       /iscsi/iqn.20...qdn/tpg1/luns> ls /
       o- / ......................................................................................... [...]
          o- backstores .............................................................................. [...]
          | o- block .................................................................. [Storage Objects: 1]
          | | o- BD1 .............................................. [/dev/sdb (2.0GiB) write-thru activated]
          | |   o- alua ................................................................... [ALUA Groups: 1]
          | |     o- default_tg_pt_gp ....................................... [ALUA state: Active/optimized]
          | o- fileio ................................................................. [Storage Objects: 0]
          | o- pscsi .................................................................. [Storage Objects: 0]
          | o- ramdisk ................................................................ [Storage Objects: 0]
          o- iscsi ............................................................................ [Targets: 1]
          | o- iqn.2018-02.net.cb.target:fqdn .................................................... [TPGs: 1]
          |   o- tpg1 ............................................................... [no-gen-acls, no-auth]
          |     o- acls .......................................................................... [ACLs: 0]
          |     o- luns .......................................................................... [LUNs: 1]
          |     | o- lun0 ........................................ [block/BD1 (/dev/sdb) (default_tg_pt_gp)]
          |     o- portals .................................................................... [Portals: 1]
          |       o- 0.0.0.0:3260 ..................................................................... [OK]
          o- loopback ......................................................................... [Targets: 0]            
       ```
       * This essentially links our backstore block (BD1) device to our iqn, all grouped under our TPG.
    4. Create name for initiators to use
       * Next we need give our backstore block a unique name that initiators can reference in order to connect to it
       ```
       /> cd /iscsi/iqn.2018-02.net.cb.target:fqdn/tpg1/acls
       /iscsi/iqn.20...qdn/tpg1/acls> create iqn.2018-02.net.cb.target:client
       Created Node ACL for iqn.2018-02.net.cb.target:client
       Created mapped LUN 0.
        
        
       /iscsi/iqn.20...qdn/tpg1/acls> ls /
       o- / ......................................................................................... [...]
          o- backstores .............................................................................. [...]
          | o- block .................................................................. [Storage Objects: 1]
          | | o- BD1 .............................................. [/dev/sdb (2.0GiB) write-thru activated]
          | |   o- alua ................................................................... [ALUA Groups: 1]
          | |     o- default_tg_pt_gp ....................................... [ALUA state: Active/optimized]
          | o- fileio ................................................................. [Storage Objects: 0]
          | o- pscsi .................................................................. [Storage Objects: 0]
          | o- ramdisk ................................................................ [Storage Objects: 0]
          o- iscsi ............................................................................ [Targets: 1]
          | o- iqn.2018-02.net.cb.target:fqdn .................................................... [TPGs: 1]
          |   o- tpg1 ............................................................... [no-gen-acls, no-auth]
          |     o- acls .......................................................................... [ACLs: 1]
          |     | o- iqn.2018-02.net.cb.target:client ..................................... [Mapped LUNs: 1]
          |     |   o- mapped_lun0 ................................................... [lun0 block/BD1 (rw)]
          |     o- luns .......................................................................... [LUNs: 1]
          |     | o- lun0 ........................................ [block/BD1 (/dev/sdb) (default_tg_pt_gp)]
          |     o- portals .................................................................... [Portals: 1]
          |       o- 0.0.0.0:3260 ..................................................................... [OK]
          o- loopback ......................................................................... [Targets: 0]
       ```
       * At this stage, the target is now fully configured to be mounted.
    5. (optional) – set username and password
       ```
       /> cd /iscsi/iqn.2018-02.net.cb.target:fqdn/tpg1/acls/iqn.2018-02.net.cb.target:client/
       /iscsi/iqn.20...target:client> set auth userid=codingbee
       Parameter userid is now 'codingbee'.
        
        
       /iscsi/iqn.20...target:client> set auth password=password
       Parameter userid is now 'password'.
        
        
       /> ls
       o- / ......................................................................................... [...]
          o- backstores .............................................................................. [...]
          | o- block .................................................................. [Storage Objects: 1]
          | | o- BD1 .............................................. [/dev/sdb (2.0GiB) write-thru activated]
          | |   o- alua ................................................................... [ALUA Groups: 1]
          | |     o- default_tg_pt_gp ....................................... [ALUA state: Active/optimized]
          | o- fileio ................................................................. [Storage Objects: 0]
          | o- pscsi .................................................................. [Storage Objects: 0]
          | o- ramdisk ................................................................ [Storage Objects: 0]
          o- iscsi ............................................................................ [Targets: 1]
          | o- iqn.2018-02.net.cb.target:fqdn .................................................... [TPGs: 1]
          |   o- tpg1 ............................................................... [no-gen-acls, no-auth]
          |     o- acls .......................................................................... [ACLs: 1]
          |     | o- iqn.2018-02.net.cb.target:client ..................................... [Mapped LUNs: 1]
          |     |   o- mapped_lun0 ................................................... [lun0 block/BD1 (rw)]
          |     o- luns .......................................................................... [LUNs: 1]
          |     | o- lun0 ........................................ [block/BD1 (/dev/sdb) (default_tg_pt_gp)]
          |     o- portals .................................................................... [Portals: 1]
          |       o- 0.0.0.0:3260 ..................................................................... [OK]
          o- loopback ......................................................................... [Targets: 0]
       />
       ```
       * Now we can exit targetcli
         ```
         /> exit
         Global pref auto_save_on_exit=true
         Last 10 configs saved in /etc/target/backup.
         Configuration saved to /etc/target/saveconfig.json
         ```
* Configure whitelist for the iSCSI pory in firewalld
  ```
  [root@iscsi-target ~]# firewall-cmd --permanent --add-service=iscsi-target
  success
  [root@iscsi-target ~]# systemctl restart firewalld
  ```
  
* Enable the target service
  ```
  [root@iscsi-target ~]# systemctl enable target
  [root@iscsi-target ~]# systemctl start target
  ```
  
## Initiator
* Initiator views the remote storage as a locally attached block device, and therefore treats the remote block device like an ordinary block device
* /etc/iscsi/initiatorname.iscsi
### Setting up an Initiator
* Install initiator software
  ```
  [root@iscsi-initiator ~]# yum isntall -y iscsi-initiator-utils
  ```
  * open-iscsi installation:  zypper -n install open-iscsi
* Configuration
  ```
  [root@iscsi-initiator ~]# cd /etc/iscsi/
  [root@iscsi-initiator iscsi]# ll
  total 20
  -rw-r--r--. 1 root root    50 Feb 26 19:44 initiatorname.iscsi
  -rw-------. 1 root root 12329 Aug  7  2017 iscsid.conf
  
  [root@iscsi-initiator iscsi]# cat /etc/iscsi/initiatorname.iscsi
  InitiatorName=iqn.1994-05.com.redhat:41d9a65147c3

  [root@iscsi-initiator iscsi]# cat /etc/iscsi/initiatorname.iscsi
  InitiatorName=iqn.2018-05.net.codingbee.iscsi-target:clientmynewscsiblockdevice1
  ```
  * if the target is password protected, then you need edit the /etc/iscsi/iscsid.conf, in particular the following section:
  ```
  .
  .
  # *************
  # CHAP Settings
  # *************
    
  # To enable CHAP authentication set node.session.auth.authmethod
  # to CHAP. The default is None.
  node.session.auth.authmethod = CHAP
    
  # To set a CHAP username and password for initiator
  # authentication by the target(s), uncomment the following lines:
  node.session.auth.username = codingbee
  node.session.auth.password = password
  .
  .
  .
  ```
* Start iscsi service
  ```
  [root@iscsi-initiator iscsi]# systemctl enable iscsi
  [root@iscsi-initiator iscsi]# systemctl start iscsi
  ```
  
  * or /etc/init.d/iscsid start
* Scan our target server for any available iSCSI storage, we do this by using the discovery mode
  ```
  [root@iscsi-initiator iscsi]# iscsiadm --mode discovery --type sendtargets --portal 192.168.14.100
  192.168.14.100:3260,1 iqn.2018-02.net.cb.target:fqdn
  ```
  * Here, we specified the target server’s ip address. 
  * And sent a request to the target server to require the target server sends us all available targets.
  * Can confirm status after discovery: iscsiadm -m node -o show

* Login iscsi target
  * Check before connecting iscsi target
    ```
    [root@iscsi-initiator ~]# lsblk --scsi
    NAME HCTL       TYPE VENDOR   MODEL             REV TRAN
    sda  2:0:0:0    disk ATA      VBOX HARDDISK    1.0  sata
    ```
      * only sda is listed here.
 
  * Login
  ```
  iscsiadm --mode node --targetname iqn.2018-02.net.cb.target:fqdn --portal 192.168.14.100 --login
  
  Logging in to [iface: default, target: iqn.2018-02.net.cb.target:fqdn, portal: 192.168.14.100,3260] (multiple)
  Login to [iface: default, target: iqn.2018-02.net.cb.target:fqdn, portal: 192.168.14.100,3260] successful.
  ```

  * Confirmation check
  ```
  [root@initiator iscsi]# lsblk --scsi
  NAME HCTL       TYPE VENDOR   MODEL             REV TRAN
  sda  2:0:0:0    disk ATA      VBOX HARDDISK    1.0  sata
  sdb  3:0:0:0    disk LIO-ORG  BD1              4.0  iscsi
  ```

* Install a filesystem
  ```
    [root@initiator ~]# mkfs.xfs -L iscsi_device /dev/sdb
    meta-data=/dev/sdb               isize=512    agcount=4, agsize=131072 blks
             =                       sectsz=512   attr=2, projid32bit=1
             =                       crc=1        finobt=0, sparse=0
    data     =                       bsize=4096   blocks=524288, imaxpct=25
             =                       sunit=0      swidth=0 blks
    naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
    log      =internal log           bsize=4096   blocks=2560, version=2
             =                       sectsz=512   sunit=0 blks, lazy-count=1
    realtime =none                   extsz=4096   blocks=0, rtextents=0
    
    [root@initiator ~]# blkid | grep sdb
    /dev/sdb: UUID="c00cfb33-2193-4d7d-80d4-f91af3468906" TYPE="xfs"
  ```

* mount
  ```
    [root@initiator iscsi]# mkdir /mnt/remotedisk
    [root@initiator iscsi]# mount /dev/sdb /mnt/remotedisk
    
    [root@initiator iscsi]# lsblk
    NAME            MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
    sda               8:0    0  64G  0 disk
    ├─sda1            8:1    0   1G  0 part /boot
    └─sda2            8:2    0  63G  0 part
      ├─centos-root 253:0    0  41G  0 lvm  /
      ├─centos-swap 253:1    0   2G  0 lvm  [SWAP]
      └─centos-home 253:2    0  20G  0 lvm  /home
    sdb               8:16   0   2G  0 disk /mnt/remotedisk
  ```

* mount persistantly using the /etc/fstab appraoch by UUID
  ```
    [root@initiator ~]# umount /dev/sdb  
  
    [root@initiator ~]# cat /etc/fstab
    
    #
    # /etc/fstab
    # Created by anaconda on Fri Feb  2 05:24:03 2018
    #
    # Accessible filesystems, by reference, are maintained under '/dev/disk'
    # See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
    #
    /dev/mapper/centos-root /                       xfs     defaults        0 0
    UUID=9e33c3c1-98df-4fe1-a66f-13ffbaa5f154 /boot                   xfs     defaults        0 0
    /dev/mapper/centos-home /home                   xfs     defaults        0 0
    /dev/mapper/centos-swap swap                    swap    defaults        0 0
    UUID=c00cfb33-2193-4d7d-80d4-f91af3468906 /mnt/remotedisk    xfs     _netdev        0 0
  ```
  * ‘_netdev’ mount option prevents the boot process hanging if there is any network related problems.

  * Confirmation check
    ```
    [root@initiator ~]# lsblk
    NAME            MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
    sda               8:0    0  64G  0 disk
    ├─sda1            8:1    0   1G  0 part /boot
    └─sda2            8:2    0  63G  0 part
    ├─centos-root 253:0    0  41G  0 lvm  /
    ├─centos-swap 253:1    0   2G  0 lvm  [SWAP]
    └─centos-home 253:2    0  20G  0 lvm  /home
    sdb               8:16   0   2G  0 disk
    
    [root@initiator ~]# mount -a
    
    [root@initiator ~]# lsblk
    NAME            MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
    sda               8:0    0  64G  0 disk
    ├─sda1            8:1    0   1G  0 part /boot
    └─sda2            8:2    0  63G  0 part
    ├─centos-root 253:0    0  41G  0 lvm  /
    ├─centos-swap 253:1    0   2G  0 lvm  [SWAP]
    └─centos-home 253:2    0  20G  0 lvm  /home
    sdb               8:16   0   2G  0 disk /mnt/remotedisk    
    ```

* Get statistics about our iSCSI mount
  ```
    [root@initiator remotedisk]# iscsiadm -m session -P 3
    iSCSI Transport Class version 2.0-870
    version 6.2.0.874-2
    Target: iqn.2018-02.net.cb.target:fqdn (non-flash)
    Current Portal: 192.168.14.100:3260,1
    Persistent Portal: 192.168.14.100:3260,1
    **********
    Interface:
    **********
    Iface Name: default
    Iface Transport: tcp
    Iface Initiatorname: iqn.2018-02.net.cb.target:client
    Iface IPaddress: 192.168.14.101
    Iface HWaddress:
    Iface Netdev:
    SID: 1
    iSCSI Connection State: LOGGED IN
    iSCSI Session State: LOGGED_IN
    Internal iscsid Session State: NO CHANGE
    *********
    Timeouts:
    *********
    Recovery Timeout: 120
    Target Reset Timeout: 30
    LUN Reset Timeout: 30
    Abort Timeout: 15
    *****
    CHAP:
    *****
    username: codingbee
    password: ********
    username_in:
    password_in: ********
    ************************
    Negotiated iSCSI params:
    ************************
    HeaderDigest: None
    DataDigest: None
    MaxRecvDataSegmentLength: 262144
    MaxXmitDataSegmentLength: 262144
    FirstBurstLength: 65536
    MaxBurstLength: 262144
    ImmediateData: Yes
    InitialR2T: Yes
    MaxOutstandingR2T: 1
    ************************
    Attached SCSI devices:
    ************************
    Host Number: 3	State: running
    scsi3 Channel 00 Id 0 Lun: 0
    Attached scsi disk sdb		State: running
  ```
  * Simply information for iscsi sesson: iscsiadm -m session --show
* Stop
  ```
  [root@initiator ~]# umount /dev/sdb
  [root@initiator ~]# systemctl stop iscsi
  ```
* Other useful commands
  * cat /proc/partitions
  * parted: a command-line tool in Linux for disk partitioning and partition resizing.
    ```
    parted --script /device \
    mklabel gpt \
    mkpart primary 1MiB 100MiB \
    mkpart primary 100MiB 200MiB \
    mkpart primary 0% 100%
    
    (上面在/dev/sdb上创建文件系统之前可以先做分区 parted)
    
    parted命令详解
    
    用法：parted [选项]... [设备 [命令 [参数]...]...]
    
    将带有“参数”的命令应用于“设备”。如果没有给出“命令”，则以交互模式运行.  
    帮助选项：
    -h, --help                    显示此求助信息
    -l, --list                    列出所有设别的分区信息
    -i, --interactive             在必要时，提示用户
    -s, --script                  从不提示用户
    -v, --version                 显示版本
    操作命令：   www.2cto.com  
    检查 MINOR                           #对文件系统进行一个简单的检查
    cp [FROM-DEVICE] FROM-MINOR TO-MINOR #将文件系统复制到另一个分区
    help [COMMAND]                       #打印通用求助信息，或关于 COMMAND 的信息
    mklabel 标签类型                      #创建新的磁盘标签 (分区表)
    mkfs MINOR 文件系统类型               #在 MINOR 创建类型为“文件系统类型”的文件系统
    mkpart 分区类型 [文件系统类型] 起始点 终止点    #创建一个分区
    mkpartfs 分区类型 文件系统类型 起始点 终止点    #创建一个带有文件系统的分区
    move MINOR 起始点 终止点              #移动编号为 MINOR 的分区
    name MINOR 名称                      #将编号为 MINOR 的分区命名为“名称”
    print [MINOR]                        #打印分区表，或者分区
    quit                                 #退出程序
    rescue 起始点 终止点                  #挽救临近“起始点”、“终止点”的遗失的分区
    resize MINOR 起始点 终止点            #改变位于编号为 MINOR 的分区中文件系统的大小
    rm MINOR                             #删除编号为 MINOR 的分区
    select 设备                          #选择要编辑的设备
    set MINOR 标志 状态                   #改变编号为 MINOR 的分区的标志
    
    
    修改标签: e2label /dev/sdb <label>
    查看标签:
           1. dumpe2fs -h /dev/sdb
           2. blkid /dev/sdb
              /dev/sdb: LABEL="CentOS_6.9_Final" TYPE="iso9660"
    按磁盘标签查询：blkid -t "LABEL=CentOS_6.9_Final" -o device
    查看某个path是否已经mount, 如果没有mount就mount它到/dev/sdb: mountpoint -q <path> || mount /dev/sdb <path> 
    用path做umount: umount <path> 
    ```
  * useful iscsiadm usage
    ```
    iscsiadm -m discovery -t st -p IP:port                  // 发现iscsi target
    iscsiadm -m node -o delete -T TARGET(IQN) -p IP:port    // 删除发现的iscsi target记录
    iscsiadm -m node                                        // 查看iscsi target的发现记录
    iscsiadm -m session                                     // 发现iscsi connect session情况
    iscsiadm -m node -T TARGET(IQN) -p OP:port -l           // 登录iscsi target，创建session
    iscsiadm -m node -T TARGET(IQN) -p OP:port -u           // 登出iscsi target   
    ```