# How to add "Screenshot" item into the popup menu from long press Power key:
    [note 1] Code base is android Pie
    [note 2] Screenshot can be done by pressing both keys: "Volume down" + "Power", 
             but here is to add a Screenshot item if you long press Power key.

### Add config_globalActionsList into config.xml
    (a) First check frameworks/base/core/res/res/values/config.xml, shoudl has below code:
    <string-array translatable="false" name="config_globalActionsList">
        <item>power</item>
        <item>restart</item>
        <item>lockdown</item>
        <item>logout</item>
        <item>bugreport</item>
        <item>screenshot</item>
    </string-array>
    
    (b) Check your overlay code should not override "config_globalActionsList"
    
### Add code for screenshot
    At frameworks/base/services/core/java/com/android/server/policy/LegacyGlobalActions.java
    (a) Search "GLOBAL_ACTION_KEY_RESTART" and add below code:
        private static final String GLOBAL_ACTION_KEY_SCREENSHOT = "screenshot";
        
    (b) Search "GLOBAL_ACTION_KEY_RESTART" and add below code that will use (a) string:
        } else if (GLOBAL_ACTION_KEY_SCREENSHOT.equals(actionKey)) {
            mItems.add(new ScreenshotAction(mContext, mWindowManagerFuncs));
        }
        
    (c) Add "ScreenshotAction" class that will new at (b):
        private class ScreenshotAction extends SinglePressAction implements LongPressAction {
            private final Context mContext;
            private final WindowManagerPolicy.WindowManagerFuncs mWindowManagerFuncs;

            public ScreenshotAction(Context context,
                WindowManagerPolicy.WindowManagerFuncs windowManagerFuncs) {
                super(com.android.internal.R.drawable.ic_screenshot, R.string.global_action_screenshot);
                /***
                ** [note] com.android.internal.R.drawable.ic_screenshot means to use:
                **        frameworks/base/core/res/res/drawable/ic_screenshot.xml 
                ***/
                
                mContext = context;
                mWindowManagerFuncs = windowManagerFuncs;
            }
            @Override
            public boolean onLongPress() {
               Log.e(TAG, "onLongPress()");
                return true;
            }
            @Override
            public void onPress() {
               Log.e(TAG, "onPress()");
            }
            @Override
            public boolean showDuringKeyguard() {
                return true;
            }
            @Override
            public boolean showBeforeProvisioning() {
                return true;
            }
        }
        
    (d) Check "global_action_screenshot" that used at (c):
        from 
            frameworks/base/core/res/res/values/symbols.xml
        or
            your overlay path
        
        should have below code:
            ...
            <java-symbol type="string" name="global_action_power_off" />
            ...
            <java-symbol type="string" name="global_action_screenshot" />
        
### Code flow:
