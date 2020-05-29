---
title: usb接口键盘转PS2接口
urlname: gnh3hv
date: 2019-03-03 23:09:24 +0800
tags: [Arduino,硬件]
categories: []
---

## 用途

公司或者主机设备没有 usb 接口，只能使用 PS/2 接口键盘。此程序完成 USB HID keyboard 协议向 PS/2 的 scan code set 2 转换。

## 需要的硬件设备

1. arduino uno R3
2. USB Host Shield

## 使用说明

1. 默认 PS/2 接口的的四条数据线中，数据段和时钟段连接在 Arduino 的（4，2）。
2. 使用 IDE 刷入 usb2ps2.ino 即可

## 源代码地址

[https://github.com/limao693/usb2ps2](https://github.com/limao693/usb2ps2)

## 参考文件

1. [USB HID to PS/2 Scan Code Translation Table](https://download.microsoft.com/download/1/6/1/161ba512-40e2-4cc9-843a-923143f3456c/translate.pdf)
2. [PS2 键盘接口说明](http://www.burtonsys.com/ps2_chapweske.htm)
3. [模拟 PS/2 键盘](https://www.arduino.cn/thread-77766-1-1.html)

## Application scenario

The company or host device does not have a usb interface and can only use the PS/2 interface keyboard. This program completes the conversion of the USB HID keyboard protocol to the PS/2 scan code set 2.

## Required hardware devices

1. Arduino uno R3
2. USB Host Shield

## Instructions for use

1. Among the four data lines of the default PS/2 interface, the data segment and the clock segment are connected to the Arduino (4, 2).
2. Use the IDE to brush usb2ps2.ino

## Source code

[https://github.com/limao693/usb2ps2](https://github.com/limao693/usb2ps2)

## reference document

1. [USB HID to PS/2 Scan Code Translation Table](https://download.microsoft.com/download/1/6/1/161ba512-40e2-4cc9-843a-923143f3456c/translate.pdf)
2. [PS2 Keyboard Interface Description](http://www.burtonsys.com/ps2_chapweske.htm)
3. [Analog PS/2 Keyboard](https://www.arduino.cn/thread-77766-1-1.html)
