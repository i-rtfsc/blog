---
title: Android 12-A/B System Update Engine-简介
top: 98
categories:
  - Android
tags:
  - Android
  - OTA
  - A/B System
description: Android 12-A/B System Update Engine-简介
comments: true
abbrlink: dd679192
date: 2021-10-27 22:06:42
---
<!--more-->
<meta name="referrer" content="no-referrer"/>


# Android 12-A/B System Update Engine
[Android 12-A/B System Update Engine-简介](https://i-rtfsc.github.io/archives/dd679192.html)
[Android 12-A/B System Update Engine-01](https://i-rtfsc.github.io/archives/f4e627d8.html)
[Android 12-A/B System Update Engine-02](https://i-rtfsc.github.io/archives/f4e511d8.html)
[Android 12-A/B System Update Engine-03](https://i-rtfsc.github.io/archives/84bf1fe3.html)
[Android 12-A/B System Update Engine-04](https://i-rtfsc.github.io/archives/1adb8a40.html)
[Android 12-A/B System Update Engine-05](https://i-rtfsc.github.io/archives/6ddcbad6.html)
[Android 12-A/B System Update Engine-06](https://i-rtfsc.github.io/archives/f4d5eb6c.html)
[Android 12-A/B System Update Engine-07](https://i-rtfsc.github.io/archives/83d2dbfa.html)
[Android 12-A/B System Update Engine-08](https://i-rtfsc.github.io/archives/136dc66b.html)



# 简介
OTA是Over-the-Air的简称，OTA升级可以理解为用户正常使用过程中进行升级，OTA 升级旨在升级基础操作系统、系统分区上安装的只读应用和/或时区规则。


# 非A/B System更新
Android 7.0及更低版本传统的OTA方式：设备上有一个Android主系统和一个Recovery系统，Android主系统运行时检测是否需要升级，如果需要升级，则将升级的数据包下载并存放到cache分区，重启系统后进入Recovery系统，并用cache分区下载好的数据更新Android主系统，更新完成后重新启动进入Android主系统。
如果更新失败，设备重启后就不能正常使用了，唯一的办法就是重新升级，直到成功为止。
OTA更新流程：

- Android设备会与 OTA 服务器进行定期确认，获知是否有更新可用，包括更新软件包的 URL 和向用户显示的描述字符串。
- 将更新下载到 cache 或 userdata 分区，并根据 /system/etc/security/otacerts.zip 中的证书验证加密签名。系统提示用户安装更新。
- Android设备重新启动进入恢复模式，引导 recovery 分区中的内核和系统（而非 boot 分区中的内核）启动。
- recovery 分区的二进制文件由 init 启动。它会在 /cache/recovery/command 中寻找将其指向下载软件包的命令行参数。
- 恢复操作会根据 /res/keys（包含在 recovery 分区中的 RAM 磁盘的一部分）中的公钥来验证软件包的加密签名。
- 从软件包中提取数据，并根据需要使用该数据更新 boot、system或vendor分区。system分区上的某个新文件包含新的 recovery 分区的内容。
- 设备正常重启。
   - 加载最新更新的 boot 分区，在最新更新的 system 分区中装载并开始执行二进制文件。
   - 作为正常启动的一部分，系统会根据所需内容（预先存储为 /system 中的一个文件）检查 recovery 分区的内容。二者内容不同，所以 recovery 分区会被所需内容重新刷写（在后续引导中，recovery 分区已经包含新内容，因此无需重新刷写）。
> 系统更新完成！更新日志可以在 /cache/recovery/last_log.**XXX** 中找到


## 分区

- bootloader：存放用于引导linux的bootloader
- boot：存放Android主系统的linux kernel文件和用于挂载system和其他分区的ramdisk
- system：Android主系统分区，包括Android的系统应用程序和库文件
- vendor：Android主系统分区，主要是包含开发厂商定制的一些应用和库文件，很多时候开发厂商也直接将这个分区的内容直接放入system分区
- userdata：用户数据分区，存放用户数据，包括用户安装的应用程序和使用时生成的数据
- cache：临时存放数据的分区，通常用于存放OTA的升级包
- recovery：存放Recovery系统的linux kernel文件和ramdisk
- misc：存放Android主系统和Recovery系统跟bootloader通信的数据


## 文OTA
Android 4.4及更低版本使用文件 OTA 更新，确保设备包含类似的文件内容、权限和模式，但允许时间戳和底层存储的布局等元数据在各设备之间因更新方法而异。
在进行基于文件的 OTA期间，Android会尝试在文件系统层更改系统分区的内容（逐个更改）。更新无法保证按照一致的顺序写入文件、具有一致的上次修改时间或超级块、甚至将块放置在块设备上的同一位置。因此，在启用dm-verity的设备上执行基于文件的OTA将会失败；进行OTA尝试后，设备不会启动。
## 块OTA
Android 5.0及更高版本使用块OTA更新，以确保每个设备使用的分区完全相同。块OTA不会比较各个文件，也不会分别计算各个二进制补丁程序，而是将整个分区处理为一个文件并计算单个二进制补丁程序，以确保生成的分区刚好包含预期的位数。这样一来，设备系统映像就能够通过fastboot或OTA达到相同的状态。
在执行基于块的OTA期间，Android为设备提供两个块映像（而非两组文件）之间的差异。该更新使用以下方法之一在块级别（位于文件系统下方）针对相应构建服务器来检查设备版本：

- 完整更新：复制整个系统映像很容易操作且易于生成补丁程序，但同时导致生成的映像较大，因而增加了应用补丁程序的成本。
- 增量更新：使用二进制文件differ工具可生成较小的映像且易于应用补丁程序，但在生成补丁程序本身时却占用大量内存。


# A/B System更新
Android从7.0开始引入新的OTA升级方式，A/B System Updates，这里将其叫做A/B系统。 A/B系统就是设备上有A和B两套可以工作的系统（用户数据只有一份，为两套系统共用），简单来讲，可以理解为一套系统分区，另外一套为备份分区。其系统版本可能一样；也可能不一样，其中一个是新版本，另外一个旧版本，通过升级，将旧版本也更新为新版本。当然，设备出厂时这两套系统肯定是一样的。
> Android7.0上传统OTA方式和新的A/B系统方式都存在，只是编译时只能选择其中的一种OTA方式。由于A/B系统在分区上与传统OTA的分区设计不一样，二者无法兼容，所以7.0以前的系统无法通过OTA方式升级为A/B系统。
> Android系统不是由一个分区组成，其系统包括boot分区的kernel和ramdisk，system和vendor分区的应用程序和库文件，以及userdata分区的数据。


## 分区

- bootloader：存放用于引导linux的bootloader
- boot_a和boot_b：分别用于存放两套系统各自的linux kernel文件和用于挂载system和其他分区的ramdisk
- system_a和system_b：Android主系统分区，分别用于存放两套系统各自的系统应用程序和库文件
- vendor_a和vendor_b：Android主系统分区， 分别用于存放两套系统各自的开发厂商定制的一些应用和库文件，很多时候开发厂商也直接将这个分区的内容直接放入system分区
- userdata：用户数据分区，存放用户数据，包括用户安装的应用程序和使用时生成的数据
- misc或其他名字分区：存放Android主系统和Recovery系统跟bootloader通信的数据，由于存放方式和分区名字没有强制要求，所以部分实现上保留了misc分区（代码中可见Brillo和Intel的平台），另外部分实现采用其他分区存放数据（Broadcom机顶盒平台采用名为eio的分区）。


### 区别
传统分区和A/B系统分区的主要区别在于A/B系统：

- boot，system和vendor系统分区从传统的一套变为两套，叫做slot A和slot B
- 不再需要cache和recovery分区
- misc分区不是必须


> 关于cache和misc分区：
> - 仍然有部分厂家保留cache分区，用于一些其他的用途，相当于temp文件夹，但不是用于存放下载的OTA数据。
> - 部分厂家使用misc分区存放Android系统同bootloader通信数据，另有部分厂家使用其它名字的分区存储这类数据。

![absystem.png](https://cdn.nlark.com/yuque/0/2021/png/1759879/1635317064936-277e35b4-d7c6-45a7-859c-7a4e933299e4.png#clientId=u654c695b-19fc-4&from=ui&id=u89f99052&margin=%5Bobject%20Object%5D&name=absystem.png&originHeight=401&originWidth=877&originalType=binary&ratio=1&size=34503&status=done&style=none&taskId=u62e3ac0e-32b4-4793-994e-c09cda8e02e)


## 状态
### 属性
对于A/B系统的slot A和slot B分区，其都存在以下三个属性：

- active：系统的活动分区标识，这是一个排他属性，系统只能有一个分区设置为active属性，启动时bootloader选取设置为active的分区进行启动。
- bootable：分区可启动标识，设置为bootable的分区表明该分区包含了一个完整的可以启动的系统。
- successful：分区成功运行标识，设置为successful的分区表明该分区在上一次启动或当前启动中可以正确运行。


### 场景
![absystem_status.png](https://cdn.nlark.com/yuque/0/2021/png/1759879/1635317948605-f4f14cf9-e4da-4c06-83cb-880e37528be4.png#clientId=u654c695b-19fc-4&from=ui&id=u03d98af6&margin=%5Bobject%20Object%5D&name=absystem_status.png&originHeight=696&originWidth=783&originalType=binary&ratio=1&size=56382&status=done&style=none&taskId=u10178ec4-9c72-45b5-959f-f67d3cc2c54)


颜色说明：

- 当前运行的系统用绿色方框表示，当前没运行的系统用灰色方框表示
- 属性标识为红色，表示该状态下相应属性被设置，标识为灰色标识该状态下属性没有设置或设置为相反属性


场景详情：

- Normal cases（普通场景）：最常见的情形，例如设备出厂时，A分区和B分区都可以成功启动并正确运行，所以两个分区都设置为bootable和successful，但由于是从B分区启动，所以只有B分区设置为active。
- Update in progress（升级中）：B分区检测到升级数据，在A分区进行升级，此时将A分区标识为unbootable，另外清除successful标识；B分区仍然为active，bootable和successful。
- Update applied, reboot pending（更新完成，等待重启）：B分区将A分区成功更新后，将A分区标识为bootable。另外，由于重启后需要从A分区启动，所以也需要将A分区设置为active，但是由于还没有验证过A分区是否能成功运行，所以不设置successful；B分区的状态变为bootable和successful，但没有active。
- System rebooted into new update（从新系统成功启动）：设备重启后，bootloader检测到A分区为active，所以加载A分区系统。进入A系统后如果能正确运行，需要将A分区标识为successful。对比第1个普通场景，A和B系统都设置为bootable和successful，但active从B分区切换到A分区。至此，B分区成功更新并切换到A分区，设备重新进入普通场景。


### [举例](https://proandroiddev.com/evolution-of-android-update-system-5385b9bc860a)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1759879/1635335976691-6009874f-e661-422e-b57c-8c12d8915a64.png#clientId=uc5c03fbe-ab21-4&from=paste&id=uc33bb4ce&margin=%5Bobject%20Object%5D&name=image.png&originHeight=379&originWidth=700&originalType=url&ratio=1&size=167574&status=done&style=none&taskId=u39e15fe6-f434-442f-a65e-688d8f866ee)
[

](https://proandroiddev.com/evolution-of-android-update-system-5385b9bc860a)


## 优点

- OTA更新可以在系统运行期间进行，而不会打断用户。用户可以在OTA期间继续使用其设备。在更新期间，唯一的一次宕机发生在设备重新启动到更新后的磁盘分区时。
- 更新后，重新启动所用的时间不会超过常规（非AB升级方式）重新启动所用的时间。
- 如果OTA失败，设备会启动到OTA之前的磁盘分区，并且仍然可以使用。并且客户端可以重新尝试进行更新。
- 如果 OTA 更新已应用但无法启动，设备将重新启动回旧分区，并且仍然可以使用。客户端可以重新尝试进行更新。
- 缓存分区不再用于存储OTA更新包，因此无需确保缓存分区的大小要足以应对日后的更新。
- dm-verity可保证设备将使用未损坏的启动映像。如果设备因OTA错误或 dm-verity问题而无法启动，则可以重新启动到旧映像。（Android 验证启动不需要 A/B 更新。）


# 主要变量及模块


## AB_OTA_UPDATER
```
http://aospxref.com/android-11.0.0_r21/xref/device/google/sunfish/BoardConfig-common.mk

AB_OTA_UPDATER := true
```
不同的设备生产商，定义的位置不一样。比如一加的就定义在：device/oneplus/sm8350-common/BoardConfigCommon.mk
AB_OTA_UPDATER是A/B系统的主要开关变量，设置后：

- recovery系统内不再具有操作cache分区的功能，[device.cpp](http://aospxref.com/android-11.0.0_r21/xref/bootable/recovery/recovery_ui/device.cpp)
- recovery系统使用不同的方式来解析升级文件，[install.cpp](http://aospxref.com/android-11.0.0_r21/xref/bootable/recovery/install/install.cpp)
- 生成A/B系统相关的META文件


## AB_OTA_PARTITIONS
```
http://aospxref.com/android-11.0.0_r21/xref/device/google/sunfish/BoardConfig-common.mk

AB_OTA_PARTITIONS += \
     boot \
     system \
     vbmeta \
     dtbo \
     product \
     system_ext \
     vbmeta_system 
```
将A/B系统可升级的分区写入文件$(zip_root)/META/ab_partitions.txt，如一加9 pro手机：out/target/product/lemonadep/obj/PACKAGING/target_files_intermediates/lineage_lemonadep-target_files-eng.solo/META/ab_partitions.txt
每个手机厂定义AB_OTA_PARTITIONS的内容都不太一样，比如一加：
```
device/oneplus/sm8350-common/BoardConfigCommon.mk

AB_OTA_PARTITIONS += \
    boot \
    dtbo \
    odm \
    product \
    system \
    system_ext \
    vbmeta \
    vbmeta_system \
    vendor \
    vendor_boot
```

## BOARD_BUILD_SYSTEM_ROOT_IMAGE
搭载Android 9的所有新设备都必须使用system-as-root（BOARD_BUILD_SYSTEM_ROOT_IMAGE必须为true），它可以将ramdisk.img合并到system.img，也就是说它的rootfs是集成到system.img中。[关于system-as-root，可以参考这篇文章。](https://www.shangyexin.com/2019/01/08/android-9-system-as-root/)

## update_engine模块
```
http://aospxref.com/android-11.0.0_r21/xref/device/google/sunfish/device.mk

# A/B support
PRODUCT_PACKAGES += \
    otapreopt_script \
    cppreopts.sh \
    update_engine \
    update_verifier


PRODUCT_PACKAGES += \
    update_engine_sideload \
    ...

# The following modules are included in debuggable builds only.
PRODUCT_PACKAGES_DEBUG += \
    ...
    update_engine_client  
```
编译update_engine、update_verifier和update_engine_sideload模块，并安装相应的应用。debug版本还编译一个update_engine_client客户端。
> 有的手机厂商不编译update_engine_sideload
> 非A/B升级的方式调用[RecoverySystem.installPackage()](http://aospxref.com/android-11.0.0_r21/xref/frameworks/base/core/java/android/os/RecoverySystem.java)，可以参考[这篇文章](https://skytoby.github.io/2019/Android%20OTA%E5%8D%87%E7%BA%A7%E6%B5%81%E7%A8%8B%E5%88%86%E6%9E%90/)的流程。
> A/B升级的方式入口是[UpdateEngine.applyPayload()](http://aospxref.com/android-11.0.0_r21/xref/frameworks/base/core/java/android/os/UpdateEngine.java)，直接调到了[update_engine](http://aospxref.com/android-11.0.0_r21/xref/system/update_engine/)模块。[update_engine](http://aospxref.com/android-11.0.0_r21/xref/system/update_engine/)模块非常复杂，后续会继续更新文档记录了解这个代码模块的过程。


# 参考
[OTA 更新](https://source.android.google.cn/devices/tech/ota)
[OTA制作及升级过程笔记](https://blog.csdn.net/teliduxing1029/article/details/51536560)


