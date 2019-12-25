# ComiNode-TM

本文档内容描述了与蓝牙Mesh透传模块`ComiNode-TM`通信的协议格式。

## 序

协议框架如下：

![](./table_temp.png)

## 通用接口(本地)

透传模块具备如下通用接口：

- 心跳
- 状态查询
- 进入配网模式
- 退出配网模式
- 重置网络
- ACK



### ACK

当载荷长度为`2`，且只包含`0xAA, 0x55`的本地通信前缀时，认为是`ACK`包



### 心跳

心跳包包含模块本身的所有状态参数。在以下几种条件下会发送心跳包：

1. 模块上电时会主动发送一次心跳包
1. 收到查询状态指令时会发送一次心跳包
1. 连续`60s`未发生有效通信时会发送一次心跳包

当连续`3`次`心跳包#3`没有得到有效回复时，模块会拉低`timeout`引脚`1s`，可用于复位`MCU`

**载荷定义如下：**

| 序号 | 内容     | 备注                                                         |
| ---- | -------- | ------------------------------------------------------------ |
| 5    | 0xAA     | 本地通信前缀                                                 |
| 6    | 0x55     | 本地通信前缀                                                 |
| 7    | 0x01     | 0x01=心跳包                                                  |
| 8    | 配网状态 | 0x00=未在配网模式且未配网<br>0x01=在配网模式且未配网<br>0x02=在配网模式且正在配网<br>0x03=已配网 |
| 9    | 保留     |                                                              |
| 10   | 保留     |                                                              |



### 状态查询

强制获取一次心跳包

**载荷定义如下：**

| 序号 | 内容 | 备注          |
| ---- | ---- | ------------- |
| 5    | 0xAA | 本地通信前缀  |
| 6    | 0x55 | 本地通信前缀  |
| 7    | 0xA1 | 0xA1=强制心跳 |
| 8    | 保留 |               |



### 进入配网模式

使模块进入配网模式，可以被网关搜索到并配网。如果已经在配网模式，则不产生任何效果。

### 退出配网模式

使模块退出配网模式，将不能被网关搜索到并配网。如果模块已经配网、正在配网，或者未在配网模式，则不产生任何效果。

**载荷定义如下：**

| 序号 | 内容     | 备注                                   |
| ---- | -------- | -------------------------------------- |
| 5    | 0xAA     | 本地通信前缀                           |
| 6    | 0x55     | 本地通信前缀                           |
| 7    | 0xA2     | 0xA2=配网控制                          |
| 8    | 配网状态 | 0x00=退出配网模式<br>0x01=进入配网模式 |
| 9    | 保留     |                                        |



### 重置网络

删除模块保存的配网信息，使得模块恢复到未配网状态。

**载荷定义如下：**

| 序号 | 内容 | 备注             |
| ---- | ---- | ---------------- |
| 5    | 0xAA | 本地通信前缀     |
| 6    | 0x55 | 本地通信前缀     |
| 7    | 0xAF | 0xAF=重置网络    |
| 8    | 0x77 | 重置网络固定后缀 |
| 9    | 0xEE | 重置网络固定后缀 |
| 10   | 0xFF | 重置网络固定后缀 |
| 11   | 0x00 | 重置网络固定后缀 |



## 测试指令

目前支持`4`个字节载荷用于测试。

当前模块使用一个`io`口作为测试输出。

指令`AA 55 01`将置`io`口为高，协议层完整传输内容（当随机数取00时）为：`5E 00 5E 03 5D AA 55 01 08`

指令`AA 55 00`将置`io`口为低。



## 测试评估设备1号 - 电动床 - 控制协议(试行)

设备具有两个电机，可单独控制升降。

### 功能列表如下：

- 背部升高
- 背部降低
- 腿部升高
- 腿部降低
- 背部腿部同时升高
- 背部腿部同时降低
- 复位

### 协议载荷设计如下：

| byte0              | byte1    | byte2          | byte3          |
| ------------------ | -------- | -------------- | -------------- |
| `0xF0`（设备标记） | 电机控制 | `0xFF`（预留） | `0xFF`（预留） |

其中，`byte1`按`bit`定义如下：

| bit  | 含义          |
| ---- | ------------- |
| 7    | 背部电机使能  |
| 6    | 1=升高 0=降低 |
| 5    | 腿部电机使能  |
| 4    | 1=升高 0=降低 |
| 3    | 预留          |
| 2    | 预留          |
| 1    | 预留          |
| 0    | 1=复位        |

#### Byte1按功能列表示例如下：

- 背部升高: `二进制0b11000000`=`十六进制0xC0`
- 背部降低: `二进制0b10000000`=`十六进制0x80`
- 腿部升高: `二进制0b00110000`=`十六进制0x30`
- 腿部降低: `二进制0b00100000`=`十六进制0x20`
- 背部腿部同时升高: `二进制0b11110000`=`十六进制0xF0`
- 背部腿部同时降低: `二进制0b10100000`=`十六进制0xA0`
- 复位: `二进制0bxxxxxxx1`=`十六进制0x01` 只要最低位为1，则忽略其他位，视为复位信号



以下为`腿部降低`的完整指令示例：

`0x5E` `0xAB` `0xF5` `0x04` `0xF1` `0xF0` `0x20` `0xFF` `0xFF` `0x0A`

注释如下：

| 字节 | 注释                                               |
| ---- | -------------------------------------------------- |
| 0x5E | 固定前缀                                           |
| 0xAB | 随机数                                             |
| 0xF5 | `0x5E`异或`0xAB`                                   |
| 0x04 | 载荷长度                                           |
| 0xF1 | 载荷长度`0x04`异或`0xF5`                           |
| 0xF0 | 设备标记                                           |
| 0x20 | 腿部降低命令                                       |
| 0xFF | 预留                                               |
| 0xFF | 预留                                               |
| 0x0A | 校验和=`0xF0^0x5`+`0x20^0x6`+`0xFF^0x7`+`0xFF^0x8` |

