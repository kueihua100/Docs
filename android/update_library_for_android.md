# How to update library for android?

## For C/C++ code:
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
      xxx/vendor/lib/hw
      
    4. How to push to target?
      # adb connect your_ip_address:5555
      # adb root + ctrl C
      # adb connect your_ip_address:5555
      # adb remount
      # adb push audio.usb.default.so /vendor/lib/hw
      # adb shell sync
      Reboot your target system and to check it~
    
## For java code:
