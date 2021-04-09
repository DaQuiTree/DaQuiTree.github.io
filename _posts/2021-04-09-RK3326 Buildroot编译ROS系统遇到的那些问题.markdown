---
layout: post
title:  "libpcl移植"
date:   2021-04-09 8:08:48 +0800
categories: jekyll update
---

## 简介：

本文主要使用RK3326 SDK，记录了在使用buildroot构造系统以及添加对ROS系统支持过程中遇到的大大小小的问题和它们的解决办法。下文以E代指error错误，S代指solution解决方法，A代指Analysis分析。



## 正文：

### E1：host-python-empy 3.3.3

```bash
>>> host-python-empy 3.3.3 Installing to host directory
(cd /home/daquitree/RKSDK/rk3326/buildroot/output/rockchip_rk3326_robot64/build/host-python-empy-3.3.3//; PATH="/home/daquitree/RKSDK/rk3326/buildroot/output/rockchip_rk3326_robot64/host/bin:/home/daquitree/RKSDK/rk3326/buildroot/output/rockchip_rk3326_robot64/host/sbin:/home/daquitree/bin:/home/daquitree/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/home/daquitree/bin:" PYTHONNOUSERSITE=1  /home/daquitree/RKSDK/rk3326/buildroot/output/rockchip_rk3326_robot64/host/bin/python setup.py install --prefix=/home/daquitree/RKSDK/rk3326/buildroot/output/rockchip_rk3326_robot64/host --root=/ --single-version-externally-managed )
usage: setup.py [global_opts] cmd1 [cmd1_opts] [cmd2 [cmd2_opts] ...]
   or: setup.py --help [cmd1 cmd2 ...]
   or: setup.py --help-commands
   or: setup.py cmd --help

error: option --single-version-externally-managed not recognized

```

### S1:

修改empy 3.3.3文件夹下的setup.py文件

```bash
#!/usr/bin/env python
#
# $Id: setup.py.pre 3116 2004-01-14 02:53:02Z max $ $Date: 2004-01-13 18:53:02 -0800 (Tue, 13 Jan 2004) $

#from distutils.core import setup
from setuptools import setup	#修改位置

DESCRIPTION = "A templating system for Python."

...
```

### A1:

distutils是老的包管理方式，我查看了一下python-empy.mk中设定的是setuptools包管理方式。而empy 3.3.3直接解压后是setup.py指定的是distutils。二者需做统一。



------



### E2：em

```
-- catkin 0.7.14
/home/daquitree/RKSDK/rk3326/buildroot/output/rockchip_rk3326_robot64/host/bin/python: can't find '__main__' module in '/home/daquitree/RKSDK/rk3326/buildroot/output/rockchip_rk3326_robot64/host/lib/python2.7/site-packages/em'
```

### S2&A2：

由于site-packages文件夹下存在em.py和em文件夹，cmake在编译时默认识别了em文件夹。因此删除掉site-packages/em文件夹即可。



------



### S3: gmapping编译

```bash
../../host/aarch64-buildroot-linux-gnu/sysroot/usr/include/flann/ext/lz4.h:249:72: error: conflicting declaration ‘typedef struct LZ4_streamDecode_t LZ4_streamDecode_t’

```

### S3&A3:

由于编译过程中host中存在两个lz4.h，host/usr/include/lz4.h 与 host/usr/include/flann/ext/lz4.h。须统一为后者。由于是报重复定义的错，因此仅保留host/usr/include/lz4.h中的报错行即可，即注释掉 host/usr/include/flann/ext/lz4.h中的报错行。



------



## 结语

在完成基于ros相关的APP的编译后，buildroot构建就告一段落了。未来还会有新的硬件加入到平台，也就需要相应的驱动和库的移植。本文会持续更新用以记录遇到的各种编译问题......