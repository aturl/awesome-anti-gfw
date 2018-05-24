# Raspberry Pi 编译安装 ZeroTier

Raspberry Pi 3 B 操作系统为最新版的 Raspbian

```
uname -a
Linux Raspberry Pi 4.14.42-v7+ #1114 SMP Mon May 21 16:39:21 BST 2018 armv7l GNU/Linux
```

如果安装已编译好的二进制安装包，可以在官方下载：

[zerotier-one_1.2.10_*.deb](https://download.zerotier.com/RELEASES/1.2.10/dist/debian/bionic/pool/main/z/zerotier-one/)

将 deb 文件 ```zerotier-one_1.2.10_armhf.deb``` 下载到本地后，执行安装。

```
wget https://download.zerotier.com/RELEASES/1.2.10/dist/debian/bionic/pool/main/z/zerotier-one/zerotier-one_1.2.10_armhf.deb
sudo dpkg -i zerotier-one_1.2.10_armhf.deb
```

假如要从源代码编译安装 ZeroTier，需要 GCC/G++ 4.9.3 或 CLANG/CLANG++ 3.4.2 版本要求。

``` 
sudo apt update
sudo apt install clang
```
或

```
sudo apt install gcc
```

查看 clang 版本 ```clang++ --version```

```
clang version 3.8.1-24+rpi1 (tags/RELEASE_381/final)
Target: armv6--linux-gnueabihf
Thread model: posix
InstalledDir: /usr/bin
```

查看 gcc 版本 ```gcc --version```

```
gcc (Raspbian 6.3.0-18+rpi1+deb9u1) 6.3.0 20170516
Copyright (C) 2016 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

实际上这一版本的 Raspbian 系统从源安装的 clang 和 gcc 已满足最低版本要求，可以略过。 

之前在 Raspberry Pi 3 编译出错，以为是 clang 或 gcc 的版本问题，所以安装了 clang 和 gcc 最新版，只装一种就行，不用两样都装。不管是 clang 6 还是 gcc 8.1，都可以参考下面的链接内容来安装。

编译安装 clang 6 Raspberry Pi - [Install Clang 6 and compile C++17 programs](https://solarianprogrammer.com/2018/04/22/raspberry-pi-raspbian-install-clang-compile-cpp-17-programs/)

编译安装 gcc 8.1 Raspberry Pi - [Install GCC 8 and compile C++17 programs](https://solarianprogrammer.com/2017/12/08/raspberry-pi-raspbian-install-gcc-compile-cpp-17-programs/)

依然出错，类似下面的提示（这只是安装记录，错误原因在下面）。

```
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make-linux.mk:263: recipe for target 'one' failed
make: *** [one] Error 1
```

实际错误的原因是源码中指定的 ARM 架构版本问题，应该将 armv5 修改为 armv7

修改源码 ```make-linux.mk``` 文件中 236 - 241 行的 ```=-march=armv5``` 为 ```=-march=armv7```，共 4 处。

```vi make-linux.mk
```

```
		override CFLAGS+=-march=armv7 -mfloat-abi=soft -msoft-float -mno-unaligned-access -marm
		override CXXFLAGS+=-march=armv7 -mfloat-abi=soft -msoft-float -mno-unaligned-access -marm
		ZT_USE_ARM32_NEON_ASM_CRYPTO=0
	else
		override CFLAGS+=-march=armv7 -mno-unaligned-access -marm
		override CXXFLAGS+=-march=armv7 -mno-unaligned-access -marm
```    

修改后开始编译安装

```
sudo make && sudo make install
```

执行 ```sudo make``` 会有以下输出，说明安装文件已准备就绪：

```
...
strip --strip-all zerotier-one
ln -sf zerotier-one zerotier-idtool
ln -sf zerotier-one zerotier-cli
```

继续执行 ```sudo make install``` 会完成编译安装，输出以下内容：

```
kdir -p /usr/sbin
rm -f /usr/sbin/zerotier-one
cp -f zerotier-one /usr/sbin/zerotier-one
rm -f /usr/sbin/zerotier-cli
rm -f /usr/sbin/zerotier-idtool
ln -s zerotier-one /usr/sbin/zerotier-cli
ln -s zerotier-one /usr/sbin/zerotier-idtool
mkdir -p /var/lib/zerotier-one
rm -f /var/lib/zerotier-one/zerotier-one
rm -f /var/lib/zerotier-one/zerotier-cli
rm -f /var/lib/zerotier-one/zerotier-idtool
ln -s ../../../usr/sbin/zerotier-one /var/lib/zerotier-one/zerotier-one
ln -s ../../../usr/sbin/zerotier-one /var/lib/zerotier-one/zerotier-cli
ln -s ../../../usr/sbin/zerotier-one /var/lib/zerotier-one/zerotier-idtool
mkdir -p /usr/share/man/man8
rm -f /usr/share/man/man8/zerotier-one.8.gz
cat doc/zerotier-one.8 | gzip -9 >/usr/share/man/man8/zerotier-one.8.gz
mkdir -p /usr/share/man/man1
rm -f /usr/share/man/man1/zerotier-idtool.1.gz
rm -f /usr/share/man/man1/zerotier-cli.1.gz
cat doc/zerotier-cli.1 | gzip -9 >/usr/share/man/man1/zerotier-cli.1.gz
cat doc/zerotier-idtool.1 | gzip -9 >/usr/share/man/man1/zerotier-idtool.1.gz
```

编译安装完成，查看安装情况：

查看 ```sudo zerotier-cli info```

输出以下内容：

```
zerotier-cli: missing port and zerotier-one.port not found in /var/lib/zerotier-one
```
上面的提示是因为没有启动 ```zerotier-one``` 引起的，可以添加到系统服务中，由 systemctl 管理。

```
cd ~/ZeroTierOne-1.2.10
sudo cp debian/zerotier-one.service /etc/systemd/system
sudo systemctl daemon-reload
sudo systemctl enable zerotier-one
sudo systemctl start zerotier-one
```
设置和启动系统服务完成，再次查看 ```sudo zerotier-cli info```

```
sudo zerotier-cli info
200 info 1927139b92 1.2.10 ONLINE
```

显示状态 200 在线，版本号为 1.2.10
