# Xok/ExOS Revival — 復活計畫

**讓 MIT 的實驗性外核心與函式庫作業系統重新運作：2026 年 11 月 13 日實現互動式 Shell。**

**語言：** [English](README.md) · [简体中文](README.zh-CN.md) · **繁體中文**

> **專案狀態：** 這是一項歷史作業系統的重現與相容性修復實驗，**不代表完整恢復，更不是可用於正式生產環境的作業系統**。目前已在 QEMU 中完成原生登入與互動式指令執行，但尚未在另一套全新環境中獨立驗證從零開始的完整建置流程。

## 復活里程碑

**2026 年 11 月 13 日**，修復後的 **Xok/ExOS** 在 **QEMU i386** 中啟動、存取 **NFS 根檔案系統**、顯示原生登入提示、完成測試帳號登入，並在 ExOS 互動式 Shell 中執行：

```console
# echo HELLO_EXOS
HELLO_EXOS
#
```

這是**登入後親自輸入並執行**的指令，而非啟動腳本預先輸出的文字。這是一套源自 1990 年代的研究系統，在約 28 年後重新達成可觀察互動的里程碑。

Xok、ExOS 及其研究成果的原始作者，是 MIT 等機構的研究人員。**本專案為獨立的歷史軟體保存與相容性修復工作，並非 MIT 官方發行版本，也不是重新原創一套作業系統。**

## 什麼是 Xok 與 ExOS？

**Xok** 是 MIT 面向 x86 的 **Exokernel（外核心）**：核心著重於安全地配置與多工使用硬體資源，而不是強制替所有應用程式提供一致的高階作業系統抽象。**ExOS** 是其上的 **LibOS（函式庫作業系統）**，在使用者空間提供類 Unix 功能。本專案著重於復原既有的歷史實作，而非開發新外核心。

原始資源：

