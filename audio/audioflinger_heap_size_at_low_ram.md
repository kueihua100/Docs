## AudioFlinger heap size at low ram

### **[note]**

    Enbale low ram device is added below property: 
    PRODUCT_PROPERTY_OVERRIDES += ro.config.low_ram=true
    
[Low RAM google doc, please refer to here](https://source.android.com/devices/tech/perf/low-ram)
<!-- comment messages -->

### **Issue:**
At low RAM environment, Enter Youtube -> playback with speaker outpur -> Headphone plugged-in, 
will have below logcat log:

    E AudioFlinger: not enough memory for AudioTrack size=524512 
    D MemoryDealer:   AudioTrack (0xb02884e0, size=1048576)
    D MemoryDealer:     0: 0xb0288560 | 0x00000000 | 0x000800E0 | A 
    D MemoryDealer:     1: 0xb02885a0 | 0x000800E0 | 0x0007FF20 | F 
    D MemoryDealer:   size allocated: 524512 (512 KB)
    E AudioFlinger: createTrack_l() initCheck failed -12; no control block?
    E IAudioFlinger: createTrack returned error -12
    E AudioTrack: AudioFlinger could not create track, status: -12 output 0
    W AudioTrack: restoreTrack_l(): createTrack_l failed, do not retry
    W AudioTrack: restoreTrack_l() failed status -12, retries 0
    E AudioTrack-JNI: Error -12 during AudioTrack native read

### Reason 1
As Headphone plugged-in, APP willl call create audio track API, but share buffer has not enough buffer (needs 512KB), 
and under low_ram configuration, audioflinger default only setup total share heap buffer = 1MB.

### Reason 2: AudioService during Construction, will call AudioService.java::readAndSetLowRamDevice() to setup mClientSharedHeapSize

    JAVA Call Stack:
    StackTrace: dalvik.system.VMStack.getThreadStackTrace(Native Method) 
    StackTrace: java.lang.Thread.getStackTrace(Thread.java:1538)
    StackTrace: com.android.server.audio.AudioService.readAndSetLowRamDevice(AudioService.java:7310)
    StackTrace: com.android.server.audio.AudioService.<init>(AudioService.java:836)
    StackTrace: com.android.server.audio.AudioService$Lifecycle.<init>(AudioService.java:664)
    StackTrace: java.lang.reflect.Constructor.newInstance0(Native Method)
    StackTrace: java.lang.reflect.Constructor.newInstance(Constructor.java:343)
    StackTrace: com.android.server.SystemServiceManager.startService(SystemServiceManager.java:98)
    StackTrace: com.android.server.SystemServer.startOtherServices(SystemServer.java:1257)
    StackTrace: com.android.server.SystemServer.run(SystemServer.java:431)
    StackTrace: com.android.server.SystemServer.main(SystemServer.java:294)
    StackTrace: java.lang.reflect.Method.invoke(Native Method)
    StackTrace: com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:493)
    StackTrace: com.android.internal.os.ZygoteInit.main(ZygoteInit.java:838)

    At AudioService.java
    private static void readAndSetLowRamDevice()
    {
        ...
        final int status = AudioSystem.setLowRamDevice(isLowRamDevice, totalMemory);
        ...
    }
    
    At AudioFlinger.cpp
    status_t AudioFlinger::setLowRamDevice(bool isLowRamDevice, int64_t totalMemory)
    {
        ...
        mClientSharedHeapSize =
            isLowRamDevice ? kMinimumClientSharedHeapSizeBytes
                    : mTotalMemory < 2 * GB ? 4 * kMinimumClientSharedHeapSizeBytes
                    : mTotalMemory < 3 * GB ? 8 * kMinimumClientSharedHeapSizeBytes
                    : mTotalMemory < 4 * GB ? 16 * kMinimumClientSharedHeapSizeBytes
                    : 32 * kMinimumClientSharedHeapSizeBytes;
        ...
    }
    [note 1] mClientSharedHeapSize is the buffer pool used to for audio track.
    [note 2] If is low_ram, mClientSharedHeapSize = 1MB.
    [note 3] If is not low_ram, mClientSharedHeapSize = according to RAM size, could be 1MB x 4,  x 8,  x16, x 32 
    
