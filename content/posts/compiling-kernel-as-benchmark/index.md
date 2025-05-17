---
title: "不同机器编译Linux内核的时间"
date: 2023-04-26T22:54:00+08:00
summary: "测试各种不同的机器上编译Kernel的时间"
showtoc: false
authors:
- admin
---

## 说明
### 编译设置
```bash
make defconfig # 默认配置
make clean     # 确保从0开始构建
time make -j16 # 并行作业数16
```


## E楼Ⅱ区203的台式电脑

### 配置
+ OS：Open Kylin
+ CPU： 12 th Gen Intel Core i7-12700 物理核12个，超线程后逻辑核（操作系统视角）有20个
+ RAM: 16 GB
### 结果
```sh
Kernel: arch/x86/boot/bzImage is ready  (#1)

real    2m2.012s
user    26m13.276s
sys     3m9.367s
```

## E楼Ⅱ区207的台式电脑

### 配置
+ OS：Ubuntu 18.04
+ CPU： Intel Core i5-9500（物理核6个，没有超线程）
+ RAM： 8 GB

### 结果
```sh
Kernel: arch/x86/boot/bzImage is ready  (#1)

real    3m29.480s
user    17m34.758s
sys     1m69.298s
```

## Gitpod上的虚拟机资源（免费）
### 配置
+ OS：Ubuntu 22.04
+ CPU：8 cores（Gitpod定义，详细查看[Gitpod Docs](https://www.gitpod.io/docs/configure/workspaces/workspace-classes#workspace-classes)）
+ RAM：16 GB

### 结果
```sh
Kernel: arch/x86/boot/bzImage is ready  (#1)

real    5m14.017s
user    30m24.471s
sys     4m8.968s
```


## 我的笔记本电脑
### 配置
+ OS：Windows Subsystem Linux Ubuntu 22.04
+ CPU：11 th Gen Intel Core i5-1135G7 物理核4个，超线程后逻辑核有8个

### 结果
```sh
Kernel: arch/x86/boot/bzImage is ready  (#1)

real    5m4.384s
user    35m47.059s
sys     2m27.786s
```

## 补充
我的笔记本上其实也装了Ubuntu 22.04，但一编译内核就发热，风扇狂转，自动关机了，也不知道原因，因此只能在Windows Subsystem Linux下测个数据，个人感觉WSL的效率已经很高了。
