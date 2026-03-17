# 模拟磁盘Io慢的方法<no value>



### 0.1.1. 创建一个指定大小的空文件
``` shell
dd if=/dev/zero of=/tmp/100M bs=1M count=100
```
### 0.1.2. 将指定文件虚拟成块设备
``` shell
losetup /dev/loop0 /tmp/100M
```

### 0.1.3. 将指定虚拟设备映射成带延迟的逻辑设备

``` shell
#将指定虚拟块设备映射成读写均延迟100ms的逻辑设备 dm-delay
dmsetup create dm-delay --table "0 `blockdev --getsz /dev/loop0` delay /dev/loop0 0 100"

#blockdev 命令用于指定设备的大小
#dmsetup create 用于创建逻辑设备,创建好的逻辑设备将会在/dev/mapper目录下存在
```

### 0.1.4. 制作文件系统并挂载
``` shell
# 将指定的设备制作成ext4文件系统
mkfs.ext4 /dev/mapper/dm-delay
# 挂载文件系统到指定路径
mount /dev/mapper/dm-delay /mnt/delay
```

### 0.1.5. 卸载挂载目录
```shell
umount -f /mnt/delay
```

### 0.1.6. 删除逻辑设备

``` shell
dmsetup remove dm-delay
```

### 0.1.7. 卸载虚拟设备
```
losetup -d /dev/loop0
```






