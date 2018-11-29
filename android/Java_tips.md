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
