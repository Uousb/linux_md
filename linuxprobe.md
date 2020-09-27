[TOC]

### 重定向

| 命令 < file    | 输入重定向，将文件作为标准的输入             |
| -------------- | -------------------------------------------- |
| 命令 > file    | 标准输出重定向，清空原有文件数据             |
| 命令 2> file   | 错误输出重定向，清空原有文件数据             |
| 命令 >> file   | 标准输出重定向，追加原有文件内容后面         |
| 命令 2>> file  | 错误输出重定向，追加原有文件内容后面         |
| 命令 2>&1 file | 标准输出与错误输出共同追加到原有文件内容后面 |

#### tee 多重重定向



### 接受输入的参数

| $?         | 返回上一条执行状态                                           |
| ---------- | ------------------------------------------------------------ |
| $0         | 返回脚本名称                                                 |
| $#         | 统计脚本参数个数                                             |
| $*         | 是以一个单字符串显示所有向脚本传递的参数，与位置变量不同，参数可超过``9``个 |
| $@         | 传递给脚本的所有参数列表                                     |
| $$         | 脚本当前运行的进程ID                                         |
| $1,$2..... | 传递给脚本的第一个，第二个参数。。。                         |

#### read 接受用户传入的值

`read -p "请输入值" VAR1`

### 判断

#### 1、逻辑判断

| &&   | -a   | 与：前面执行成功才执行后面 |
| ---- | ---- | -------------------------- |
| \|\| | -o   | 或：前面执行失败才执行后面 |
| !    | !    | 非：判断值取反             |



#### 2、字符串比较

| =    | 判断字符串是否相等   |
| :--- | -------------------- |
| ！=  | 判断是字符串是否不等 |
| -z   | 判断字符串是否为空   |



例：判断当前是否为root用户

```shell
[ $USER != root ] && echo "user" ||echo "root"
```



#### 3、文件判断

测试语句格式：`[ 条件表达式 ]` 表达式前后有一个空格

| -d   | 测试文件是否为目录             |
| ---- | ------------------------------ |
| -f   | 测试是否为**一般文件**         |
| -e   | 测试文件是否**存在**           |
| -r   | 测试当前用户是否有**读取**权限 |
| -w   | 测试当前用户是否有**写入**权限 |
| -x   | 测试当前用户是否有**执行**权限 |



#### 4、整数比较

| -eq  | 等于     |
| ---- | -------- |
| -ne  | 不等于   |
| -gt  | 大于     |
| -lt  | 小于     |
| -le  | 小于等于 |
| -ge  | 大于等于 |

例：判断内存是否小于等于1024MB

```shell
FreeMem=`free -m | grep Mem | awk '{print $4}'`
[ $FreeMem -lt 1024 ] && echo "Insufficient Memory"
```



### 流程控制

#### 1、if 

```shell
if 条件测试1
	then 命令序列1
elif 条件测试2
	then 命令序列2
else
	命令序列3
fi

```



#### 2、for

```shell
for 变量 in 取值列表
do
	命令序列
done
```

例：批量创建用户

```shell
#!/bin/bash
read -p "enter account default password:" PASS_WD

for $USER_CODE in `cat user.txt`
do
    id $USER_CODE &>/dev/null
    if [ $? -qe 0 ];then
        echo "account existing"
    else
        echo "createing $USER_CODE ... "
        useradd -s -M /sbin/nologin &>/dev/null
        echo "$PASS_WD"|passwd --stdin $USER_CODE
        if [ $? -qe 0 ];then
            echo "create success"
        else
            echo "create failure"
        fi
	fi
done
```

#### 3、while

```shell
while 条件测试操作
do
	命令序列
done
```

例:

```shell
#!/bin/bash
while true
do
        read -p '请输入成绩,或者按Q退出程序: ' ach
        if [ $ach = Q ];then
                exit 0
        else
                if [ $ach -ge 1 ] && [ $ach -le 100 ];then
                if [ $ach -ge 80 ] && [ $ach -le 100 ];then
                        echo "score:$ach very good"
                elif [ $ach -ge 60 ] && [ $ach -le 80 ];then
                        echo "acore:$ach is good"
                else
                        echo "score:$ach"
                fi
                else
                        echo 'score error!'
                fi
        fi
done
```



#### 4、case

```shell
case 变量值 in 
模式1 )
	命令序列1
	;;
模式2 )
	命令序列2
	;;
......
* )
	默认命令序列
esac
```



### 计划任务服务程序

#### 1、单次任务 `at`

| at 23：00 | 设置任务时间 |
| --------- | ------------ |
| at -l     | 查看任务     |
| atrm      | 删除任务     |

```shell
echo "systemctl restart httpd" | at 23:00   ##设置单次任务，晚上23：00重启httpd服务
```



#### 2、定时任务`crontab`

###### ![](/home/lion/Pictures/2020-08-11 15-33-38屏幕截图.png)

| crontab -l         | 查看定时任务                       |
| ------------------ | ---------------------------------- |
| crontab -e         | 编辑定时任务                       |
| crontab -r         | 删除定时任务                       |
| crontab -e -u user | **`root用户编辑其他用户定时任务`** |

例：每周一、三、五凌晨3：25使用tar命令备份wwwroot目录

```shell
crontab -l
25 3 * * 1,3,5 /usr/bin/tar -czvf backup.tar.gz /home/wwwroot
```

> + crond服务中命令参数要使用绝对路径，使用`whereis`查询
> + 任务中包含多条命令语句，应该每行仅填写一条
> + 可以使用 `#`开头进行注释
> + `分`字段必须有数值，不能为`空`或者`*`，`日`和`星期`不能同时使用，会冲突

