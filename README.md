# Xok/ExOS Revival

**Reviving MIT's experimental exokernel and library OS — interactive shell demonstrated on November 13, 2026.**

**Languages:** **English** · [简体中文](README.zh-CN.md) · [繁體中文](README.zh-TW.md)

> **Project status:** An experimental historical-system revival, **not** a complete restoration or a production-ready operating system. The original Xok kernel, ExOS user environment, and selected utilities have been brought far enough to log in and execute commands in QEMU. A clean-room, end-to-end rebuild from these instructions has **not yet been independently validated**.

## The milestone

On **2026-11-13**, the revived **Xok/ExOS** environment booted in **QEMU i386**, accessed an **NFS root filesystem**, displayed the native login prompt, accepted a test-account login, and ran an interactive ExOS shell command:

```console
# echo HELLO_EXOS
HELLO_EXOS
#
```

This was an **interactive command entered after login**, not merely text printed by a startup script. The work brings software originating in the late 1990s back to an observable, usable research milestone in 2026, approximately 28 years later.

The original MIT project and its authors deserve credit for Xok, ExOS, and the associated research. **This revival is an independent preservation and compatibility effort, not an official MIT release or an original operating-system implementation.**

## What are Xok and ExOS?

**Xok** is MIT's x86 **exokernel**: its design emphasizes protected allocation and multiplexing of hardware resources rather than imposing one fixed set of high-level OS abstractions. **ExOS** is a **library operating system (LibOS)** that implements Unix-like facilities above Xok. This repository focuses on reviving an existing historical implementation, not designing a new exokernel.

Original research and source archives:

- [MIT PDOS — Exokernel overview](https://pdos.csail.mit.edu/archive/exo/)
- [MIT PDOS — original Exopc distribution, source snapshot and licensing information](https://pdos.csail.mit.edu/archive/exo/distrib.html)
- [MIT PDOS — historical documentation](https://pdos.csail.mit.edu/archive/exo/docs.html)

The original distribution includes a source snapshot dated **June 22, 2000**. Although the research and many source files date to the 1990s, the archival source snapshot should not be described as a purely 1998 release.

## Verified progress

| Component / test | Result |
| --- | --- |
| Xok kernel booted in QEMU i386 | ✅ Verified |
| Tulip PCI networking worked after a PCI bus-mastering fix | ✅ Verified |
| NFS root filesystem could be read and mounted | ✅ Verified |
| Hostname setup progressed successfully | ✅ Verified |
| Native ExOS `/bin/sh` compiled and executed a startup-script command | ✅ Verified |
| `consoled` displayed a login prompt | ✅ Verified |
| Native `/usr/bin/login` accepted a test login | ✅ Verified |
| Interactive shell executed `echo HELLO_EXOS` | ✅ Verified |
| Full set of ExOS userland programs | ⚠️ Not verified |
| IRQ 15 stability, general NFS write support | ⚠️ Unresolved |

## Tested environment

The successful setup used **two distinct machines/environments**:

| Role | Environment |
| --- | --- |
| QEMU host | WSL Ubuntu on a Windows host |
| Emulated target | `qemu-system-i386`, 64 MiB RAM, Tulip PCI NIC, floppy boot image |
| NFS server and build VM | Debian GNU/Linux 2.2 (`potato`) running in VMware |
| Cross-compiler used in the VM | GCC **2.7.2.3** for `i386-unknown-openbsd2` |
| Linker | GNU ld **2.8.1** (BFD 2.8.1) |
| Example exported NFS root | `/srv/xok-root` |

The following IPs and paths describe **one successful lab**, not a portable default: Xok guest `192.168.50.10`, QEMU user-network gateway `192.168.50.1`, and NFS server `192.168.220.134`. The lab also required networking between the QEMU guest and the Debian VM; that routing/forwarding is **not completely specified by the QEMU command alone**.

### Obtain Debian 2.2 yourself

This project does **not** distribute the author's working Debian VMware virtual machine. Install the original system and set up the toolchain yourself using the documentation and future reproduction notes:

- [Debian's official 2.2 (`potato`) release information](https://www.debian.org/releases/potato/)
- [Debian historical CD-image archive](https://cloudfront.debian.net/cdimage/archive/) — includes some historical, user-provided 2.2 images; check architecture, revision, and integrity.
- [Debian archived `potato` packages](https://archive.debian.org/debian/dists/potato/)

**Important:** stock Debian 2.2 does **not** automatically provide the historical custom OpenBSD cross-toolchain used here. Its exact acquisition/build procedure is a remaining documentation task; simply downloading Debian and running `make` is not guaranteed to reproduce the result.

## Boot the already-prepared environment

**Prerequisites:** a compatible `xok-nfs.img` boot image, the prepared and accessible ExOS NFS root, a running Debian NFS server, and correctly configured networking. The command below does **not** create these prerequisites.

On the WSL Ubuntu QEMU host, from the directory containing `xok-nfs.img`:

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

At the FreeDOS prompt, run:

```text
boot bsd
```

When `login:` appears, authenticate using the **locally provisioned test account**. Do not assume any published image or root filesystem shares the original lab's credentials. After obtaining a shell, verify with:

```sh
echo HELLO_EXOS
```

If you see `No such file or directory` for `xok-nfs.img`, check your working directory. If booting stalls at NFS mount, check the NFS service, the guest/server addresses, and routing first.

## Building selected ExOS user programs

The commands below reflect **successful experiments in the already-configured historical Debian VM**, not a complete toolchain/bootstrap tutorial. They assume the `exopc` tree and required static libraries have already been built.

The critical compatibility change was to make GCC forward group-linking options to the linker using `-Wl,`:

```bash
# Run from the exopc source-tree root inside the Debian build VM.
make -C bin/sh \
  CROSS=/usr/local/openbsd-cross/bin/ \
  CC_VER= STATIC=1 \
  LD_RULE='$(CC) -o $@ $(LDFLAGS) -Wl,--start-group $^ -Wl,--end-group $(GCC_LIB)'
```

The same command-line `LD_RULE` override was successfully used to build `bin/login` and `bin/pwd_mkdb` by replacing `bin/sh` with those directories. `consoled` was also built and deployed in the research environment. The override is a **recorded invocation**, not evidence that a permanent Makefile patch is included in this repository.

Relevant deployed paths in the successful environment were:

```text
/bin/sh
/bin/consoled
/usr/bin/login
/usr/sbin/pwd_mkdb
```

A temporary workaround made the historical account's `/bin/csh` path point to `/bin/sh` (`/bin/csh -> sh`). **This is not a genuine C Shell**, and should not be presented as full `csh` support.

## NFS root and data safety

The lab used an NFS export of `/srv/xok-root`. A **read-only** export is preferred for repeatable boot tests. The historical root contains account-related files such as `/etc/master.passwd`, `/etc/pwd.db`, and `/etc/spwd.db`. **Do not publicly upload the private, unreviewed root tarball or the VMware VM**: account hashes, old snapshots, logs, and machine-specific data may be embedded.

A temporary writable NFS export was tried to rebuild BSD password databases; the ExOS client continued to report `Read-only file system`. Native `pwd_mkdb -c /etc/master.passwd` **did** succeed, but full database regeneration and general NFS writes were **not** restored. The working interactive login used a compatibility shell-path workaround instead. Revert any experiment-only `rw,no_root_squash` export to a restrictive configuration once testing ends.

## Key fixes and remaining work

**Working fixes / workarounds identified:**

1. Enabled **PCI bus mastering** for the emulated Tulip device in the historical driver (`sys/dev/if_de_init.c`), restoring network activity.
2. Passed static-library grouping options through the old GCC driver as `-Wl,--start-group` and `-Wl,--end-group`, enabling successful user-program links.
3. Restored missing NFS-root directories and deployed `sh`, `consoled`, and `login` to the paths expected at boot.
4. Used a temporary `/bin/csh -> sh` symbolic link for the account's pre-existing BSD password-database entry.

**Still open:** recurring `unaccounted IRQ 15` messages; native C Shell restoration; correctly rebuilding compatible BSD password databases; general NFS write behavior; broader userland/system testing; clean, independently verified end-to-end reconstruction.

> **Patch publication status:** Exact minimal patches are to be extracted and reviewed against the original source snapshot before they are advertised as downloadable files. This README does not claim that unpublished patches or binary assets already exist in GitHub Releases.

## Releases and reproducibility

The intention is to publish documented, reviewed reproduction materials and suitable boot artifacts through this repository's **Releases** page when available. The private Debian VM is **not** part of the public release. Public NFS-root artifacts must be sanitized and their permissions/symlinks preserved; attach checksums when published.

For history and reproducibility, please record the precise source snapshot, compiler/linker versions, QEMU version, VM networking, checksums, and every patch applied. The November 13 milestone is **verified in the original research environment**, not yet a universally reproducible one-command install.

## Safety and licensing

- This is historical, unsupported OS software. **Use isolated virtual networks; do not expose Debian 2.2 or ExOS to untrusted networks.**
- The original Exopc distribution has **mixed copyright and licensing terms**. Refer to [MIT's source-distribution licensing information](https://pdos.csail.mit.edu/archive/exo/distrib.html), preserve notices in redistributed files, and audit individual components before repackaging. No blanket new license is claimed over MIT or third-party code.
- Do not publish passwords, password hashes, SSH keys, raw VM snapshots, or unsanitized NFS filesystems.
- The project is **independent and unofficial**; original Xok/ExOS authorship remains with its historical contributors.

## Acknowledgment

Thanks to the original MIT PDOS Xok/ExOS researchers and contributors for making the design and historical source available. This revival preserves and studies their work rather than replacing or claiming authorship of it.
