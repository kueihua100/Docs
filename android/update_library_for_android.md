# How to update library for android?

## [Case 1] Vendor C/C++ code:
    Use android default usbaudio as an example:
    1. Code path: pie/hardware/libhardware/modules/usbaudio    
    2. From makefile: Android.bp
      cc_library_shared {
          name: "audio.usb.default",
          relative_install_path: "hw",
          vendor: true,
          srcs: ["audio_hal.c"],
          shared_libs: [
              "liblog",
              "libcutils",
              "libtinyalsa",
              "libaudioutils",
              "libalsautils",
          ],
          cflags: ["-Wno-unused-parameter"],
          header_libs: ["libhardware_headers"],
      }
      
      [note] From "name" tag, the library is "audio.usb.default.so".
      
    3. Library output path: 
      out/xxx/vendor/lib/hw
      
    4. How to push to target?
      # adb connect your_ip_address:5555
      # adb root + ctrl C
      # adb connect your_ip_address:5555
      # adb remount
      # adb push audio.usb.default.so /vendor/lib/hw
      # adb shell sync
      Reboot your target system and to check it~
      
## Framework java code:
    Use frameworks\base\services\core\java\com\android\server\audio\AudioService.java as an example:
    After modifid, ...
    
    1. If you just mm under frameworks/base/services/core/ folder
      1-a) From frameworks/base/services/core/Android.bp:
        java_library_static {
            ...
        }
        ...
        java_library {
            name: "services.core",
            static_libs: ["services.core.priorityboosted"],
        }
    
      1-b) it generates:
        services.core.jar   <== under out/xxx/system/framework/oat/arm
        services.core.odex  <== under out/xxx/system/framework/oat/arm
        services.core.vdex  <== under out/xxx/system/framework/oat/arm
        But adb push these libraries, will not work.
      
    2. Should mm under frameworks/base/services/
      2-a) From frameworks/base/services/Android.bp
        java_library {
            name: "services",
            ...
        }
        
      2-b) it generates:
        services.core.jar   <== under out/xxx/system/framework
        services.jar        <== under out/xxx/system/framework
        services.jar.prof   <== under out/xxx/system/framework
        services.core.odex  <== under out/xxx/system/framework/oat/arm
        services.core.vdex  <== under out/xxx/system/framework/oat/arm
        services.art        <== under out/xxx/system/framework/oat/arm
        services.vdex       <== under out/xxx/system/framework/oat/arm
        services.odex       <== under out/xxx/system/framework/oat/arm
        
        # adb push services.core.jar services.jar services.jar.prof /system/framework
        # adb push services.core.odex services.core.vdex services.art services.vdex services.odex /system/framework/oat/arm
        
## Framework C/C++ code:
    Use frameworks/av/services/audioflinger as an example:
    1. From makefile Android.mk:
        ...
        LOCAL_MODULE:= libaudioflinger
        ...
    2. After moddifies the c/C++ code under audioflinger folder,
       Should mm under audioflinger folder, then the it generates
       libaudioflinger.so  <== under out/xxx/system/lib
       
       # adb push libaudioflinger.so /system/lib
