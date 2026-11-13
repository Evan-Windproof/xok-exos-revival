# Xok/ExOS Revival — 复活计划

**让 MIT 的实验性外核与库操作系统重新运行：2026 年 11 月 13 日实现交互式 Shell。**

**语言：** [English](README.md) · **简体中文** · [繁體中文](README.zh-TW.md)

> **项目状态：** 这是一次历史操作系统复现与兼容性修复实验，**不代表完整恢复，更不是可用于生产的操作系统**。当前已经在 QEMU 中完成原生登录和交互式命令执行，但尚未在另一套全新环境中独立验证从零到成功的完整构建流程。

## 复活里程碑

**2026 年 11 月 13 日**，修复后的 **Xok/ExOS** 在 **QEMU i386** 中启动，访问 **NFS 根文件系统**，显示原生登录提示符，完成实验账户登录，并在 ExOS 的交互式 Shell 中执行：

```console
# echo HELLO_EXOS
HELLO_EXOS
#
```

这条命令是**登录后手动输入并执行**的，不是启动脚本预先打印的文字。这是一个源于 20 世纪 90 年代的研究系统在约 28 年后重新实现可观察交互的里程碑。

Xok、ExOS 以及其研究成果的原始作者是 MIT 等机构的研究人员。**本项目是独立的历史软件保存与兼容性修复工作，不是 MIT 官方发行版，也不是重新原创一个操作系统。**

## Xok 和 ExOS 是什么？

**Xok** 是 MIT 面向 x86 的 **Exokernel（外核）**：内核主要负责安全地分配和复用硬件资源，而不是强制为所有应用提供统一的高级操作系统抽象。**ExOS** 是运行于其上的 **LibOS（库操作系统）**，在用户态提供类 Unix 的能力。本项目恢复的是一套已经存在的历史实现，而非开发新外核。

原始资料：

