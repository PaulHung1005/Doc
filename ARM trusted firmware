# ARM trusted firmware

## To do
- [x] 自行編譯並安裝 QEMU
    [學會如何在 Linux 上編譯開源應用程式](https://linux.vbird.org/linux_basic/centos7/0520source_code_and_tarball.php)
- [X] 在 QEMU 上執行 TF-A
    [QEMU virt Armv8-A](https://trustedfirmware-a.readthedocs.io/en/latest/plat/qemu.html)
- [x] 觀察 TF-A 的開機流程
    [QEMU GDB usage](https://www.qemu.org/docs/master/system/gdb.html)
- [x] 編譯與執行 OP-TEE
    [Build and Run OP-TEE](https://optee.readthedocs.io/en/latest/building/index.html)
- [x] 追蹤完整 TF-A 開機流程

## 基礎知識
### Linux 啟動流程
#### UEFI (Unified Extensible Firmware Interface) 
有別於傳統 BIOS(Basic Input/Output System) 使用組合語言開發，且僅支援 16bit 資料運算不同。 UEFI 使用 C 語言撰寫，可依據平台性能支援 32bit 或 64bit ，並提供低階的作業系統功能，藉以實現精美的 UI(User Interface) 介面或網路開機等功能。在硬體資源的控制上， BIOS 使用中斷 (Interrupt Request, IRQ) 進行管理，而 UEFI 則採用輪詢 (polling) 方式。
    
#### Boot Loader
提供下列功能
1. 開機選單：提供多系統開機選項。
2. 載入核心檔案：將作業系統核心解壓縮並載入到記憶體中執行。
3. 轉交其他 boot loader：依據第一點的選項，將開機管理功能轉交給其他作業系統的 boot loader。
    
多系統開機流程如下圖範例所示，硬碟中共存在 Windows(W) 與 Linxu(L) 兩個作業系統， MBR 指向 W 的 boot loader 提供了 W 和 L 兩個開機選項。當使用者選擇 L ，則 W 的 boot loader 會將開機管理工作轉移至 L 的 boot loader ，並開始載入 Linux 核心接續開機流程。
![image](https://hackmd.io/_uploads/rkSLctRUA.png)

#### 啟動流程
1. 電腦啟動電源後，會自動讀取 BIOS 或 UEFI(Unified Extensible Firmware Interface) BIOS 載入硬體資訊及進行硬體系統的自我測試 (Power-on Self Test, POST) ，完成後會依據設定讀取第一個可開機裝置，如 USB、光碟機或硬碟等。
2. 從可開機裝置的主要開機紀錄區 (MBR, Master Boot Record) 載入與執行開機管理程式 (boot loader)。
3. Boot loader 依據設定解壓縮並載入目標 Linux 核心到記憶體中執行，重新偵測硬體資訊與載入驅動程式。
4. Linux 核心呼叫 systemd 程式，載入系統平時運作所需的所有外部程式。

#### 虛擬檔案系統 (Initial RAM Filesystem, initramfs)
又被稱為 Initial RAM Disk ，即為 initrd 系統命令。 boot loader 運作時會同時將 Linux 核心與 initramfs 掛載到記憶體中。 Linxu 核心剛載入至記憶體時，會因為核心沒有編譯硬碟驅動程式，無法讀取硬碟中的資料掛載根目錄下的驅動程式。故需使用 initramfs 模擬 Linux 核心的根目錄，透過此虛擬檔案系統偵測與載入實際的硬體驅動程式，讓核心可透過虛擬檔案系統取得驅動程式，以及載入實際的根目錄檔案系統。

細部的 initramfs 運作說明可以參照 initrd 的 man page 描述，並可以透過 lsinitramfs 命令，顯示包含 Linux 核心開機必要模組的 initramfs 映像檔內容。
```shell
$ lsinitramfs /boot/initrd.img
```
需注意的是 initrd.img 檔案是連結檔，實際所使用的映像檔可以使用 ls 命令進行觀看。
```shell
$ ls -al initrd.img
lrwxrwxrwx 1 root root 27  六  19 20:28 initrd.img -> initrd.img-6.5.0-41-generic
```
    
Linux 核心完成實際檔案系統載入後便會將 initramfs 釋放，並呼叫 systemd 執行接續的開機流程。 systemd 用於建立軟體執行環境，以啟動作業系統的各項服務

> [主機規劃與磁碟分割](https://linux.vbird.org/linux_basic/centos7/0130designlinux.php#partition_table)
> [開機流程、模組管理與 Loader](https://linux.vbird.org/linux_basic/centos7/0510osloader.php)

### ELF (Executable and Linkable Format) 檔案
程式碼經過編譯器與連結器輸出為可執行的目的檔 (.axf/.elf)，該檔案中存在多個用於儲存程式碼與變數等資訊的區間 (section) 。
![image](https://hackmd.io/_uploads/BkBheCZIC.png)

區間內容在編譯過程中透過連結器進行分配與處理，下列為常見的區間種類，實際應用上使用者也可依據自身需求定義新的區間。須注意下列的 .data 與 .bss 為全域變數，區域變數只有在執行過程中儲存於 stack 中。
- .text: text section ，程式的機械碼指令，通常是唯讀
- .rodata: read only data section ，存放如使用 const 宣告的唯讀全域變數
- .data: data section with initial value ，有給予初始值的全域變數
- .bss: data section without initial value ，未給予初始值的全域變數

> [嵌入式系統建構：開發運作於STM32的韌體程式](http://wiki.csie.ncku.edu.tw/embedded/Lab19/stm32-prog.pdf)

### ARMv8 架構
![image](https://hackmd.io/_uploads/Hk-aPhTS0.png)
- Architecture: 描述系統主要功能及行為 (e.g. ARMv8-A, ARMv9-A)，並明確定義下列功能
    - Instruction set
    - Register set
    - Exception model
    - Memory model
    - Debug, trace, and profiling
- Micro-architecture: 描述系統的細部規格 (e.g. Cortex-A8, Cortex-A65AE)，明確定義下列規格
    - Pipline 長度與種類 (in-order/out-of-order)
    - Cache 數量與大小
    - option feature
- PE (Processing element): 處理單元 (e.g. ARMv8-A, ARMv9-A)， manual 文件中對 ARM 處理器的架構與行為的抽象稱呼 (The behavior of an abstract machine)，避免在文件描述上與 micro-architecture 混淆。
- Execution State: 定義處理單元的運算環境 (e.x. AArch64 和 AArch32)，內容包含下列項目。自 ARMv8-A 架構開始可同時支援 AArch64 和 AArch32 ，AArch64 支援 A64 指令集，可處理 32 或 64-bit 的資料與運算； AArch32 支援 T32 與 A32 指令集，負責執行 32-bit 資料運算，且與 ARMv7-A 相容。 需注意 PE 僅能在處理器重置或 EL 層級變更時更動 execution state 。
    - 支援的暫存器資料寬度 (register widths)
    - 支援的指令集 (instruction sets)
    - Exception Level
    - Virtual Memory System Architecture (VMSA)
    - Programmer's model
- Exception: 概念與 interrupt 相似，但 interrupt 在 AArch64 中定義為外部觸發的 exception 。當 exception 出現時， PE 會將 program counter (PC) 切換至 exception handler 進行處理，此動作稱為 take exception 。當 exception handler 執行完畢後會回到原本的程式碼繼續執行，此動作稱為 return from exception 。
    - Synchronous exception
    - Asynchronous exception
- EL (Exception Level)：例外層級，共有四個層級，數字越大優先權 (privilege) 越高，此優先會影響可存取的記憶體 (memory privilege) 及可使用的處理器資源，需注意不同的 micro-architecture 支援的 EL 層級有所差異，使用前請查閱 TRM (Technical Reference Manual) 文件確認細節。
    - EL0：無特權模式 (unprivileged)， user mode 應用程式運作層級
    - EL1：作業系統核心模式 (OS kernel mode)，又可細分為 secure 與 normal 模式
    - EL2：虛擬機器監視器模式 (Hypervisor mode)，又可細分為 secure 與 normal 模式
    - EL3：TrustZone monitor mode
    ![image](https://hackmd.io/_uploads/BkSRP2aSC.png)

- Memory Privilege: 透過軟體建立 MMU (memory management unit) 的記憶體存取權限對應表，在 EL0 層級需有 unprivileged access permission 才可存取、在 EL1 、EL2 與 EL3 下需有 priviledge access permission 才可存取。
- EL 層級所使用的 execution state 由下一個更高 EL 層級的暫存器決定， execution State 可在處理器重置或 EL 層級變更時變更，但需注意 EL 層級變更時切換有下列限制，藉此實現可在 AArch64 上運作 AArch32 ，但無法在 AArch32 下運作 AArch64 。
    - EL 層級由低轉高時僅可切換至 AArch64
    - EL 層級由高轉低時僅可切換至 AArch32
    ![image](https://hackmd.io/_uploads/B1iJ_naH0.png)

- EL 層級切換僅在下列五種狀態下發生：
    - 執行 exception 任務 (taking an exception) 
        增加或維持於相同的 EL 層級
        - EL0 => EL1: SVC (system call)
        - EL1 => EL2: HVC (hypervisor call)
        - EL2 => EL3 / EL1 => EL3: SMC (secure monitor call)
    ![image](https://hackmd.io/_uploads/BkxA_87tR.png)
    - 完成 exception 任務後返回
        減少或維持於相同的 EL 層級
    - 重置處理器 (reset)
    - During Debug state
    - 離開 Debug state 

- Secutiry States: 安全狀態，分為 secure state 與 non-secure state 兩種狀態，安全狀態必須透過 EL3 層級切換，且不同架構的 EL 層級可支援的安全狀態有所差異。以 ARMv8-A 為例， EL3 層級固定為 secure state ；但在 ARMv9-A 中，若有實作 RME (Realm Management Extension) 則可同時支援兩種安全狀態。
    - secure state (secure world): PE 可存取 secure 與 non-secure 的實體記憶體區間，以及存取所有暫存器
    - non-secure state (normal world): PE 僅可存取 non-secure 的實體記憶體區間和 non-secure 的暫存器
    ![image](https://hackmd.io/_uploads/rJHZdhaB0.png)

- RME (Realm Management Extension): 於 ARMv9-A 架構推出的新功能，新增下列兩個種安全狀態，且 EL3 僅支援 root state ，以實現 TrustZone 機制。
    - Realm stat: PE 僅可存取 non-secure 與 Realm 物理記憶體區間
    - Root state: PE 可完整存取所有物理記憶體區間，且僅有 EL3 支援此狀態
- Processor State (PSTATE)
有別於 ARMv7 使用 CPSR (Current Program Status Register) 儲存當前 CPU 狀態，ARMv8 使用 PSTATE (Processor State) 儲存相關資訊，此暫存器的細部定義如下，需注意 EL0 僅可存取 N, Z, C, V ，而其他 Exception Level 則可存取所有 PSTAT 內容。
![image](https://hackmd.io/_uploads/BJ5sKw7YC.png)
    - N：Negative condition flag
    - Z：Zero condition flag
    - C: Carry condition flag
    - V：Overflow condition flag
    - SS：Software step bit
    - IL：Illegal execution state bit
    - D：Debug mask bit
    - A：SError mask bit
    - I：IRQ mask bit
    - F：FIQ mask bit
    - M：Execution state that the exception was taken from. A value of 0 indicates AArch64.
    - M[3:2]：Exception level
        - 2'b00 = EL0
        - 2'b01 = EL1
        - 2'b10 = EL2
        - 2'b11 = EL3
    - M[1]：nRW, Execution state
        - 0 = 64-bit
        - 1 = 32-bit
    - M[0]：SP, Stack Pointer selector
        - 0 = SP_EL0
        - 1 = SP_ELn

- [Secure Configuration Register, SCR_EL3](https://developer.arm.com/documentation/ddi0601/2020-12/AArch64-Registers/SCR-EL3--Secure-Configuration-Register?lang=en#fieldset_0-0_0)
PE 的 secure state (安全狀態)紀錄於 64bits 長度的 SCR_EL3 暫存器中，其 bit\[0] NS 用於標示當下的 Exception level 運作於哪個 secure state。
    - 1'b0：EL0 與 EL1 運作於 secure state
    - 1'b1：非 EL3 的 exception level 運作於 non-secure state
    
> [成大資工 Wiki - ARMv8](https://wiki.csie.ncku.edu.tw/embedded/ARMv8)
> [回顧 ARM 架構](https://hackmd.io/@owlfox/Bkcen7LeL/https%3A%2F%2Fhackmd.io%2Fs%2FrykYKYXjg)
> [Learn the architecture - Introducing the Arm architecture](https://developer.arm.com/documentation/102404/0201)
> [Arm Architecture Reference Manual for A-profile architecture](https://developer.arm.com/documentation/ddi0487/latest/)


### [ARM trusted firmware (ATF)](https://github.com/ARM-software/arm-trusted-firmware)
- 透過硬體資源的控制，將執行環境分割為 secure processing environment (SPE) 和 non secure processing environment (NSPE) 。 SPE 提供需要安全保護的服務 (e.g. 韌體更新、資料加解密)； NSPE 提供一般使用者的應用程式執行環境。兩者之間的資料交換需透過特定的 API 來傳輸，達到資料的操作權限控管。
![image](https://hackmd.io/_uploads/Sy70PK8PC.png)
- ATF 通常具備下列五種元件 (component) ，這五種元件由低至高同時代表著系統啟動時的順序。
    - Boot Loader stage 1 (BL1) AP Trusted ROM
    - Boot Loader stage 2 (BL2) Trusted Boot Firmware
    - Boot Loader stage 3-1 (BL31) EL3 Runtime Software
    - Boot Loader stage 3-2 (BL32) Secure-EL1 Payload (optional)
    - Boot Loader stage 3-3 (BL33) Non-trusted Firmware
    ![image](https://hackmd.io/_uploads/rJa7unaHC.png)

- SMC (Secure Monitor Call): Normal world 與 Secure world 對 BL31 發送的呼叫訊號，使用寫入特定暫存器數值進行實作。
- ERET (Exception Return): BL31 對於 SMC 回傳的執行結果。
![image](https://hackmd.io/_uploads/r1oS_nTSC.png)

- Secure-EL1 Payload: 泛指在 Secure-EL1 層級下運作的軟體，包含但不限於 Trusted OS 。 Secure-EL1 Dispatcher 負責初始化 Secure-EL1 Payload 和處理 Secure world 與 EL1 或 EL2 層級下的 Normal world 之間的溝通。溝通方式有：
    - 以 Fast SMC (定義於 SMC Calling Convention (SMCCC)）直接達成
    - 透過 BL31 的 PSCI SMC 間接達成

- PSCI (Power System Control Interface): 提供系統控制 CPU 下列運作狀態 控制介面。
    - 控制 CPU 核進入閒置模式 (idle)
    - 動態新增或關閉 CPU 核 (hotplug)
    - Secondary core boot
    - 將 trusted OS context 切換至其他核上運算
    - 系統關機或重置
- SCMI (System Control and Management Interface): 控制系統時脈、電源等的 API 介面。
- Boot up procedure
    實際的啟動流程與細節，不同的晶片流程會有所差異，故實際的流程需參考晶片供應商提供的資料。
    1. BL1 於 EL3 層級下執行，並載入於 Secure EL1 層級下運作的 BL2
    2. BL2 載入 BL31 、 BL32 與 BL33 ，並將需在 EL3 層級下運作的 BL31 記憶體位址傳遞給 BL1，由在 EL3 層級下運作的 BL1 觸發 BL31 運作
    3. BL31 觸發 BL32 與 BL33 開始運作，完成啟動流程後的 BL31 用於處理 secure 與 normal world 之間的 SMC 與 PSCI
        (1) BL32 在 Secure-EL1 層級下運作，載入並執行 trusted OS kernel 
        (2) BL33 依據所執行的任務而在不同的 EL 層即下運作。non secure OS 運作於 EL1 ； hypervisor 運作於 EL2 下
    ![image](https://hackmd.io/_uploads/B1nIu3aSA.png)

- SCP (System Control Processor) firmware: 負責底層硬體系統 (low-levle hardware system) 的電源啟動時序、硬體系統初始化、系統時脈管理和處理 OS Power Managment (OSPM) 的命令。
    - SCP boot ROM (SCP_BL1): 系統 cold reset 時第一個執行的程式，使用 Cortex-M3 處理器上的SCP 模組進行運算，執行基本的硬體初始化設定 (e.g. generic timer, clock, UART console)，並啟動 AP 處理器 (通常為 CPU0)
    - SCP runtime firmware (SCP_BL2): 初始化 SCP 週邊模組，設定系統控制器 (system controllers) 如 memory controller 和 coherent interconnect 。完成系統初始化設定後，透過 Message Handling Unit (MHU) 處理 SCMI 傳來的任務，如處理器電源管理與 Dynamic Voltage and Frequency Scaling (DVFS)
- AP (Application Processor) firmware: 使平台在 SCP 完成初始化後，接續處理系統開機流程，載入 Secure world image (e.g. Secure partition maanger, Trusted OS) ，使系統可運作 Rich Operating System (ROS) 。
    - AP trusted ROM (AP_BL1): 在 TF-A 中被稱為 AP Boot ROM
    - AP trusted boot firmware (AP_BL2)
    - AP EL3 runtime firmware (AP_BL31)
    - AP secure EL1 payload (AP_BL32)
    - AP Normal world firmware (AP_BL33)

- CPU 啟動流程
    - 共分為冷開機 (cold boot) 和暖開機 (warm boot) 兩種模式
        - 冷開機：系統從未送電的配置系開機
        - 暖開機：系統在有送電的狀態下重新啟動
    - 當系統擁有多個 CPU 時，系統會自動挑選其中一顆作為主要 CPU ，處理所有開機流程，剩餘的 CPU 則會初始化至安全模式後閒置，直到系統完成必要初始化後才會啟動剩餘的 CPU 。
    ![image](https://hackmd.io/_uploads/SygiiUmYA.png)


> [Understanding ARM Trusted Firmware using QEMU](https://lnxblog.github.io/2020/08/20/qemu-arm-tf.html)
> [Secure Boot using Trusted Firmware-M](https://yushuanhsieh.github.io/post/2022-01-28-tfm_boot/)
> [How ARM Systems are Booted: An Introduction to the ARM Boot Flow](https://www.youtube.com/watch?v=GXFw8SV-51g)
> [ARM Developer - SCP/AP Firmware](https://developer.arm.com/documentation/108028/0000/RD-TC22-software/Software-components)
> [Arm Power State Coordination Interface](https://developer.arm.com/documentation/den0022/latest/)
> 
:::warning
疑問
1. TF-A 介紹資料中提到 SCP (System Control Processor) 與 AP (Application Processor)，並分別有各自的韌體執行不同工作，為何特別需要獨立劃分 SCP 進行相關的開機流程與系統管理？
:::

## 編譯並安裝 QEMU
QEMU 模擬器可提供多樣平台和系統的模擬環境，省去硬體購置與架設成本，提升系統開發與實驗的便利性。以下為在 Ubuntu 22.04 上編譯並安裝 QEMU 9.0.0 ，首先確認安裝環境：
```shell
$ gcc --version
gcc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0

$ make --version
GNU Make 4.3

$ lscpu
Architecture:            x86_64
  CPU op-mode(s):        32-bit, 64-bit
  Address sizes:         39 bits physical, 48 bits virtual
  Byte Order:            Little Endian
CPU(s):                  8
  On-line CPU(s) list:   0-7
Vendor ID:               GenuineIntel
  Model name:            12th Gen Intel(R) Core(TM) i3-1215U
    CPU family:          6
    Model:               154
    Thread(s) per core:  2
    Core(s) per socket:  6
    Socket(s):           1
    Stepping:            4
    CPU max MHz:         4400.0000
    CPU min MHz:         400.0000
    BogoMIPS:            4992.00
```

下載 [QEMU 原始碼](https://download.qemu.org/)進行編譯與安裝，參考 QEMU 官網提供的安裝流程，在安裝 QEMU 前需先安裝下列軟體：
- git ：檔案版本控管軟體
- glib2.0-dev ：GTK C utility library ，可透過 zlib1g-dev 取得
- libfdt-devel ：Flat Device Trees manipulation library 

```shell
$ sudo apt-get install git libglib2.0-dev libfdt-dev libpixman-1-dev zlib1g-dev ninja-build
```

在目標資料夾下使用 Git 下載最新版的 QEMU 原始碼。
```shell
git clone https://github.com/qemu/qemu
```

在該資料夾中建立 build 資料夾，執行下列步驟進行編譯， QEMU 要編譯的檔案很多會跑好一陣子(以 9.0.0 版本為例，要編譯與連結的項目多達 9307 項)。
```shell
$ mkdir build
$ cd build
$ ../configure
$ make
```

執行 configure 時出現下列錯誤，系統提示未安裝 python3-vene 套件：
```shell
$ ../configure
python determined to be '/usr/bin/python3'
python version: Python 3.10.12

*** Ouch! ***

Python's ensurepip module is not found.
It's normally part of the Python standard library, maybe your distribution packages it separately?
Either install ensurepip, or alleviate the need for it in the first place by installing pip and setuptools for '/usr/bin/python3'.
(Hint: Debian puts ensurepip in its python3-venv package.) 



ERROR: python venv creation failed
```

解決辦法：安裝 python3-venv
```shell
$ sudo apt-get install python3-venv
```

重新執行 configure 過程又出現失敗，系統提示找不到 Flex 程式。 Flex (fast lexical analyzer generator) 是一種詞法分析程式，通常與 bison 一起運作，缺什麼補什麼，就把兩個程式一起安裝吧!
```shell
...
Program scripts/decodetree.py found: YES (/home/booleanii/linux2024/ATF/toolchain/qemu/build/pyvenv/bin/python3 /home/booleanii/linux2024/ATF/toolchain/qemu/scripts/decodetree.py)
Program flex found: NO

../target/hexagon/meson.build:292:8: ERROR: Program 'flex' not found or not executable

A full log can be found at /home/booleanii/linux2024/ATF/toolchain/qemu/build/meson-logs/meson-log.txt

ERROR: meson setup failed
```

解決辦法：安裝 Flex 與 bison 
```shell
$ sudo apt-get install flex
$ sudo apt-get install bison
```

完成編譯後使用 make install 安裝 QEMU
```shell
$ sudo make install
```

安裝完成後，可使用下列命令確認 QEMU 是否安裝成功，若安裝成功會顯示當前的版本資訊。
```shell
$ qemu-system-x86_64 --version
QEMU emulator version 9.0.50 (v9.0.0-1123-g74abb45dac)
Copyright (c) 2003-2024 Fabrice Bellard and the QEMU Project developers
```

或是輸入 qemu- 後連續按下兩次 tab 鍵，若出現下面的 qemu 輸出，代表安裝成功。
```shell
$ qemu-
qemu-aarch64              qemu-or1k                 qemu-system-microblazeel
qemu-aarch64_be           qemu-ppc                  qemu-system-mips
qemu-alpha                qemu-ppc64                qemu-system-mips64
qemu-arm                  qemu-ppc64le              qemu-system-mips64el
qemu-armeb                qemu-pr-helper            qemu-system-mipsel
qemu-cris                 qemu-riscv32              qemu-system-or1k
...
```

上述顯示資訊代表目前 QEMU 可支援的 CPU 種類，細部 Micro-architecture 支援清單可透過下列命令顯示。若清單上沒有想要的 Micro-architecture ，則可以檢查是否有新的 QEMU 版本被釋出。
```shell
$ qemu-system-aarch64 -M virt -cpu help
Available CPUs:
  a64fx
  arm1026
  arm1136
  arm1136-r2
  arm1176
  arm11mpcore
  arm926
  arm946
  cortex-a15
  cortex-a35
  cortex-a53
  cortex-a55
  cortex-a57
  cortex-a7
  cortex-a710
  cortex-a72
  cortex-a76
  cortex-a8
  cortex-a9
...
```
:::warning
疑問：
1. qemu-system-aarch64 和 qemu-system-arm 的支援清單僅有些許的不同，兩者在功能面上的差異為何？
:::

> [QEMU on Linux hosts](https://wiki.qemu.org/Hosts/Linux)
> [鳥哥 - 軟體安裝：原始碼與 Tarball](https://linux.vbird.org/linux_basic/centos7/0520source_code_and_tarball.php)

## 在 QEMU 上執行 TF-A
在 QEMU 中建立 Armv8-A 環境，並使用 TrustedFirmware-A (TF-A) 載入 non-secure OS。

### Toolchain
Toolchain 版本應盡可能與最小需求相同，否則不保證可以正常運作。 ATF 對於 Armv7-A 與 Armv8-A 裝置支援下列所有 Cross-compiler ；但對於 AArch32 與 AArch64 則需使用 arm-none-eabi 與 aarch64-none-elf 。

|       Program        | Min supported version |                      Note                       |
|:--------------------:|:---------------------:|:-----------------------------------------------:|
|     Arm Compiler     |         6.18          |                 cross-compiler                  |
|   Arm GNU Compiler   |         13.2          |                 cross-compiler                  |
|      Clang/LLVM      |        11.0.0         |                 cross-compiler                  |
| Device Tree Compiler |         1.4.7         |                 cross-compiler                  |
|       GNU make       |         3.81          |                                                 |
|       mbed TLS       |         3.6.0         |    For Trusted Board Boot and Measured Boot.    |
|       OpenSSL        |         1.0.0         | For cert_create, encrypt_fw, and fiptool tools. |
|        QCBOR         |          1.2          |        For DICE Protection Environment.         |
|       Node.js        |          16           |        For building TF-A documentation.         |
|        Poetry        |         1.3.2         |        For building TF-A documentation.         |
|        Sphinx        |         2.4.4         |        For building TF-A documentation.         |

若不撰寫文件，實際所需的安裝程式：
 - [Arm GNU Compiler](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)
     - Cross Compiler 讓編譯過程不受當下的平台限制，如在 x86 系統上編譯在 ARM 系統執行的程式，更可透過在性能較高的平台上進行編譯節省時間。 
     - Cross Compiler 的命名遵循 arch-vendor-(os-)abi 規範。 
         - arch: 代表運作平台 (e.g ARM, x86_64) 。
         - vendor: 代表提供此 cross compiler 的組織/廠商名稱，如標示為 none 則代表此編譯器是由開源社群共同維護 (e.g. GNU) 。
         - os: 用於標示目標平台所使用的作業系統，未標示代表執行檔運作於無作業系統的環境下 (俗稱 bare-metal)。 
         - abi: 標示所使用的 application binary interface 種類，如 eabi 代表 ARM EABI 、 gnueabi 代表 GNU EABI 。
     - 以下述三個 cross compiler 為範例
         - aarch64-none-elf: 用於編譯 AArch64 ELF bare-metal 程式 
         - arm-none-eabi: 用於編譯 AArch32 bare-metal 程式
         - arm-none-linux-gnueabihf: 用於編譯帶有浮點數運算模組 (hard float) 的 AArch32 GNU/Linux 程式
 - GNU make
 - Device Tree Compiler
    ```shell
    $ sudo apt install device-tree-compiler
    ```
 - OpenSSL
    ```shell
    $ sudo apt install openssl
    ...
    $ openssl version
    OpenSSL 3.0.2 15 Mar 2022 (Library: OpenSSL 3.0.2 15 Mar 2022)
    ```
> [TF-A Prerequisites](https://trustedfirmware-a.readthedocs.io/en/latest/getting_started/prerequisites.html#node-js)

### Install aarch64-linux-gnu
使用 apt install 安裝 aarch64-linux-gnu cross compiler ：
```shell
$ sudo apt install gcc-aarch64-linux-gnu
```

由版本資訊可發現 aarch64-linux-gnu-gcc 版本為 11.4.0 ，後續實測此版本可以正常編譯。
```shell
$ aarch64-linux-gnu-gcc --version
aarch64-linux-gnu-gcc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
```

### Install aarch64-none-elf
直接至 [Arm GNU Toolchain Downloads](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads) 網頁中下載對應平台的壓縮檔，解壓縮後可於其 bin 資料夾中看到 cross compiler 的執行檔。執行編譯前需指定編譯器執行檔所處的路徑 `path-to-aarch64-gcc` ：
```shell
$ export CROSS_COMPILE=<path-to-aarch64-gcc>/bin/aarch64-none-elf-
```

### Build non-TF image (Normal world UEFI)
建立 Normal world 開機所需的 UEFI 映像檔，參照下列命令，至 EDK2 Github 下載相關原始碼並進行編譯：
```shell
$ git clone https://github.com/tianocore/edk2.git
$ cd edk2
$ git submodule update --init
$ make -C BaseTools
$ source edksetup.sh
$ export GCC5_AARCH64_PREFIX=aarch64-linux-gnu-
$ build -a AARCH64 -t GCC5 -p ArmVirtPkg/ArmVirtQemuKernel.dsc
```

實際操作過程在 make -C BaseTools 失敗，看起來是編譯 EDK2 時出現問題。EDK2 為用於建立 UEFI 應用的開發環境， EDK2 BaseTools 則為使用 Python 建立相關功能的版本。
```shell
Finished building BaseTools C Tools with HOST_ARCH=X64
make[1]: Leaving directory '/home/booleanii/linux2024/ATF/nonTF_img/edk2/BaseTools/Source/C'
make -C Source/Python
make[1]: Entering directory '/home/booleanii/linux2024/ATF/nonTF_img/edk2/BaseTools/Source/Python'
make[1]: Nothing to be done for 'all'.
make[1]: Leaving directory '/home/booleanii/linux2024/ATF/nonTF_img/edk2/BaseTools/Source/Python'
make -C Tests
make[1]: Entering directory '/home/booleanii/linux2024/ATF/nonTF_img/edk2/BaseTools/Tests'
/bin/sh: 1: python: not found
make[1]: *** [GNUmakefile:11: test] Error 127
make[1]: Leaving directory '/home/booleanii/linux2024/ATF/nonTF_img/edk2/BaseTools/Tests'
make: *** [GNUmakefile:19: Tests] Error 2
```

依據 GNUmakefile 的錯誤提示，其在 Tests 資料夾中，因無法使用 python 命令而中斷。 Ubuntu 自 20.04 版本開始，內建 python 3 而非 python 。透過安裝 python-is-python3 ，使用 python 3 假冒 python ，提供 python 命令。
```shell
$ sudo apt install python-is-python3
...
$ python --version
Python 3.10.12
$ python3 --version
Python 3.10.12
```

安裝python-is-python3 後，便可完成 make -C BaseTools 命令。接續執行剩餘步驟，在 build 階段又出現問題：
```shell
/bin/sh: 1: iasl: not found
make: *** [GNUmakefile:434: /home/booleanii/linux2024/ATF/nonTF_img/edk2/Build/ArmVirtQemuKernel-AARCH64/DEBUG_GCC5/AARCH64/MdeModulePkg/Universal/Disk/RamDiskDxe/RamDiskDxe/OUTPUT/RamDisk.aml] Error 127


build.py...
 : error 7000: Failed to execute command
	make tbuild [/home/booleanii/linux2024/ATF/nonTF_img/edk2/Build/ArmVirtQemuKernel-AARCH64/DEBUG_GCC5/AARCH64/MdeModulePkg/Universal/Disk/RamDiskDxe/RamDiskDxe]


build.py...
 : error F002: Failed to build module
	/home/booleanii/linux2024/ATF/nonTF_img/edk2/MdeModulePkg/Universal/Disk/RamDiskDxe/RamDiskDxe.inf [AARCH64, GCC5, DEBUG]

- Failed -
Build end time: 00:38:51, Jun.03 2024
Build total time: 00:00:36
```

根據系統提示看起來缺少 isal 命令，該命令可藉由安裝 acpica-tools 取得，其為 Intel ACPI Component Architecture (ACPICA) 編譯器，用於將 ACPI Source Language (ASL) 檔案編譯為 ACPI Machine Language (AML) 檔案。透過下列命令進行 ACPICA 安裝與驗證：
```shell
$ sudo apt install acpica-tools
...
$ $ iasl -v

Intel ACPI Component Architecture
ASL+ Optimizing Compiler/Disassembler version 20200925
Copyright (c) 2000 - 2020 Intel Corporation
```

重新執行編譯，若獲得下列系統提示代表編譯成功，映像檔 QEMU_EFI.fd 位於 <edk2 路徑>/Build/ArmVirtQemuKernel-AARCH64/DEBUG_GCC5/FV/ 資料夾中。
```shell
Fd File Name:QEMU_EFI (/home/booleanii/linux2024/ATF/nonTF_img/edk2/Build/ArmVirtQemuKernel-AARCH64/DEBUG_GCC5/FV/QEMU_EFI.fd)

Generate Region at Offset 0x0
   Region Size = 0x8000
   Region Name = DATA

Generate Region at Offset 0x8000
   Region Size = 0x1F8000
   Region Name = FV

Generating FVMAIN_COMPACT FV

Generating FVMAIN FV
#######
Fd File Name:QEMU_VARS (/home/booleanii/linux2024/ATF/nonTF_img/edk2/Build/ArmVirtQemuKernel-AARCH64/DEBUG_GCC5/FV/QEMU_VARS.fd)

Generate Region at Offset 0x0
   Region Size = 0x40000
   Region Name = DATA

Generate Region at Offset 0x40000
   Region Size = 0x40000
   Region Name = DATA

Generate Region at Offset 0x80000
   Region Size = 0x40000
   Region Name = None

GUID cross reference file can be found at /home/booleanii/linux2024/ATF/nonTF_img/edk2/Build/ArmVirtQemuKernel-AARCH64/DEBUG_GCC5/FV/Guid.xref

FV Space Information
FVMAIN [99%Full] 6934080 (0x69ce40) total, 6934056 (0x69ce28) used, 24 (0x18) free
FVMAIN_COMPACT [51%Full] 2064384 (0x1f8000) total, 1056232 (0x101de8) used, 1008152 (0xf6218) free

- Done -
Build end time: 23:25:27, Jun.03 2024
Build total time: 00:00:30
```

### Build rootfs image (Normal World Initial RAM Filesystem)
建立 Normal world 開機所需的虛擬檔案系統 (initramfs) 映像檔，透過 Git 下載 Buildroot 原始碼並進行編譯，需注意最後的編譯需要一段時間。
```shell
git clone git://git.buildroot.net/buildroot.git
cd buildroot
make qemu_aarch64_virt_defconfig
utils/config -e BR2_TARGET_ROOTFS_CPIO
utils/config -e BR2_TARGET_ROOTFS_CPIO_GZIP
make olddefconfig
make
```
編譯完成後可在 output/images 資料夾下得到 rootfs.cpio.gz 檔案。

### Build Linux image (Normal Wrold OS)
建立 Normal world 下運作的 Linux 映像檔，首先使用 git clone 下載最新的 Linux 核心，並進入該資料夾中。
```shell
$ git clone https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git
$ cd linux-stable
```

第一次編譯 Linux 核心前，可先執行 make mrproper 將編譯過程的目標檔案以及設定檔移除，避免編譯過程使用錯誤的設定檔。後續重新編譯時，因不需刪除設定檔，只需使用 make clean 清除先前編譯到的 .o 檔案。
```shell
$ make mrproper
```

在進行 Linux 核心編譯前需設定核心功能配置，配置設定檔放在 arch/<平台名稱>/configs 資料夾內，若不使用現成的設定檔，編譯過程需手動選擇要如何配置相關設定，如果看不懂也可以使用 ? 觀看說明 (極度不建議這個方法，設定數量多到會懷疑人生)。另外一種設定方式，可使用 make menuconfig 以 GUI 介面設定核心配置。
```shell
$ make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig
$ make Image
...
kexec jump (KEXEC_JUMP) [N/y/?] (NEW) ?

CONFIG_KEXEC_JUMP:

Jump between original kernel and kexeced kernel and invoke
code in physical address mode via KEXEC

Symbol: KEXEC_JUMP [=n]
Type  : bool
Defined at kernel/Kconfig.kexec:90
  Prompt: kexec jump
  Depends on: ARCH_SUPPORTS_KEXEC_JUMP [=y] && KEXEC [=y] && HIBERNATION [=y]
  Location:
    -> General setup
      -> Kexec and crash features
        -> kexec jump (KEXEC_JUMP [=n])
```

在這邊選擇使用默認的設定 defconfig 進行編譯，編譯完成的映像檔會儲存於 arch/<平台名稱>/boot 資料夾中。
```shell
$ make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig
$ make -j2 ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image
```

編譯過程出現 fatal error ，內容為找不到 openssl/bio.h 檔案，雖然先前建立 toolchain 時有安裝 openssl ，但在 /usr/include 路徑下並沒有該檔案。
```shell
certs/extract-cert.c:21:10: fatal error: openssl/bio.h: No such file or directory
   21 | #include <openssl/bio.h>
      |          ^~~~~~~~~~~~~~~
compilation terminated.
```

解決方法為安裝 OpenSSL 的開發套件，安裝完成後即可在 /usr/include 路徑下找到 bio.h 檔案，重新執行 Linux 核心編譯便可成功完成。並在 linux/arch/arm64/boot/ 路徑下獲得 Image Linux 核心映像檔。
```shell
$ sudo apt update
$ sudo apt install libssl-dev
```

### Build TrustedFirmware-A (TF-A Bootloader)
TF-A 是基於 ARM Trusted Firmware 的一個分支，其應用於 ARMv8-A 架構，專案原始碼可透過 [GitHub](https://github.com/ARM-software/arm-trusted-firmware) 取得，下載完成後開啟 arm-trusted-firmware 資料夾。
```shell
$ git clone https://github.com/ARM-software/arm-trusted-firmware
$ cd arm-trusted-firmware
```
編譯 TF-A 前需執行 cross compiler 設定，因目標是在 QEMU 中模擬 ARMv8-A 64bit 系統，故選用 AArch64 cross compiler 進行編譯，`PLAT` 選項設定為 qemu ，若此選項未設定則會默認為使用 FVP (Fixed Virtual Platform) 。
```shell
$ export CROSS_COMPILE=<path-to-aarch64-gcc>/bin/aarch64-none-elf-
$ make PLAT=qemu 
```

### Booting via semi-hosting on QEMU
完成 TF-A 編譯後，移動當前路徑至 build/qemu/release 資料夾，可在資料夾中看到編譯完成的映像檔：
- bl1.bin
- bl2.bin
- bl31.bin

將方才編譯完成的 rootfs.cpio.gz 、 QEMU_EFI.fd 和 Linux 核心 (Image) 映像檔複製到此資料夾內，須注意下列檔案名稱需要修改：
- QEMU_EFI.fd -> bl33.bin

除了 bl1.bin 檔需特別標示在選項中外， QEMU 啟動時會自動載入其餘的映像檔。
```shell
$ qemu-system-aarch64 -nographic -machine virt,secure=on -cpu cortex-a57  \
    -kernel Image                           \
    -append "console=ttyAMA0,38400 keep_bootcon"   \
    -initrd rootfs.cpio.gz -smp 2 -m 1024 -bios bl1.bin   \
    -d unimp -semihosting-config enable=on,target=native
```
使用到的 QEMU 命令選項說明如下：
- -nographic: 當 QEMU 編譯時設定為支援視窗化介面，於開啟時會自動使用視窗介面，需使用此選項強制使用 command line 介面
- -machine: 指定模擬的機器（平台），[virt](https://www.qemu.org/docs/master/system/arm/virt.html) 代表模擬對象為虛擬硬體平台，無實際硬體平台的系統特性限制。secure 選項代表是否要模擬 ARM Security Extension (TrustZone)
- -cpu: 指定模擬的 CPU 型號
- [-kernel](https://www.qemu.org/docs/master/system/invocation.html#hxtool-8): normal world 的 kernel 映像檔
- -append: kernel command line
- -initrd: 載入 normal mode 所需的 initial ram disk 映像檔
- -smp: 模擬的對稱式多處理器 (Symmetric multiprocessing CPU) 的核數量 (1 個 socket N 個核)
- -m: 模擬的 RAM 記憶體空間 (Unit: MB)，若設定為 1G 則為 1GB
- -bios: BIOS 檔案，該檔案將存入 ROM 或 FLASH 區間中，於系統開機時進行讀取與執行
- -d: 開啟 Log 紀錄功能，可使用 -d help 命令觀看詳細說明，設定為 unimp 可紀錄 unimplemented functionality
- [-semihosting-config](https://www.qemu.org/docs/master/about/emulation.html#semihosting): 由於晶片並不一定可以透過 I/O 完整讀取內部資訊，故實際硬體開發上會需要 ICE (In-circuit emulator) 讀取內部資訊，而 QEMU 中透過 semihosting 建立相關機制，讓 Host 端的程式可與 Guest 進行互動，請注意此功能可讓 Guest 直接存取 host 端的檔案系統。

當系統成功完成開機流程時，將看到下列登入訊息，輸入 root 後即可進入 normal world 下運作的 Linux 系統。
```shell
[    0.886981] Freeing unused kernel memory: 10176K
[    0.888107] Run /init as init process
Saving 256 bits of non-creditable seed for next boot
Starting syslogd: OK
Starting klogd: OK
Running sysctl: OK
Starting network: udhcpc: started, v1.36.1
udhcpc: broadcasting discover
udhcpc: no lease, forking to background
OK

Welcome to Buildroot
buildroot login:
```

若要退出 QEMU ，可透過 ctrl + a 後再按下 x 鍵，或是直接將該終端機關閉即可。

## 觀察 TF-A 的開機流程
目標為使用 GDB 分析 TF-A 開機流程，在此之前可以先使用 TF-A 提供的 Memory Layout 工具，了解 Bootloader 的記憶體使用情形。

### Bootloader Image Memory Layout
每個 bootloader (BL) 映像檔依據資料內容可分為 PROGBITS 和 NOBITS 兩大區域。除了 BL31 的 NOBITS 可由使用者另外定義於映像檔中的擺放位置外，所有的 PROGBITS 會被放置在映像檔的起始位置，接續為 NOBITS 區域以節省映像檔大小，細部的擺放位置會在 linker script 中描述。
1. static content: 靜態項目，實際的變數內容會被儲存於映像檔中，隸屬於 ELF 檔案中 的 PROGBITS 區域。
2. run-time content: 執行階段項目，映像檔中僅紀錄描述該變數在執行階段會被宣告與儲存的位置 (metadata) ，隸屬於 ELF 檔案中 的 NOBITS 區域。

當 BL31 的 NOBITS 要擺放在非緊鄰 PROGBITS 區域後，需設定下列參數：
- SEPARATE_NOBITS_REGION: 設定為 1
- BL31_NOBITS_BASE
- BL31_NOBITS_LIMIT

所有的 BL 映像檔必須具備下列條件：
- .bss 區間中的變數在被 C 語言執行前皆須被初始化為 0
- 共享記憶體區間 (coherent memory) 須被初始化為 0
- 記憶體管理單元 (MMU) 的設定程式碼須完成共享記憶體與唯讀記憶體 (Read Only Memory, ROM) 區間的位址配置設定。若 SEPARATE_CODE_AND_RODATA 設定為 1 ，則需額外定義唯讀記憶體的空間配置細節。

BL1 儲存於 ROM 區間，運作時直接從 ROM fetch 指令進處理器中執行，然而其 .data 區間的變數在運作時須在可讀寫記憶體中宣告與使用，在記憶體中的存放位址是從該記憶體的最高位址往回儲存。

不同的運算平台的記憶體空間配置可能會有所不同，當前的 TF-A 韌體版本尚未支援映像檔動態載入 (Dynamic Image Loading) ，開發人員須依據平台規格設定映像檔在執行時期的記憶體擺放位置 (link map) ，避免映像檔的使用空間重疊問題。 

BL 映像檔的 link map 存放於 &lt;TF-A PATH>/build/&lt;platform>&lt;/platform>/&lt;build-type>/bl&lt;x>/ 資料夾下的 bl&lt;x>.map 檔案中， &lt;x> 代表 BL階段代號 (e.g. bl1, bl2, bl31)。以下列 bl1.map 內容為例，對 BL1 使用的 RAM 與 ROM 記憶體區間進行描述。可看到 BL1 所使用的 RAM 區間為 0x0000_0000_0e0e_e000 開始到 0x0000_0000_0e0f_6000 。

```shell
0x000000000e0ee000                __BL1_RAM_START__ = ADDR (.data)
0x000000000e0f6000                __BL1_RAM_END__ = .
0x0000000000005a00                __DATA_ROM_START__ = LOADADDR (.data)
0x0000000000000105                __DATA_SIZE__ = SIZEOF (.data)
0x0000000000005b05                __BL1_ROM_END__ = (__DATA_ROM_START__ + __DATA_SIZE__)
```

> [Memory Layout of TF-A BL Images](https://github.com/ARM-software/arm-trusted-firmware/blob/master/docs/design/firmware-design.rst#memory-layout-of-bl-images)

### [TF-A Memory Layout Tool](https://trustedfirmware-a.readthedocs.io/en/latest/tools/memory-layout-tool.html)
透過此工具可以將 TF-A 的記憶體配置資訊進行彙整與視覺化，但在使用前需安裝相關的 Python 套件。
1. 首先是用於安裝與管理 Python 應用程式的 fpipx ，安裝方式如下：
```shell
$ sudo apt update
$ sudo apt install pipx
$ pipx ensurepath
```
2. 接續使用 pipx 安裝 poetry ，此程式用於管理 Python 的虛擬環境、套件間的相依性和套件的封装 (packaging)。
```shell
$ pipx install poetry
$ poetry install --with memory
```

注意須在 TF-A 資料夾下執行 memory 安裝，否則系統會回報下列錯誤訊息。
```shell
$ poetry install --with memory
    
Poetry could not find a pyproject.toml file in /home/booleanii or its parents
```

安裝完成後可透過 --help 參數確認是否正常運作。
```shell
$ poetry run memory --help
Usage: memory [OPTIONS]

Options:
  -r, --root PATH                 Root containing build output.
  -p, --platform TEXT             The platform targeted for analysis.
                                  [default: fvp]
  -b, --build-type [debug|release]
                                  The target build type.
  -f, --footprint                 Generate a high level view of memory usage
                                  by memory types.
  -t, --tree                      Generate a hierarchical view of the modules,
                                  segments and sections.
  --depth INTEGER                 Generate a virtual address map of important
                                  TF symbols.
  -s, --symbols                   Generate a map of important TF symbols.
  -w, --width INTEGER
  -d                              Display numbers in decimal base.
  --no-elf-images                 Analyse the build's map files instead of ELF
                                  images.
  --help                          Show this message and exit.
```

在 TF-A 資料夾下，使用 run memory 命令顯示 ELF 檔案的 memory layout 情形。需注意如未設定平台參數 -p ， poetry 將會默認找尋並分析 FVP 平台的 ELF 檔案內容。從下列的執行結果可以看到，先前建立的 BL1 、 BL2 和 BL31 映像檔的記憶體配置情形。
```shell
$ poetry run memory -s -p qemu
build-path: /mnt/data_partition/ATF/toolchain/arm-trusted-firmware/build/qemu/release
Memory Layout:
            +----------__BL1_RAM_END__----------+-----------------------------------+-----------------------------------+
0x00e0f6000 +--------__XLAT_TABLE_END__---------+                                   |                                   |
0x00e0f0000 +-------__XLAT_TABLE_START__--------+                                   |                                   |
            +------__BASE_XLAT_TABLE_END__------+                                   |                                   |
0x00e0ef9e0 +------------__BSS_END__------------+                                   |                                   |
            +-----__BASE_XLAT_TABLE_START__-----+                                   |                                   |
            +---__PMF_PERCPU_TIMESTAMP_END__----+                                   |                                   |
            +-------__PMF_TIMESTAMP_END__-------+                                   |                                   |
0x00e0ef9c0 +------__PMF_TIMESTAMP_START__------+                                   |                                   |
            +-----------__BSS_START__-----------+                                   |                                   |
0x00e0ef140 +----------__STACKS_END__-----------+                                   |                                   |
0x00e0ee140 +---------__STACKS_START__----------+                                   |                                   |
0x00e0ee105 +---------__DATA_RAM_END__----------+                                   |                                   |
            +---------__BL1_RAM_START__---------+                                   |                                   |
0x00e0ee000 +--------__DATA_RAM_START__---------+                                   |                                   |
0x00e0d8000 |                                   |                                   +--------__XLAT_TABLE_END__---------+
0x00e0d2000 |                                   |                                   +-------__XLAT_TABLE_START__--------+
            |                                   |                                   +------__BASE_XLAT_TABLE_END__------+
0x00e0d18a0 |                                   |                                   +------------__BSS_END__------------+
            |                                   |                                   +-----__BASE_XLAT_TABLE_START__-----+
            |                                   |                                   +---__PMF_PERCPU_TIMESTAMP_END__----+
            |                                   |                                   +-------__PMF_TIMESTAMP_END__-------+
0x00e0d1880 |                                   |                                   +------__PMF_TIMESTAMP_START__------+
            |                                   |                                   +-----------__BSS_START__-----------+
0x00e0cc080 |                                   |                                   +----------__STACKS_END__-----------+
0x00e0ac080 |                                   |                                   +---------__STACKS_START__----------+
            |                                   |                                   +-----------__RELA_END__------------+
0x00e0ac070 |                                   |                                   +----------__RELA_START__-----------+
0x00e0ac000 |                                   |                                   +----------__RODATA_END__-----------+
0x00e0ab4c8 |                                   |                                   +-----__RODATA_END_UNALIGNED__------+
            |                                   |                                   +----------__CPU_OPS_END__----------+
            |                                   |                                   +------------__GOT_END__------------+
0x00e0ab4b0 |                                   |                                   +-----------__GOT_START__-----------+
            |                                   |                                   +---------__CPU_OPS_START__---------+
            |                                   |                                   +------__FCONF_POPULATOR_END__------+
            |                                   |                                   +-----__FCONF_POPULATOR_START__-----+
            |                                   |                                   +-------__PMF_SVC_DESCS_END__-------+
0x00e0ab0f0 |                                   |                                   +------__PMF_SVC_DESCS_START__------+
            |                                   |                                   +---------__RODATA_START__----------+
            |                                   |                                   +------__TEXT_END_UNALIGNED__-------+
0x00e0aa000 |                                   |                                   +-----------__TEXT_END__------------+
0x00e0a0000 |                                   |                                   +----------__TEXT_START__-----------+
0x00e079000 |                                   +--------__XLAT_TABLE_END__---------+                                   |
0x00e073000 |                                   +-------__XLAT_TABLE_START__--------+                                   |
            |                                   +------__BASE_XLAT_TABLE_END__------+                                   |
0x00e072660 |                                   +------------__BSS_END__------------+                                   |
            |                                   +-----__BASE_XLAT_TABLE_START__-----+                                   |
            |                                   +---__PMF_PERCPU_TIMESTAMP_END__----+                                   |
            |                                   +-------__PMF_TIMESTAMP_END__-------+                                   |
0x00e072640 |                                   +------__PMF_TIMESTAMP_START__------+                                   |
            |                                   +-----------__BSS_START__-----------+                                   |
0x00e072240 |                                   +----------__STACKS_END__-----------+                                   |
0x00e071240 |                                   +---------__STACKS_START__----------+                                   |
0x00e071000 |                                   +----------__RODATA_END__-----------+                                   |
            |                                   +----------__CPU_OPS_END__----------+                                   |
            |                                   +---------__CPU_OPS_START__---------+                                   |
            |                                   +------__FCONF_POPULATOR_END__------+                                   |
            |                                   +-----__FCONF_POPULATOR_START__-----+                                   |
            |                                   +------------__GOT_END__------------+                                   |
            |                                   +-----------__GOT_START__-----------+                                   |
            |                                   +-------__PMF_SVC_DESCS_END__-------+                                   |
            |                                   +------__PMF_SVC_DESCS_START__------+                                   |
0x00e070c10 |                                   +-----__RODATA_END_UNALIGNED__------+                                   |
            |                                   +---------__RODATA_START__----------+                                   |
            |                                   +------__TEXT_END_UNALIGNED__-------+                                   |
0x00e070000 |                                   +-----------__TEXT_END__------------+                                   |
0x00e06b000 |                                   +----------__TEXT_START__-----------+                                   |
0x000005b05 +----------__BL1_ROM_END__----------+                                   |                                   |
            +----------__CPU_OPS_END__----------+                                   |                                   |
            +--------__DATA_ROM_START__---------+                                   |                                   |
            +------------__GOT_END__------------+                                   |                                   |
            +-----------__GOT_START__-----------+                                   |                                   |
            +-----__RODATA_END_UNALIGNED__------+                                   |                                   |
0x000005a00 +----------__RODATA_END__-----------+                                   |                                   |
            +---------__CPU_OPS_START__---------+                                   |                                   |
            +------__FCONF_POPULATOR_END__------+                                   |                                   |
            +-----__FCONF_POPULATOR_START__-----+                                   |                                   |
            +-------__PMF_SVC_DESCS_END__-------+                                   |                                   |
0x000005700 +------__PMF_SVC_DESCS_START__------+                                   |                                   |
            +---------__RODATA_START__----------+                                   |                                   |
            +------__TEXT_END_UNALIGNED__-------+                                   |                                   |
0x000004000 +-----------__TEXT_END__------------+                                   |                                   |
0x000000000 +----------__TEXT_START__-----------+-----------------------------------+-----------------------------------+
                            bl1                                 bl2                                 bl31
```
    
如果只是要觀察記憶體的使用概況，則可使用 -f 參數獲得更精簡的資訊。由下列資訊可看出 BL1 儲存於 Address 0x0000_0000 ~ 0x0000_5A00 之間的 Trusted ROM 中，並於執行過程將資量放至於 0x0E0E_E000 ~ 0x0E0F_6000 的 Trusted RAM 中； BL2 則是使用 0x0E06_B000 ~ 0x0E07900 的 Trusted RAM； BL31 使用 0x0E0A_0000 ~ 0x0E0D_8000 之間的 Trusted RAM 中。


```shell
$ poetry run memory -f -p qemu
build-path: /mnt/data_partition/ATF/toolchain/arm-trusted-firmware/build/qemu/release
+----------------------------------------------------------------------------+
|                         Memory Usage (bytes) [RAM]                         |
+-----------+------------+------------+------------+------------+------------+
| Component |   Start    |   Limit    |    Size    |    Free    |   Total    |
+-----------+------------+------------+------------+------------+------------+
|    BL1    |    e0ee000 |    e100000 |       8000 |       a000 |      12000 |
|    BL2    |    e06b000 |    e0a0000 |       e000 |      27000 |      35000 |
|    BL31   |    e0a0000 |    e100000 |      38000 |      28000 |      60000 |
+-----------+------------+------------+------------+------------+------------+ 

+----------------------------------------------------------------------------+
|                         Memory Usage (bytes) [ROM]                         |
+-----------+------------+------------+------------+------------+------------+
| Component |   Start    |   Limit    |    Size    |    Free    |   Total    |
+-----------+------------+------------+------------+------------+------------+
|    BL1    |          0 |      20000 |       5a00 |      1a600 |      20000 |
+-----------+------------+------------+------------+------------+------------+ 
```

映像檔中的區間資訊，可以使用 -t 參數以樹狀結構顯示。
```shell
$ poetry run memory -t -p qemu
build-path: /mnt/data_partition/ATF/toolchain/arm-trusted-firmware/build/qemu/release
name                                     start        end       size
bl1                                          0      32000      32000
├── 00                                       0       5a00       5a00
│   ├── .text                                0       4000       4000
│   └── .rodata                           4000       5a00       1a00
├── 01                                 e0ee000    e0ee105        105
│   └── .data                          e0ee000    e0ee105        105
├── 02                                 e0ee140    e0ef140       1000
│   └── .stacks                        e0ee140    e0ef140       1000
├── 04                                 e0ef140    e0ef9e0        8a0
│   └── .bss                           e0ef140    e0ef9e0        8a0
└── 03                                 e0f0000    e0f6000       6000
    └── .xlat_table                    e0f0000    e0f6000       6000
bl2                                    e06b000    e0a0000      35000
├── 00                                 e06b000    e071000       6000
│   ├── .text                          e06b000    e070000       5000
│   └── .rodata                        e070000    e071000       1000
└── 01                                 e071000    e079000       8000
    ├── .data                          e071000    e071201        201
    ├── .stacks                        e071240    e072240       1000
    ├── .bss                           e072240    e072660        420
    └── .xlat_table                    e073000    e079000       6000
bl31                                   e0a0000    e100000      60000
├── 00                                 e0a0000    e0aa000       a000
│   └── .text                          e0a0000    e0aa000       a000
└── 01                                 e0aa000    e0d8000      2e000
    ├── .rodata                        e0aa000    e0ac000       2000
    ├── .data                          e0ac000    e0ac06c         6c
    ├── .stacks                        e0ac080    e0cc080      20000
    ├── .bss                           e0cc080    e0d18a0       5820
    └── .xlat_table                    e0d2000    e0d8000       6000
```

> [Python Poetry Introduction](https://python-poetry.org/docs/)

### Rebuild TrustedFirmware-A (TF-A)
如同使用 gcc 編譯時需要加入 -g 選項，為了要讓 GDB 可以追蹤 TF-A bootloader 的執行流程，需在 TF-A 編譯時建立可被 GDB 追蹤的 symbol 。參考 Build TrustedFirmware-A (TF-A) 章節中的描述，於 make 命令增加 DEBUG 選項。
```shell
$ export CROSS_COMPILE=<path-to-aarch64-gcc>/bin/aarch64-none-elf-
$ make PLAT=qemu DEBUG=1
```

編譯完成後，可在 build/qemu/debug 資料夾中看到 bootloader 映像檔。同樣將 rootfs.cpio.gz 、 QEMU_EFI.fd 和 Linux 核心映像檔複製到該資料夾中，並修改為指定的檔案名稱。
- bl1.bin
- bl2.bin
- bl31.bin
- rootfs.cpio.gz 
- bl33.bin
- Image

### Install gdb-multiarch
由於 GDB 通常只支援 host 電腦的 CPU 架構除錯，故需先確認當前的 GDB 是否支援 aarch64 架構。在 GDB 中使用 set architecture 命令搭配 tab 鍵，列出 GDB 支援的 CPU 種類，從顯示結果可看出目前 GDB 並不支援 aarch64 ，需安裝支援多種 CPU 架構的 gdb-multiarch 。
```shell
(gdb) set architecture 
auto               i386:intel         i386:x64-32:intel  i386:x86-64:intel
i386               i386:x64-32        i386:x86-64        i8086
```
gdb-multiarch 安裝方式：
```shell
$ sudo apt-get upgrade
$ sudo apt-get install gdb-multiarch
```
完成安裝後透過 gdb-multiarch 命令開啟，使用 set architecture 命令搭配 tab 鍵，可以看到其支援的 CPU 架構數量明顯多了不少，並可以看到 aarch64 在支援的清單中。
```shell
$ gdb-multiarch 
...

(gdb) set architecture 
Display all 200 possibilities? (y or n)
aarch64                        mips:4120
aarch64:armv8-r                mips:4300
aarch64:ilp32                  mips:4400
alpha                          mips:4600
alpha:ev4                      mips:4650
alpha:ev5                      mips:5000
alpha:ev6                      mips:5400
arm                            mips:5500
arm_any                        mips:5900
armv2                          mips:6000
armv2a                         mips:7000
armv3                          mips:8000
armv3m                         mips:9000
armv4                          mips:gs264e
armv4t                         mips:gs464
armv5                          mips:gs464e
armv5t                         mips:interaptiv-mr2
armv5te                        mips:isa32
armv5tej                       mips:isa32r2
armv6                          mips:isa32r3
armv6-m                        mips:isa32r5
armv6k                         mips:isa32r6
armv6kz                        mips:isa64
--More--
```
### Connect GDB to QEMU
QEMU 使用 -s 選項可透過 TCP port 1234 連接 GDB 進行除錯，而透過 -S 可要求 QEMU 等待 GDB 連線後才開始執行。開啟兩個終端機，一邊執行新增 -s 和 -S 選項的 qemu 命令 。
```shell
$ qemu-system-aarch64 -nographic -machine virt,secure=on -cpu cortex-a57  \
    -kernel Image                           \
    -append "console=ttyAMA0,38400 keep_bootcon"   \
    -initrd rootfs.cpio.gz -smp 2 -m 1024 -bios bl1.bin   \
    -d unimp -semihosting-config enable=on,target=native   \
    -s -S
```

另外一邊執行 gdb-multiarch 命令，使用 -ex 選項於啟動 gdb 時執行 file 命令載入被追蹤檔案，載入檔案的路徑可使用執行 gdb 時的位置資料夾開始的相對路徑，或是直接使用絕對路徑。
```shell
$ gdb-multiarch -ex "file /mnt/data_partition/ATF/toolchain/arm-trusted-firmware/build/qemu/debug/bl1/bl1.elf"
```

亦可在進入 gdb 後使用 file 命令載入被追蹤檔案。
```shell
(gdb) file /mnt/data_partition/ATF/toolchain/arm-trusted-firmware/build/qemu/debug/bl1/bl1.elf
```

請注意 file 命令僅能載入一個可追蹤檔案，要同時追蹤 bl1 以外的 bootloader 行為，可以使用 add-symbol-file 命令載入其他 elf 檔。
```shell
(gdb) add-symbol-file bl2/bl2.elf 
add symbol table from file "bl2/bl2.elf"
(y or n) y
Reading symbols from bl2/bl2.elf..
```

完成被追蹤檔案載入後，使用 TCP port 1234 與 QEMU 連接。
```shell
(gdb) target remote localhost:1234
```

可將上述命令撰寫為 scrip file ，進入GDB後使用 source 命令載入，省去實驗過程重複輸入指令的時間。需注意部分的中斷條件會拉慢 QEMU 模擬速度，可待接近該中斷位置時再加入進行觀察。
```shell
file bl1/bl1.elf
add-symbol-file bl2/bl2.elf
add-symbol-file bl31/bl31.elf
target remote localhost:1234
watch *0xe0ee000
watch $pc > 0x60000000
break bl1_entrypoint
break bl2_entrypoint
break bl2_load_images
break smc
break bl1_print_next_bl_ep_info
break bl31_entrypoint
break el3_exit
```

```shell
(gdb) source GDB_Script 
add symbol table from file "bl2/bl2.elf"
add symbol table from file "bl31/bl31.elf"
bl1_run_bl2_in_root () at bl1/aarch64/bl1_entrypoint.S:86
86		adrp	x20, bl2_ep_info
Hardware watchpoint 1: *0xe0ee000
Breakpoint 2 at 0x0: file bl1/aarch64/bl1_entrypoint.S, line 86.
Breakpoint 3 at 0xe06b000: file bl2/aarch64/bl2_entrypoint.S, line 22.
Breakpoint 4 at 0xe06b090: file bl2/bl2_image_load_v2.c, line 36.
Breakpoint 5 at 0x0: smc. (2 locations)
Breakpoint 6 at 0x3a8: file bl1/bl1_main.c, line 210.
Breakpoint 7 at 0xe0a0000: file bl31/aarch64/bl31_entrypoint.S, line 30.
Breakpoint 8 at 0x0: smc. (2 locations)
Breakpoint 9 at 0x4420: el3_exit. (2 locations)
```


>[QEMU GDB usage](https://www.qemu.org/docs/master/system/gdb.html)

### QEMU virt ARMv8-A TF-A Boot Flow via semi-hosting option
![semihosting_boot](https://hackmd.io/_uploads/SkWetumYA.png)

1. Run BL1 from Trusted ROM
    - Exception Level: EL3
    - Function: bl1_entrypoint
    - 從 Trusted ROM 記憶體位址 0x0_0000_0000 開始執行 AP Boot ROM 開機流程。

2. Load BL1 data to Trusted RAM
    - Exception Level: EL3
    - Function: bl1_main
    - 將 BL1 運作所需的資料儲存至 Trusted RAM 的 0x0_0E0E_E000 到 0x0_0E0F_6000 區間中。

3. Load BL2 Image to Trusted RAM
    - Exception Level: EL3
    - Function: bl1_main
    - BL1 透過 load_auth_image 函式載入 Image ID 1 的 BL2 映像檔到 Trusted RAM 0x0_0E06_B000 位址。

4. Run BL2 from Trusted RAM
    - Exception Level: Secure-EL1
    - Function: bl2_entrypoint
    - 完成 BL1 初始化階段，呼叫 el_exit 函式並接著開始執行 Trusted Boot Firmware 開機流程。

5. Load BL31 Image to Trusted RAM
    - Exception Level: Secure-EL1
    - Function: bl2_main
    - BL2 透過 load_auth_image 函式，以 semi-hosting 的途徑，將 Image ID 3 的 BL31 映像檔載入至 Trusted RAM 0x0_0E0E_A000 位址。

6. Load BL33 Image to Non-Trusted RAM
    - Exception Level: Secure-EL1
    - Function: bl2_main
    - BL2 透過 load_auth_image 函式，以 semi-hosting 的途徑，將 Image ID 5 的 BL33 映像檔載入至 Non-Trusted RAM 0x0_6000_0000 位址。

7. Handoff to BL31 by SMC
    - Exception Level: EL3
    - Function: smc_handler64 
    - 在 Secure EL1 下運作的 BL2 無法直接切換至 EL1 下的 BL33，需透過 SMC 呼叫透過 BL1 切換至 BL31，再由 BL31 將控制權切換至 BL33。相關流程可在 bl1_print_next_bl_ep_info 函式設定中斷點進行觀察。

8. Run BL31 from Trusted RAM
    - Exception Level: EL3
    - Function: bl31_entrypoint
    - 執行 EL3 Runtime Firmware 開機流程，建立運作所需的 MMU 與環境設定後，切換至 BL33。

9. Run BL33 from Non-Trusted RAM
    - Exception Level: EL1
    - 執行 Non-Trusted Firmware 開機流程，載入 Normal world OS 的 UEFI 與系統核心，完成後開始運作 Normal world 作業系統。

## OP-TEE (Open Portable Trusted Execution Environment)
為了能觀察完整的 ARM trusted firmware 的運作，選用 OP-TEE (Open Portable Trusted Execution Environment) 作為 TF-A 中沒有包含到的 BL32 Bootloader。

OP-TEE 是一個基於 ARM TrustZone 硬體隔離運算機制技術進行實作的信任運算環境 (Trusted Execution Environment, TEE) ，建立滿足下列三個目標的 TEE Internal Core API ：
- Isolation: 使用硬體功能達到 Normal World 與 Secure World 的隔離機制
- Small footprint: 於 ARM 平台上可達成低記憶體使用量
- Portability: 可移植到不同的信任運算環境與作業系統下運作

[OP-TEE 專案](https://github.com/OP-TEE)分為三大部分：
- [optee_os](https://github.com/OP-TEE/optee_os)
    於 secure EL-1 層級下運作，提供 Secure World OS 運作所需的中斷處理、執行序處理與記憶體管理等 API 功能，並且能夠建立在 Secure EL-0 層級下運作的 Trusted Application 。
- optee_client
    提供 Normal World OS 下的 Client Application 呼叫 Trusted Application 所需 API 功能。
- optee_test (xtest)
    提供 OP-TEE 的健全性測試 (sanity test)，測試 OP-TEE 韌體可否正常運作。

> [About OP-TEE](https://optee.readthedocs.io/en/latest/general/about.html)

### Build and Run OP-TEE
參考 OP-TEE 的 [Prerequisties ](https://optee.readthedocs.io/en/latest/building/prerequisites.html#prerequisites)網頁描述，安裝編譯與運作所需的程式。
```shell
$ sudo apt-get install \
    adb \
    acpica-tools \
    autoconf \
    automake \
    bc \
    bison \
    build-essential \
    ccache \
    cpio \
    cscope \
    curl \
    device-tree-compiler \
    e2tools \
    expect \
    fastboot \
    flex \
    ftp-upload \
    gdisk \
    git \
    libattr1-dev \
    libcap-ng-dev \
    libfdt-dev \
    libftdi-dev \
    libglib2.0-dev \
    libgmp3-dev \
    libhidapi-dev \
    libmpc-dev \
    libncurses5-dev \
    libpixman-1-dev \
    libslirp-dev \
    libssl-dev \
    libtool \
    libusb-1.0-0-dev \
    make \
    mtools \
    netcat \
    ninja-build \
    python3-cryptography \
    python3-pip \
    python3-pyelftools \
    python3-serial \
    python-is-python3 \
    rsync \
    swig \
    unzip \
    uuid-dev \
    wget \
    xalan \
    xdg-utils \
    xterm \
    xz-utils \
    zlib1g-dev
```

接續安裝下載 OP-TEE 專案所需的 Google Repo 工具，官方建議安裝方法如下，但實際執行會遇到系統提示權限不足的情況。
```shell
$ sudo curl https://storage.googleapis.com/git-repo-downloads/repo > /bin/repo && chmod a+x /bin/repo
```

替代方法可直接透過 apt get install 安裝，但此版本較舊，在後續使用上系統會自動偵測，並提示有新的版本可使用。
```shell
$ sudo apt get install repo

...

... A new version of repo (2.45) is available.
... New version is available at: /mnt/data_partition/ATF/optee/.repo/repo/repo
... The launcher is run from: /usr/bin/repo
!!! The launcher is not writable.  Please talk to your sysadmin or distro
!!! to get an update installed.
```

建立 OP-TEE 資料夾，並在該資料夾下使用 repo 命令下載 OP-TEE 原始碼。 `-m` 選項用於指定執行的平台，選擇 `qemu_v8.xml` 在 QEMU 中模擬 ARMv8-A 平台； `-b` 選項指定下載的專案版本，例如希望下載的版本為 3.12 版時，選項設為 `-b 3.12.0` ，若未指定則會自動下載最新版本。下載過程若出現網路異常造成下載不成功，重新再執行一次即可。
```shell
$ mkdir optee
$ cd optee
$ repo init -u https://github.com/OP-TEE/manifest.git -m qemu_v8.xml
$ repo sync -j2
```

完成原始碼下載後，進入 build 資料夾中，執行 OP-TEE 的 tool chain 編譯，此流程會花費一點時間，可以透過 -j 選項配置投入編譯的 CPU 數量。
```shell
$ cd build
$ make -j2 toolchains
```

使用 `make` 命令編譯 OP-TEE ，加上 `DEBUG=1` 選項可獲得 Debug 模式版本。
```shell!
$ make -j2 DEBUG=1
```

編譯完成後，輸出的映像檔散布在不同的資料夾下：
- TF-A 映像檔放至於 `[optee 路徑]/trusted-firmware-a/build/qemu/release` 資料夾：
    - bl1.bin
    - bl2.bin
    - bl31.bin

- BL32 映像檔案放在 `[optee 路徑]/optee_os/out/arm/core` 資料夾：
    - tee-header_v2.bin: bl32.bin
    - tee-pager_v2.bin: bl32_extra1.bin
    - tee-pageable_v2.bin: bl32_extra2.bin

- BL33 映像檔放在 `optee/u-boot` 資料夾中：
    - u-boot.bin: bl33.bin

- Secure world OS 映像檔放在 `[optee 路徑]/linux/arch/arm64/boot` 資料夾：
    - Image

- Secure world initramfs 映像檔放在 `[optee 路徑]/out-br/images` 資料夾：
    - rootfs.cpio.gz

- Normal world OS 和 initramfs 映像檔放在 `[optee 路徑]/out/bin` 資料夾中：
    - uImage
    - rootfs.cpio.uboot

執行 `make check` 命令可對編譯完成的 OP-TEE 進行健全性測試 (xtest)，此測試包含驗證是否相容於 ARM Trusted Zone 功能，並實際在 QEMU 上執行以 OP-TEE 作為 BL32 的 TF-A 開機流程，確認系統是否可正常運作。
```shell
$ make -j2 check
```

健全性測試成功將會看到下列提示訊息。
```shell
Status: PASS (1491 test cases)
Running: keyctl tests...
Status: keyctl tests successful
```

完成測試後，可以在 `optee/out/bin` 資料夾中看到 `serial0.log` 和 `serial1.log` 兩個檔案，前者紀錄測試過程的命令資訊，由此 log 檔可以看到測試方式是在 QEMU 中使用 semi-hosting 的方式載入映像檔，並使用 serail1.log 紀錄 QEMU 的執行狀態。 
```shell
spawn [open ...]
spawn sh -c /mnt/data_partition/ATF/optee/build/../qemu/build/aarch64-softmmu/qemu-system-aarch64 -nographic -smp 2 -cpu max,sme=on,pauth-impdef=on -d unimp -semihosting-config enable=on,target=native -m 1057 -bios bl1.bin -initrd rootfs.cpio.gz -kernel Image -append 'console=ttyAMA0,38400 keep_bootcon root=/dev/vda2 '  -object rng-random,filename=/dev/urandom,id=rng0 -device virtio-rng-pci,rng=rng0,max-bytes=1024,period=1000 -netdev user,id=vmnic -device virtio-net-device,netdev=vmnic -machine virt,acpi=off,secure=on,mte=off,gic-version=3,virtualization=false  -serial mon:stdio -serial file:serial1.log
```

`make run` 命令為編譯與執行 OP-TEE ，過程中會將 QEMU 運作所需的映像檔以 symbolic link 方式連結於 `optee/out/bin` 資料夾中，並建立 secure world 與 normal world 的 console 視窗，顯示 OP-TEE 運作過程中的系統資訊。
完成後透過
```shell
$ make run
```

建立 console 視窗的方式，是使用 gnome-terminal 與 nc 命令搭配 `[optee 路徑]\build\soc_term.py` 的 python 程式達成 。
```shell
$ nc -z  127.0.0.1 54320 || /usr/bin/gnome-terminal -t "Normal World" -x /mnt/data_partition/ATF/optee/build/soc_term.py 54320 & \
nc -z  127.0.0.1 54321 || /usr/bin/gnome-terminal -t "Secure World" -x /mnt/data_partition/ATF/optee/build/soc_term.py 54321 & \
while ! nc -z 127.0.0.1 54320 || ! nc -z 127.0.0.1 54321; do sleep 1; done
```

若完成編譯後只是要單純執行 OP-TEE 模擬，則可使用 `make only-run` 命令開啟 QEMU 並載入 TF-A 與 OP-TEE 。
```shell
$ make only-run
```

### Build BL32 Bootloader Image from OP-TEE project
雖然 OP-TEE 專案中可以輕鬆完成完整的 TF-A 模擬，但仔細觀察發現專案中的 TF-A 韌體版本較舊，下面就來看看如何只編譯 OP-TEE ，並整合到最新版本的 TF-A 中，將所有的 bootloader 打包為單一的映像檔。

為方便接續的觀察， BL32 編譯過程中加入 `DEBUG=1` 和 `CFG_CORE_ASLR=n` 選項， 避免 ASLR(Address Space Layout Randomization) 產生隨機的虛擬記憶體位址，在每次開機時將程式碼與映像檔載入到不同的虛擬記憶體位址上。編譯後可供 GDB 追蹤的 elf 檔為 'tee.elf' ，放置在 `[optee 路徑]/optee_os/out/arm/core/` 路徑下。
```shell
$ make optee-os DEBUG=1 CFG_CORE_ASLR=n 
```
[What is ASLR](https://static.linaro.org/connect/lvc21/presentations/lvc21-118.pdf)

編譯完成後將輸出的下列映像檔複製到 TF-A 的資料夾內，執行 TF-A 的編譯並將 BL32 一同打包為 [FIP(Firmware Image Package)](https://github.com/ARM-software/arm-trusted-firmware/blob/master/docs/design/firmware-design.rst#firmware-image-package-fip) ，由於 OP-TEE 的映像檔在未特別配置下會使用 GICv3 ，而 TF-A 默認使用 GICv2 ，故編譯時需加入 `QEMU_USE_GIC_DRIVER=QEMU_GICV3` 選項。
```shell
$ make CROSS_COMPILE=aarch64-linux-gnu- PLAT=qemu BL32=bl32.bin \
    BL32_EXTRA1=bl32_extra1.bin BL32_EXTRA2=bl32_extra2.bin \
    BL33=bl33.bin BL32_RAM_LOCATION=tdram SPD=opteed all fip \
    DEBUG=1 QEMU_USE_GIC_DRIVER=QEMU_GICV3
```

TF-A 編譯完成後於 [TF-A 路徑]/build/qemu/debug 資料夾下輸出 fip.bin ，並獲得下列提示：
```shell
Trusted Boot Firmware BL2: offset=0x128, size=0x9429, cmdline="--tb-fw"
EL3 Runtime Firmware BL31: offset=0x9551, size=0x100C4, cmdline="--soc-fw"
Secure Payload BL32 (Trusted OS): offset=0x19615, size=0x1C, cmdline="--tos-fw"
Secure Payload BL32 Extra1 (Trusted OS Extra1): offset=0x19631, size=0xD1400, cmdline="--tos-fw-extra1"
Non-Trusted Firmware BL33: offset=0xEAA31, size=0x200000, cmdline="--nt-fw"

Built /mnt/data_partition/ATF/toolchain/arm-trusted-firmware/build/qemu/debug/fip.bin successfully

  DD      qemu_fw.bios
  DD      qemu_fw.rom
Building qemu
  OD      /mnt/data_partition/ATF/toolchain/arm-trusted-firmware/build/qemu/debug/bl2/bl2.dump
  OD      /mnt/data_partition/ATF/toolchain/arm-trusted-firmware/build/qemu/debug/bl31/bl31.dump
make: Nothing to be done for 'fip'.
```

移動到 build/qemu/debug/ 資料夾中，透過 dd 命令將編譯產生的 bl1.bin 和 fip.bin 合成為 flash.bin 。
```shell
$ cd build/qemu/debug
$ dd if=bl1.bin of=flash.bin bs=4096 conv=notrunc
$ dd if=fip.bin of=flash.bin seek=64 bs=4096 conv=notrunc
```

透過 [TF-A 路徑]/tools/fiptools 下的 fiptools 工具，可分析打包好的映像檔中的資訊。
```shell
$ ./fiptool info ../../build/qemu/debug/fip.bin 
Trusted Boot Firmware BL2: offset=0x128, size=0x9429, cmdline="--tb-fw"
EL3 Runtime Firmware BL31: offset=0x9551, size=0x100C4, cmdline="--soc-fw"
Secure Payload BL32 (Trusted OS): offset=0x19615, size=0x1C, cmdline="--tos-fw"
Secure Payload BL32 Extra1 (Trusted OS Extra1): offset=0x19631, size=0xCF438, cmdline="--tos-fw-extra1"
Non-Trusted Firmware BL33: offset=0xE8A69, size=0x200000, cmdline="--nt-fw"
```

然而使用 poetry 進行分析時，卻發現無法僅能顯示片段的 BL32(tee) 的資訊，猜測應該是 poetry 只能完整顯示 secure SRAM 與 secure ROM 的使用資訊。
```shell
$ poetry run memory -s -b debug -p qemu -f -t
build-path: /mnt/data_partition/ATF/toolchain/arm-trusted-firmware/build/qemu/debug
+----------------------------------------------------------------------------+
|                         Memory Usage (bytes) [RAM]                         |
+-----------+------------+------------+------------+------------+------------+
| Component |   Start    |   Limit    |    Size    |    Free    |   Total    |
+-----------+------------+------------+------------+------------+------------+
|    BL1    |    e0ee000 |    e100000 |       8000 |       a000 |      12000 |
|    BL2    |    e06b000 |    e0a0000 |      11000 |      24000 |      35000 |
|    BL31   |    e0a0000 |    e100000 |      42000 |      1e000 |      60000 |
+-----------+------------+------------+------------+------------+------------+ 

+----------------------------------------------------------------------------+
|                         Memory Usage (bytes) [ROM]                         |
+-----------+------------+------------+------------+------------+------------+
| Component |   Start    |   Limit    |    Size    |    Free    |   Total    |
+-----------+------------+------------+------------+------------+------------+
|    BL1    |          0 |      20000 |       8370 |      17c90 |      20000 |
+-----------+------------+------------+------------+------------+------------+ 

name                                     start        end       size
bl1                                          0      32000      32000
├── 00                                       0       8370       8370
│   ├── .text                                0       6000       6000
│   └── .rodata                           6000       8370       2370
├── 01                                 e0ee000    e0ee105        105
│   └── .data                          e0ee000    e0ee105        105
├── 02                                 e0ee140    e0ef140       1000
│   └── .stacks                        e0ee140    e0ef140       1000
├── 04                                 e0ef140    e0efa60        920
│   └── .bss                           e0ef140    e0efa60        920
└── 03                                 e0f0000    e0f6000       6000
    └── .xlat_table                    e0f0000    e0f6000       6000
bl2                                    e06b000    e0a0000      35000
├── 00                                 e06b000    e074000       9000
│   ├── .text                          e06b000    e072000       7000
│   └── .rodata                        e072000    e074000       2000
└── 01                                 e074000    e07c000       8000
    ├── .data                          e074000    e074429        429
    ├── .stacks                        e074440    e075440       1000
    ├── .bss                           e075440    e075860        420
    └── .xlat_table                    e076000    e07c000       6000
bl31                                   e0a0000    e100000      60000
├── 00                                 e0a0000    e0ad000       d000
│   └── .text                          e0a0000    e0ad000       d000
└── 01                                 e0ad000    e0e2000      35000
    ├── .rodata                        e0ad000    e0b0000       3000
    ├── .data                          e0b0000    e0b00c4         c4
    ├── .stacks                        e0b0100    e0d0100      20000
    ├── .bss                           e0d0100    e0dba20       b920
    └── .xlat_table                    e0dc000    e0e2000       6000
tee                                    e100000    e216380     116380
└── 00                                 e100000    e216380     116380
    ├── .text                          e100000    e19a268      9a268
    ├── .rodata                        e19b000    e1ce310      33310
    ├── .data                          e1cf000    e1cf420        420
    ├── .bss                           e1cf420    e1dc9b8       d598
    ├── .heap1                         e1dc9b8    e1fd000      20648
    └── .nozi                          e1fd000    e216380      19380 

Memory Layout:
            +-----__BL1_RAM_END__------+--------------------------+--------------------------+--------------------------+
0x00e0f6000 +----__XLAT_TABLE_END__----+                          |                          |                          |
0x00e0f0000 +---__XLAT_TABLE_START__---+                          |                          |                          |
            +-__BASE_XLAT_TABLE_END__--+                          |                          |                          |
0x00e0efa60 +-------__BSS_END__--------+                          |                          |                          |
            +__BASE_XLAT_TABLE_START__-+                          |                          |                          |
            +_PMF_PERCPU_TIMESTAMP_END_+                          |                          |                          |
            +--__PMF_TIMESTAMP_END__---+                          |                          |                          |
0x00e0efa40 +-__PMF_TIMESTAMP_START__--+                          |                          |                          |
            +------__BSS_START__-------+                          |                          |                          |
0x00e0ef140 +------__STACKS_END__------+                          |                          |                          |
0x00e0ee140 +-----__STACKS_START__-----+                          |                          |                          |
0x00e0ee105 +-----__DATA_RAM_END__-----+                          |                          |                          |
            +----__BL1_RAM_START__-----+                          |                          |                          |
0x00e0ee000 +----__DATA_RAM_START__----+                          |                          |                          |
0x00e0e2000 |                          |                          +----__XLAT_TABLE_END__----+                          |
0x00e0dc000 |                          |                          +---__XLAT_TABLE_START__---+                          |
            |                          |                          +-__BASE_XLAT_TABLE_END__--+                          |
0x00e0dba20 |                          |                          +-------__BSS_END__--------+                          |
            |                          |                          +__BASE_XLAT_TABLE_START__-+                          |
            |                          |                          +_PMF_PERCPU_TIMESTAMP_END_+                          |
            |                          |                          +--__PMF_TIMESTAMP_END__---+                          |
0x00e0dba00 |                          |                          +-__PMF_TIMESTAMP_START__--+                          |
            |                          |                          +------__BSS_START__-------+                          |
0x00e0d0100 |                          |                          +------__STACKS_END__------+                          |
0x00e0b0100 |                          |                          +-----__STACKS_START__-----+                          |
            |                          |                          +-------__RELA_END__-------+                          |
0x00e0b00c8 |                          |                          +------__RELA_START__------+                          |
0x00e0b0000 |                          |                          +------__RODATA_END__------+                          |
0x00e0af740 |                          |                          +-__RODATA_END_UNALIGNED__-+                          |
            |                          |                          +-----__CPU_OPS_END__------+                          |
            |                          |                          +-------__GOT_END__--------+                          |
0x00e0af728 |                          |                          +------__GOT_START__-------+                          |
            |                          |                          +----__CPU_OPS_START__-----+                          |
            |                          |                          +-__FCONF_POPULATOR_END__--+                          |
            |                          |                          +__FCONF_POPULATOR_START__-+                          |
            |                          |                          +--__PMF_SVC_DESCS_END__---+                          |
0x00e0af188 |                          |                          +-__PMF_SVC_DESCS_START__--+                          |
            |                          |                          +-----__RODATA_START__-----+                          |
            |                          |                          +--__TEXT_END_UNALIGNED__--+                          |
0x00e0ad000 |                          |                          +-------__TEXT_END__-------+                          |
0x00e0a0000 |                          |                          +------__TEXT_START__------+                          |
0x00e07c000 |                          +----__XLAT_TABLE_END__----+                          |                          |
0x00e076000 |                          +---__XLAT_TABLE_START__---+                          |                          |
            |                          +-__BASE_XLAT_TABLE_END__--+                          |                          |
0x00e075860 |                          +-------__BSS_END__--------+                          |                          |
            |                          +__BASE_XLAT_TABLE_START__-+                          |                          |
            |                          +_PMF_PERCPU_TIMESTAMP_END_+                          |                          |
            |                          +--__PMF_TIMESTAMP_END__---+                          |                          |
0x00e075840 |                          +-__PMF_TIMESTAMP_START__--+                          |                          |
            |                          +------__BSS_START__-------+                          |                          |
0x00e075440 |                          +------__STACKS_END__------+                          |                          |
0x00e074440 |                          +-----__STACKS_START__-----+                          |                          |
0x00e074000 |                          +------__RODATA_END__------+                          |                          |
            |                          +-----__CPU_OPS_END__------+                          |                          |
            |                          +----__CPU_OPS_START__-----+                          |                          |
            |                          +-__FCONF_POPULATOR_END__--+                          |                          |
            |                          +__FCONF_POPULATOR_START__-+                          |                          |
            |                          +-------__GOT_END__--------+                          |                          |
            |                          +------__GOT_START__-------+                          |                          |
            |                          +--__PMF_SVC_DESCS_END__---+                          |                          |
            |                          +-__PMF_SVC_DESCS_START__--+                          |                          |
0x00e073450 |                          +-__RODATA_END_UNALIGNED__-+                          |                          |
            |                          +-----__RODATA_START__-----+                          |                          |
0x00e072000 |                          +-------__TEXT_END__-------+                          |                          |
0x00e071800 |                          +--__TEXT_END_UNALIGNED__--+                          |                          |
0x00e06b000 |                          +------__TEXT_START__------+                          |                          |
0x000008475 +-----__BL1_ROM_END__------+                          |                          |                          |
0x000008370 +----__DATA_ROM_START__----+                          |                          |                          |
            +-----__CPU_OPS_END__------+                          |                          |                          |
            +-------__GOT_END__--------+                          |                          |                          |
            +------__GOT_START__-------+                          |                          |                          |
            +-__RODATA_END_UNALIGNED__-+                          |                          |                          |
0x000008368 +------__RODATA_END__------+                          |                          |                          |
            +----__CPU_OPS_START__-----+                          |                          |                          |
            +-__FCONF_POPULATOR_END__--+                          |                          |                          |
            +__FCONF_POPULATOR_START__-+                          |                          |                          |
            +--__PMF_SVC_DESCS_END__---+                          |                          |                          |
0x000007fa8 +-__PMF_SVC_DESCS_START__--+                          |                          |                          |
            +-----__RODATA_START__-----+                          |                          |                          |
0x000006000 +-------__TEXT_END__-------+                          |                          |                          |
0x000005800 +--__TEXT_END_UNALIGNED__--+                          |                          |                          |
0x000000000 +------__TEXT_START__------+--------------------------+--------------------------+--------------------------+
                        bl1                        bl2                       bl31                        tee
```

TF-A 記憶體配置定義於 [TF-A 路徑]/plat/qemu/qemu/include/platform_def.h 中，可以看到有別於 BL1、BL2 和 BL33 載入到 secure SRAM ，BL32 將會被載入到 secure DRAM 中。
```c
/*
 * Partition memory into secure ROM, non-secure DRAM, secure "SRAM",
 * and secure DRAM.
 */
#define SEC_ROM_BASE                    0x00000000
#define SEC_ROM_SIZE                    0x00020000

#define NS_DRAM0_BASE                   ULL(0x40000000)
#define NS_DRAM0_SIZE                   ULL(0xc0000000)

#define SEC_SRAM_BASE                   0x0e000000
#define SEC_SRAM_SIZE                   0x00100000

#define SEC_DRAM_BASE                   0x0e100000
#define SEC_DRAM_SIZE                   0x00f00000

#define SECURE_GPIO_BASE                0x090b0000
#define SECURE_GPIO_SIZE                0x00001000
#define SECURE_GPIO_POWEROFF            0
#define SECURE_GPIO_RESET               1

/* Load pageable part of OP-TEE 2MB above secure DRAM base */
#define QEMU_OPTEE_PAGEABLE_LOAD_BASE   (SEC_DRAM_BASE + 0x00200000)
#define QEMU_OPTEE_PAGEABLE_LOAD_SIZE   0x00400000

/*
 * ARM-TF lives in SRAM, partition it here
 */

#define SHARED_RAM_BASE                 SEC_SRAM_BASE
#define SHARED_RAM_SIZE                 0x00001000
```

### Analyze Boot Sequence of TF-A
啟動 QEMU 前需先建立 secure world 與 normal world 的 console 視窗，並在 QEMU 的命令中新增 `-serail` 選項， `-bios` 選項設定為以 flash.bin 作為開機的映像檔。
```shell
$ qemu-system-aarch64 -nographic -machine virt,secure=on,gic-version=3 \
    -cpu cortex-a57 -kernel Image \
    -append "console=ttyAMA0,38400 keep_bootcon"   \
    -initrd rootfs.cpio.gz -smp 2 -m 1024 -bios flash.bin   \
    -d unimp -semihosting-config enable=on,target=native   \
    -s -S \
    -serial tcp:127.0.0.1:54320 \
    -serial tcp:127.0.0.1:54321
```

使用 GDB 分析時，需要注意雖然把所有 bootloader 透過 FIP 打包為單一映像檔，仍要以 add-symbol-file 命令載入要觀察的 bootloader elf 檔。分析的過程中會需要開啟下列四個 console 視窗：
- QEMU
- GDB
- Secure world console
- Normal world console
![Screenshot from 2024-09-29 16-18-58](https://hackmd.io/_uploads/S188CF800.png)

增加 OP-TEE 後的 TF-A 運作資訊顯示在 normal world 視窗中，可以觀察到 BL2 階段載入中更多的映像檔，映像檔 ID 定義在 tbbr_img_def_exp.h 中。
- 0: FIP_IMAGE_ID
    - FIP_IMAGE_ID
- 1: BL2_IMAGE_ID
    - Trusted Boot Firmware BL2
- 2: SCP_BL2_IMAGE_ID
    - SCP Firmware SCP_BL2
- 3: BL31_IMAGE_ID
    - EL3 Runtime Firmware BL31
- 4: BL32_IMAGE_ID
    - Secure Payload BL32 (Trusted OS)
- 5: BL33_IMAGE_ID
    - Non-Trusted Firmware BL33
- 21: BL32_EXTRA1_IMAGE_ID
    - Secure Payload BL32_EXTRA1 (Trusted OS Extra1)
- 22: BL32_EXTRA2_IMAGE_ID
    - Secure Payload BL32_EXTRA2 (Trusted OS Extra2)
- INVALID_IMAGE_ID: U(0xFFFFFFFF)
    - include/export/common/bl_common_exp.h

從 TF-A 顯示的訊息，可看到在 BL2 階段出現轉交控制權到 BL32 的提示，以 `Handoof to BL32` 作為關鍵字進行搜尋，發現該提示資訊定義於 `qemu_bl2_setup.c` ，並由 `bl2_load_images` 函式透過 while 進行呼叫，沒有真的轉交控制權到 BL32 。
```shell
INFO:    BL2: Loading image id 4
INFO:    Loading image id=4 at address 0xe100000
INFO:    Image id=4 loaded: 0xe100000 - 0xe10001c
INFO:    OPTEE ep=0xe100000
INFO:    OPTEE header info:
INFO:          magic=0x4554504f
INFO:          version=0x2
INFO:          arch=0x1
INFO:          flags=0x0
INFO:          nb_images=0x1
INFO:    Handoff to BL32
INFO:    Using default arguments
INFO:    BL2: Loading image id 21
```

實際將控制權轉交至 BL32 的時間點發生在 BL31 階段， BL31 執行 BL32 初始化的過程中，會透過 `opteed_setup` 函式將 BL32 的進入點函式 `opteed_init` 指定到 `bl32_init`， 在該函式中呼叫 `opteed_enter_sp` 將控制權轉移到於 S—EL1 層級下運作的 BL32 ，執行完成後再回到 bl31_main 函式中。

BL32 交回控制權給 BL31 後，會以 timer interrupt 方式定期中斷 Normal World OS ，並在 S-EL1 層級下執行 `periodic_callback` 函式。可在 secure world 視窗中看到下列系統提示：
```shell
I/TC: Primary CPU switching to normal world boot
D/TC:0   periodic_callback:136 seconds 1 millis 189 count 1
D/TC:0   periodic_callback:136 seconds 2 millis 189 count 2
D/TC:0   periodic_callback:136 seconds 3 millis 188 count 3
D/TC:0   periodic_callback:136 seconds 4 millis 188 count 4
D/TC:0   periodic_callback:136 seconds 5 millis 188 count 5
D/TC:0   periodic_callback:136 seconds 6 millis 188 count 6
```
完整的啟動流程描述如下，圖片頗傷眼，但這是目前試到可以用 Graphviz 畫出的完整流程，之後完全熟悉 Graphviz 後再來重新畫一張。
![BL32_boot](https://hackmd.io/_uploads/r1sceAIAA.png)

1. Run BL1 from Trusted ROM
    - Exception Level: EL3
    - Function: bl1_entrypoint
    - 從 Trusted ROM 記憶體位址 0x0_0000_0000 開始執行 AP Boot ROM 開機流程。

2. Load BL1 data to Trusted SRAM
    - Exception Level: EL3
    - Function: bl1_main
    - 將 BL1 運作所需的資料儲存至 Trusted RAM 的 0x0_0E0E_E000 到 0x0_0E0F_6000 區間中。

3. Load BL2 Image to Trusted SRAM
    - Exception Level: EL3
    - Function: bl1_main
    - BL1 透過 load_auth_image 函式載入 Image ID 1 的 BL2 映像檔到 Trusted RAM 0x0_0E06_B000 位址。

4. Run BL2 from Trusted SRAM
    - Exception Level: Secure-EL1
    - Function: bl2_entrypoint
    - 完成 BL1 初始化階段，呼叫 el_exit 函式並接著開始執行 Trusted Boot Firmware 開機流程。

5. Load BL31 Image to Trusted SRAM
    - Exception Level: Secure-EL1
    - Function: bl2_main
    - BL2 透過 load_auth_image 函式，以 semi-hosting 的途徑，將 Image ID 3 的 BL31 映像檔載入至 Trusted SRAM 0x0_0E0E_A000 位址。

6. Load BL32 Image to Trusted DRAM
    - Exception Level: Secure-EL1
    - Function: bl2_main
    - BL2 透過 load_auth_image 函式，以 semi-hosting 的途徑，將 Image ID 4 的 BL32 映像檔載入至 Trusted DRAM 0x0_0E10_0000 位址。

7. Load BL32 EXTRA1 Image to Trusted DRAM
    - Exception Level: Secure-EL1
    - Function: bl2_main
    - BL2 透過 load_auth_image 函式，以 semi-hosting 的途徑，將 Image ID 4 的 BL32 映像檔載入至 Trusted DRAM 0x0_0E10_0000 位址。

8. Load BL33 Image to Non-Trusted RAM
    - Exception Level: Secure-EL1
    - Function: bl2_main
    - BL2 透過 load_auth_image 函式，以 semi-hosting 的途徑，將 Image ID 5 的 BL33 映像檔載入至 Non-Trusted SRAM 0x0_6000_0000 位址。

7. Handoff to BL31 by SMC
    - Exception Level: EL3
    - Function: smc_handler64 
    - 在 Secure EL1 下運作的 BL2 無法直接切換至 EL1 下的 BL33，需透過 SMC 呼叫透過 BL1 切換至 BL31，再由 BL31 將控制權切換至 BL33。相關流程可在 bl1_print_next_bl_ep_info 函式設定中斷點進行觀察。

8. Run BL31 from Trusted SRAM
    - Exception Level: EL3
    - Function: bl31_entrypoint
    - 執行 EL3 Runtime Firmware 開機流程，建立運作所需的 MMU 與環境設定後，切換至 BL33。

9. Run BL32 from Trusted SRAM
    - Exception Level: Secure-EL1
    - Function: _start
    - 執行 Secure Payload 開機流程，建立 Secure OS 運作所需的 MMU 與環境設定後，切換至 BL31。

10. Run BL33 from Non-Trusted RAM
    - Exception Level: EL1
    - 執行 Non-Trusted Firmware 開機流程，載入 Normal world OS 的 UEFI 與系統核心，完成後開始運作 Normal world 作業系統。
