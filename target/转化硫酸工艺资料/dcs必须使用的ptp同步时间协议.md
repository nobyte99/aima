使用 `linuxptp` 软件包将 Linux 系统时钟与 PTP 硬件时钟（PHC）同步，核心工具是 `phc2sys`。它的任务就是将高精度的 PTP 硬件时钟时间“传递”给系统时钟（`CLOCK_REALTIME`）[](https://manpages.debian.org/testing/linuxptp/phc2sys.8.en.html)[](https://github.com/open-edge-platform/edge-ai-suites/blob/release-2026.0.0/metro-ai-suite/deterministic-threat-detection/docs/user-guide/how-to-guides/configure-ptp.md)[](https://techdocs.broadcom.com/us/en/storage-and-ethernet-connectivity/ethernet-nic-controllers/bcm957xxx/adapters/Configuration-adapter/precision-time-protocol/configuring-ptp.html)。

根据你的网络环境和需求，同步方式主要有以下三种。

### ⚙️ 方案一：自动化同步（推荐）

如果网络中有正在运行的 `ptp4l` 服务，这是最省事的方式。`phc2sys` 会自动从 `ptp4l` 获取同步状态并执行相应操作[](https://manpages.debian.org/testing/linuxptp/phc2sys.8.en.html)[](https://docs.redhat.com/zh-cn/documentation/openshift_container_platform/4.15/pdf/networking/OpenShift_Container_Platform-4.15-Networking-zh-CN.pdf#109#23)[](https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/7/pdf/system_administrators_guide/Red_Hat_Enterprise_Linux-7-System_Administrators_Guide-zh-CN.pdf#77#50)。

bash

# 系统时钟作为时钟源或同步对象，由 ptp4l 的状态决定
sudo phc2sys -a -r

- `-a`：从正在运行的 `ptp4l` 服务中读取需要同步的时钟，并自动调整同步方向[](https://manpages.debian.org/testing/linuxptp/phc2sys.8.en.html)。
    
- `-r`：指示 `phc2sys` 也需要考虑同步系统时钟（`CLOCK_REALTIME`）[](https://manpages.debian.org/testing/linuxptp/phc2sys.8.en.html)[](https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/7/pdf/system_administrators_guide/Red_Hat_Enterprise_Linux-7-System_Administrators_Guide-zh-CN.pdf#77#50)。
    

### 🎯 方案二：手动指定同步

当需要明确指定源和目标的设备时，可以手动配置。该命令将网卡 `enp1s0` 的硬件时钟（作为源）同步到系统时钟（作为目标）[](https://github.com/open-edge-platform/edge-ai-suites/blob/release-2026.0.0/metro-ai-suite/deterministic-threat-detection/docs/user-guide/how-to-guides/configure-ptp.md)[](https://techdocs.broadcom.com/us/en/storage-and-ethernet-connectivity/ethernet-nic-controllers/bcm957xxx/adapters/Configuration-adapter/precision-time-protocol/configuring-ptp.html)[](https://doc.miivii.com/BRD601-THOR-USER-MANUAL-CH/7.0-NonLTS/wiki/BRD601-THOR-PTP.html)。

bash

sudo phc2sys -s enp1s0 -c CLOCK_REALTIME -m -w

- `-s enp1s0`：指定 **源时钟** 为网络接口 `enp1s0` 关联的 PTP 硬件时钟（PHC）[](https://manpages.debian.org/testing/linuxptp/phc2sys.8.en.html)[](https://github.com/open-edge-platform/edge-ai-suites/blob/release-2026.0.0/metro-ai-suite/deterministic-threat-detection/docs/user-guide/how-to-guides/configure-ptp.md)。你需要将 `enp1s0` 替换为实际支持 PTP 的网卡名。
    
- `-c CLOCK_REALTIME`：指定 **目标时钟** 为系统时钟，这也是默认值，因此该选项通常可以省略[](https://manpages.debian.org/testing/linuxptp/phc2sys.8.en.html)。
    
- `-w`：等待 `ptp4l` 进入同步状态后才开始工作，确保不会在未同步时做出错误调整[](https://manpages.debian.org/testing/linuxptp/phc2sys.8.en.html)[](https://github.com/open-edge-platform/edge-ai-suites/blob/release-2026.0.0/metro-ai-suite/deterministic-threat-detection/docs/user-guide/how-to-guides/configure-ptp.md)[](https://techdocs.broadcom.com/us/en/storage-and-ethernet-connectivity/ethernet-nic-controllers/bcm957xxx/adapters/Configuration-adapter/precision-time-protocol/configuring-ptp.html)。
    
- `-m`：将信息打印到标准输出，方便观察同步过程[](https://docs.redhat.com/zh-cn/documentation/openshift_container_platform/4.15/pdf/networking/OpenShift_Container_Platform-4.15-Networking-zh-CN.pdf#109#23)[](https://doc.miivii.com/BRD601-THOR-USER-MANUAL-CH/7.0-NonLTS/wiki/BRD601-THOR-PTP.html)[](https://docs.carnegierobotics.com/docs/time/ptp_setup/linux.html)。
    

### 🌐 方案三：集成多种时间源 (`timemaster`)

对于更复杂的场景，例如网络中存在多个 PTP 域或需要将 NTP 作为 PTP 的备份时，可以使用 `timemaster` 工具[](https://manpages.debian.org/testing/linuxptp/timemaster.8.en.html)[](https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/7/pdf/system_administrators_guide/Red_Hat_Enterprise_Linux-7-System_Administrators_Guide-zh-CN.pdf#77#50)。它能统一管理 `ptp4l`、`phc2sys` 和 NTP 服务（如 `chronyd`），让系统从多个时间源中选择最可靠的进行同步[](https://manpages.debian.org/testing/linuxptp/timemaster.8.en.html)[](https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/7/pdf/system_administrators_guide/Red_Hat_Enterprise_Linux-7-System_Administrators_Guide-zh-CN.pdf#77#50)。

### ⚙️ 几个重要的调优选项

`phc2sys` 提供了不少参数，让你能根据硬件和网络环境调整同步行为[](https://manpages.debian.org/testing/linuxptp/phc2sys.8.en.html)：

- `-R <更新速率>`：指定每秒更新目标时钟的次数，默认是每秒1次。如果对精度有更高要求，可以适当提高此值，例如 `-R 16` 表示每秒更新16次[](https://manpages.debian.org/testing/linuxptp/phc2sys.8.en.html)[](https://docs.redhat.com/zh-cn/documentation/openshift_container_platform/4.15/pdf/networking/OpenShift_Container_Platform-4.15-Networking-zh-CN.pdf#109#23)。
    
- `-N <读取次数>`：指定每次更新时读取源时钟的次数，系统会从多次读取中选最快的一次来更新。这有助于消除系统调度的随机延迟，默认是5次[](https://manpages.debian.org/testing/linuxptp/phc2sys.8.en.html)[](https://docs.redhat.com/zh-cn/documentation/openshift_container_platform/4.15/pdf/networking/OpenShift_Container_Platform-4.15-Networking-zh-CN.pdf#109#23)。
    
- `-S <步进阈值>`：设置一个以秒为单位的阈值。当时间偏差超过这个值时，`phc2sys` 会直接“步进”（跳跃式）调整时钟，而不是缓慢地通过频率调整来追赶。默认为0.0秒，即不步进[](https://manpages.debian.org/testing/linuxptp/phc2sys.8.en.html)。
    
- `-E <伺服算法>`：选择时钟伺服算法，`pi` 是默认的 PI 控制器，`linreg` 是自适应的线性回归控制器，后者可能在某些场景下表现更好，且无需手动调节 PI 参数[](https://manpages.debian.org/testing/linuxptp/phc2sys.8.en.html)[](https://docs.redhat.com/zh-cn/documentation/red_hat_enterprise_linux/7/pdf/system_administrators_guide/Red_Hat_Enterprise_Linux-7-System_Administrators_Guide-zh-CN.pdf#77#50)[](https://sourceforge.net/p/linuxptp/mailman/linuxptp-users/thread/10b08bc9ad274a229a68c8f77be6768a%40pgs.com/#msg35969516)。
    

### ✅ 验证同步状态

命令执行后，控制台会持续输出类似下面的日志，可以通过观察这些数值来判断同步效果：

- `phc offset`：表示 PTP 硬件时钟与系统时钟之间的**时间偏移量**，单位通常是纳秒（ns）。这个值越小越好[](https://github.com/open-edge-platform/edge-ai-suites/blob/release-2026.0.0/metro-ai-suite/deterministic-threat-detection/docs/user-guide/how-to-guides/configure-ptp.md)[](https://doc.miivii.com/BRD601-THOR-USER-MANUAL-CH/7.0-NonLTS/wiki/BRD601-THOR-PTP.html)[](https://sourceforge.net/p/linuxptp/mailman/linuxptp-users/thread/10b08bc9ad274a229a68c8f77be6768a%40pgs.com/#msg35969516)。
    
- `freq`：表示对时钟频率的**调整值**，单位是 ppb（十亿分之一）。数值稳定表示时钟频率已锁定[](https://github.com/open-edge-platform/edge-ai-suites/blob/release-2026.0.0/metro-ai-suite/deterministic-threat-detection/docs/user-guide/how-to-guides/configure-ptp.md)[](https://sourceforge.net/p/linuxptp/mailman/linuxptp-users/thread/10b08bc9ad274a229a68c8f77be6768a%40pgs.com/#msg35969516)。
    

**建议**：开始之前，先用 `ethtool -T <网卡名>` 确认网卡支持硬件时间戳（输出中包含 `hardware-transmit` 和 `hardware-receive`）[](https://doc.miivii.com/BRD601-THOR-USER-MANUAL-CH/7.0-NonLTS/wiki/BRD601-THOR-PTP.html)。并从“方案二”开始，用 `-m` 选项观察日志来确认一切工作正常。