---
title: 如何学习Android Audio
categories:
  - Android
tags:
  - Android
  - Audio
description: 介绍Android Audio基本概念以及如何学习
comments: true
abbrlink: c7d23d1f
date: 2021-11-09 21:37:11
---
<!--more-->
<meta name="referrer" content="no-referrer"/>

# 音频框架
![N|Solid](https://source.android.google.cn/devices/audio/images/ape_fwk_audio.png)

# 音频术语
- PCM 脉冲编码调制
    对应的数据格式有几种:	Q0.15	Q0.7 1	Q0.23	Q0.31	浮点数
    [数据格式定义](https://source.android.google.cn/devices/audio/data_formats):https://source.android.google.cn/devices/audio/data_formats

- 奈奎斯特频率
    可由离散信号以指定采样率的一半表示的最大频率分量。例如，由于人类的听力范围可达到近 20 kHz，因此数字音频信号的采样率必须至少有 40 kHz 才能代表该范围。在实践中，44.1 kHz 和 48 kHz 的采样率比较常用，对应的奈奎斯特频率分别为 22.05 kHz 和 24 kHz。如需了解详情，请参阅奈奎斯特频率和听力范围。

- 每样本位数或位深
    每个样本的信息位数。

- 声道
    单个音频信息流，通常与一个录音位置或播放位置相对应。

-  Hz
    采样率或帧率的单位。

- 采样率或帧率
    每秒帧数。“帧率”这一用法更为准确，但业内习惯使用“采样率”来表示帧率。

- FIFO
    先进先出。用于实现数据队列先进先出的硬件模块或软件数据结构。谈到音频时，存储在队列中的数据通常是音频帧。FIFO 可通过环形缓冲区来实现。

- 立体声
    两个声道。

- 音量
    响度，音频信号的主观强度。

- 帧
    某个时间点上的样本集，每个声道对应一个样本。

- 每缓冲区帧数
    同时从一个模块传递到另一个模块的帧数。音频 HAL 接口会使用每缓冲区帧数这一概念。

- 欠载/溢出
    类似underrun和overrun

[更多音频术语](https://source.android.google.cn/devices/audio/terminology):https://source.android.google.cn/devices/audio/terminology


# 手机的音量管理

Android上的音量集中在AudioService AudioPolicyManager AudioFlinger 3个地方，
他们的设置过程是从AudioService -> AudioPolicyManager -> AudioFlinger
AudioFlinger 主要是去获取音频设备的属性，比如他的音量多大，是不是静音
AudioPolicyManager 会去加载音频场景，音量曲线，音频类型
AudioService 负责管理不同音频流类型的音量，设备音量，用户交互。其中音量框的展示调节交互是在SystemUI中实现IVolumeController接口来绑定的


## 音频场景
为了适配不同场景的需求。比如铃声，音乐，通知，系统，耳机，蓝牙等，Andorid里定义了很多音频场景来满足。
        
AudioAttributes中有很多使用场景枚举，这些会决定在AudioPolicy侧选用对应的策略执行

```sh
    /**
     * Content type value to use when the content type is unknown, or other than the ones defined.
     */
    public final static int CONTENT_TYPE_UNKNOWN = 0;
    /**
     * Content type value to use when the content type is speech.
     */
    public final static int CONTENT_TYPE_SPEECH = 1;
    /**
     * Content type value to use when the content type is music.
     */
    public final static int CONTENT_TYPE_MUSIC = 2;
    /**
     * Content type value to use when the content type is a soundtrack, typically accompanying
     * a movie or TV program.
     */
    public final static int CONTENT_TYPE_MOVIE = 3;
    /**
     * Content type value to use when the content type is a sound used to accompany a user
     * action, such as a beep or sound effect expressing a key click, or event, such as the
     * type of a sound for a bonus being received in a game. These sounds are mostly synthesized
     * or short Foley sounds.
     */
    public final static int CONTENT_TYPE_SONIFICATION = 4;
    
    /**
     * Usage value to use when the usage is unknown.
     */
    public final static int USAGE_UNKNOWN = 0;
    /**
     * Usage value to use when the usage is media, such as music, or movie
     * soundtracks.
     */
    public final static int USAGE_MEDIA = 1;
    /**
     * Usage value to use when the usage is voice communications, such as telephony
     * or VoIP.
     */
    public final static int USAGE_VOICE_COMMUNICATION = 2;
```

AudioSteamType 音频流类型：
```sh
    /** Used to identify the default audio stream volume */
    public static final int STREAM_DEFAULT = -1;
    /** Used to identify the volume of audio streams for phone calls */
    public static final int STREAM_VOICE_CALL = 0;
    /** Used to identify the volume of audio streams for system sounds */
    public static final int STREAM_SYSTEM = 1;
    /** Used to identify the volume of audio streams for the phone ring and message alerts */
    public static final int STREAM_RING = 2;
    /** Used to identify the volume of audio streams for music playback */
    public static final int STREAM_MUSIC = 3;
    /** Used to identify the volume of audio streams for alarms */
    public static final int STREAM_ALARM = 4;
    /** Used to identify the volume of audio streams for notifications */
    public static final int STREAM_NOTIFICATION = 5;
    /** Used to identify the volume of audio streams for phone calls when connected on bluetooth */
    public static final int STREAM_BLUETOOTH_SCO = 6;
    /** Used to identify the volume of audio streams for enforced system sounds in certain
     * countries (e.g camera in Japan) */
    @UnsupportedAppUsage
    public static final int STREAM_SYSTEM_ENFORCED = 7;
    /** Used to identify the volume of audio streams for DTMF tones */
    public static final int STREAM_DTMF = 8;
    /** Used to identify the volume of audio streams exclusively transmitted through the
     *  speaker (TTS) of the device */
    public static final int STREAM_TTS = 9;
    /** Used to identify the volume of audio streams for accessibility prompts */
    public static final int STREAM_ACCESSIBILITY = 10;
    
    
    //对齐音频流，代表哪些音量类型是同步一起变化的。
    private final int[] STREAM_VOLUME_ALIAS_VOICE = new int[] {
        AudioSystem.STREAM_VOICE_CALL,      // STREAM_VOICE_CALL
        AudioSystem.STREAM_RING,            // STREAM_SYSTEM
        AudioSystem.STREAM_RING,            // STREAM_RING
        AudioSystem.STREAM_MUSIC,           // STREAM_MUSIC
        AudioSystem.STREAM_ALARM,           // STREAM_ALARM
        AudioSystem.STREAM_RING,            // STREAM_NOTIFICATION
        AudioSystem.STREAM_BLUETOOTH_SCO,   // STREAM_BLUETOOTH_SCO
        AudioSystem.STREAM_RING,            // STREAM_SYSTEM_ENFORCED
        AudioSystem.STREAM_RING,            // STREAM_DTMF
        AudioSystem.STREAM_MUSIC,           // STREAM_TTS
        AudioSystem.STREAM_MUSIC            // STREAM_ACCESSIBILITY
    };
```

## 音频流的音量初始化
每一个音频流类型都会创建一个VolumeStreamState实例，并且在开机后会加载每一个流的最大，最小音量。
并且从VolumeStreamState.readSettings()接口从Settings中初始化音量等级。

```sh
    private void createStreamStates() {
        int numStreamTypes = AudioSystem.getNumStreamTypes();
        VolumeStreamState[] streams = mStreamStates = new VolumeStreamState[numStreamTypes];
```


VolumeStreamState.readSettings
```
                for (int i = 0; remainingDevices != 0; i++) {
                    int device = (1 << i);
                    if ((device & remainingDevices) == 0) {
                        continue;
                    }
                    remainingDevices &= ~device;

                    // retrieve current volume for device
                    // if no volume stored for current stream and device, use default volume if default
                    // device, continue otherwise
                    int defaultIndex = (device == AudioSystem.DEVICE_OUT_DEFAULT) ?
                            AudioSystem.DEFAULT_STREAM_VOLUME[mStreamType] : -1;
                    int index;
                    if (!hasValidSettingsName()) {
                        index = defaultIndex;
                    } else {
                        String name = getSettingNameForDevice(device);
                        index = Settings.System.getIntForUser(
                                mContentResolver, name, defaultIndex, UserHandle.USER_CURRENT);
                    }
                    if (index == -1) {
                        continue;
                    }

                    mIndexMap.put(device, getValidIndex(10 * index));
                }
```
最大最小音量：

```sh
   /** Maximum volume index values for audio streams */
    protected static int[] MAX_STREAM_VOLUME = new int[] {
        5,  // STREAM_VOICE_CALL
        7,  // STREAM_SYSTEM
        7,  // STREAM_RING
        15, // STREAM_MUSIC
        7,  // STREAM_ALARM
        7,  // STREAM_NOTIFICATION
        15, // STREAM_BLUETOOTH_SCO
        7,  // STREAM_SYSTEM_ENFORCED
        15, // STREAM_DTMF
        15, // STREAM_TTS
        15  // STREAM_ACCESSIBILITY
    };
    
    /** Minimum volume index values for audio streams */
    protected static int[] MIN_STREAM_VOLUME = new int[] {
        1,  // STREAM_VOICE_CALL
        0,  // STREAM_SYSTEM
        0,  // STREAM_RING
        0,  // STREAM_MUSIC
        1,  // STREAM_ALARM
        0,  // STREAM_NOTIFICATION
        0,  // STREAM_BLUETOOTH_SCO
        0,  // STREAM_SYSTEM_ENFORCED
        0,  // STREAM_DTMF
        0,  // STREAM_TTS
        1   // STREAM_ACCESSIBILITY
    };
```


## 音量调节

![流程图](http://assets.processon.com/chart_image/5abcf222e4b018c271d42392.png)

音量的设置过程：
1，首先判断当前哪个音频流处于激活状态，如果没有则默认调节指定的默认音频流（MUSIC）
2，获取到调节的音频流后，继续从音频流获取对应的音频设备
3，增减音量，并设置到当前的音量到播放线程（同步对齐其他音频流）
4，AudioFlinger根据类型，下发给AudioHal或者设置到播放线程上生效
5, 保存音量到Settings数据库中


## 音量曲线
不同的场景，不同的设备，音量的调节幅度，上下限都是不一样，可以参考audio_policy_volumes.xml中的配置
实际也是为了满足相同音量等级下，不同场景的，不同设备的细微调整


```sh
    <volume stream="AUDIO_STREAM_MUSIC" deviceCategory="DEVICE_CATEGORY_HEADSET">
        <point>1,-6000</point>
        <point>13,-5500</point>
        <point>20,-4200</point>
        <point>27,-3700</point>
        <point>33,-3300</point>
        <point>40,-2900</point>
        <point>47,-2500</point>
        <point>53,-2050</point>
        <point>60,-1700</point>
        <point>66,-1400</point>
        <point>73,-1100</point>
        <point>80,-800</point>
        <point>87,-500</point>
        <point>93,-250</point>
        <point>100,0</point>
    </volume>
    
    <volume stream="AUDIO_STREAM_MUSIC" deviceCategory="DEVICE_CATEGORY_SPEAKER">
        <point>1,-5500</point>
        <point>13,-4400</point>
        <point>20,-3800</point>
        <point>27,-3000</point>
        <point>33,-2200</point>
        <point>40,-2000</point>
        <point>47,-1800</point>
        <point>53,-1400</point>
        <point>60,-1200</point>
        <point>66,-1000</point>
        <point>73,-800</point>
        <point>80,-600</point>
        <point>87,-400</point>
        <point>93,-200</point>
        <point>100,0</point>
    </volume>
```


# AudioTrack创建流程
![类图](http://dtimage.oss-cn-shanghai.aliyuncs.com/2017/0627/27182315gmwx.jpg)
图解：可以从图上看到，AudioFlinger这个类涵盖了很多成员类
AudioTrack 中实际持有的是TrackHandler，他实现了IAudioTrack的接口，用于控制对应在AudioFlinger里创建的Track，比如start stop pause这些控制操作
AudioRecord 也是类似的，持有RecordHandle，实现了IAudioRecord接口，控制录音的start， stop等操作

Track：在AudioFlinger上的播放/录音 客户端，于AudioTrack，AudioRecord 是一一对应的关系
ThreadBase 是继承Thread的一个在AudioFlinger中最基础的单元线程
PlaybackThread 继承ThreadBase，属于播放线程
RecordThread 继承ThreadBase，属于录音线程
AudioFlinger主要是如下几个播放线程以及录音线程组成
1，MixerThread ：混音线程，该线程上可以把多个Track
2，DirectOutputThread：直输线程，该线程上一般就一个Track，所以不能混音
3，OffloadThread：继承DirectOutputThread，一般用于播放一些非PCM格式的数据
4，MmapPlaybackThread：用于AAudio创建的一种新播放线程（图片比较早，还没有列出来）
5，RecordThread：录音线程
6，MmapCaptureThread：用于AAudio创建的一种新录音线程

AudioEffect：音效处理


![AudioTrack创建流程图](http://assets.processon.com/chart_image/5c076f96e4b0eb0da01fe56c.png)

从内存分配方式分为以下两种，静态和动态
MODE_STATIC	应用进程将回放数据一次性付给 AudioTrack，适用于数据量小、时延要求高的场景
MODE_STREAM	用进程需要持续调用 write() 写数据到 FIFO，写数据时有可能遭遇阻塞（等待 AudioFlinger::PlaybackThread 消费之前的数据），基本适用所有的音频场景

MODE_STREAM模式下，从写入数据的方式又可以细分如下：
TRANSFER_SYNC  对应走AudioTrack.write
TRANSFER_CALLBACK 对应走cbf通知应用写入

创建比较重要的接口：
getMinBufferSize() 接口，字面意思是返回最小数据缓冲区的大小，它是声音能正常播放的最低保障，从函数参数来看，返回值取决于采样率、采样深度、声道数这三个属性。MODE_STREAM 模式下，应用程序重点参考其返回值然后确定分配多大的数据缓冲区。如果数据缓冲区分配得过小，那么播放声音会频繁遭遇 underrun，underrun 是指生产者（AudioTrack）提供数据的速度跟不上消费者（AudioFlinger::PlaybackThread）消耗数据的速度，反映到现实的后果就是声音断续卡顿，严重影响听觉体验。

```sh
/* static */ size_t AudioSystem::calculateMinFrameCount(
        uint32_t afLatencyMs, uint32_t afFrameCount, uint32_t afSampleRate,
        uint32_t sampleRate, float speed /*, uint32_t notificationsPerBufferReq*/)
{
    // Ensure that buffer depth covers at least audio hardware latency
    uint32_t minBufCount = afLatencyMs / ((1000 * afFrameCount) / afSampleRate);
    if (minBufCount < 2) {
        minBufCount = 2;
    }

    ALOGV("calculateMinFrameCount afLatency %u  afFrameCount %u  afSampleRate %u  "
            "sampleRate %u  speed %f  minBufCount: %u" /*"  notificationsPerBufferReq %u"*/,
            afLatencyMs, afFrameCount, afSampleRate, sampleRate, speed, minBufCount
            /*, notificationsPerBufferReq*/);
    return minBufCount * sourceFramesNeededWithTimestretch(
            sampleRate, afFrameCount, afSampleRate, speed);
}
```

举例：采样率44.1kHz 采样深度是16bit， 声道数是立体声
一般来说选到的afSampleRate=48kHz，afFrameCount=1920， speed=1.000000
计算出来 2*（（1920*（44100/48000）+1 +1 ）* 1.000000 +1 +1）* 2（位宽）* 2（声道数）= 14144
+1 这种是为了调节四舍五入的问题



# AudioServer启动流程
从Android7.0 开始，Audio服务就从MediaServer中独立出来AudioServer，单独处理音频相关事务

## main_audioserver.cpp
```sh
        ···
        AudioFlinger::instantiate();
        AudioPolicyService::instantiate();
        ···
```

## AudioPolicyService.cpp

    ···
    void AudioPolicyService::onFirstRef()
    {
        {
            Mutex::Autolock _l(mLock);
    
            // start audio commands thread
            mAudioCommandThread = new AudioCommandThread(String8("ApmAudio"), this);
            // start output activity command thread
            mOutputCommandThread = new AudioCommandThread(String8("ApmOutput"), this);
    
            mAudioPolicyClient = new AudioPolicyClient(this);
            mAudioPolicyManager = createAudioPolicyManager(mAudioPolicyClient);
        }
        // load audio processing modules
        sp<AudioPolicyEffects>audioPolicyEffects = new AudioPolicyEffects();
        {
            Mutex::Autolock _l(mLock);
            mAudioPolicyEffects = audioPolicyEffects;
        }
    
        mUidPolicy = new UidPolicy(this);
        mUidPolicy->registerSelf();
    
        mSensorPrivacyPolicy = new SensorPrivacyPolicy(this);
        mSensorPrivacyPolicy->registerSelf();
    }
    ···

## AudioPolicyManager.cpp
```sh
    ···
    loadConfig();//初始化配置 =》 deserializeAudioPolicyXmlConfig（） 加载"audio_policy_configuration.xml" 策略文件
    initialize();
    ···
```

## AudioPolicyManager::initialize()
```sh
    ···
    // Retrieve the Policy Manager Interface 策略执行逻辑
    mEngine = engineInstance->queryInterface<AudioPolicyManagerInterface>();
    if (mEngine == NULL) {
        ALOGE("%s: Failed to get Policy Engine Interface", __FUNCTION__);
        return NO_INIT;
    }
    mEngine->setObserver(this);

    //加载硬件设备模块，并打开设备
    for (const auto& hwModule : mHwModulesAll) {
        ···
        sp<SwAudioOutputDescriptor> outputDesc = new SwAudioOutputDescriptor(outProfile,
                                                                             mpClientInterface);
        audio_io_handle_t output = AUDIO_IO_HANDLE_NONE;
        status_t status = outputDesc->open(nullptr, DeviceVector(supportedDevice),
                                           AUDIO_STREAM_DEFAULT,
                                           AUDIO_OUTPUT_FLAG_NONE, &output);
        ...
        addOutput(output, outputDesc); //add to mOutputs
        ...
        sp<AudioInputDescriptor> inputDesc =
                new AudioInputDescriptor(inProfile, mpClientInterface);

        audio_io_handle_t input = AUDIO_IO_HANDLE_NONE;
        status_t status = inputDesc->open(nullptr,
                                          availProfileDevices.itemAt(0),
                                          AUDIO_SOURCE_MIC,
                                          AUDIO_INPUT_FLAG_NONE,
                                          &input);
        ...
    }
```

## AudioFlinger.openOutput

    #define AUDIO_HARDWARE_MODULE_ID_PRIMARY "primary"
    #define AUDIO_HARDWARE_MODULE_ID_A2DP "a2dp"
    #define AUDIO_HARDWARE_MODULE_ID_USB "usb"
        static const char * const audio_interfaces[] = {
        AUDIO_HARDWARE_MODULE_ID_PRIMARY,
        AUDIO_HARDWARE_MODULE_ID_A2DP,
        AUDIO_HARDWARE_MODULE_ID_USB,
    };

```sh
    ...
    AudioStreamOut *outputStream = NULL;
    //最终会走到对应的audio hal去打开具体的设备以及stream
    status_t status = outHwDev->openOutputStream(
            &outputStream,
            *output,
            devices,
            flags,
            config,
            address.string());
    ...
```


# 拓展学习
 - 设备切换时，路由策略选择
 - AudioEffect的流程学习
 - AudioHal中的设备切换，路由切换等
 - soundtrrigger 与audioserver的关系
