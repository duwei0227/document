---
layout: post
title: Linux系统格式化U盘为fat32
categories: [其他]
permalink: other/linux_format_to_fat32.html

---


### 

### 一、插入 U 盘并确定设备名称

1. 打开终端。
2. 输入以下命令以列出所有的磁盘设备，找到 U 盘的设备名称（通常类似于 `/dev/sda`，具体取决于你插入的设备）：

   ```bash
   lsblk
   ```

   可以看到类似下面的输出：

   ```bash
   duwei@probiecoder:~$ lsblk
   NAME         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
   sda            8:0    1   7.5G  0 disk 
   ├─sda1         8:1    1   2.1G  0 part /run/media/duwei/Fedora-WS-Live-40-1-14
   ├─sda2         8:2    1  12.5M  0 part 
   └─sda3         8:3    1   300K  0 part 
   zram0        252:0    0     8G  0 disk [SWAP]
   nvme0n1      259:0    0 476.9G  0 disk 
   ├─nvme0n1p1  259:1    0   100M  0 part 
   
   ```
   
   在此示例中，`sda` 是 U 盘的设备名称。

### 二、卸载 U 盘

在格式化之前，需要卸载 U 盘。假设 U 盘的设备名是 `/dev/sda1`，可以运行以下命令来卸载：

```bash
sudo umount /dev/sda1
```

### 三、格式化 U 盘为 FAT32

使用 `mkfs.vfat` 命令将 U 盘格式化为 `FAT32`：

```bash
sudo mkfs.vfat -F 32 /dev/sda
```

> **注意**：`/dev/sda` 是整个磁盘，不要加分区编号（如 `/dev/sda1`）。请根据实际设备名替换 `/dev/sda`。



如果报错提示如下信息，需要调整命令参数：

```shell
mkfs.fat 4.2 (2021-01-31)
mkfs.vfat: Partitions or virtual mappings on device '/dev/sda', not making filesystem (use -I to override)
```

```shell
sudo mkfs.vfat -I -F 32 /dev/sda
```



### 四、验证格式化

格式化完成后，可以再次使用 `lsblk` 命令查看设备，确保 U 盘已成功格式化为 `FAT32`。

```bash
lsblk -f
```

输出中应该显示 U 盘的文件系统类型为 `vfat`，这表示 FAT32 文件系统。

```shell
duwei@probiecoder:~$ lsblk -f
NAME         FSTYPE FSVER LABEL  UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda          vfat   FAT32 PROBIE 49F5-F4C7                               7.4G     1% /run/media/duwei/PROBIE

```



### 可选：为分区设置卷标

如果你想为 U 盘设置一个卷标，可以使用 `-n` 选项，例如：

```bash
sudo mkfs.vfat -F 32 -n PROBIE /dev/sda
```

这样，U 盘的卷标将设置为 `PROBIE`。
