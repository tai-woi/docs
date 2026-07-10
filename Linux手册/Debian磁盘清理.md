
+ 来源：![Deepseek](#bot-message-square "purple")
+ 编辑：[太亮](/贡献者/太亮.md)
+ 修订日期：2026-7-6

Debian 本身是一个轻量且干净的系统，但“消耗磁盘空间”通常不是系统固定占用的，而是默认配置下的某些行为或用户习惯导致的。以下是 Debian 中特别容易“吃”硬盘的几大元凶，以及对应的排查思路：

未清理的旧内核
====

Debian 默认不会自动删除旧内核。每次你执行 `apt upgrade` 更新内核时，旧内核的 vmlinuz、initrd.img 以及相关模块都会保留在 `/boot` 和 `/lib/modules` 下。

- 典型表现：`/boot` 分区写满，导致无法安装新内核。
- 解决：查看已安装内核 `dpkg --list | grep linux-image`，保留当前使用的（`uname -r`），其余用 `sudo apt purge linux-image-旧版本号` 删除。


APT 包管理器的缓存
====

Debian 默认永久保存下载过的所有 `.deb` 安装包（/var/cache/apt/archives）。如果你长期不清理，这个文件夹很容易膨胀到 几 GB 甚至更大。

- 解决：定期执行 `sudo apt clean`（清空所有缓存）或 `sudo apt autoclean`（仅删除无法下载的过时包）。


系统日志的持久化存储
====

Debian 默认将 systemd 日志（journal）保存在内存中，但如果你安装过 `systemd-journal-remote` 或修改过配置文件开启了持久化（`Storage=persistent`），日志会写入 `/var/log/journal`。

- 典型表现：如果某个服务疯狂报错，日志文件能在几小时内涨到 10GB+。
- 解决：检查大小 `journalctl --disk-usage`，限制最大占用 `sudo journalctl --vacuum-size=200M`。


桌面环境的缩略图缓存
====

如果你使用 GNOME、KDE 或 XFCE，文件管理器会为图片和视频生成缩略图（~/.cache/thumbnails）。当你的照片目录很大时，这个隐藏文件夹能轻松吃掉 数 GB 空间。

- 解决：直接删除该目录下的文件（系统重启后会重新生成）。


Web 应用日志
====

如 Nginx/Apache/MySQL，这是服务器上最可怕的空间杀手。默认配置下，access.log 和 error.log 从不轮转（rotate）或轮转策略太宽松。

- 典型表现：一个高并发网站一周的访问日志可能膨胀到 几十 GB。
- 解决：检查 `/var/log/` 下的大文件，修改 `/etc/logrotate.d/` 配置，缩短轮转周期（如 daily）并启用压缩（compress）。


Docker / Podman 的存储驱动
====

如果你在 Debian 上跑 Docker，默认的存储目录 `/var/lib/docker` 会存储所有拉取的镜像层、容器卷和构建缓存。

- 典型表现：即使容器停止了，镜像依然占用空间；`docker system df` 显示虚悬镜像（dangling images）巨大。
- 解决：定期执行 `docker system prune -a -f`（注意会删除未使用的镜像）。


备份工具的默认快照
====

例如 timeshift 或 deja-dup。Timeshift 默认使用 RSYNC 模式，将快照保存在 `/timeshift` 目录。如果采用“定时快照”且不限制数量，它会把整个系统文件复制多份，轻松吃掉 50GB+。

- 解决：检查根目录下是否有该文件夹，调整保留策略或更改存储位置到外置硬盘。


快速定位
====

如果你发现硬盘突然红了，请按顺序执行以下命令：

```bash
# 1. 查看根目录下哪些文件夹最大（深度1）
sudo du -sh /* 2>/dev/null | sort -hr | head -10

# 2. 专门查 /var（日志和缓存重灾区）
sudo du -sh /var/* 2>/dev/null | sort -hr | head -10

# 3. 查用户家目录的隐藏缓存（如果有图形界面）
du -sh ~/.cache/* 2>/dev/null | sort -hr | head -10
```

最后提醒：Debian 自身的“/usr”和“/lib”是极其稳定的，不会无故变大。99% 的空间异常都集中在 `/var`、`/home`（用户缓存）和 `/boot`（旧内核）这三个目录。定期清理上述行为，就能让系统保持清爽。
