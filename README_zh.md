# MIDI组件<a name="ZH-CN_TOPIC_MIDI_001"></a>

* [简介](#section_intro)
* [基本概念](#section_concepts)


* [目录](#section_dir)
* [编译构建](#section_build)
* [使用说明](#section_usage)
* [接口说明](#section_api)
* [开发步骤](#section_steps)


* [相关仓](#section_related)

## 简介<a name="section_intro"></a>

midi_framework部件为OpenHarmony系统提供了统一的MIDI设备访问、数据传输及协议处理能力，使得应用能够直接调用系统提供的接口实现外部MIDI设备的发现、连接以及高性能的指令收发功能。

**图 1** MIDI组件架构图<a name="fig_midi_arch"></a>

### 基本概念<a name="section_concepts"></a>

* **MIDI (Musical Instrument Digital Interface)**
乐器数字接口，是电子乐器、计算机和其他音频设备之间交换音乐信息（如音符、控制参数等）的行业标准协议。
* **UMP (Universal MIDI Packet)**
通用 MIDI 数据包。这是 MIDI 2.0 规范引入的一种全新的数据格式，用于承载 MIDI 消息。与传统的字节流不同，UMP 基于 32 位字（Word）构建，支持更高的带宽、更低的抖动以及更丰富的通道控制能力，同时向后兼容 MIDI 1.0 消息。
* **MIDI 1.0 Byte Stream**
传统的 MIDI 数据传输方式，基于串行字节流。在处理旧设备或传统接口时，系统通常需要将 UMP 数据包转换回字节流格式。
* **MIDI 端口 (MIDI Port)**
MIDI 设备上用于输入或输出数据的逻辑接口。一个 MIDI 设备可以拥有多个输入端口和输出端口，每个端口独立传输 MIDI 数据流。

架构中主要模块的功能说明如下：

**表 1** 模块功能介绍

| 模块名称            | 功能                                                                                                               |
| ------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **MIDI客户端**      | 提供MIDI客户端创建、销毁及回调注册的接口，是应用与服务交互的入口。                                                 |
| **设备发现与管理**  | 提供当前系统连接的MIDI设备枚举、设备信息查询、设备上下线事件监听接口以及通过MAC地址链接蓝牙设备的接口。            |
| **连接会话管理**    | 管理应用端与服务端的连接会话，处理端口（Port）的打开、关闭及多客户端路由分发逻辑。                                 |
| **数据传输**        | 基于共享内存与无锁环形队列（Shared Ring Buffer）实现跨进程的高性能MIDI数据传输。                                   |
| **协议处理器**      | 用于处理MIDI 2.0（UMP）与MIDI 1.0（Byte Stream）之间的协议解析与自动转换。                                         |
| **USB设备适配**     | 负责与MIDI HDI交互，间接访问ALSA驱动，实现标准USB MIDI设备的指令读写与控制。                                       |
| **BLE设备适配**     | 负责对接系统蓝牙服务，实现BLE MIDI设备的指令读写与控制。                                                           |
| **IPC通信模块**     | 定义并实现客户端与服务端之间的跨进程通信接口，处理请求调度。                                                       |
| **MIDI硬件驱动HDI** | 提供MIDI硬件驱动的抽象接口，通过该接口对服务层屏蔽底层硬件差异（如声卡设备节点），向服务层提供统一的数据读写能力。 |

## 目录<a name="section_dir"></a>

仓目录结构如下：

```
/foundation/multimedia/midi_framework      # MIDI组件业务代码
├── bundle.json                            # 部件描述与编译配置文件
├── config.gni                             # 编译配置参数
├── figures                                # 架构图等资源文件
├── frameworks                             # 框架层实现
│   └── native
│       ├── midi                           # MIDI客户端核心逻辑 (Client Context)
│       ├── midiutils                      # 基础工具库
│       └── ohmidi                         # Native API 接口实现
├── interfaces                             # 接口定义
│   ├── inner_api                          # 系统内部接口头文件 (C++ Client)
│   └── kits                               # 对外接口头文件 (Native C API)
├── sa_profile                             # 系统服务配置文件
├── services                               # MIDI服务层实现
│   ├── common                             # 服务端通用组件 (Futex同步, 共享内存, 无锁队列, UMP协议处理)
│   ├── etc                                # 进程启动配置 (midi_server.cfg)
│   ├── idl                                # IPC通信接口定义
│   └── server                             # 服务端核心逻辑 (设备管理, USB驱动适配, 客户端连接管理)
└── test                                   # 测试代码
    ├── fuzzer                             # Fuzzing测试用例
    └── unittest                           # 单元测试用例

```

## 编译构建<a name="section_build"></a>

根据不同的目标平台，使用以下命令进行编译：

**编译32位ARM系统midi_framework部件**

```bash
./build.sh --product-name {product_name} --ccache --build-target midi_framework

```

**编译64位ARM系统midi_framework部件**

```bash
./build.sh --product-name {product_name} --ccache --target-cpu arm64 --build-target midi_framework

```

> **说明：**
> `{product_name}` 为当前支持的平台名称，例如 `rk3568`。

## 使用说明<a name="section_usage"></a>

### 接口说明<a name="section_api"></a>

midi_framework部件向开发者提供了C语言原生接口（Native API），主要涵盖客户端管理、设备管理及端口操作。主要接口及其功能如下：

**表 2** 接口说明

| 接口名称                  | 功能描述                                         |
| ------------------------- | ------------------------------------------------ |
| **OH_MidiClientCreate**   | 创建MIDI客户端实例，初始化上下文环境。           |
| **OH_MidiClientDestroy**  | 销毁MIDI客户端实例，释放相关资源。               |
| **OH_MidiGetDevices**     | 获取当前系统已连接的MIDI设备列表及设备详细信息。 |
| **OH_MidiOpenDevice**     | 打开指定的MIDI设备，建立连接会话。               |
| **OH_MidiCloseDevice**    | 关闭已打开的MIDI设备，断开连接。                 |
| **OH_MidiGetDevicePorts** | 获取指定设备的端口信息。                         |
| **OH_MidiOpenInputPort**  | 打开设备的指定输入端口，准备接收MIDI数据。       |
| **OH_MidiClosePort**      | 关闭指定的输入端口，停止数据传输。               |

### 开发步骤<a name="section_steps"></a>

可以使用此仓库内提供的 Native API 接口实现 MIDI 设备访问。以下步骤描述了如何开发一个基础的 MIDI 数据接收功能：

1. 使用 **OH_MidiClientCreate** 接口创建客户端实例。
```cpp
OH_MidiClient *client = nullptr;
OH_MidiCallbacks callbacks = {nullptr, nullptr}; // 系统回调暂为空
OH_MidiStatusCode ret = OH_MidiClientCreate(&client, callbacks, nullptr);

```


2. 使用 **OH_MidiGetDevices** 接口获取连接的设备列表。
> **注意**：该接口采用两次调用模式。第一次调用获取设备数量，第二次调用获取实际数据。


```cpp
size_t devCount = 0;
// 第一次调用：获取数量
OH_MidiGetDevices(client, nullptr, &devCount);

if (devCount > 0) {
    std::vector<OH_MidiDeviceInformation> devices(devCount);
    // 第二次调用：获取详情
    OH_MidiGetDevices(client, devices.data(), &devCount);
}

```


3. 使用 **OH_MidiOpenDevice** 打开指定的设备。
```cpp
OH_MidiDevice *device = nullptr;
// 假设打开列表中的第一个设备
int64_t targetDeviceId = devices[0].midiDeviceId;
ret = OH_MidiOpenDevice(client, targetDeviceId, &device);

```


4. 使用 **OH_MidiGetDevicePorts** 获取端口信息，确认端口方向。
```cpp
size_t portCount = 0;
OH_MidiGetDevicePorts(device, nullptr, &portCount);
std::vector<OH_MidiPortInformation> ports(portCount);
OH_MidiGetDevicePorts(device, ports.data(), &portCount);

```


5. 定义数据接收回调函数，并使用 **OH_MidiOpenInputPort** 打开输入端口。
```cpp
// 回调函数实现
void OnMidiReceived(void *userData, const OH_MidiEvent *events, size_t eventCount) {
    for (size_t i = 0; i < eventCount; ++i) {
        // 处理 events[i].data (字节流或UMP)
    }
}

// 在遍历端口时打开 Input 类型的端口
if (ports[i].direction == MIDI_PORT_DIRECTION_INPUT) {
    OH_MidiPortDescriptor desc = {ports[i].portIndex, MIDI_PROTOCOL_1_0};
    OH_MidiOpenInputPort(device, desc, OnMidiReceived, nullptr);
}

```


6. 数据处理完成后，依次调用 **OH_MidiClosePort**、**OH_MidiCloseDevice** 和 **OH_MidiClientDestroy** 释放资源。
```cpp
OH_MidiClosePort(device, portIndex);
OH_MidiCloseDevice(device);
OH_MidiClientDestroy(client);

```



## 相关仓<a name="section_related"></a>

[媒体子系统](https://gitcode.com/openharmony/docs/blob/master/zh-cn/readme/媒体子系统.md)

**[midi_framework](https://gitcode.com/openharmony/midi_framework-sig)**