（ ， ） 表示多个时间段（8,9,12”表示8月、9月和12月）

（ - ）表示连续周期（例如字段“日”的取值为“12-15”，则表示每月的12～15日）

（ / ）表示执行任务的间隔时间（例如“*/2”表示每隔2分钟执行一次任务）





### 用户及权限管理

#### 用户身份

> 用户UID为0：系统超级管理员
>
> 用户UID为1~999：为系统用户
>
> 用户UID为1000+：为普通用户

**用户信息**

```shell
/etc/passwd
/etc/shadow
```

**组信息位置**

```shell
/etc/group
/etc/gshadow
```

`/etc/passwd`用户信息文件分析（每项用`:`隔开）

```shell
例如：jack:X:503:504:::/home/jack/:/bin/bash
jack　　	# 用户名
X　	　	# 口令、密码
503　　	# 用户id（0代表root、普通新建用户从500开始）
504　　	# 所在组
:　　		# 描述
/home/jack/　　	#用户主目录
/bin/bash　　		#用户Shell
```

`/etc/shadow`组信息文件分析

```shell
例如：lx:$?$:18411:0:99999:7:::
lx　　	# 用户名
$!$　　	# 被加密的口令
18411　　	# 密码最后一次修改时间，计算`date -d "1970-01-01 13801days"`
0　　		# 最小修改时间间隔,0代表随时可以修改
99999　　	# 密码有效期，99999代表永久
7　　		# 到7天时提醒
*　　		# 禁用天数
*　　		# 过期天数
*		 #保留字段
```



**useradd**

| -d   | 指定用户家目录                       |
| ---- | ------------------------------------ |
| -e   | 账户到期时间，YYYY-MM-DD             |
| -u   | 指定账户UID                          |
| -g   | 指定一个初始的用户基本组(必须已存在) |
| -G   | 指定一个或多个扩展组                 |
| -N   | 不创建与用户名相同的的基本用户组     |
| -s   | 指定用户的shell                      |

**groupadd**

**usermod**

| -c    | 修改用户账号的备注                           |
| ----- | -------------------------------------------- |
| -d -m | 重新指定用户家目录，并把源目录下文件迁移过去 |
| -e    | 修改账号到期时间，（YYYY-MM-DD）             |
| -g    | 变更所属用户组                               |
| -aG   | 变更用户扩展组                               |
| -l    | 变更用户名称，（usermod -l newuser olduser） |
| -L    | 锁定                                         |
| -U    | 解锁账号                                     |
| -s    | 变更shell                                    |
| -u    | 修改用户UID                                  |
| -a    | 把用户追加到扩展组中，仅与-G一起使用         |

> 在使用-G进行扩展组变更时，会覆盖原有扩展组，-aG进行单纯追加扩展组

**passwd**

| -l      | 锁定用户，禁止登录                                           |
| ------- | ------------------------------------------------------------ |
| -u      | 解锁用户，允许登录                                           |
| -d      | 删除密码，允许使用空密码登录                                 |
| -e      | 强制用户下次登录时修改密码                                   |
| -S      | 显示密码相关信息，是否锁定、加密算法名称                     |
| --stdin | 允许通过标准输入修改用户密码`echo "password"|passwd--stdin user` |

**gpasswd**

```shell
gpasswd -d user group #从group组中删除user用户
```



**userdel**

| -f   | 强制删除                     |
| ---- | ---------------------------- |
| -r   | 同时删除用户账号及家目录数据 |



#### 默认权限umask

> 设定linux新建文件时候默认文件、文件夹权限
>
> 文件默认最大权限为666，文件夹最大权限为777

计算方式`初始权限=文件(目录)最大权限 - umask权限`

```shell
[lx@db01 Desktop]$ umask
0002
```



#### 文件权限与归属

**文件类型**

| -    | 普通文件     |
| ---- | ------------ |
| d    | 目录文件     |
| l    | 链接文件     |
| b    | 块设备文件   |
| c    | 字符设备文件 |
| p    | 管道文件     |

**文件夹权限**

| r    | 能够读取目录内的文件列表           |
| ---- | ---------------------------------- |
| w    | 能够在目录内新增、删除、重命名文件 |
| x    | 能够进入该目录                     |



#### 文件的特殊权限

**SUID**

> **用户在运行某个程序时，如果该程序有SUID权限，则程序运行为进程时，进程的属主不是发起者，而是程序文件所属的属主**
>
> 1. 一般指针对可执行二进制文件设置SUID，非二进制文件设置没有意义
> 2. 命令执行者（非属主）在执行过程中获得文件的**属主**权限
> 3. 获得**属主**权限的行为只发生在执行过程中

```shell
chmod u+s a.txt
----------.  1 root root    95 Aug 13 14:41 a.sh
```



**SGID**

> **参考SUID设计，不同在于程序运行为进程时，用户获取的不再是文件的所有者的临时权限，而且获取到文件所属组的权限**
>
> 1. 让执行者临时拥有所属组的权限（对拥有执行权限的二进制程序进行设置）
> 2. 在某个目录中创建的文件自动继承该目录的用户组（只可以对目录进行设置）

```shell
chmod -R 2700 00
drwxrwsrwx. 2 barr barry    43 Aug 13g 15:38 00
```



**SBIT**

> **仅对目录有效，目录设置SBIT权限后，目录内文件仅有属主与root才能删除文件，其他用户无法删除**
>
> 1. 仅对目录有效
> 2. 仅对other设置

```shell
chmod o+t 00/
drwxrwsrwt. 2 barr barry    43 Aug 13g 15:38 00
```



#### 文件隐藏属性

**chattr**

> 给文件添加隐藏属性

| i    | 无法对文件进行修改；若对目录设置，则只能修改其中的子文件内容而不能新建或删除文件 |
| ---- | ------------------------------------------------------------ |
| a    | 仅能补充（追加）内容，无法覆盖、删除内容                     |
| S    | 文件内容变更后立即同步到硬盘                                 |
| s    | 彻底从硬盘删除，不可恢复（使用0填充源文件所在硬盘区域）      |
| A    | 不可修改文件或者目录最后访问时间（atime）                    |
| b    | 不可修改文件或者目录的读取时间                               |
| D    | 检查压缩文件中的错误                                         |
| d    | 使用dump命令备份时忽略本文件、目录                           |
| c    | 默认将文件或者目录进行压缩                                   |
| u    | 当删除文件后依然保留在硬盘中的数据，方便恢复                 |
| t    | 让文件系统支持尾部合并（tail-merging）                       |
| x    | 可以直接访问压缩文件中的内容                                 |

```shell
[root@db01 Desktop]# chattr +a a.txt
[root@db01 Desktop]# rm a.txt
rm: cannot remove 'a.txt': Operation not permitted
```



**lsattr**

> 查看文件隐藏属性

```shell
[root@db01 Desktop]# lsattr a.txt
-----a-------------- a.txt
```



#### 文件访问控制列表(ALC)

**setfacl**

> 对某个指定的用户进行单独的权限控制，可对用户组（g）进行访问控制操作

| -m   | 设定用户或组ACL权限                                 | setfacl -m u:user:rwx /root    |
| ---- | --------------------------------------------------- | ------------------------------ |
| -x   | 删除指定用户的ACL权限                               | setfacl -x u :user /root       |
| -b   | 删除所有ACL权限                                     | setfacl -b /root               |
| -d   | 设定默认ACL权限，只对目录生效，新建立文件都有此权限 | setfacl -m d:u:user:rwx /root  |
| -R   | 递归设定ACL权限，所有已存在都文件都会继承父目录权限 | setfacl -m u:user:rwx -R /root |
| -k   | 删除默认ACL权限                                     | setfacl -k /root               |

```shell
setfacl -Rm u:lx:rwx /root 	###设定lx用户对/root目录递归访问权限
setfacl -Rm g:lx:rwx /root 	###设定lx组对/root目录递归访问权限
setfacl-m d:u:lx:rx /root 	###设定lx用户对/root目录下新建文件默认rx权限
```



**getfacl**

> 查看文件访问控制列表ACL

```shell
getfacl /root
```



#### su与sudo服务

**1、su**

> 切换账号

| -    | 切换账号后，使用切换后账号环境变量登录系统个，不使用则原账号环境变量 |
| ---- | ------------------------------------------------------------ |
| -l   | 与 - 类似                                                    |
| -m   | 使用切换前账号环境变量登录系统                               |



**2、sudo**

> 使用其他用户身份执行指令（通常为root账号）
>
> 配置文件**/etc/sudoers**，使用**visudo**进行配置

| -u   | 指定用户身份执行命令   |
| ---- | ---------------------- |
| -b   | 将指令放到后台执行     |
| -k   | 立即刷新sudo密码有效期 |

`使用者账号		登陆者来源主机名 =(可切换身份)			可执行指令`

```shell
root	ALL=(ALL) 	ALL		##用户
%wheel	ALL=(ALL)	ALL		##加上%带代表组

lx	ALL=(root)	NOPASSWD: ALL ##用户lx可以sudo使用全部指令并且免密码认真
```

| 使用者账号       | 规定哪些账号或者组可以使用sudo                             |
| ---------------- | ---------------------------------------------------------- |
| 登陆者来源主机名 | 指定信任客户端计算机连接，默认全部信任                     |
| 可切换身份       | 指定可切换成什么身份来执行后续指令，默认root可切换为任何人 |
| 可执行命令       | 可用该身份执行或不允许执什么指令，须填写绝对路径           |

`ALL是特别关键词，代表任何身份、主机或者指令`

**sudo 有限制的指令操作**

> 在指令设定值前面加上`！`，代表不可执行

```shell
[root@db01 ~]# visudo
lx      ALL=(ALL)       !/usr/bin/passwd, /usr/bin/passwd [A-Za-z]*, /usr/bin/passwd root
```

`允许lx使用sudo命令，允许使用passwd 任意字符，但是passwd 与passwd root 不允许使用`

**使用别名批量管理sudo**

指令别名：Cmnd_Alias

账户别名：User_Alias

主机别名：Host_Alias

```shell
[root@db01 ~] # visudo
User_alias ADMPW = user1, user2, user3, myuser1, myuser2
Cmnd_Alias ADMPCOM = !/usr/bin/passwd, /usr/bin/passwd [A-Za-z]*, !/usr/bin/passwd root
ADMPw	ALL=(root)	ADMPCOM
```

> 通过User_Alias建立一个账号别名ADMPW，通过Cmnd_Alias建立命令别名ADMPCOM

使用sudo登录切换为root

```shell
[root@db01 ~] # visudo
lx	ALL=(root)	/usr/bin/su -
[lx@db01 ~]$ sudo su -
[root@db01 ~] whoami
root
[lx@db01 ~]$ sudo -i
[root@db01 ~] whoami
root
```





### 存储结构与磁盘划分



MBR（主引导记录）下，最多支持4个主分区，超过4个分区需要将第一个扇区中的16字节的空间指向另外一个分区（扩展分区）；

主分区或扩展分区的编号从1开始，到4结束。逻辑分区从编号5开始

<img src="/home/lion/Document/linuxprobe.assets/2020-08-16 14-09-38屏幕截图.png" style="zoom:67%;" />

