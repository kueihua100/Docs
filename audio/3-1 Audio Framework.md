Audio Framework
--------------------------------
![3-1-1](/audio/res/3-1-1.png?raw=true "3-1-1")

* Application framework

  The application framework includes the app code, which uses the android.media APIs to interact with audio hardware. Internally, this code calls corresponding JNI glue classes to access the native code that interacts with audio hardware.

* JNI

  The JNI code associated with android.media calls lower level native code to access audio hardware. JNI is located in frameworks/base/core/jni/ and frameworks/base/media/jni.

* Native framework

  The native framework provides a native equivalent to the android.media package, calling Binder IPC proxies to access the audio-specific services of the media server. Native framework code is located in frameworks/av/media/libmedia.

* Binder IPC

  Binder IPC proxies facilitate communication over process boundaries. Proxies are located in frameworks/av/media/libmedia and begin with the letter "I".

* Media server

  The media server contains audio services, which are the actual code that interacts with your HAL implementations. The media server is located in frameworks/av/services/audioflinger.

* HAL

  The HAL defines the standard interface that audio services call into and that you must implement for your audio hardware to function correctly. The audio HAL interfaces are located in hardware/libhardware/include/hardware. For details, see audio.h.

* Kernel driver

  The audio driver interacts with your hardware and HAL implementation. You can use Advanced Linux Sound Architecture (ALSA), Open Sound System (OSS), or a custom driver (HAL is driver-agnostic).

![3-1-2](/audio/res/3-1-2.png?raw=true "3-1-2")

* Audio Application Framework：音频应用框架

    AudioTrack：负责回放数据的输出，属 Android 应用框架 API 类
    
    AudioRecord：负责录音数据的采集，属 Android 应用框架 API 类
    
    AudioSystem： 负责音频事务的综合管理，属 Android 应用框架 API 类

* Audio (Native) Framework：音频本地框架 

    AudioTrack：负责回放数据的输出，属 Android 本地框架 API 类
    
    AudioRecord：负责录音数据的采集，属 Android 本地框架 API 类

    AudioSystem： 负责音频事务的综合管理，属 Android 本地框架 API 类

* Audio Services：音频服务 

    AudioPolicyService：音频策略的制定者，负责音频设备切换的策略抉择、音量调节策略等

    AudioFlinger：音频策略的执行者，负责输入输出流设备的管理及音频流数据的处理传输

* Audio HAL：音频硬件抽象层，负责与音频硬件设备的交互，由 AudioFlinger 直接调用

![3-1-3](/audio/res/3-1-3.png?raw=true "3-1-3")
![3-1-4](/audio/res/3-1-4.png?raw=true "3-1-4")