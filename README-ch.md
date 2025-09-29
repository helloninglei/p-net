# p-net Profinet 设备协议栈

## 网络资源

- 源代码仓库: [https://github.com/rtlabs-com/p-net](https://github.com/rtlabs-com/p-net)
- 文档: [https://rt-labs.com/docs/p-net](https://rt-labs.com/docs/p-net)
- 持续集成: [https://github.com/rtlabs-com/p-net/actions](https://github.com/rtlabs-com/p-net/actions)
- RT-Labs（协议栈集成、认证服务和培训）: [https://rt-labs.com](https://rt-labs.com)

## p-net

Profinet 设备协议栈实现。主要特性：
- Profinet v2.4
  - 符合性类别 A 和 B
  - 实时类别 1
  - 多以太网端口
- 易于使用
  - 广泛的文档和入门指南
  - 在 Raspberry Pi 上构建和运行示例应用程序只需 30 分钟
- 可移植
  - 使用 C 语言编写
  - 支持 Linux、RTOS 或裸机运行
  - 提供支持的移植层源代码

RT-Labs Profinet 协议栈 p-net 用于 Profinet 设备实现。它易于使用且占用空间小，特别适合资源有限、效率至关重要的嵌入式系统。该协议栈提供完整的源代码，包括移植层和示例应用程序。

同时也支持使用 C++（任何版本）进行应用程序开发。

平台的主要要求是能够发送和接收原始以太网第 2 层帧。

### 功能特性：
- 多以太网端口（目前仅支持 Linux）
- TCP/IP
- LLDP
- SNMP
- RT（实时）
- 地址解析
- 参数化
- 过程 IO 数据交换
- 报警处理
- 可配置的模块和子模块数量
- 裸机或操作系统
- 提供移植层
- 支持 I&M0 - I&M4。支持设备的 I&M 数据，但不支持单个模块的 I&M 数据。

### 限制或尚未实现的功能：

- 这是一个设备协议栈，意味着不支持 IO 控制器/主站/PLC 端
- 无媒体冗余（不支持 MRP）
- 遗留启动模式未完全实现
- 不支持 RT_CLASS_UDP
- 不支持 DHCP
- 无快速启动
- 无 MC 组播设备到设备通信
- 不支持共享设备（连接到多个控制器）
- 仅支持完整连接，不支持有限的"DeviceAccess"连接类型
- 不支持 iPar（参数服务器）
- 不支持时间同步
- 报警时不支持 UDP 帧（仅实现默认报警机制）
- 不支持 ProfiDrive 或 ProfiSafe 配置文件

本软件采用双重许可，包括 GPL 版本 3 和商业许可。如果您打算在商业产品中使用此协议栈，则可能需要购买许可证。详情请参见 LICENSE.md。

## 要求

平台必须能够发送和接收原始以太网第 2 层帧，以太网驱动程序必须能够处理完整大小的帧。出于性能原因，还应避免数据复制。

- cmake 3.14 或更高版本

### Linux 要求：

- gcc 4.6 或更高版本
- 查看文档中的"Linux 的实时特性"页面，了解如何改善 Linux 定时

### rt-kernel 要求：

- Workbench 2020.1 或更高版本

我们使用的微控制器示例是 Infineon XMC4800，它具有运行在 144 MHz 的 ARM Cortex-M4，2 MB Flash 和 352 kB RAM。它运行 rt-kernel，我们已测试了 9 个 Profinet 插槽，每个插槽有 8 个数字输入和 8 个数字输出（每个一位）。这些值每毫秒发送和接收一次（PLC 看门狗设置为 3 毫秒）。

## 入门指南

请参阅文档中的教程: [https://rt-labs.com/docs/p-net/tutorial.html](https://rt-labs.com/docs/p-net/tutorial.html)

注意克隆时需要包含子模块：

```
git clone --recurse-submodules https://github.com/rtlabs-com/p-net.git
```

## 依赖项

一些平台相关部分位于 OSAL 仓库和 cmake-tools 仓库中。

- [https://github.com/rtlabs-com/osal](https://github.com/rtlabs-com/osal)
- [https://github.com/rtlabs-com/cmake-tools](https://github.com/rtlabs-com/cmake-tools)

这些会在安装过程中自动下载。

p-net 协议栈不包含第三方组件。其外部依赖项包括：

- C 库
- 操作系统（如果使用）
- 对于符合性类别 B，您需要 SNMP 实现。在 Linux 上使用 net-snmp（BSD 许可证）[http://www.net-snmp.org](http://www.net-snmp.org)

用于构建、测试和文档的工具（不会包含在最终的二进制文件中）：

- cmake（BSD 3-clause 许可证）[https://cmake.org](https://cmake.org)
- gtest（BSD-3-Clause 许可证）[https://github.com/google/googletest](https://github.com/google/googletest)
- Sphinx（BSD 许可证）[https://www.sphinx-doc.org](https://www.sphinx-doc.org)
- Doxygen（GPL v2）[https://www.doxygen.nl](https://www.doxygen.nl/index.html)
- clang-format（Apache License 2.0）[https://clang.llvm.org](https://clang.llvm.org/docs/ClangFormat.html)

## 贡献

欢迎贡献。如果您想贡献代码，需要签署贡献者许可协议并通过电子邮件或实体邮件发送给我们。更多信息请访问 [https://rt-labs.com/contribution](https://rt-labs.com/contribution)。