GPT（全局唯一标识磁盘分区表）分区是一种新的磁盘管理方式，使用UEFI启动。解决了MBR只能四个分区的缺点。理论上对磁盘分区数量没有限制。

文件系统

| Ext3 | 日志文件系统，能在异常宕机时避免文件丢失并自动修复           |
| ---- | ------------------------------------------------------------ |
| Ext4 | Ext3改进版，最大支持1EB，无限多子目录，支持批量分配block块，提高读写效率 |
| XFS  | 高性能日志文件系统，最大支持18EB。RHEL7后默认文件管理系统    |

虚拟文件接口VFS（Virtual File System）

为了方便用户在众多文件系统中进行统一操作而不用去关系底层硬盘结构，linux内核中软件层为用户提供了VFS接口，用户在对文件进行操作时统一对虚拟文件系统进行操作。

实际文件系统在VFS下隐藏了自己的特性和细节，这样用户在日常使用时觉得文件系统没有区别。

<img src="/home/lion/Document/linuxprobe.assets/2020-08-18 16-10-12屏幕截图.png" style="zoom:67%;" />

**inode（索引节点）**

每个存储设备或者存储设备的分区被格式化为文件系统后有分为两个部分，一部分为inode区，另一部分为block区。通过inode对每个文件进行信息索引，操作系统根据指令能够通过inode值最快的找到相对于的文件。

**superblock**记录此filesystem的整体信息，包括inode/block总量、使用量、剩余量，以及文件系统的格式与相关信息等

**inode**用来记录文件的属性，包括文件大小、属主、用户组、读写权限等同时记录此文件的数据所在的block号码。一个文件占用一个inode

**block**用来存储具体的数据，若文件过大过长，则占用多个block。

特殊作用：

1. 如果文件包含特殊字符无法正常删除，可通过直接删除inode节点号来达到删除文件目的
2. 打开文件或者重新命名文件，只改变文件名，不影响inode号
3. 打开一个文件后，系统以inode号来识别文件，不考虑文件名

**硬链接**

```shell
ln 源文件 目标文件
```

多个inode指向一个文件，可以使用不同文件名访问同一个文件，对文件修改会影响到所有文件名。删除一个文件名不影响其他文件名的访问

**软连接**

```shell
ln -s 源文件 目标文件
```

源文件和目标文件的inode不同，无法伦达源文件还是目标文件最终都是访问源文件。如果删除源文件。目标文件将无法访问。类似win下快捷方式

**lsblk**

> 列出系统上的所有磁盘列表

```shell
lsblk -ip ##列出素有磁盘列表完整文件名
```

**blkid**

> 列出装置的UUID等参数

```shell
[root@db01 ~]# blkid /dev/sda1
/dev/sda1: UUID="48f10bce-b08e-4b02-a813-dc8b9293aacb" TYPE="xfs" PARTLABEL="Linux filesystem" PARTUUID="7400e20c-2616-4407-9402-ba7fc328c88d"
```





#### 挂载设备

当需要使用硬盘设备或者分区中的数据时，需要先将其与一个已经存在的文件系统进行关联。

```shell
mount /dev/sdb2 /backup	###将硬盘sdb2挂载到/backup目录下
```

| -t   | 指定文件系统的类型                   |
| ---- | ------------------------------------ |
| -a   | 挂载所有在/etc/fstab中定义的文件系统 |

> 挂载目录如果不为空，挂载后文件将不可见，需要卸载挂载后才可见

**/etc/fatab**	系统启动时自动挂载

```shell
设备文件	挂载目录	格式类型	文件系统参数	是否备份(dump)	是否自检(fsck)
UUID=08ef4047-d8f0-4f3b-a37f-9021d82e99f0	/	xfs	defaults	0	0
/dev/sdb2	/backup	ext4	defaults	0	0
```

| 设备文件 | 一般为设备路径+设备名称或者label name也可是唯一识别码UUID（建议使用） |
| -------- | ------------------------------------------------------------ |
| 挂载目录 | 指定要挂载到的目录，需要挂载钱创建好                         |
| 格式类型 | 指定文件系统格式，如Ext3，Ext4，XFS，SWAP，ios9660（光盘）   |
| 权限选项 | 若设置为Defaults，默认权限为rw,suid,dev.exec,auto,nouser,async |
| 是否备份 | 若为1则开机后自动dump进行磁盘备份，为0则不备份g              |
| 是否自检 | 若为1则开机后进行磁盘检查，为0则不检查                       |

1. **当进入单人维护模式是，根目录通常为只读，需要重新挂载为rw权限**

```shell
mount -o remount,rw,auto /	##重新挂载 / 目录，并将加入rw，auto参数
```

2. 硬链接不能连接目录，某些程序对软连接不友好，可以将目录挂载 mount --bind（并非挂载文件系统）

   ```shell
   mount --bind /var /mnt/var	##将var目录挂载至/mnt/var目录
   ```

3. 挂载loop文件（iso光盘镜像文件），linux中直接挂载iso文件，读取内容

   ```shell
   mount -o loop /tmp/centos7.iso /mnt/centos7		#挂载镜像文件
   ```

   

撤销挂载**umount**

| -f   | 强制卸载                  |
| ---- | ------------------------- |
| -l   | 立刻卸载文件系统          |
| -n   | 不更新/etc/mtab情况下卸载 |

```shell
umount /dev/sdb2	###卸载已经挂载的/dev/sdb2设备文件
```



#### 磁盘分区

使用**parted**查看磁盘分区类型；MBR分区表中使用**fdisk**，GPT分区表中使用**gdisk**

