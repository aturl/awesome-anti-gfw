# 在 Windows 8.1 系统安装 ZeroTier 1.8.* 的简单记录

假如在 Windows 8.1 系统安装 ZeroTier 1.8.* 遇到安装错误，不能启动服务或创建虚拟网卡，有些情况可能用以下方法得以解决：

如果安装 ZeroTier 1.8.* 时失败，停止已启动的服务并卸载安装的程序。

下载安装 ZeroTier 1.6.6 (https://download.zerotier.com/RELEASES/1.6.6/dist/ZeroTierOne.msi)

在 ```C:\ProgramData\ZeroTier\One\tap-windows\x64``` 目录内找到 ```zttap300.inf``` 文件，点击右键执行“安装”。

下载安装 ZeroTier 1.8.6/7 并覆盖安装 (https://download.zerotier.com/RELEASES/1.8.6/dist/ZeroTierOne.msi)

详见 ZeroTier 知识库 https://zerotier.atlassian.net/wiki/spaces/SD/pages/35455014/PORT+ERROR+on+Windows
