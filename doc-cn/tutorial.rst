# 教程

在本教程中，我们将在 Raspberry Pi（嵌入式 Linux 开发板）上运行 p-net Profinet 设备协议栈及其示例应用程序。您可以选择在 Raspberry Pi 上连接两个 LED 灯和两个按钮，以便更方便地与示例应用程序交互。

我们将使用第二个 Raspberry Pi 作为 PLC（可编程逻辑控制器 = IO 控制器），运行 Codesys 软 PLC。

完成本教程所需的必要硬件：

* 1 个 Raspberry Pi 作为 IO 设备
* 1 个 Raspberry Pi 作为 IO 控制器（或者西门子 PLC）
* 1 个以太网交换机
* 3 根以太网线缆

可选硬件：

* 用于 Raspberry Pi 的键盘、鼠标和显示器（作为 IO 设备）。如果没有这些设备，使用 USB 转串口线与 Raspberry Pi 通信也很有帮助（因为它会自动更改 IP 地址）。
* 用于连接到 Raspberry Pi（作为 IO 设备）的 LED 灯和按钮。

在 Raspberry Pi 上运行 p-net 大约需要 30 分钟。之后示例应用程序将等待传入连接。另外需要一个小时来设置另一个 Raspberry Pi 作为 IO 控制器（PLC）并研究示例应用程序数据。

## 示例应用程序描述

示例应用程序实现了一个 Profinet IO 设备，具有一个 IO 模块，包含 8 个数字输入和 8 个数字输出。示例使用连接到输入的两个按钮和连接到输出之一的一个 LED（我们称之为"数据 LED"）。

第二个 LED 连接为 Profinet"信号 LED"，可以通过闪烁来识别特定的 IO 设备（如果您有很多设备）。

![教程概览](illustrations/TutorialOverview.png)

IO 设备上的 LED1（"数据 LED"）由 IO 控制器（PLC）控制，通常会闪烁。通过 IO 设备上的 Button1，可以告诉 PLC 打开和关闭 LED 的闪烁。

Button2 触发从 IO 设备向 PLC 发送报警。

* Button1：设置周期数据值
* Button2：触发报警，设置诊断等
* LED1：数据 LED
* LED2：信号 LED

可以研究产生的以太网流量（见下文）。

## 高级用户注意事项

IO 设备示例应用程序可以运行在：

* Raspberry Pi（如本教程所述）
* 其他嵌入式 Linux 开发板
* Linux 笔记本电脑（或 Virtualbox 中的 Linux 虚拟机）
* 运行 RTOS（如 RT-kernel）的嵌入式开发板

除了在第二个 Raspberry Pi 上运行的 Codesys 软 PLC 外，您还可以使用西门子 Simatic PLC。请参阅本文档的另一页。

## 可用文件

p-net 仓库中的 `sample_app` 目录包含此示例的源代码。它还包含一个 GSD 文件（用 GSDML 编写），该文件告诉 IO 控制器如何与 IO 设备通信。

示例应用程序中依赖于运行 Linux 或 RTOS 的部分位于 `src/ports` 中。

## 模块和插槽

插槽是您可以放置模块的位置。

示例应用程序的 GSDML 文件定义了这些模块：

| 模块 | 输入数据（到 IO 控制器） | 输出数据（来自 IO 控制器） |
|------|------------------------|--------------------------|
| 8 位输入 + 8 位输出 | 1 字节 | 1 字节 |
| 8 位输入 | 1 字节 |  |
| 8 位输出 |  | 1 字节 |

示例应用程序有 4 个插槽（除了用于 DAP 模块的插槽 0），对于此示例应用程序，任何模块都可以放入插槽 1 到 4 中的任意一个。

在此示例中，我们将在插槽 1 中使用"8 位输入 + 8 位输出"模块。

## 设置 IO 设备 Raspberry Pi 运行 p-net

