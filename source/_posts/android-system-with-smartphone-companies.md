---
title: Android系统演进之手机厂商
categories:
  - Android
tags:
  - Android
  - Binder
description: Android手机厂商除了定制一些符合本土化的需求，还需要做什么？
comments: true
abbrlink: eb89ec1b
date: 2021-10-22 22:33:28
---
<!--more-->
<meta name="referrer" content="no-referrer"/>



# 功能
- 后置指纹、侧边指纹、屏下指纹
- 刘海屏、水滴屏、挖孔屏
- 高刷（90HZ、120HZ、144HZ）
- 快充（大电流、大电压）
- 多摄像头
- 多扬声器
- NFC

 
# 稳定

## 系统自身稳定

- Android大版本升级时AOSP引入的bug
- 厂商新Feature引入的bug


## 三方APP兼容

- APP对新Android版本适配不及时
- APP保活
- APP热更新


# 功耗

## 问题

- Wifi scans
- Bluetooth scans
- Sensor
- GPS
- Network
- CPU
- WakeLock
- Alarm


## 解决
电量的消耗是由于各个sensor硬件的运行，各个软件进程、操作系统的cpu、gpu计算导致的。

### 控制应用启动

- 一像素Activity保活
- 循环播放无声音频保活
- 借用TileService保活
- Broadcast唤醒
- Provider唤醒
- job scheduler唤醒
- 父子service唤醒
- 多进程监控唤醒
- 模拟binder调用唤醒


### 控制后台进程运行

- 定时清理进程、杀死后台运行的进程
- 冻结进程
- 当前应用跑在CPU大核、后台应用跑在CPU小核上
- AES(Energy-Aware Scheduling)调度器方案


### 待机功耗优化

- 解决应用持锁导致系统不休眠
- 解决网络请求、Alarm等中断唤醒AP
- 解决wifi、blutooth、gps、modem、nfc在后台工作


### 场景功耗优化

- 针对特殊app，单独优化
- 减少layer：部分app多余layer，可以通过SurfaceFlinger进行移除
- 长连接：微信微博等app长连接问题可以在kernel层进行netfilter过滤动态限制，减少同步频率


> 了解Android M之后的DozeMode、Device IDLE针对系统灭屏后一段时间后的策略。


# 优化方向

## CPU

- 更优的调度器：比如关注是top-app应用的性能，其它后台进程可以做适当的限制。可以了解Linux主流的CFS（Completely Fair Scheduler）、AES(Energy-Aware Scheduling)调度器方案
- 在合适的时机开关cpu核心数量
- 不同task任务绑定不同大小核：cpuset绑定独占cpu核心，taskset绑定任务指定cpu核心运行
- 控制cpu核心的频率：governor 对cpu的电压频率进行控制；cpu的各种模式：on-demand、performance等
- 各个进程task根据不同的调度策略policy、优先级、进行cpu资源的动态占用，比如top-app、media进程等可能就需要设置较高的实时性策略
- cgroups的cpu控制能限制指定task任务组cpu资源的，比如限制后台进程cpu使用率超过50%，开源节流-减少后台浪费，给前台任务更多资源
- 开放接口给top-app，让合作的app可以根据自身的场景切换设置不同的cpu使用策略，比如cpu boost
- 配置dtsi，cpu超频-为了性能


> 分析工具：
> - 使用ftrace和systrace进行cpu使用情况分析，可以发现各个进程状态、cpu状态、cpu频率繁忙和空闲情况等
> - 基于sampling采样方案
> - 基于instrument的方案：所有函数进入退出进行跟踪
> - hook方案：统计关键点的执行次数和时间
> - 转火焰图分析，快速发现系统瓶颈
> 


> 系统开发：
> 注cpu的饱和度，部分建议为cpu核心的0.7左右。
> android为了减少主线程任务，单独起了RenderThread线程进行draw frame。

> APP开发：
> 对app端来说，在cpu方面要多关注多线程问题，是否使用线程池，毕竟频繁创建线程还是有开销的，直接进行线程缓存，可以节约时间。线程数量是否太少，不能充分利用多核性能？是否太多造成调度延迟。
> - io密集型：可能就要多启动些线程，毕竟线程io block后，干不了其它事情，cpu都空闲出来了，增加线程可以增大抢占cpu的概率
> - cpu密集型：一般cpu-core + 1 个线程；如果系统把cpu资源都给你了，你要每个core都跑一个线程，留1个做备用应对任务调度


## GPU

- 优化ui布局，或交互方式，减少layer层级和对象数量；
- 部分数据可以缓存，比如字体资源等，各类资源纹理等缓存cache可以适当tuning调大；
- 裁剪分辨率，减少数据量；
- gpu的clock，频率tuning；
- gpu数据由串行改为并发执行；
- Adreno和 mali 2类gpu本身调度算法性能优化；



> app ui响应input操作后，通过measure、layout、draw，把具体合成功能给gpu操作。
> android 开始使用skia软件绘制，后面为了提高性能，引入gpu绘制；再后来引入vulkan。


## memory优化
内存不足可能会导致出现各种莫名其妙的问题；内存不足出现各种页回收，碎片整理，swap交换。都可能导致卡顿。

- 维护系统可用内存最低阈值，减少卡顿

早期手机内存不大，一般采用kill杀后台进程方案；后续内存大了，就更多采用的冻结方案。
android整合利用linux的lmk：low memory killer方案，在内存不足时杀优先级低的进程，根据不同内存阈值，进行查杀，直到恢复指定内存数量。
但是默认参数在国内不适用，各家也自己tuning适配的参数。 lmk在app启动阶段，不要进行kill操作，这样可以减少卡顿。
> 内存问题主要注意进程的内存泄漏，可以自己实现内存溢出监控-进程的绝对值和相对值/总内存的百分比阈值，也可以使用cgroups的内存控制功能实现。
> 注意：不要出现swap交换。

- 维护top app可用内存减少卡顿

app默认有最大heap使用限制，默认192MB。部分应用启动需要消耗很多内存，超过gc阈值，就会触发gc回收，造成卡顿。
系统在app启动时候，可用默认调高阈值，启动完后退后台再gc调整恢复阈值。
也可以结合cgroups的 memory模块进行进程的内存限制。尤其是native的进程的限制，避免出现内存泄漏导致系统不稳定。
> 大型app，国民app也可以针对性的设置更大内存配额。减少gc卡顿

- 后台、待机时内存回收和碎片整理

内存问题，大多是容量不够导致的问题。各类方案就是想方设法保证有充足的可用内存。
手机系统和传统的pc或服务器系统的使用场景不一样：

   - 某时刻就那么几个可见app，和1个top app，大部分都是后台app进程；
   - 手机有工作时间和待机时间--人总要睡觉。

内存优化，也可以基于性质，对后台进程进行碎片整理；长时间待机时进行更激进的整体碎片整理和内存优化。disk的trim 也是利用这个特点进行待机碎片整理，也避免开机disk check的耗时。


## 优化内存分配算法

- kernel

系统对大内存按page进行分配； 小内存按slub/slab分配。
> 内存碎片问题，系统运行一段时间后，ddr上留下各种各样的空洞，导致alloc时连续空间不够。 导致page compact页面压缩。

- 用户层

malloc的分配方案也很多，高性能线程安全的有：google的tcmalloc、Facebook的jemalloc。 
android使用dlmalloc和jemalloc。

   - tcmalloc
      - 类似kernel分配方案
      - 各个线程存有ThreadCache局部缓存，进程使用CentralCache中心缓存
      - 小对象走局部缓存
      - 参考slub/slab 分成100+个class等级
      - 大对象走中心缓存：从1page，2pages、3pages,..数组+链表形式
      - 同时增加了回收功能
   - jemalloc
      - 采用线程局部cache
      - 按大小分类small、large、huge
      - 采用chunk方式分割虚拟内存
      - 比tcmalloc开销大点，性能好点



## io/disk和filesystem
用户空间进程io系统调用到vfs虚拟文件系统； vfs再标准化各类文件系统：ext4、f2fs等； 文件系统在对接generic block 块设备； block dev 发起读写请求高io调度器； io调度器类似cpu的进程调度器，看能否合并某些page以期批量处理数据，减少磁盘io操作； 时机一到，触发具体block的读写流程，交由各类块设备driver驱动进行读写磁盘。

### read流程

- 发起system call，系统调用 转到kernel内核态，用户线程block阻塞等待
- vfs通过open时候的dirname找到inode对应的block上的位置返回fd，做个token标识
- 通过读取的文件offset偏移，找到磁盘block对应的物理页映射到page cache缓存，方便下次直接读取缓存，减少磁盘io开销
- 数据copy to user 系统调用返回进程，进程阻塞返回继续执行


### write流程

- 通过fd找到对应inode的对应offset的page缓存页，没有就先预读加载进kernel
- 更新数据到page cache，直接返回用户进程
- kernel再选择时机一起回写到磁盘block，默认是5s


