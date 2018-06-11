# class.C 精简系协议 *[v1.0.1](#!dev/changelog.md)*



## 序

协议格式如下：

| bytes | 0    | 1    | 2    | 3    | 6~n  |
| :---- | :--- | :--- | :--- | :--- | :--- |
| -     |流控   | 长度   | 接口索引 | 操作数  | 参数   |

## 约定

`->MCU`指发送至MCU

`MCU->`指从MCU发出

其他同理。



### 流控

使用一个`byte`，每发一个数据包，此`byte`加一。使用此`byte`作为最近发送数据包的唯一标识。可用于数据完整性的验证以及数据丢失的检测。

ps. 每条通信逻辑链路的流控由发起方维持。



## 设备端接口

接口均使用`接口索引+操作数+参数`的描述方式

Note: 无参数返回默认表示`ACK`

| 接口   | 说明                           |
| ---- | ---------------------------- |
| [时间日期](#!dev/classC/timedate.md)`0x01` | 本地时间与日期的设置与获取接口 |
| [指针控制](#!dev/classC/hand.md)`0x02` |手表指针的控制接口|
| [提醒](#!dev/classC/notify.md)`0x03`   |提醒信息的更新与删除|
| [节电时间](#!dev/classC/powersave.md)`0x04` |设置节电时间的专门接口|
| [闹钟](#!dev/classC/alarm.md)`0x05`   |设置闹钟信息的专门接口|
| [系统信息](#!dev/classC/sysinfo.md)`0x06` |获取如版本号等系统信息|
| [系统操作](#!dev/classC/sysctrl.md)`0x07` |执行如重启等系统操作|
| [数据操作](#!dev/classC/data.md)`0x08` |获取和配置如计步等数据|

## 移动端接口

| 接口   | 说明                   |
| ---- | -------------------- |
| [应用功能](#!dev/classB/m_func.md)`0x81` | 移动端的功能接口 |
| [系统信息](#!dev/classB/m_info.md)`0x82` | 获取移动端的信息 |