**fdisk**

```shell
fdisk /dev/sda	##使用fdisk工具对sda硬盘设备进行分区操作
```

| m    | 查看全部可用命令       |
| ---- | ---------------------- |
| n    | 添加新分区             |
| d    | 删除某个分区           |
| l    | 列出所有可用的分区类型 |
| t    | 改变某个分区类型       |
| p    | 查看分区表信息         |
| w    | 保存并退出             |
| q    | 不保存直接退出         |

> 在创建好分区后，可能遇到系统没有把分区信息同步给内核的情况，需要使用 `partprobe -s `命令来手动同步

**gdisk**

```shell
gdisk /dev/sda	##使用gdisk工具对sda硬盘设备进行分区操作
```

基本操作与fdisk命令类似

**mkfs** 磁盘格式化

格式化命令，使用具体mkfs.**命令来格式化为指定的文件格式

> mkfs         mkfs.cramfs  mkfs.ext2    mkfs.ext3    mkfs.ext4    mkfs.fat     mkfs.minix   mkfs.msdos   mkfs.vfat    mkfs.xfs

```shell
mkfs.xfs /dev/sda1	##格式化/dev/sda1分区为xfs文件系统
```

**xfs可以使用多个数据流来读写系统，以增加速度，因此agcount可以和cpu的核心数来做搭配。**

如：cpu有四个物理核心，但是超线程后有8个逻辑核心。可以将agcount设定为8

```shell
mkfs.xfs -f -d agcount=8 /dev/sda1	##强制将sda1系统格式化，并将agcont设置为8
```



**添加交换分区SWAP**，1划分独立分区；2建立虚拟文件

+ **划分独立分区**

1. 使用分区工具，分区出空间

   ```shell
   fdisk /dev/sda	##在sda磁盘上新建分区sda2
   ```

2. 建立和设置SWAP分区

   ```shell
   mkswap /dev/sda2	##使用mkswap格式化建立swap分区
   ```

3. 激活交换分区

   ```shell
   swapon /dev/sda2	##激活swap分区，使用`free -m` 或者`swapon -s`查看swap情况
   ```

4. 设置`/etc/fstab`，使其开机自动挂载

   ```shell
   vim /etc/fstab	##写入fstab
   /dev/sda2	swap	swap	defaults	0	0
   ```




+ **建立虚拟文件方式**

  1. 使用dd命令新增文件

     ```shell
     dd if=/dev/zere of=/swapfile bs=1M count=512	#建立一个512M的空文件
     ```

  2. 使用mkswap格式化为swap文件系统

     ```shell
     mkswap /swapfile
     ```

  3. 激活swap分区

     ```shell
     swapon /swapfile
     ```

  4. 使用`swapon -s`查看swap，写入`/etc/fstab`

     ```shell
     vim /etc/fstab	##写入fstab
     /dev/sda2	swap	swap	defaults	0	0
     ```

     



#### 磁盘空间配额quota

1. 群组限制：限制某一群组所能使用的最大磁盘配额
2. 用户限制：限制某一用户的最大磁盘配额
3. 限制某一目录的磁盘配额

软限制：当容量到达软限制时会提示用户，但仍然允许用户在限定的额度内继续使用

硬限制：当容量达到应限制时会提示用户，且强制终止用户的操作

在`/etc/fstab`中设置文件系统启用`quota` 

```shell
/dev/sda2 /home                   xfs     defaults,usrquota,grpquota        0 0
```

针对quota限制主要有三项：

1. `uquota`/`usrquota`/`quota`	--针对使用者账号设定
2. `gquota`/`grpquota`              --针对群组的设定
3. `pquota`/`prjquota`               --针对单一目录设定，**但不可以与grpquota同时存在**



**针对xfs格式的文件系统：**

```shell
xfs_quota -x -c "指令" [挂载点]
```

-x 专家模式，后续才可以加入-c的指令参数

-c 后面接指令

| print  | 列出主机内文件系统参数等                                     |
| ------ | ------------------------------------------------------------ |
| df -h  | 与原本df一样，可加上-b（block）、-i（inode）、-h（加上单位） |
| report | 列出目前quota项目,有-ugr（user、group、project）及-bi（block、inode）等参数可选 |
| state  | 说明目前支持quota的文件系统的信息，有没有启动相关情况        |



**xfs_quota设定对user，group进行限制**

| limit | 实际限制的项目，可对user、group及project进行限制（u、g、p）  |
| ----- | ------------------------------------------------------------ |
|       | bsoft/bhard为block的soft、hard限制值，可以加单位M/G          |
|       | isoft/ihard为inode的soft、hard限制值                         |
|       | name为用户名、群组名                                         |
| timer | 用来设定grace time，也可以针对user、group以及block、inode设定 |

```shell
xfs_quota -x -c "limit -u[g] b[i]soft=M b[i]hard=M user[group]" /挂载点
xfs_quota -x -c "limit -u bsoft=100M bhard=200M user_name" /home
xfs_quota -x -c "limit -g bsof=200M bhard=1G group_name" /home
```

**xfs_quota设定宽限期（grace time）**

```shell
xfs_quota -x -c "timer -u -b 14days"	##设定user grace time 为14天
xfs_quota -x -c "timer -g -b 15d"		##设定group grace time 为15天
```

**xfs_quota设定对project（目录）进行限制**

1. 修改`/etc/fstab`支持`prjquota`参数

   ```shell
   /dev/sda3 /home                   xfs     defaults,uquota,pquota	0 0
   ```