- [MIT PDOS：Exokernel 專案介紹](https://pdos.csail.mit.edu/archive/exo/)
- [MIT PDOS：Exopc 原始原始碼發行頁、快照及授權資訊](https://pdos.csail.mit.edu/archive/exo/distrib.html)
- [MIT PDOS：歷史技術文件](https://pdos.csail.mit.edu/archive/exo/docs.html)

MIT 原始發行頁提供日期為 **2000 年 6 月 22 日**的原始碼快照。雖然研究成果和許多檔案來自 1990 年代，但不能把該份具體快照直接稱為「1998 年發行版」。

## 已驗證的成果

| 項目 | 結果 |
| --- | --- |
| Xok 核心在 QEMU i386 啟動 | ✅ 已驗證 |
| 修復 Tulip PCI 匯流排主控設定後恢復網路通訊 | ✅ 已驗證 |
| NFS 根檔案系統可存取及掛載 | ✅ 已驗證 |
| 主機名稱初始化成功推進 | ✅ 已驗證 |
| 原生 `/bin/sh` 編譯成功並執行啟動腳本 | ✅ 已驗證 |
| `consoled` 顯示登入提示 | ✅ 已驗證 |
| 原生 `/usr/bin/login` 完成測試帳號登入 | ✅ 已驗證 |
| 互動式 Shell 執行 `echo HELLO_EXOS` | ✅ 已驗證 |
| 全部 ExOS 使用者空間程式 | ⚠️ 尚未完整驗證 |
| IRQ 15 穩定性與一般 NFS 寫入 | ⚠️ 尚未解決 |

## 成功實驗使用的環境

這次實驗使用了**兩套不同的環境**：

| 用途 | 環境 |
| --- | --- |
| QEMU 主機環境 | Windows 主機上的 WSL Ubuntu |
| QEMU 模擬目標 | `qemu-system-i386`、64 MiB 記憶體、Tulip PCI 網卡、軟碟開機映像 |
| 建置與 NFS 伺服器 | VMware 中的 Debian GNU/Linux 2.2（`potato`） |
| 交叉編譯器 | 針對 `i386-unknown-openbsd2` 的 GCC **2.7.2.3** |
| 連結器 | GNU ld **2.8.1**（BFD 2.8.1） |
| 實驗 NFS 根目錄 | `/srv/xok-root` |

以下位址都是**本次實驗設定，並非適合所有人的預設值**：Xok 用戶端 `192.168.50.10`、QEMU 使用者網路閘道 `192.168.50.1`、NFS 伺服器 `192.168.220.134`。QEMU 用戶端與 Debian 虛擬機之間仍需要適當的路由或轉送，**只複製 QEMU 指令並不會自動完成全部網路設定**。

### 自行取得 Debian 2.2

本專案**不會公開散布作者正在使用的 Debian VMware 虛擬機**。有興趣重現的開發者可自行安裝 Debian 2.2，並依照紀錄建立工具鏈：

- [Debian 官方 Debian 2.2（potato）版本資訊](https://www.debian.org/releases/potato/)
- [Debian 歷史光碟映像存檔](https://cloudfront.debian.net/cdimage/archive/)：含部分使用者提供的 Debian 2.2 歷史映像，請確認架構、版本與檔案完整性。
- [Debian potato 歷史套件存檔](https://archive.debian.org/debian/dists/potato/)

**請注意：** 原生 Debian 2.2 **並未內建**本次使用的特定 OpenBSD 交叉編譯工具鏈。工具鏈的確切取得與建置程序仍待補充；單純下載 Debian 並執行 `make`，無法保證重現成功。

## 啟動已準備好的環境

**前提：** 已有相容的 `xok-nfs.img`、準備完成且可存取的 ExOS NFS 根目錄、正在執行的 Debian NFS 服務，以及正確的網路設定。以下指令**不會自動建立這些前置條件**。

在 WSL Ubuntu 中，進入存放映像的目錄：

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

進入 FreeDOS 後輸入：

```text
boot bsd
```

出現 `login:` 時，使用**本機預先設定的測試帳號**登入。請勿假設任何公開映像或根檔案系統都沿用原實驗環境的密碼。登入 Shell 後執行：

```sh
echo HELLO_EXOS
```

若出現找不到 `xok-nfs.img` 的錯誤，請先確認終端機目前的工作目錄；若停在 NFS 掛載階段，應優先檢查 NFS 服務、雙方 IP 及路由設定。

## 建置部分 ExOS 使用者空間程式

以下指令來自**已成功設定的歷史 Debian 實驗虛擬機**，並非完整的交叉工具鏈安裝教學。它假設 `exopc` 原始碼樹和相關靜態函式庫都已準備完成。

關鍵相容性修復是使用 `-Wl,`，將靜態函式庫群組選項正確傳遞給舊版 GCC 所呼叫的連結器：

```bash
# 在 Debian 虛擬機中，從 exopc 原始碼根目錄執行。
make -C bin/sh \
  CROSS=/usr/local/openbsd-cross/bin/ \
  CC_VER= STATIC=1 \
  LD_RULE='$(CC) -o $@ $(LDFLAGS) -Wl,--start-group $^ -Wl,--end-group $(GCC_LIB)'
```

將 `bin/sh` 替換為 `bin/login` 和 `bin/pwd_mkdb`，相同的 `LD_RULE` 命令列覆寫方式也曾成功建置對應程式。實驗中也成功建置並部署了 `consoled`。請注意：這裡記錄的是**實際使用過的命令列覆寫方式**，不表示儲存庫已包含永久套用的 Makefile 修補檔。

成功環境中的部署路徑：

```text
/bin/sh
/bin/consoled
/usr/bin/login
/usr/sbin/pwd_mkdb
```

為暫時避開舊 BSD 密碼資料庫中的 `/bin/csh` 設定，實驗使用 `/bin/csh -> sh` 符號連結。**這不是真正的 C Shell，也不代表支援全部 csh 語法。**

## NFS 根目錄與隱私安全

實驗將 `/srv/xok-root` 作為 NFS 匯出目錄。一般重現測試建議使用**唯讀匯出**。歷史根目錄含 `/etc/master.passwd`、`/etc/pwd.db`、`/etc/spwd.db` 等帳戶相關檔案。**未經清理的私人根目錄封存檔及整台 VMware 虛擬機，不應直接公開上傳**：其中可能含密碼雜湊、快照、紀錄與機器專屬設定。

實驗曾嘗試暫時開放 NFS 寫入來重建 BSD 密碼資料庫，但 ExOS 用戶端仍顯示 `Read-only file system`。原生 `pwd_mkdb -c /etc/master.passwd` **唯讀檢查成功**，但完整資料庫重建及一般 NFS 寫入功能**仍未恢復**。最後能進行互動登入，是透過 Shell 路徑的相容性替代方案。如果測試時曾使用 `rw,no_root_squash`，結束後應恢復較嚴格的設定。

## 關鍵修復與待辦事項

**本次實際使用的修復或暫時方案：**

1. 在歷史 Tulip 驅動 `sys/dev/if_de_init.c` 中啟用 **PCI Bus Master**，恢復模擬網卡通訊。
2. 使用 `-Wl,--start-group` 與 `-Wl,--end-group` 修正舊版 GCC 的靜態函式庫群組參數傳遞，成功連結使用者空間程式。
3. 補齊 NFS 根目錄結構，並將 `sh`、`consoled`、`login` 部署到啟動所需位置。
4. 利用暫時的 `/bin/csh -> sh` 符號連結，相容舊 BSD 密碼資料庫中的預設 Shell 路徑。

**仍待解決：** 持續出現的 `unaccounted IRQ 15`、真正的 C Shell、相容 BSD 密碼資料庫的正式重建、NFS 寫入、更多使用者空間元件測試，以及在全新環境中獨立完成完整重現。

> **修補檔狀態：** 最小化修補檔仍需從實驗原始碼正式擷取，並與原始快照核對後才可發行。本 README 不宣稱 GitHub Releases 已存在尚未上傳的修補檔或映像檔。

## 發行與可重現性

未來計畫在審核完成後，透過儲存庫的 **Releases** 發布適合公開的映像、重現資料與校驗碼。**私人 Debian VMware 虛擬機不公開。** 公開 NFS 根目錄封存檔前，必須審核與清理帳戶資料，同時保留必要的檔案權限及符號連結。

後續重現紀錄應註明準確的原始碼快照、GCC/ld/QEMU 版本、網路拓樸、校驗碼以及全部修補內容。2026 年 11 月 13 日的成果已在**原研究環境中實際驗證**，但不等於已具備「一行指令從零安裝」的流程。

## 安全、授權與署名

- 這是停止維護的歷史系統軟體。**僅應在隔離的虛擬網路中研究，勿將 Debian 2.2 或 ExOS 直接暴露在不受信任的網路上。**
- 原始 Exopc 發行版包含**多種著作權及授權條款**。請參考 [MIT 原始發行頁的授權資訊](https://pdos.csail.mit.edu/archive/exo/distrib.html)，保留原有聲明，重新散布前逐項核對，不能對 MIT 或第三方程式碼一概宣稱新的授權條款。
- 不要公開帳戶密碼、密碼雜湊、SSH 私鑰、未經清理的虛擬機快照或 NFS 根目錄。
- 本專案**並非 MIT 官方專案**；Xok/ExOS 原始成果仍歸其歷史作者及貢獻者所有。

## 致謝

感謝 MIT PDOS 的原始研究人員與所有歷史貢獻者公開 Xok/ExOS 的設計、文件與原始碼。本專案旨在保存、研究與重現他們的成果，而非取代原專案或主張原始作者身分。
