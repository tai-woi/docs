
+ 来源：![Deepseek](#bot-message-square "purple")
+ 编辑：[太亮](/贡献者/太亮.md)
+ 修订日期：2026-7-8

通过`/etc/fstab`永久挂载的方式最稳定、最常用，适用于开机自动挂载内部硬盘、固定分区或网络存储。

基本步骤
====

1. 查看磁盘UUID：`sudo blkid`
2. 创建挂载点（目标文件夹）：`sudo mkdir /mnt/mydata`
3. 编辑`/etc/fstab`：`sudo micro /etc/fstab`。在末尾添加一行，格式：`[设备] [挂载点] [文件系统] [挂载参数] [dump] [pass]`
4. 测试挂载：`sudo mount -a`
5. 无报错即成功，重启后会自动挂载。


挂载选项
====

针对 `/etc/fstab` 中 `[设备]`、`[挂载点]`、`[文件系统]`、`[挂载参数]`、`[dump]`、`[pass]` 这 6 个字段的所有常用可选值和注意事项：

设备
----

| 写法 | 示例 | 推荐度 | 说明 |
|------|------|--------|------|
| UUID | `UUID=123e4567-e89b-12d3-a456-426614174000` | ⭐⭐⭐⭐⭐ | 最稳定，设备插拔或换接口都不变 |
| PARTUUID | `PARTUUID=abc123-...` | ⭐⭐⭐⭐ | GPT分区表的唯一ID，比UUID更底层 |
| 设备文件 | `/dev/sdb1` | ⭐⭐ | 简单但不推荐，重启后盘符可能变化 |
| LABEL | `LABEL=MyDisk` | ⭐⭐⭐ | 适合移动硬盘，但卷标可能重复 |
| 网络路径 | `//192.168.1.100/share` | ⭐⭐⭐ | 用于CIFS/SMB网络共享 |
| 服务器:/路径 | `server:/export/data` | ⭐⭐⭐ | 用于NFS网络文件系统 |


挂载点
----

+ 必须是已存在的空目录（如 `/mnt/data`、`/media/usb`）
+ 建议放在 `/mnt/` 下（临时挂载）或 `/media/` 下（可移动设备）
+ 绝对不能挂载到 `/`、`/etc`、`/boot` 等系统关键目录，会导致系统崩溃


文件系统
----

| 文件系统类型 | 适用场景 | 需安装的包 |
|-------------|---------|-----------|
| `ext4` | Linux默认，最常用 | 内置 |
| `ext3` / `ext2` | 旧版Linux | 内置 |
| `ntfs-3g` | Windows NTFS分区 | `ntfs-3g` |
| `vfat` | FAT32/U盘（兼容性好） | 内置 |
| `exfat` | exFAT大文件U盘 | `exfat-fuse` + `exfat-utils` |
| `btrfs` | 新版Linux高级文件系统 | 内置（需安装btrfs-progs） |
| `xfs` | 高性能大容量存储 | `xfsprogs` |
| `iso9660` | 光盘/DVD | 内置 |
| `auto` | 自动检测（不建议） | 可能猜错 |
| `cifs` / `smb3` | Windows网络共享 | `cifs-utils` |
| `nfs` / `nfs4` | NFS网络共享 | `nfs-common` |
| `swap` | 交换分区 | 内置 |


挂载参数
----

挂载参数最复杂、最关键，多个选项用逗号隔开，不能有空格。

### 常用基础选项

| 选项 | 作用 |
|------|------|
| `defaults` | 默认参数（相当于 `rw,suid,dev,exec,auto,nouser,async`） |
| `rw` | 读写挂载 |
| `ro` | 只读挂载 |
| `noatime` | 不更新访问时间，提升性能（SSD推荐） |
| `relatime` | 仅在修改时更新访问时间（默认） |
| `async` | 异步读写（性能好，有掉数据风险） |
| `sync` | 同步读写（安全但慢，U盘推荐） |

### 用户权限控制

| 选项 | 作用 |
|------|------|
| `user` | 允许普通用户挂载（需配合 `noexec` 等） |
| `nouser` | 仅root可挂载（默认） |
| `users` | 允许所有用户挂载和卸载 |
| `uid=1000` | 设置文件所有者的用户ID |
| `gid=1000` | 设置文件所有者的组ID |
| `umask=022` | 设置权限掩码（fat/ntfs常用） |
| `dmask=000` / `fmask=111` | 分别设置目录/文件的权限掩码 |

### 启动和行为控制

| 选项 | 作用 |
|------|------|
| `auto` | 开机自动挂载（默认） |
| `noauto` | 开机不自动挂载，需手动 `mount -a` 或`mount 挂载点` |
| `nofail` | 挂载失败不报错，不阻塞启动（外部设备必备！） |
| `_netdev` | 网络设备挂载，等待网络就绪（NFS/CIFS必须加） |
| `x-systemd.automount` | systemd按需挂载（访问时才挂载） |

### 针对特定文件系统的选项

#### NTFS/extFAT（解决中文乱码和权限）

```
defaults,uid=1000,gid=1000,iocharset=utf8,umask=022
```

#### NFS/CIFS网络共享：

```
defaults,_netdev,noatime,vers=3.0  （NFS版本）
defaults,_netdev,username=user,password=pass,uid=1000,gid=1000  （CIFS需要）
```

#### SSD优化

```
defaults,noatime,discard  （discard开启TRIM）
```


dump备份标志
----

| 值 | 含义 |
|----|------|
| `0` | 不备份（绝大多数情况用这个） |
| `1` | 由dump程序备份（几乎没人用） |


pass开机磁盘检查
----

| 值 | 含义 |
|----|------|
| `0` | 不检查（U盘、外置硬盘、网络共享必须用0） |
| `1` | 仅根分区 `/` 使用（只能有一个） |
| `2` | 其他需要检查的内置分区（按顺序检查） |

错误提示：

+ 如果给外部设备（U盘、NTFS、网络盘）设置 `2`，开机可能因检查超时而卡住。
+ 根目录必须为 `1`，其他Linux分区为 `2`，外部设备为 `0`。


完整示例
====

内置 ext4 硬盘
----

```
UUID=12345678  /mnt/data  ext4  defaults,noatime  0  2
```


NTFS 外置硬盘
----

普通用户可读写，不阻塞启动。

```
UUID=87654321  /mnt/win  ntfs-3g  defaults,uid=1000,gid=1000,iocharset=utf8,umask=022,nofail  0  0
```


U盘自动检测
----

不阻塞启动。

```
UUID=ABCD-1234  /mnt/usb  vfat  defaults,user,noatime,nofail,umask=000  0  0
```


NFS网络共享
----

必须等网络。

```
server:/export  /mnt/nfs  nfs  defaults,_netdev,noatime,vers=4.2  0  0
```


开机不自动挂载
----

手动触发。

```
UUID=xx-xx  /mnt/manual  ext4  defaults,noauto,user  0  0
```


查看当前挂载
====

```bash
mount | grep 挂载点
```

可以看到当前生效的选项，方便调试。
