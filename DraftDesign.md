# 潦草设计
## 工具开发目的
- 简化从裸金属设备到具备开发条件或部署自己的服务的过程，简化自己手动安装系统，配置开发环境的步骤

## 说明
开发者当前配置一台机器做开发过程：
选择系统镜像，安装 
安装：选择语言，选择时区....reboot
更新git、安装dotnet、docker

## 目标
- 选择系统镜像，配置基本参数，自动安装系统
- (可选) 一键更新git；一键安装docker；一键安装dotnet

## 问题分解
### 前置条件检查方案
#### 检查程序是否正常运行
1. 程序运行

自动测试：
下载文件服务样例测试文件；
访问http://server:port/bootfile/{Mac}/{IP}/{UUID}接口，检查是否能正常访问；

程序运行后打开前端网页，检查是否能正常打开


#### 程序中检查
1. 填入目标机器的BMC ip，用户名，密码，检查是否能连接控制
    已知可能会出现的错误：
```bash
    Object reference not set to an instance of an object. #redfish接口错误

    The input does not contain any JSON tokens. # ip账号密码错误

    BMC Login Faild. # ip账号密码错误

```
true->下一步，false->根据错误反馈不同提示、提示检查网络

2. 挂载镜像时
挂载镜像时如果失败，需要提示用户检查网络环境，检查本地文件服务，检查目标机器所在网络是否能访问本地文件服务；
true->下一步

3. 机器重启时
本地记录重启机器的uuid，时间，开始重启计时5分钟内没有访问以下内容视为网络出错:
本地服务开启接口，目标机器执行脚本chain http://server:port/bootfile/{Mac}/{IP}/{UUID}，接收到请求后，根据Mac，IP，UUID判断机器网络正常
记录ip，mac，uuid，时间。

4. 机器重启时，获取文件系统内核文件镜像，安装脚本
文件服务日志记录，IP，获取的文件






### 系统安装需要功能
    引导服务：
    当机器访问引导服务api时，根据机器的uuid返回对应的引导脚本，引导安装系统
    GetScriptsByUuid(string uuid) -> return string scripts

    生成启动镜像：
    界面上选择安装配置时，生成对应的安装脚本和ipxe启动镜像
    Buildiso( string script ) -> return iso

    Redfish：
    控制目标机器，将启动镜像挂载，控制机器重启
    Redfish( string ip,string uname,string pwd ,iso) -> return bool

    文件服务：
    提供安装系统需要的镜像，内核，文件系统，安装脚本等文件

贴图：
![a9869a88539ec7fe8e7384f1cf213bf.png](https://img1.imgtp.com/2023/07/24/h8RueY4p.png)

    
### 安装系统配置如何生效
    选择的时区、语言、网络等配置生效方式：
    根据选择的系统镜像，生成对应的自动安装脚本，安装脚本中包含了选择的配置信息，安装完成后，配置生效

### 服务管理
    ssh过去执行脚本或命令。
