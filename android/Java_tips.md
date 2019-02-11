# Java tips:
## How to enable java code's Log.isLoggable?
    At frameworks\base\services\core\java\com\android\server\audio\AudioService.java:
    
    AudioService.java:: handleDeviceConnection() {
        if (DEBUG_DEVICES) {
            Slog.i(TAG, "handleDeviceConnection(" + connect + " dev:" + Integer.toHexString(device)
                    + " address:" + address + " name:" + deviceName + ")");
        }
        ...
    }
    
    [note 1] 
        private static final String TAG = "AudioService";
        protected static final boolean DEBUG_DEVICES = Log.isLoggable(TAG + ".DEVICES", Log.DEBUG);

    [note 2] 
        frameworks/base/core/java/android/util/Log.java
        /**
         * Checks to see whether or not a log for the specified tag is loggable at the specified level.
         *
         *  The default level of any tag is set to INFO. This means that any level above and including
         *  INFO will be logged. Before you make any calls to a logging method you should check to see
         *  if your tag should be logged. You can change the default level by setting a system property:
         *      'setprop log.tag.&lt;YOUR_LOG_TAG> &lt;LEVEL>'
         *  Where level is either VERBOSE, DEBUG, INFO, WARN, ERROR, ASSERT, or SUPPRESS. SUPPRESS will
         *  turn off all logging for your tag. You can also create a local.prop file that with the
         *  following in it:
         *      'log.tag.&lt;YOUR_LOG_TAG>=&lt;LEVEL>'
         *  and place that in /data/local.prop.
         *  ...
         */
        public static native boolean isLoggable(String tag, int level);

    [note 3]
        # setprop log.tag.AudioService.DEVICES D   <== turn on DEBUG_DEVICES
        # stop     <== stop android framework
        # star     <== start android framework

## How to add call stack at java code?
    Add below java code piece:
    private void printCallStack() 
    {
        StackTraceElement[] elements = Thread.currentThread().getStackTrace();
        for (StackTraceElement element : elements) {
            Log.d("Java_StackTrace", element.toString());
        }
    }

## How to de-compile APK?
Tools is under:
    https://drive.google.com/drive/u/0/folders/1D0JoMJnTshtjW2Mkhmh4JTsZ1BYHFpwu

## How to generate deodexed .jar file?
    java code -------------------> .smali ---------------------> .dex
    
    a) https://bitbucket.org/JesusFreke/smali/overview
    b) use baksmali.jar: disassembler (*.odex + *.oat + *.jar) into deodexed *.smali files
    c) use smali.jar:    assembler deodexed *.samli files into deodexed *.jar
      
#### [note]
  https://forum.xda-developers.com/android/software-hacking/tooll-03-12-fulmics-deodexer-1-0-t3512081
  https://www.reddit.com/r/PokemonGoSpoofing/comments/alchoq/mini_guide_android_root_smalli_patcher_deodexing/
  .dex/.oat: https://www.jianshu.com/p/389911e2cdfb
    
## DexPatcher
https://www.xda-developers.com/dexpatcher-patch-android-apks-using-java/  

    a) Making patch dex files more simpler and allowing developers to completely avoid dealing with Smali.
    b) Devs can write patches in Java alone and have DexPatcher handle everything else.
    c) The main advantage is having easily readable and manageable patch files. 
       And Patching APKs also becomes more convenient in general.
#### source: 
  [https://github.com/DexPatcher/dexpatcher-tool](https://github.com/DexPatcher/dexpatcher-tool)

## Example of disassembling (xx.jar or xx.dex) to xx.smali files
    a) decode xx.jar to xx.dex
      (1) use 7z tool to unzip xx.jar
      (2) or use command: jar -xf xx.jar
      
    b) decode xx.dec to xx.smali
      (1) download dexpatch scripts: https://github.com/DexPatcher/dexpatcher-tool-scripts
      (2) enter "dexpatcher-tool-scripts/bundled/dex2jar"
      (3) mv xx.dex files to "dexpatcher-tool-scripts/bundled/dex2jar"
      (4) use command: ./d2j-dex2smali.sh xx.dex  <-- will generate xx.samli files