> block driver 是和disk配套的代码控制器，完成block的读写和校验，毕竟磁盘是死的。代码才是活的。
> 参考方案：
> _新的更适合移动设备的flash规范，新的文件系统类型_
> - 根据具体具体业务类型，选择直接read/write， mmap，还是直接io等
> - 关注io操作的次数和buffer大小，按page大小倍速读写，可以减少io次数
> - io调度器电梯算法的请求队列和调度策略设置:cfs、noop、deadline
> - 顺序io可以适当增大预读size
> 

> 各类android 抓取log方案，就采用的是mmap方案，由于进程vma和kernel地址空间是共享的，可以减少1层系统调用开销。
> huawei：iotrace, io monitor

> 对app端来说，sqlite和shared preference性能影响较大。
> - sqlite的优化： page大小、buffer大小；wal模式，关闭统计功能，mmap支持
> - shared preference重写优化：比如mmap方式替换；按uid通过binder进行本地server端更新



## 网络优化

- 设备外的路由间转发
- 网卡收发数据; 网络层ip
- 传输层tcp、udp
- 应用层


## app启动优化

- apk redex优化
- 尽量保证热启动
- 启动时候禁止kill，gc等耗时任务
- Launcher优先响应click事件，其它任务异步化或延迟执行
- view对click事件提前串行执行
- 启动动画单独设计和速率等调整，这个很影响速度
- app进程预创建和缓存-可以节约30-50ms左右
- 耗时class预加载/异步加载
- app的renderthread等预先启动，gpu等环境预先初始化和缓存-节约几十ms
- app的theme、layout等和业务逻辑无关的资源可以预先load和缓存-可以节约上百ms
- app的lib so使用监控，预mmap
- 系统层so合并和缓存，提升so加载速度
- app 减少不必要业务路径，延迟不重要逻辑
- 系统级dns缓存-可以节约100~400ms左右
- app启动流程task化，充分利用多核cpu，避免锁等待
- ams启动大锁优化，有限保证top-app和system_server各个核心线程占有cpu优势资源
- 各类boost和top-app的大核独占绑定、优先级调高，后台进程优先级调低
## app运行优化

- 非主activity等class，可以在startActivity后手动classloader.loadClass()
- looper里消息分类和优先级：类型数组->优先级链表/数组->同优先级消息链表
- tp报点优化：缩短休眠唤醒间隔、提高采样率，优化坐标计算算法
- 不等sync信号，直接先处理input事件流程，点击提前处理，滑动阈值、摩擦力参数优化,input也可以单独线程处理
- view对click事件考虑串行不走post，click effect异步化或优先处理onclick
- 子线程分担ui线程input/../measure/layout任务-facebook方案可以参考
- app动态分辨率-部分app降低分辨率不影响体验
- app自身各类listview\recycleview的item view细化缓存
- app自身layout优化，自定义层级、减少层级等
- framework层measure时动态优化不必要的viewgroups，减少层级
- view跨context缓存，切换activity时更新view的context相关内容
- gpu数据并行化
- SurfaceFlinger过滤多余layer，动态fps
- 其它系统级：app线程的cpu核心绑定，cpu\gpu频率，场景化boost，预留足够内存，减少不必要的io，提高cpu、disk、net 优先级。irq balance。cgroups可以限制cpu、mem、disk 、net、io，要充分利用保证top-app优先使用
- 注意thermal导致的限速降频等
- 整个系统为了高扩展性移植性等架构，多采用分层结构，但客制化对应的系统可以打破原有架构设计，根据top-app等android特有性能设计更好优先级、实时性的场景化方案，比如优先响应top-app的网络请求，io请求等
- 机器码优化运行，减少字节码解析运行--目前几乎都是arm芯片，google可以考虑放弃通用性，单独编译成机器码级别app


> 其实，google现在也越来越关注开发者意见了。系统方可以开放2类api
> - app启动加速api:提供api给app 预加载缓存，提高冷启动速度
> - app运行调频api: 微信把厂商提供给它的boost api封装成Hardcoder也开源的


 
# 安全

- 芯片安全漏洞
- OS漏洞
   - 反编译
   - IEMI、MAC
- 应用
   - permission（国内流氓的应用是不给权限就退出应用，国内各厂商提供的passive permission control更适合隐私控制，app能正常使用，但是敏感权限控制可以拒绝某些数据读取申请。）
   - 热更新
   - webview

copy from：https://zhuanlan.zhihu.com/p/138828988