# midi_framework部件

## 简介

midi_framework部件为OpenHarmony系统提供了统一的MIDI设备访问、数据传输及协议处理能力，使得应用能够直接调用系统提供的接口实现外部MIDI设备的发现、连接以及高性能的指令收发功能。

midi_framework部件主要具备以下功能：

* **设备发现与连接**：支持MIDI设备的枚举、信息查询、热插拔监听（USB MIDI设备）以及设备连接。
* **数据传输**：支持基于UMP（Universal MIDI Packet）协议的高性能数据发送与接收。

**图 1** midi_framework部件架构图

架构中主要模块的功能说明如下：

**表 1** 模块功能介绍

| 模块名称            | 功能                                                                                                           |
| ------------------- | -------------------------------------------------------------------------------------------------------------- |
| **MIDI客户端**      | 提供MIDI客户端创建、销毁及回调注册的接口，是应用与服务交互的入口。                                             |
| **设备发现与管理**  | 提供当前系统连接的MIDI设备枚举、设备信息查询及USB设备热插拔事件监听接口。                                      |
| **连接会话管理**    | 管理应用端与服务端的连接会话，处理输入/输出端口（Port）的打开、关闭及多客户端路由分发逻辑。                    |
| **数据传输**        | 基于共享内存与无锁环形队列（Shared Ring Buffer）实现跨进程的高性能MIDI数据传输。                               |
| **协议处理器**      | 用于处理MIDI 2.0（UMP）与MIDI 1.0（Byte Stream）之间的协议解析与自动转换。                                     |
| **USB设备适配**     | 负责与底层ALSA驱动交互，实现标准USB MIDI设备的指令读写与控制。                                                 |
| **IPC通信**         | 定义并实现客户端与服务端之间的跨进程通信接口，处理请求调度。                                                   |
| **MIDI硬件驱动HDI** | 提供MIDI硬件驱动的抽象接口，通过该接口对服务层屏蔽底层硬件差异（如ALSA节点），向服务层提供统一的数据读写能力。 |

## 目录

```
/foundation/multimedia/midi_framework
├── bundle.json                            # 部件描述与编译配置文件
├── config.gni                             # 编译配置参数
├── figures                                # 架构图等资源文件
├── frameworks                             # 框架层实现
│   └── native
│       ├── midi                           # MIDI客户端核心逻辑 (Client Context)
│       ├── midiutils                      # 基础工具库
│       └── ohmidi                         # Native API (NDK) 接口实现
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

## 编译构建

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

## 说明

### 接口说明

midi_framework部件向开发者提供了C语言原生接口（Native API），主要涵盖客户端管理、设备管理及端口操作。主要接口及其功能如下：

**表 2** 接口说明

| 接口名称                  | 功能描述                                         |
| ------------------------- | ------------------------------------------------ |
| **OH_MidiClientCreate**   | 创建MIDI客户端实例，初始化上下文环境。           |
| **OH_MidiClientDestroy**  | 销毁MIDI客户端实例，释放相关资源。               |
| **OH_MidiGetDevices**     | 获取当前系统已连接的MIDI设备列表及设备详细信息。 |
| **OH_MidiOpenDevice**     | 打开指定的MIDI设备，建立连接会话。               |
| **OH_MidiCloseDevice**    | 关闭已打开的MIDI设备，断开连接。                 |
| **OH_MidiGetDevicePorts** | 获取指定设备的输入端口信息。                     |
| **OH_MidiOpenInputPort**  | 打开设备的指定输入端口，准备接收MIDI数据。       |
| **OH_MidiClosePort**      | 关闭指定的输入或输出端口，停止数据传输。         |

## 相关仓

[媒体子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/媒体子系统.md)

**[midi_framework](https://gitcode.com/openharmony/midi_framework-sig)**