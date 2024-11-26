# linuxNode

[TOC]





## 文件系统管理

### 文件系统简介

​		一个硬盘的主分区总共最多只能分四个。扩展分区只能有一个，也算是主分区的一种；其不能存储数据和格式化，必须再划分为逻辑分区才能使用。

<img src="images/linuxNote/image-20241125233849486.png" alt="image-20241125233849486" style="zoom:50%;" />

<img src="images/linuxNote/image-20241125234006344.png" alt="image-20241125234006344" style="zoom:50%;" />

​		==ext、ext2、ext3、ext4和xfs的特点==

### 常用命令



```shell
# 文件系统占用大小查看命令
# 包括文件大小和被系统占用的空间
# 常用: df -h
df [选项] [挂载点]

# 用于显示目录或文件占用的磁盘空间
# 只包含文件大小
du -sh [目录路径]	# 查看该目录下所有文件大小总和
du -h --max-depth=1 | sort -hr	# 查看目录的一级子目录的大小，并降序排序

# 文件系统修复命令
fsck [选项] 分区设备文件名

# 显示磁盘状态命令
dumpe2fs 分区设备文件名
```

​		

### 文件系统挂载

```shell
# 文件系统的挂载
mount		# 查询系统中已经挂载的设备
mount -a	# 依据配置文件/etc/fstab的内容进行挂载
mount [-t 文件系统] [-L 卷标名] [-o 特殊选项] 设备文件名 挂载点	# 挂载设备，可以直接 mount 设备文件名 挂载点

# 假设要将光盘/dev/sr0挂载到/mnt/sr文件夹上（可以是任意文件夹）
mount /dev/sr0 /mnt/sr
# 卸载，以下两个指令都可以
unmount /dev/sr0
unmount /mnt/sr

```

​		mount的特殊选项

<img src="images/linuxNote/image-20241126220737791.png" alt="image-20241126220737791" style="zoom:40%;" />

### 文件系统分区

```shell
fdisk -l		# 查看所有能被识别到的存储设备，并显示分区信息
# 假设新加入的硬盘的系统分配的设备名为/dev/sdb，对其进行分区
fdisk /dev/sdb	# 然后就进入分区的交互引导过程
```

<img src="images/linuxNote/image-20241126222641571.png" alt="image-20241126222641571" style="zoom:40%;" />

按如下划分第一个分区，名为/dev/sdb1，从磁盘起始位置2GB大小：

<img src="images/linuxNote/image-20241126223444704.png" alt="image-20241126223444704" style="zoom:50%;" />

​		同理还可以继续划分主分区和扩展分区，然后在扩展分区中划分逻辑分区。划分好后输入w保存退出。

```shell
# 重新读取分区表
partprobe
# 格式化分区
mkfs -t ext4 /dev/sdb1
# 挂载分区
mount /dev/sdb1 /disk1
```

### 文件系统的自动挂载

​		在/etc/fstab中内容如下所示：

```shell
# <file system>							<mount point>   <type>  	<options>       <dump>  <pass>
UUID=d5abbd65-9681-490f-ab70-bf3844091a33 /               ext4    errors=remount-ro 0       1
/dev/sdb1                                 /home/sdlm/data ext4    defaults         0       0

```

- 第一字段：分区设备文件名或者UUID（dumpe2fs可以查看）

- 第二字段：挂载点
- 第三字段：文件系统格式
- 第四字段：挂载参数，可以默认使用 defaults，该参数详见mount的特殊选项表格
- 第五字段：指定分区是否被dump备份，0代表不备份，1代表每天备份，2 代表不定期备份
- 第六字段：指定分区是否被fsck检测，0代表不检测，其他数组代表优先级，越小越高。一般设置为0，系统分区设置1

​		一般在修改后，直接调用`mount  -a`查看是否修改正确。