由于 PLC 通常会更改 IO 设备的 IP 地址，我们建议您将键盘、鼠标和显示器连接到运行 p-net 示例应用程序的 Raspberry Pi。或者您可以使用 USB 转串口线从笔记本电脑与 Raspberry Pi 通信。

要设置 Raspberry Pi 上的 Raspbian，并可选择连接按钮和 LED，请参阅准备 Raspberry Pi。

Linux 示例应用程序通过写入文件来控制 LED，例如 `/sys/class/gpio/gpio17/value`。LED 状态更改时，文件中将写入 `0` 或 `1`。这是通过脚本完成的，便于适应您的硬件。

如果您没有物理 LED，可以使用替代脚本将 LED 输出写入纯文本文件。使用方法如下所述。

## 安装依赖项

您的 Raspberry Pi 需要通过 LAN 或 WiFi 连接到互联网才能下载软件。

要在 Raspberry Pi 上编译 p-net，您需要较新版本的 [cmake](file://c:\Mac\Home\Desktop\PLC开发\workspaces\p-net\cmake\Linux.cmake)。安装它：

```bash
sudo apt update
sudo apt install snapd
sudo reboot
sudo snap install cmake --classic
```

验证安装的版本：

```bash
cmake --version
```

将安装的版本与 p-net 所需的最低版本进行比较（见第一页）。

您还需要 `git` 来下载 p-net。使用以下命令安装：

```bash
sudo apt install git
```

## 下载和编译 p-net

创建目录：

```bash
mkdir /home/pi/profinet/
cd /home/pi/profinet/
```

克隆源代码：

```bash
git clone --recurse-submodules https://github.com/rtlabs-com/p-net.git
```

这将克隆包含子模块的仓库。

然后创建并配置构建：

```bash
cmake -B build -S p-net
```

构建代码：

```bash
cmake --build build --target install
```

我们使用 `install` 目标来安装用于操作 IP 设置、控制 LED 等的脚本。

默认行为是将 LED 输出写入常规文件，而不是控制真实 LED。如果您在 Raspberry Pi 上连接了真实 LED，请启用 LED 控制脚本：

```bash
mv build/set_profinet_leds build/set_profinet_leds.disabled
mv build/set_profinet_leds.raspberrypi build/set_profinet_leds
```

## 高级用户注意事项

如果您已经克隆了仓库但没有使用 `--recurse-submodules` 标志，则在 `p-net` 文件夹中运行：

```bash
git submodule update --init --recursive
```

调整一些设置的替代 cmake 命令：

```bash
cmake -B build -S p-net -DCMAKE_BUILD_TYPE=Debug -DBUILD_TESTING=OFF -DBUILD_SHARED_LIBS=ON -DUSE_SCHED_FIFO=ON
```

您可以为构建文件夹选择任何名称，例如如果您想要构建不同的配置。

如果您想启用并行构建，可以在 [make](file://c:\Mac\Home\Desktop\PLC开发\workspaces\p-net\cmake\Linux.cmake) 中使用 `-j` 标志。

根据您安装 cmake 的方式，您可能需要运行 `snap run cmake` 而不是 [cmake](file://c:\Mac\Home\Desktop\PLC开发\workspaces\p-net\cmake\Linux.cmake)。

可以指定子模块仓库的位置。详情请见本页末尾。

## 运行示例应用程序

在构建目录中运行示例应用程序：

```bash
cd build
```

IO 设备示例应用程序的用法：

```none
pi@pndevice-pi:~/profinet/build$ ./pn_dev -h

Sample application for p-net Profinet device stack.

Wait for connection from IO-controller.
Then read buttons (input) and send to controller.
Listen for application LED output (from controller) and set application LED state.
It will also send a counter value (useful also without buttons and LED).
Button1 value is sent in the periodic data.
Button2 cycles through triggering an alarm, setting diagnosis and creating logbook entries.

Also the mandatory Profinet signal LED is controlled by this application.

The LEDs are controlled by the script set_profinet_leds
located in the same directory as the application binary.
A version for Raspberry Pi is available, and also a version writing
to plain text files (useful for demo if no LEDs are available).

Assumes the default gateway is found on .1 on same subnet as the IP address.

Optional arguments:
   --help       Show this help text and exit
   -h           Show this help text and exit
   -v           Incresase verbosity. Can be repeated.
   -f           Reset to factory settings, and store to file. Exit.
   -r           Remove stored files and exit.
   -g           Show stack details and exit. Repeat for more details.
   -i INTERF    Name of Ethernet interface to use. Defaults to eth0
   -s NAME      Set station name. Defaults to rt-labs-dev  Only used
               if not already available in storage file.
   -b FILE      Path (absolute or relative) to read Button1. Defaults to not read Button1.
   -d FILE      Path (absolute or relative) to read Button2. Defaults to not read Button2.
   -p PATH      Absolute path to storage directory. Defaults to use current directory.

p-net revision: 0.1.0+bb4177a
```

启用以太网接口并设置初始 IP 地址：

```bash
sudo ifconfig eth0 192.168.0.50 netmask 255.255.255.0 up
```

运行示例应用程序：

```bash
从 p-net/tree/master 下载P-Net主体源代码，解压到设备中，目录结构~/profinet/p-net。
从 rtlabs-com/osal 下载osal源代码，解压到p-net源码根文件夹中，目录名osal（~/profinet/p-net/osal）。
编辑CMakeLists.txt，屏蔽46行的#include(AddOsal)，并在228行添加add_subdirectory (osal)
mkdir build && cd build
cmake 
make j8
make install 
cd install
sudo ./pn_dev -v
sudo ./pn_dev -v -v -v -v -v -i enp0s5 -s "rt-labs-dev" -b button1.txt -d button2.txt
```

示例输出：

```none
pi@pndevice-pi:~/profinet/build$ sudo ./pn_dev -v

** Starting Profinet sample application 0.1.0+bb4177a **
Number of slots:      5 (incl slot for DAP module)
P-net log level:      3 (DEBUG=0, FATAL=4)
App verbosity level:  1
Number of ports:      1
Network interfaces:   eth0
Button1 file:
Button2 file:
Station name:         rt-labs-dev
Management port:      eth0
Physical port [1]:    eth0
Current hostname:     pndevice-pi
Current IP address:   192.168.0.50
Current Netmask:      255.255.255.0
Current Gateway:      192.168.0.1
Storage directory:    /home/pi/profinet/build

Profinet signal LED call-back. New state: 0
Network script for eth0:  Set IP 192.168.0.50   Netmask 255.255.255.0   Gateway 192.168.0.1   Permanent: 1   Hostname: rt-labs-dev   Skip setting hostname: true
Module plug call-back
Pull old module.    API: 0 Slot:  0    Slot was empty.
Plug module.        API: 0 Slot:  0 Module ID: 0x1
Submodule plug call-back.
Pull old submodule. API: 0 Slot:  0                   Subslot: 1      Subslot was empty.
Plug submodule.     API: 0 Slot:  0 Module ID: 0x1    Subslot: 1 Submodule ID: 0x1 "DAP Identity 1"
                     Data Dir: NO_IO In: 0 Out: 0 (Exp Data Dir: NO_IO In: 0 Out: 0)
Submodule plug call-back.
Pull old submodule. API: 0 Slot:  0                   Subslot: 32768      Subslot was empty.
Plug submodule.     API: 0 Slot:  0 Module ID: 0x1    Subslot: 32768 Submodule ID: 0x8000 "DAP Interface 1"
                     Data Dir: NO_IO In: 0 Out: 0 (Exp Data Dir: NO_IO In: 0 Out: 0)
Submodule plug call-back.
Pull old submodule. API: 0 Slot:  0                   Subslot: 32769      Subslot was empty.
Plug submodule.     API: 0 Slot:  0 Module ID: 0x1    Subslot: 32769 Submodule ID: 0x8001 "DAP Port 1"
                     Data Dir: NO_IO In: 0 Out: 0 (Exp Data Dir: NO_IO In: 0 Out: 0)
Waiting for connect request from IO-controller
```

IP 设置存储到文件中。如果您意外地在 IP 设置错误时运行了应用程序，请使用以下命令删除存储的设置：

```bash
sudo ./pn_dev -r
```

现在您已经成功在 Raspberry Pi 上安装了示例应用程序！要查看它的实际效果，您需要将其连接到 PLC。

## 设置 PLC

我们建议您使用 Codesys 软 PLC。
在第二个 Raspberry Pi 上安装 Raspberry Pi OS。不需要串口线或 LED。

请参阅准备 Raspberry Pi 和使用 Codesys 软 PLC，了解如何将其设置为 IO 控制器（PLC）。

通过以太网交换机连接两个 Raspberry Pi 开发板和您的笔记本电脑。

## 连接到 PLC 时 Linux 示例应用程序的输出

这是启用详细输出时 Linux 示例应用程序启动时的典型输出：

```none
pi@pndevice-pi:~/profinet/build$ sudo ./pn_dev -v -b /sys/class/gpio/gpio22/value -d /sys/class/gpio/gpio27/value

** Starting Profinet sample application 0.1.0+bb4177a **
Number of slots:      5 (incl slot for DAP module)
P-net log level:      3 (DEBUG=0, FATAL=4)
App verbosity level:  1
Number of ports:      1
Network interfaces:   eth0
Button1 file:         /sys/class/gpio/gpio22/value
Button2 file:         /sys/class/gpio/gpio27/value
Station name:         rt-labs-dev
Management port:      eth0
Physical port [1]:    eth0
Current hostname:     pndevice-pi
Current IP address:   192.168.0.50
Current Netmask:      255.255.255.0
Current Gateway:      192.168.0.1
Storage directory:    /home/pi/profinet/build

Profinet signal LED call-back. New state: 0
Network script for eth0:  Set IP 0.0.0.0   Netmask 0.0.0.0   Gateway 0.0.0.0   Permanent: 1   Hostname: rt-labs-dev   Skip setting hostname: true
No valid default gateway given. Skipping setting default gateway.
Module plug call-back
Pull old module.    API: 0 Slot:  0    Slot was empty.
Plug module.        API: 0 Slot:  0 Module ID: 0x1
Submodule plug call-back.
Pull old submodule. API: 0 Slot:  0                   Subslot: 1      Subslot was empty.
Plug submodule.     API: 0 Slot:  0 Module ID: 0x1    Subslot: 1 Submodule ID: 0x1 "DAP Identity 1"
                     Data Dir: NO_IO In: 0 Out: 0 (Exp Data Dir: NO_IO In: 0 Out: 0)
Submodule plug call-back.
Pull old submodule. API: 0 Slot:  0                   Subslot: 32768      Subslot was empty.
Plug submodule.     API: 0 Slot:  0 Module ID: 0x1    Subslot: 32768 Submodule ID: 0x8000 "DAP Interface 1"
                     Data Dir: NO_IO In: 0 Out: 0 (Exp Data Dir: NO_IO In: 0 Out: 0)
Submodule plug call-back.
Pull old submodule. API: 0 Slot:  0                   Subslot: 32769      Subslot was empty.
Plug submodule.     API: 0 Slot:  0 Module ID: 0x1    Subslot: 32769 Submodule ID: 0x8001 "DAP Port 1"
                     Data Dir: NO_IO In: 0 Out: 0 (Exp Data Dir: NO_IO In: 0 Out: 0)
Waiting for connect request from IO-controller

Network script for eth0:  Set IP 192.168.0.50   Netmask 255.255.255.0   Gateway 0.0.0.0   Permanent: 0   Hostname: rt-labs-dev   Skip setting hostname: true
No valid default gateway given. Skipping setting default gateway.
Module plug call-back
Pull old module.    API: 0 Slot:  1    Slot was empty.
Plug module.        API: 0 Slot:  1 Module ID: 0x32
Submodule plug call-back.
Pull old submodule. API: 0 Slot:  1                   Subslot: 1      Subslot was empty.
Plug submodule.     API: 0 Slot:  1 Module ID: 0x32   Subslot: 1 Submodule ID: 0x1 "Input 8 bits output 8 bits"
                     Data Dir: INPUT_OUTPUT In: 1 Out: 1 (Exp Data Dir: INPUT_OUTPUT In: 1 Out: 1)
Connect call-back. AREP: 1  Status codes: 0 0 0 0
Callback on event PNET_EVENT_STARTUP   AREP: 1
New data status callback. AREP: 1  Data status changes: 0x35  Data status: 0x35
   Run, Valid, Primary, Normal operation, Evaluate data status
Parameter write call-back. AREP: 1 API: 0 Slot:  1 Subslot: 1 Index: 123 Sequence:  2 Length: 4
Bytes: 00 00 00 05
Parameter write call-back. AREP: 1 API: 0 Slot:  1 Subslot: 1 Index: 124 Sequence:  3 Length: 4
Bytes: 00 00 00 06
Dcontrol call-back. AREP: 1  Command: PRM_END
Callback on event PNET_EVENT_PRMEND   AREP: 1
Set input data and IOPS for slot  0 subslot 1 "DAP Identity 1"  size 0 IOXS_GOOD
Set input data and IOPS for slot  0 subslot 32768 "DAP Interface 1"  size 0 IOXS_GOOD
Set input data and IOPS for slot  0 subslot 32769 "DAP Port 1"  size 0 IOXS_GOOD
Set input data and IOPS for slot  1 subslot 1 "Input 8 bits output 8 bits"  size 1 IOXS_GOOD
Set output IOCS         for slot  1 subslot 1 "Input 8 bits output 8 bits"
Application will signal that it is ready for data.
Callback on event PNET_EVENT_APPLRDY   AREP: 1
Setting outputs to default values.
Ccontrol confirmation call-back. AREP: 1  Status codes: 0 0 0 0
Callback on event PNET_EVENT_DATA   AREP: 1
Setting outputs to default values.
```

确切的输出将取决于您在设置 PLC 时使用的模块等。

## 输入按钮和 LED，或用于模拟的文件

如果您使用纯文件作为输出而不是 LED，请使用以下命令查看"数据 LED"的文件：

```bash
watch -n 0.1 cat /home/pi/profinet/build/pnet_led_1.txt
```

如果您想使用物理输入按钮，您必须首先正确设置按钮的 GPIO 文件：

```bash
echo 22 > /sys/class/gpio/export
echo 27 > /sys/class/gpio/export
```

然后：

```bash
sudo ./pn_dev -v -b /sys/class/gpio/gpio27/value -d /sys/class/gpio/gpio22/value
```

也可以使用纯文件作为输入而不是物理按钮：

```bash
touch /home/pi/profinet/build/button1.txt
touch /home/pi/profinet/build/button2.txt
sudo ./pn_dev -v -b /home/pi/profinet/build/button1.txt -d /home/pi/profinet/build/button2.txt
```

手动向文件写入 `1` 或 `0` 来模拟按钮按下和释放：

```bash
echo 1 > /home/pi/profinet/build/button1.txt
echo 0 > /home/pi/profinet/build/button1.txt
```

如果您只有一个终端，需要在后台运行 `pn_dev` 才能运行这些命令。这可以通过在启动 `pn_dev` 的命令末尾添加 `&` 来实现。之后使用 `sudo pkill pn_dev` 杀死 `pn_dev` 进程。

## 研究产生的通信

LED1 默认应该闪烁。按下 Button1 切换 LED1 闪烁。

通过按下 Button2 可以触发报警、添加诊断等。请查看控制台中的输出。

在"捕获和分析以太网数据包"页面中描述了如何研究网络流量。如果您对启动期间发送的不同数据包或周期数据负载感兴趣，请参阅"示例应用程序详细信息"页面。

## 调整日志级别

p-net 协议栈中有日志记录，描述与 PLC 的交互。

如果您想更改 p-net 协议栈的日志级别，请在 `build` 目录中运行 `ccmake .`。它将启动一个菜单程序。移动到 LOG_LEVEL 条目，按 Enter 更改为 DEBUG。按 [c](file://c:\Mac\Home\Desktop\PLC开发\workspaces\p-net\src\common\pf_cpm.c) 保存，按 `q` 退出。

您需要重新构建项目以使更改生效。
 
## 下一步

太好了！您成功运行了示例应用程序。

尝试闪烁 Profinet 信号 LED。请参阅"使用 Codesys 软 PLC"页面上的描述。

要启用示例应用程序在开机时自动启动，请参阅"在 Raspberry Pi 上安装 Raspberry Pi OS"页面。

对于 Profinet 成员，"ART tester" 工具可用于一致性测试。对示例应用程序运行一致性测试以验证协议栈是否合规。请参阅本文档中关于一致性测试的单独页面。

要试验符合性类别 B 的 SNMP 功能，请参阅"网络拓扑检测"页面。

现在是时候开始开发您自己的应用程序了。您可以使用示例应用程序作为起始模板。通过修改可用模块以及它们发送和接收的数据类型进行实验。相应地修改您的 GSDML 文件，以向 PLC 配置工具解释 IO 设备行为。

有一个单独的页面提供了一些关于如何编写应用程序的想法。记住要不时运行"ART tester"来验证您保持合规性。

## 时序问题

如果在没有实时补丁的 Linux 机器上运行，可能会遇到超时问题。可能看起来像：

```none
Callback on event PNET_EVENT_ABORT. Error class: 253 Error code: 6
```

其中错误代码通常是 5 或 6。
请参阅本文档中的"Linux 实时属性"页面了解解决方案，以及"使用 Codesys 软 PLC"页面了解解决方法。

## 调试编译问题

要显示更多编译详细信息，请使用：

```bash
cmake --build build -v
```

## 故障排除

如果您的 IO 设备 Raspberry Pi 出现网络问题，请重新运行上面给出的 `ifconfig` 命令。

如果您在建立与 PLC 的连接时遇到问题，请将其直接连接到您的笔记本电脑并在相应的以太网接口上运行 Wireshark 程序。研究 DCP 和 LLDP 帧以查看当前的 PLC 设置。本文档的另一页详细介绍了 Wireshark 的使用。LLDP 帧中的"管理地址"块显示了 PLC 的 IP 地址。还有其他描述 MAC 地址和端口 ID 的块。您可以在某些 DCP 帧中找到预期的 IO 设备站名。

## 高级用户：OSAL

OSAL 是一个通用的操作系统抽象库，可以被系统中的多个项目使用。为了避免多个库副本的冲突问题，它已被移到自己的仓库中。

`cmake-tools` 是一个包含 RT-Labs 项目通用 CMake 实用程序的仓库。它包含一个 CMake 脚本 `AddOsal.cmake`，简化了 OSAL 的使用。它支持两种不同的使用场景：

### 1) 自动下载和构建 OSAL

在 CMake 配置期间，如果在系统中找不到 OSAL，它将自动下载并构建。对大多数用户来说，这将是默认设置。
通过执行以下命令运行 CMake 配置：

```bash
cmake -B build -S p-net
```

### 2) 外部 OSAL

在 CMake 配置期间，如果找到 OSAL，则 p-net 将只链接到外部库。
如果 OSAL 安装在默认位置如 `/usr/include` 或 `/usr/local/include`，CMake 将找到外部 OSAL 库。这可能适用于原生构建或具有暂存文件夹的交叉编译 Linux 系统。

通过在配置期间设置 `Osal_DIR`，CMake 也可以被告知已安装的 OSAL 版本的路径，如下所示：

```bash
cmake -B build -S p-net -DOsal_DIR=/path/to/osal/install/cmake
```

当在 OSAL 构建目录中运行以下命令时，会产生安装文件夹：

```bash
make install
```

或类似命令。