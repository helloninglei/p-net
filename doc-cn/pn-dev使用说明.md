根据您提供的文档和代码信息，[pn_dev](file://c:\Mac\Home\Desktop\PLC开发\workspaces\p-net\build\pn_dev) 是 p-net 协议栈的示例应用程序可执行文件。以下是详细的使用方法：

## 基本用法

编译依赖库：
git clone --recurse-submodules https://github.com/rtlabs-com/osal.git
cd osal && mkdir build && cd build && cmake .. && make

### 0.编译
```bash
cd p-net && mkdir build && cd build
cmake .. -DLOG_LEVEL_VALUES=DEBUG -DUSE_SCHED_FIFO=ON -DUSE_SCHED_FIFO=ON
make 
```

### 1. 查看帮助信息
```bash
./pn_dev -h
# 或
./pn_dev --help
```

### 2. 基本运行
```bash
sudo ./pn_dev
```

### 3. 带详细输出的运行
```bash
sudo ./pn_dev -v
```

## 命令行参数详解

根据文档，[pn_dev](file://c:\Mac\Home\Desktop\PLC开发\workspaces\p-net\build\pn_dev) 支持以下参数：

- `--help` 或 `-h`: 显示帮助文本并退出
- `-v`: 增加详细程度，可以重复使用以增加详细级别
- `-f`: 重置为出厂设置并存储到文件，然后退出
- `-r`: 删除存储的文件并退出
- `-g`: 显示协议栈详细信息，重复使用可显示更多信息
- `-i INTERFACE`: 指定要使用的以太网接口名称，默认为 eth0
- `-s NAME`: 设置站名，默认为 rt-labs-dev（仅在存储文件中不可用时使用）
- `-b FILE`: 读取 Button1 的路径（绝对或相对），默认不读取 Button1
- `-d FILE`: 读取 Button2 的路径（绝对或相对），默认不读取 Button2
- `-p PATH`: 存储目录的绝对路径，默认使用当前目录

## 实际使用示例

### 1. 基本运行示例
```bash
# 设置网络接口
sudo ifconfig eth0 192.168.0.50 netmask 255.255.255.0 up

# debian 12以上使用ip命令设置网络接口
# 先查看网卡信息
ip link show
# 设置网络接口(enp0s5是上面查到的网卡名称)
sudo ip addr add 192.168.0.50/24 dev enp0s5

# 运行应用程序, 多个-v 可以增加详细程度
sudo ./pn_dev -v -v -v -v -v -i enp0s5 -s my-profinet-device -b button1.txt -d button2.txt
```

### 2. 指定网络接口和站名
```bash
sudo ./pn_dev -v -i eth0 -s my-profinet-device
```

### 3. 使用文件模拟按钮输入
```bash
# 创建按钮文件
touch button1.txt
touch button2.txt

# 运行应用并指定按钮文件
sudo ./pn_dev -v -b button1.txt -d button2.txt
```

### 4. 指定存储目录
```bash
sudo ./pn_dev -v -p /home/pi/profinet/storage
```

## 按钮和LED操作

### 物理按钮操作
如果连接了物理按钮，需要先设置 GPIO：
```bash
echo 22 > /sys/class/gpio/export
echo 27 > /sys/class/gpio/export
```

然后运行应用：
```bash
sudo ./pn_dev -v -b /sys/class/gpio/gpio22/value -d /sys/class/gpio/gpio27/value
```

### 文件模拟按钮操作
使用文件模拟按钮时，可以通过以下方式操作：
```bash
# 模拟 Button1 按下
echo 1 > button1.txt

# 模拟 Button1 释放
echo 0 > button1.txt

# 模拟 Button2 按下
echo 1 > button2.txt

# 模拟 Button2 释放
echo 0 > button2.txt
```

### 查看LED状态
如果使用文件模拟LED，可以查看LED状态：
```bash
# 查看数据LED状态
cat pnet_led_1.txt

# 实时监控LED状态
watch -n 0.1 cat pnet_led_1.txt
```

## 功能说明

根据代码分析，`pn_dev` 应用程序具有以下功能：

1. **Profinet IO设备功能**：实现了一个具有8个数字输入和8个数字输出的IO模块
2. **按钮功能**：
   - Button1：设置周期数据值
   - Button2：循环触发报警、设置诊断和创建日志条目
3. **LED功能**：
   - LED1：数据LED
   - LED2：Profinet信号LED
4. **报警处理**：支持发送过程报警
5. **诊断功能**：支持添加、更新和删除诊断信息
6. **日志功能**：支持创建日志条目

## 典型运行输出

运行时会显示类似以下的输出：
```
** Starting Profinet sample application x.x.x **
Number of slots:      5 (incl slot for DAP module)
P-net log level:      3 (DEBUG=0, FATAL=4)
App verbosity level:  1
Number of ports:      1
Network interfaces:   eth0
Station name:         rt-labs-dev
...
Waiting for connect request from IO-controller
```

## 注意事项

1. **权限要求**：需要使用 `sudo` 运行，因为应用程序需要访问网络接口
2. **网络配置**：确保指定的网络接口存在且配置正确
3. **存储文件**：IP设置等信息会存储到文件中，如需重置可使用 `-r` 参数
4. **实时性要求**：在没有实时补丁的Linux系统上可能会遇到时序问题

这样就可以正确使用 [pn_dev](file://c:\Mac\Home\Desktop\PLC开发\workspaces\p-net\build\pn_dev) 示例应用程序了。