2. 设定`/etc/projects`与`/etc/projid`

   ```shell
   echo "11:/home/myquota" >> /etc/projects	##将项目标识符与目录写入`/etc/projects`
   echo "11:myquotaject" >> /etc/projid	##将项目名称与标识符写入`/etc/projid`
   ```

   > / etc / projects：将数字项目标识符映射到目录树。
   > / etc / projid：数字项目标识符到项目名称的映射。

   

3. 使用xfs_quota，初始化项目名称

   ```shell
   xfs_quota -x -c "project -s myquotaject"
   ```

   > ​	使用`xfs_quota -x -c "print" /home`查看项目目录是否开启quota

4. 设定xfs_quota，对project（目录）进行限制

   ````shell
   xfs_quota -x -c "limit -p bsoft=400M bhard=500M myquotaject" /home
   ````



**编辑quota配额限制**

| -u   | 指定修改用户quota限制   |
| ---- | ----------------------- |
| -g   | 指定修改用户组quota限制 |



**xfs_quota 管理额外指令对照表**

| disable | 暂时取消quota显示，可对u、g、p一起或单独使用                 |
| ------- | ------------------------------------------------------------ |
| enable  | 启用quota功能                                                |
| off     | 完全关闭quota功能，无法使用enable再次启用，除非执行**remore**操作 |
| remore  | 移除所有quota限制，quota必须在off状态下才可以执行，可以对u、g、p一起或单独使用 |

```shell
xfs_quota -x -c "disable -up" /home #暂停user、project的quota功能
xfs_quota -x -c "off -p" /home	#关闭quota功能，同时取消project限制
xfs_quota -x -c "remove -u" /home	#移除user的quota限制
```



### RAID和LVM

#### RAID

通过软件实现RAID，分为RAID0、RAID1、RAID5、RAID6、RAID10等

mdadm 用于查看创建、管理和查看raid

> 任何不以"-"开头的参数均视为设备名称

| 创建模式                          | 管理模式                | 查看                           |
| --------------------------------- | ----------------------- | ------------------------------ |
| -C ：创建RAID                     | -f ：标记指定磁盘为损坏 | -D：查看raid详细信息           |
| -l ：指定raid级别                 | -a ：添加磁盘           | cat /proc/mdstat :查看raid状态 |
| -a ：{yes\|no} 自动创建RAID设备   | -r ：移除磁盘           |                                |
| -c ：CHUNK_SIZE 指定块大小，单位k |                         |                                |
| -x ：指定备份盘数量               |                         |                                |

建立RAID10

```shell
mdadm -Cv /dev/md0 -a yes -n 4 -l 10 /dev/sd{a,b,c,d}
	##	-C创建RAID;v显示过程;/dev/md0创建的RAID设备名;-a yes自动创建设备文件;-n 4使用4块硬盘;-l 10使用RAID10 方案;/dev/sda 为硬盘设备名称
```

建立RAID5，并使用一个备份盘

```shell
mdadm -Cv /dev/md0 -a yes -n 3 -l 5 -x 1 /dev/sd{a,b,c,d}
	##	-x 1使用一块磁盘作为备份盘，当raid中有磁盘出现故障，备份盘自动替换上
```

添加一块磁盘进入RAID与删除RAID中的一块磁盘

```shell
mdadm /dev/md0 -a /dev/sde	#将/dev/sde添加到raid中
mdadm /dev/md0 -r /dev/sdc	#将raid中/dev/sde磁盘移除
```

```shell
mdadm -S /dev/md0	#停止设备
mdadm -A -s /dev/md0	#激活设备
mdadm -R /dev/md0	#强制启动
mdadm --zero-superblock /dev/sda	#删除raid信息
```



#### LVM

> LVM是硬盘分区和文件系统之间添加一个逻辑层，提供了一个抽象的卷组，可以把多块硬盘进行卷组合并。用户不必关心武力硬盘设备的底层架构和布局，既可以实现对硬盘分区的动态调整。

![img](/home/lion/Document/linuxprobe.assets/逻辑卷.png)

物理存储介质：LVM存储介质，可以是硬盘分区、整个硬盘、RAID等。必须初始化为LVM物理卷才能与LVM结合使用

物理块PE：基本单元，物理卷PV中可以分配的最小存储单元，大小可以指定，默认为4MB

物理卷PV：LVM的基本存储逻辑快，与`物理存储介质`相比包含LVM相关管理参数，常见物理卷可是硬盘分区也可是整个硬盘

卷组VG：VG建立在物理卷之上，一个卷组可以包含多个物理卷，而且卷组创建之后也可以继续向其中添加新的物理卷

逻辑卷LV：LV使用VG中空闲的资源建立的，并且逻辑卷在建立之后可以动态的扩展或缩小空间。

**LVM常用命令**

| 功能/命令           | 物理卷管理PV | 卷组管理VG  | 逻辑卷管理LV         | 文件系统xfs、ext4        |
| ------------------- | ------------ | ----------- | -------------------- | ------------------------ |
| 扫描 scan           | pvs、pvscan  | vgs、vgscan | lvs、lvscan          | lsblk、blkid             |
| 创建 create         | pvcreate     | vgcreate    | lvcreate             | mkfs.xfs、mkfs.ext4      |
| 显示 display        | pvdisplay    | vgdisplay   | lvdisplay            | df、mount                |
| 扩展 extend         |              | vgextend    | lvextend（lvresize） | xfs_growfs、rezise2fs    |
| 减小 reduce         |              | vgreduce    | lvreduce（lvresize） | xfs不支持减小、resize2fs |
| 删除 remore         | pvremove     | vgremove    | lvremove             | umount、重新格式化       |
| 改变容量 resize     |              |             | lvreize              | xfs_growfs、resize2fs    |
| 改变属性 att rebute | pvchange     | vgchange    | lvchange             | /etc/fstab、remount      |

