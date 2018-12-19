# How to enable ASAN for audio HAL code?

#### [Note]  
    Enable ASAN need to add ASAN compile flags to both binary and library files.

#### ASAN compile flags:
    [note]: modify below ":=" or "+=" part accoording to your code.
    
    LOCAL_CLANG := true
    LOCAL_CFLAGS +=  -Wall -O0 -fsanitize=address -fno-omit-frame-pointer
    LOCAL_SANITIZE := address #alignment bounds null unreachable integer
    LOCAL_SANITIZE_DIAG := alignment bounds null unreachable integer
    LOCAL_SHARED_LIBRARIES += libclang_rt.asan-arm-android

#### audio service binary  
    a) name: android.hardware.audio@2.0-service
    b) Makefile path: 
       pie/hardware/interfaces/audio/common/all-versions/default/service/Android.mk

#### audio HAL library
    Due to different vendors has their own path, below is using qcom as an example:
    a) name: see "LOCAL_MODULE" from pie/hardware/qcom/audio/xxx/Android.mk
    b) Makefile path: pie/hardware/qcom/audio/xxx/Android.mk
