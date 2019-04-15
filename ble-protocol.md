[TOC]

# 版本修订记录

| 版本 |   日期     |  作者 |             备注           |
| ---- | --------- | ----- | --------------------------|
| 1.0  | 2019-4-16 |  张凯 |             创建           |

# 概述
此协议规定了APP与蓝牙终端之间的通讯方式以及协议规范。
# 通讯
## 通讯接口
* 蓝牙
## 通讯方式
* APP为蓝牙从机（central）， 蓝牙终端为主机（peripheral），采用一问一答的方式。
## 数据顺序
* 数据顺序采用大端模式，即高位在前，地位在后。

# 消息定义
## 消息帧结构
![icon](FrameStructure.png)
### 帧头
* 固定1字节 0x68
### 长度
* 表示数据与长度，用两个字节表示，采用大端模式
### 数据加密类型
* 指示数据域的加密类型：无加密（0），RSA加密（1），AES加密（3）三种，用一个字节长度表示。
### 数据域
* 数据域参考[通讯协议数据域](#datafield)
### 校验
* 校验方法采用CRC16，校验示例代码参考附录[校验](#checksum)


# 通讯协议数据域
<span id="datafield"> </span>

## APP发送公钥指令

### APP发送公钥（APP -> BLE）
APP与蓝牙终端建立蓝牙连接后，app需要发送公钥和一个随机数到蓝牙终端，json数据包为透明数据，不加密。

``` json
{
    "cmd":"SendPubKey",
    "pubkey":"XXXXXXXXXXXXXXX",
    "random":xxxxxxx
}
```

### 蓝牙终端接收公钥应答（APP -> BLE）
APP与蓝牙终端建立蓝牙连接后，并接收到app发过来的公钥以及随机数，则本地生成一个随机数，然后发送到APP端，且数据包要通过接收到的公钥加密。接收到的公钥本地需要check，app接收应答后需要判断check正确性，[check](#check)方法见附录。
1. 应答成功
``` json
{
    "cmd":"SendPubKeyAck",
    "ack":0,
    "random":xxxxxxx,
    "pubkeycheck":xxxxxxx
}
```

2. 应答失败
``` json
{
    "cmd":"SendPubKeyAck",
    "ack":X
}
```
***

## 获取设备基本信息
APP获取蓝牙终端相关信息（ID，软件版本，硬件版本，蓝牙通讯协议版本），用于APP获取蓝牙终端信息，以便从后台请求设备的鉴权码。

### 获取设备基本信息请求（APP -> BLE）
``` json
{
    "cmd":"BaseInfoReq"
}
```

### 获取设备基本信息应答（BLE -> APP）
1. 应答成功
``` json
{
    "cmd":"BaseInfoReqAck",
    "ack":0,
    "info":{id:"XXXXXXXX",hver:"1.0.0",sver:"1.0.0",pver:"1.0.0",},
}
```

2. 应答失败
``` json
{
    "cmd":"BaseInfoReqAck",
    "ack":X
}
```
***

## 鉴权请求
APP与蓝牙终端建立加密通道，且从服务器端获取到鉴权码，则可向蓝牙终端发起鉴权请求，鉴权成功蓝牙终端才可让APP获取更多数据。
### 鉴权请求（APP -> BLE）
``` json
{
    "cmd":"AuthReq",
    "authkey":xxxxxxxx
}
```
### 鉴权请求应答（BLE -> APP）
[check](#check)方法见附录。
1. 应答成功
``` json
{
    "cmd":"AuthReqAck",
    "ack":0,
    "authkeycheck":xxxxxxxx
}
```
2. 应答失败
``` json
{
    "cmd":"AuthReqAck",
    "ack":X
}
```
***

## 设置
### 设置类命令结构
#### 设置
``` json
{
    "cmd":"Setting",
    "type":"name",
    "info":{......}
}
```
#### 设置应答
* 应答成功
``` json
{
    "cmd":"SettingAck",
    "type":"name",
    "ack":0
}
```
*应答失败
``` json
{
    "cmd":"SettingAck",
    "type":"name",
    "ack":X
}
```
### 设置命令类型

***

## 查询
### 查询类命令结构
#### 请求
``` json
{
    "cmd":"Query",
    "type":"name",
}
```
#### 应答
* 应答成功
``` json
{
    "cmd":"QueryAck",
    "type":"name",
    "ack":0,
    "info":{......}
}
```
* 应答失败
``` json
{
    "cmd":"QueryAck",
    "type":"name",
    "ack":X
}
```
### 查询命令类型

***

# 附录

## check
<span id="check"> </span>

## 校验
<span id="checksum"> </span>
