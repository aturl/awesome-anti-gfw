如果安装已编译好的二进制安装包，可以在官方下载：

[zerotier-one_1.2.10_*.deb ](https://download.zerotier.com/RELEASES/1.2.10/dist/debian/bionic/pool/main/z/zerotier-one/)

客户端以 Raspberry Pi 3 B，操作系统为最新版的 Raspbian

```
uname -a
Linux Raspberry Pi 4.14.42-v7+ #1114 SMP Mon May 21 16:39:21 BST 2018 armv7l GNU/Linux
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

因为在 Raspberry Pi 3 编译出错，所以安装了 clang 和 gcc 最新版

编译安装 clang 6 Raspberry Pi - Install Clang 6 and compile C++17 programs[https://solarianprogrammer.com/2018/04/22/raspberry-pi-raspbian-install-clang-compile-cpp-17-programs/]

编译安装 gcc 8.1 Raspberry Pi - Install GCC 8 and compile C++17 programs[https://solarianprogrammer.com/2017/12/08/raspberry-pi-raspbian-install-gcc-compile-cpp-17-programs/]

依然出错，类似下面的提示

```
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make-linux.mk:263: recipe for target 'one' failed
make: *** [one] Error 1
```

修改源码 ```make-linux.mk``` 文件中 236 - 241 行的 ```=-march=armv5``` 为 ```=-march=armv7```，共 4 处。

修改后开始编译安装

```
sudo make && sudo make install
```
执行 ```sudo make``` 会有以下输出，说明成功

```
...
strip --strip-all zerotier-one
ln -sf zerotier-one zerotier-idtool
ln -sf zerotier-one zerotier-cli

继续执行 ```sudo make install``` 会完成编译安装，输出以下内容

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

编译安装完成，开始设置客户端。

查看 ```sudo zerotier-cli info```

输出一下内容

```
zerotier-cli: missing port and zerotier-one.port not found in /var/lib/zerotier-one
```
```
cd ~/ZeroTierOne-1.2.10
sudo cp debian/zerotier-one.service /etc/systemd/system
sudo systemctl daemon-reload
sudo systemctl enable zerotier-one
sudo systemctl start zerotier-one
```
设置和启动系统服务完成，再次查看 ```sudo zerotier-cli info```

```