- [MIT PDOS：Exokernel 项目介绍](https://pdos.csail.mit.edu/archive/exo/)
- [MIT PDOS：Exopc 原始源码发行页、源码快照及许可证信息](https://pdos.csail.mit.edu/archive/exo/distrib.html)
- [MIT PDOS：历史技术文档](https://pdos.csail.mit.edu/archive/exo/docs.html)

MIT 原始发行页提供 **2000 年 6 月 22 日**的源码快照。虽然研究成果和许多文件来自 1990 年代，但不能把这份具体快照直接称为“1998 年发行版”。

## 已验证的成果

| 项目 | 结果 |
| --- | --- |
| Xok 内核在 QEMU i386 启动 | ✅ 已验证 |
| 修复 Tulip PCI 总线主控配置后恢复网络通信 | ✅ 已验证 |
| NFS 根文件系统可访问及挂载 | ✅ 已验证 |
| 主机名初始化成功推进 | ✅ 已验证 |
| 原生 `/bin/sh` 编译成功并执行启动脚本 | ✅ 已验证 |
| `consoled` 显示登录提示符 | ✅ 已验证 |
| 原生 `/usr/bin/login` 完成实验账户登录 | ✅ 已验证 |
| 交互式 Shell 执行 `echo HELLO_EXOS` | ✅ 已验证 |
| ExOS 全部用户程序 | ⚠️ 未完整验证 |
| IRQ 15 稳定性与通用 NFS 写入 | ⚠️ 尚未解决 |

## 本次成功使用的环境

这次实验使用了**两套不同的环境**：

| 用途 | 环境 |
| --- | --- |
| QEMU 宿主环境 | Windows 主机中的 WSL Ubuntu |
| QEMU 模拟目标 | `qemu-system-i386`、64 MiB 内存、Tulip PCI 网卡、软盘启动镜像 |
| 构建与 NFS 服务端 | VMware 中的 Debian GNU/Linux 2.2（`potato`） |
| 交叉编译器 | 面向 `i386-unknown-openbsd2` 的 GCC **2.7.2.3** |
| 链接器 | GNU ld **2.8.1**（BFD 2.8.1） |
| 实验 NFS 根目录 | `/srv/xok-root` |

以下地址均是**本次实验配置，并非适用于所有人的默认值**：Xok 客户端 `192.168.50.10`、QEMU 用户网络网关 `192.168.50.1`、NFS 服务端 `192.168.220.134`。QEMU 客户端与 Debian 虚拟机之间还需要正确的路由或转发，**仅复制 QEMU 命令并不能自动完成整个网络配置**。

### 自行获取 Debian 2.2

本项目**不公开分发作者正在使用的 Debian VMware 虚拟机**。有兴趣复现的开发者可以自行安装 Debian 2.2，并根据记录搭建工具链：

- [Debian 官方 Debian 2.2（potato）版本说明](https://www.debian.org/releases/potato/)
- [Debian 历史光盘镜像归档](https://cloudfront.debian.net/cdimage/archive/)：包含部分用户提供的 Debian 2.2 历史镜像，请核对架构、版本及文件完整性。
- [Debian potato 历史软件包仓库](https://archive.debian.org/debian/dists/potato/)

**注意：** 原生 Debian 2.2 **不自带**本次使用的特定 OpenBSD 交叉工具链。工具链的准确获取与构建过程仍需补写；仅下载 Debian 后直接运行 `make`，无法保证复现成功。

## 启动已经准备好的环境

**前提：** 已有兼容的 `xok-nfs.img`、准备完毕且可访问的 ExOS NFS 根目录、正在运行的 Debian NFS 服务以及正确的网络配置。以下命令**不会自动生成这些依赖**。

在 WSL Ubuntu 中，进入存放镜像的目录：

```bash
cd ~/os-research/xok-boot

qemu-system-i386 \
  -m 64 \
  -drive file=xok-nfs.img,if=floppy,format=raw \
  -boot a \
  -nic none \
  -netdev user,id=xoknet,net=192.168.50.0/24,host=192.168.50.1,ipv6=off \
  -device tulip,netdev=xoknet,mac=52:54:00:12:34:56 \
  -object filter-dump,id=xokdump,netdev=xoknet,file=/tmp/xok-root-test.pcap
```

进入 FreeDOS 后执行：

```text
boot bsd
```

出现 `login:` 后，使用**本地预先配置的实验账户**登录。不能假定任何公开镜像或根文件系统都使用原实验室的账户密码。登录进入 Shell 后执行：

```sh
echo HELLO_EXOS
```

如果提示找不到 `xok-nfs.img`，先确认终端的工作目录。如果卡在 NFS 挂载，优先检查 NFS 服务、双方 IP 和路由配置。

## 编译部分 ExOS 用户态程序

下面的命令来自**已经配置成功的历史 Debian 实验虚拟机**，不是完整的交叉工具链安装教程。它假设 `exopc` 源码树及相关静态库已经准备好。

关键的兼容性修复是利用 `-Wl,` 把静态库分组选项正确传递给旧版 GCC 调用的链接器：

```bash
# 在 Debian 虚拟机中，从 exopc 源码根目录运行。
make -C bin/sh \
  CROSS=/usr/local/openbsd-cross/bin/ \
  CC_VER= STATIC=1 \
  LD_RULE='$(CC) -o $@ $(LDFLAGS) -Wl,--start-group $^ -Wl,--end-group $(GCC_LIB)'
```

把 `bin/sh` 替换为 `bin/login` 和 `bin/pwd_mkdb`，同样的 `LD_RULE` 命令行覆盖方式也曾成功构建对应程序。实验中还成功构建并部署了 `consoled`。注意：这里展示的是**实际用过的命令行覆盖方式**，不代表仓库已经包含永久生效的 Makefile 补丁。

成功环境中使用的部署路径：

```text
/bin/sh
/bin/consoled
/usr/bin/login
/usr/sbin/pwd_mkdb
```

为了暂时绕过旧 BSD 密码数据库中的 `/bin/csh` 配置，实验使用了 `/bin/csh -> sh` 符号链接。**这不是真正的 C Shell，也不意味着支持全部 csh 语法。**

## NFS 根目录与隐私安全

实验使用 `/srv/xok-root` 作为 NFS 导出目录。建议正常复现时使用**只读导出**。历史根目录包含 `/etc/master.passwd`、`/etc/pwd.db`、`/etc/spwd.db` 等账户相关文件。**未经清理的私人根目录归档和整台 VMware 虚拟机不能直接公开发布**：其中可能包含密码哈希、快照、日志及机器专属配置。

实验曾尝试临时开放 NFS 写入，用以重新生成 BSD 账户数据库，但 ExOS 客户端仍报告 `Read-only file system`。原生 `pwd_mkdb -c /etc/master.passwd` **只读检查成功**，但完整的数据库重建以及通用 NFS 写入功能**尚未恢复**。最终能够交互登录，依靠的是 Shell 路径兼容方案。若曾临时使用 `rw,no_root_squash`，实验结束后应恢复为限制更严格的配置。

## 关键修复与未完成项目

**本次实际采用的修复或临时方案：**

1. 在历史 Tulip 驱动 `sys/dev/if_de_init.c` 中启用 **PCI Bus Master**，恢复模拟网卡通信。
2. 使用 `-Wl,--start-group` 和 `-Wl,--end-group` 修复旧版 GCC 的静态库分组参数传递，成功链接用户态程序。
3. 补齐 NFS 根目录结构，将 `sh`、`consoled`、`login` 部署至启动所需的路径。
4. 用临时 `/bin/csh -> sh` 符号链接兼容旧 BSD 密码数据库中记录的默认 Shell。

**仍待解决：** 持续刷屏的 `unaccounted IRQ 15`、真正的 C Shell、兼容 BSD 密码数据库的正式重建、NFS 写入、更多用户态组件测试，以及在全新环境中独立完成从零复现。

> **补丁状态：** 最小化修复补丁仍需从实验源码中正式提取，并与原始快照核对后才能发布。此 README 不声称 GitHub Releases 已存在尚未上传的补丁或镜像。

## 发行与可复现性

计划在审查完成后通过仓库的 **Releases** 发布适合公开的镜像、复现资料及校验和。**私人 Debian VMware 虚拟机不公开。** 公开 NFS 根目录包之前必须审查并清理账户数据，同时保留必要的文件权限和符号链接。

请在后续复现记录中注明准确的源码快照、GCC/ld/QEMU 版本、网络拓扑、校验和和全部补丁。2026 年 11 月 13 日的成果是在**原实验环境中实际验证**的，并不等于已经实现“一条命令从零安装”。

## 安全、许可证与署名

- 这是已经停止维护的历史系统软件。**只应在隔离的虚拟网络中研究，不要把 Debian 2.2 或 ExOS 直接暴露到不可信网络。**
- 原始 Exopc 发行版包含**多种版权与许可证条款**。请参阅 [MIT 原始发行页中的许可证信息](https://pdos.csail.mit.edu/archive/exo/distrib.html)，保留原始声明，重新分发之前逐项核对，不能对 MIT 或第三方代码统一宣称新的许可证。
- 不要公开账户密码、密码哈希、SSH 私钥、未经清理的虚拟机快照和 NFS 根目录。
- 本项目**非 MIT 官方项目**。Xok/ExOS 的原始成果归其历史作者及贡献者所有。

## 致谢

感谢 MIT PDOS 的原始研究人员及所有历史贡献者公开 Xok/ExOS 的设计、资料与源码。本项目旨在保存、研究和复现他们的成果，而非取代原项目或主张原始作者身份。
