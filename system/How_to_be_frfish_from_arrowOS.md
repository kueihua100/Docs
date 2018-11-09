## How to be frfish from arrowOS?
### copy pixel Launcher3_pix

    a) cp -rf /home/kueihua/tmp/red_me_note_3/PixelExperience_O/packages/apps/Launcher3 ./
    b) Modify Android.mk of Launcher3
      b-1) Due to below 2 code:
         ./vendor/arrow/config/packages.mk:23:    Launcher3QuickStep \  <= = >  Launcher3
         ./Launcher3_arrow/Android.mk:    LOCAL_PACKAGE_NAME := Launcher3QuickStep
      b-2) Fix compile error:    add LOCAL_SDK_VERSION := 27 in Android.mk of Launcher3
      b-3) [note] Launcher3 will output at: ArrowOS_9/out/target/product/kenzo/system/priv-app/Launcher3

### Change bootanimation:
    a) SRC: PixelExperience_O/vendor/pixelstyle/media/bootanimation_1080.zip
    b) DST: ArrowOS_9/vendor/arrow/prebuilt/common/bootanimation.zip

### Remove ArrowOS build logo at the end: 
    a) ArrowOS_9/build/make/core/Makefile
    b) Search "Package complete" 
    
### Code modification for "packages/apps/Settings"
    a) modified: core/java/com/android/internal/widget/LockPatternUtils.java
    b) modified: core/java/com/android/internal/widget/LockPatternView.java
    c) modified: packages/apps/Settings/src/com/android/settings/applications/appops/AppOpsState.java

### make image script:
    a) build/make/tools/releasetools/*