1. 新建PV（添加分区到）

   ```shell
   pvcreate /dev/sd{a,b,c,d}	#将sda~sdd设置为lvm物理卷
   ```

   + 删除PV

     ```shell
     pvremore /dev/sda	#将物理卷中/dev/sda删除
     ```

     

2. 新建VG

   ```shell
   vgcreate storage /dev/sd[a,b,c,d]	#将物理卷中的sda,sdb,sdc,sdd建立为storage卷组
   ```

   + 扩展卷组

     ```shell
     vgextend storage /dev/sdc	#将/dev/sdc添加至卷组storage中
     ```

   + 缩小卷组

     ```shell
     vgreduce storage /dev/sdb	#将/dev/sdb从storage中移除
     ```

     

3. 新建LVM

   ```shell
   lvcreate -n vo -l 37 storage	#将卷组storage中37个基本单元建立为逻辑卷vo
   ```

   + 直接指定逻辑卷大小

     ```shell
     lvcreate -n vo -L 450M storage	#新建逻辑卷为storage，大小为450M
     ```

     

4. 扩展逻辑卷

   ```shell
   lvextend -L 290M -r /dev/storage/vo	#逻辑卷动态扩展到290M 
   ```

   + EXT4文件系统扩展后使用 **`e2fsck -f`** 检查文件系统完整性
   + XFS文件系统扩展后同步文件系统 **`xfs_qrowfs`** 

5. 缩小逻辑卷，**XFS文件系统只能动态扩展但是无法动态缩小**

   ```shell
   lvreduce -L 100M -r /dev/storage/vo		#将逻辑卷/dev/storage/vo动态减少100M
   ```

**逻辑卷LV快照**(snapshot)

> 快照区与被快照区的LV必须在同一个VG上
>
> 快照容量必须小于等于逻辑卷容量
>
> 快照仅一次有效，一旦执行还原操作后则立即会被自动删除

1. 建立快照

   ```shell
   lvcreate -L 150M -s -n SNAP /dev/storage/vo	#对逻辑卷vo创建150M空间的快照，名称为SNAP
   ```

   > -s	关键选项，创建快照

2. 快照还原

   ```shell
   lvconvert --merge /dev/storage/SNAP	#将快照SNAP进行进行还原合并到原始空间
   ```

3. 可以对快照内容进行部分还原，直接拷贝快照内容即可

删除逻辑LVM

1. 卸载lv
2. 删除逻辑卷lv
3. 删除卷组vg，直接指定卷组名称。不需要设备路径
4. 删除物理卷pvexiot



### Iptables与Firewalld防火墙

> RHEL7后，firewalld防火墙取代了iptables
>
> iptables与freewalld都不是真正的防火墙，都是指用来定义防火墙策略的管理工具。
>
> **iptables**服务把配置好的防火墙策略交由内核层面的**netfilter**网络过滤器来处理
>
> **firewalld**服务把配置好的防火墙策略交由内核层面的**nftables**包过滤框架来处理

#### Iptables

策略与规则链：iptables把用于火过滤流量的策略条目称之为规则，多条规则可以组成一个规则链，而规则链则依据数据包处理位置的不同进行分类，如下

> + 在进行路由选择前处理数据包（**PREROUTING**）
> + 处理流入的数据包（**INPUT**）
> + 处理流出的数据包（**OUTPUT**）
> + 处理转发的数据包（**FORWARD**）
> + 在进行路由选择后处理数据包（**POSTROUTING**）

处理数据包的动作：

+ 允许流量通过（**ACCEPT**）
+ 拒绝流量通过（**REJECT**）：在拒绝流量后回复一条"您的信息已经收到，但是被扔掉了"，从而让流量发送方清晰的看到数据被拒绝的响应信息
+ 记录日志信息（**LOG**）
+ 拒绝流量通过（**DROP**）：直接将流量丢弃而且不响应

常用参数

| -P          | 设置默认策略                                  |
| ----------- | --------------------------------------------- |
| -F          | 清空规则链                                    |
| -L          | 查看规则链                                    |
| -A          | 在规则链的末尾加入新规则                      |
| -I num      | 在规则链的头部加入新规则                      |
| -D num      | 删除某一条规则                                |
| -s          | 匹配来源地址IP/MASK，加感叹号！表示这个ip除外 |
| -d          | 匹配目标地址                                  |
| -i 网卡名称 | 匹配从这块网卡流入的数据                      |
| -o 网卡名称 | 匹配从这块网卡流出的数据                      |
| -p          | 匹配协议，如TCP、UDP、ICMP                    |
| -j 目标     | 指定要跳转的目标                              |
| --dport num | 匹配目标端口号                                |
| --sport num | 匹配来源端口号                                |

iptables修改后，默认下次重启后失效。需要保存后以便永久生效

```shell
iptables-save
```



将INPUT规则链的默认策略设置为拒绝：

```shell
iptable	-P INPUT DROP
```

向INPUT链中添加允许ICMP流量进入的策略规则：

```shell
iptables -I	INPUT -p icmp -j ACCEPT
```

删除INPUT规则链中允许ICMP流量，并把默认策略设置为允许：

```shell
iptables -D INPUT 1		#删除第一条规则
iptables -P INPUT ACCEPT		#设置INPUT规则为ACCEPT
```

将INPUT规则链设置为只允许指定网段(192.168.0/24)的主机访问本机的22端口，拒绝来自其他所有主机的流量：

```shell
iptables -I INPUT -s 192.168.0/24	-p tcp --dport 22 -j ACCEPT		#设置允许指定网段
iptables -A INPUT -p tcp --dport 22 -j REJECT		#规则链末端加入拒绝流量
```

向INPUT规则链中加入拒绝所有人访问本机12345端口的策略规则：

```shell
iptables -I INPUT -p tcp --dport 12345 -j REJECT		#拒绝tcp协议访问本机12345端口
iptables -I INPUT -p udp --dport 12345 -j REJRCT		#拒绝udp协议访问本机12345端口
```

向INPUT规则链中添加拒绝192.168.10.5主机访问本机80端口的策略规则：

```shell
iptables -I INPUT -s 192.168.10.5 -p tcp --dport 80 -j REJECT
```

向INPUT规则链中添加拒绝所有主机访问本机1000~1024端口的策略规则：

```shell
iptables -A INPUT -p tcp --dport 1000:1024 -j REJECT
iptables -A INPUT -p udp --dport 1000:1024 -j REJECT
```



#### Firewalld

Firewalld有CLI（命令行界面）和GUI（图形用户界面）管理方式

Firewalld支持动态更新技术并加入了区域（zone）。区域是firewalld预先准备的几套防火墙策略集合，用户可以根据生产环境的不同而选择合适的策略集合，从而实现防火墙策略之间的快速切换。

RUNTIME立即生效，重启后失效。默认配置下是此模式

PRTMANENT当前不生效，重启后生效

| 区域                     | 默认规则链                                                   |
| ------------------------ | ------------------------------------------------------------ |
| trusted                  | 允许所有的数据包                                             |
| home                     | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、mdns、ipp-client、amba-client、dhcp6-client服务相关，则允许流入 |
| internal                 | 等同于home区域                                               |
| work                     | 拒绝流入的流量，除非有流出的流量相关；而如果流量与ssh、ipp-client、ipv6-client服务相关，则允许流入 |
| **public**(当前生效区域) | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、dhcpv6-client服务相关，则允许流入 |
| external                 | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh服务相关，则允许流入 |
| dmz                      | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh服务相关，则允许流入 |
| block                    | 拒绝流入的流量，除非与流入的流量相关                         |
| drop                     | 拒绝流入的流量，除非与流入的流量相关                         |



Firewall-cmd 命令参数

| --get-dfault-zone             | 查询默认的区域名称 |
| ----------------------------- | ---- |
| --set-default-zone=<区域名称> | 设置默认的区域，使其永久生效 |
| --get-zones                   | 显示可用的区域 |
| --get-services                | 显示预先定义的服务 |
| --get-active-zones            | 显示当前正在使用的区域与网卡名称 |
| --add-source=                 | 将源自此IP或子网的流量导向指定的区域 |
| --remove-source=              | 不再将源自此IP或子网的流量导向某个指定区域 |
| --add-interface=<网卡名称>    | 将源自该网卡的所有流量都导向某个指定区域 |
| --change-interface=<网卡名称> | 将某个网卡与区域进行关联 |
| --list-all                    | 显示当前区域的网卡配置参数、资源、端口以及服务等信息 |
|	--list-all-zones	|显示所有区域的网卡配置参数、资源、端口以及服务等信息|
| --add-service=<服务名> |设置默认区域允许该服务的流量|
| --add-port=<端口号/协议> |设置默认区域允许该端口的流量|
| --remove-service=<服务名> |设置默认区域不再允许该服务的流量|
| --remove-port=<端口号/协议> |设置默认区域不再允许该端口的流量|
| --reload |让“永久生效”的配置规则立即生效，并覆盖当前的配置规则|
| --panic-on |开启应急状况模式|
| --panic-off |关闭应急状况模式|

查看firewalld服务当前所使用的区域

```shell
firewall-cmd --get-default-zone
```

查询网卡ens160所在firewalld服务中的区域

```shell
firewall-cmd --get-zone-of-interface=ens160
```

把firewalld服务中的ens160网卡的默认区域设置为external，并在系统重启后生效

```shell
firewall-cmd --permanent --zone=external --change-interface=ens160
firewall-cmd --get-zone-of-interface=ens160	#查看ens160网卡默认区域
```

把firewalld服务的当前默认区域设置为public

```shell
firewall-cmd --set-default-zone=public
firewall-cmd --get-default-zone	#查看当前默认区域
```

启动/关闭firewalld防火墙服务的应急状况模式，阻断一切网络连接（当远程控制时谨慎使用）

```shell
firewall-cmd --panic-on
firewall-cmd --panic-off
```

查询public区域是否允许请求SSH和HTTPS协议的流量

```shell
firewall-cmd --zone=public --query-service=ssh
firewall-cmd --zone=public --query-service=https
```

把firewalld服务中请求的HTTPS协议流量设置为永久允许，并立即生效

```shell
firewall-cmd --zone=public --add-service=https	#设置为运行时
firewall-cmd --permanent --zone=public --add-service=https	#设置为永久
```

把在firewalld服务中访问8080和8081端口的流量策略设置为允许，但仅限当前生效

```shell
firewalld-cmd --zone=public --add-port=8080-8081/tcp	#设置8080到80801端口
firewall-cmd --zone=public --list-ports	#查看
```

把原本访问本机888端口的流量转发到22端口，要求当前和长期均有效

> Firewall-cmd --permanent --zone=<区域> --add-forward-port=port=<源端口号>:proto=<协议>:toport=<目标端口号>:toaddr=<目标IP地址>

```shell
firewall-cmd --permanent --zone=public --add-forward-port=port=888:proto=tcp:toport=22:toaddr=192.168.10.10
```



服务的访问控制列表（tcp Wrappers）**RHEL8弃用**

/etc/hosts.allow	允许

/etc/hosts.deny	拒绝







