# Audio launcher touch sound (key tone)
1. Android 開到桌面 (launcher), 上下左有移動, 應該會有touch sound (key tone)
2. UI touch sounds are put under:   
    /xxx/pie/frameworks/base/data/sounds (Oreo and Pie both).          
    But at android Pie, launcher touch sound “Effect_Tick.ogg” was removed.         
    Needs to be added by vendor and put under /system/media/audio/ui at target board.   
    
### Touch sound initial flow:
    AudioService.java:: AudioSystemThread::run()
    -> Looper.java:: android.os.Looper.loop()
        //[note] ./frameworks/base/core/java/android/os/Looper.java
    -> Handler.java:: android.os.Handler.dispatchMessage()
        //[note] ./frameworks/base/core/java/android/os/Handler.java
    -> AudioService.java:: AudioHandler:: handleMessage()
        case MSG_SYSTEM_READY:
            onSystemReady()
    -> AudioService.java:: onSystemReady()
        sendMsg(mAudioHandler, MSG_LOAD_SOUND_EFFECTS, )
    -> AudioService.java:: AudioHandler:: handleMessage()
        case MSG_LOAD_SOUND_EFFECTS:
            boolean loaded = onLoadSoundEffects();
    -> AudioService.java:: AudioHandler:: onLoadSoundEffects()
        loadTouchSoundAssets();
        mSoundPool = new SoundPool.Builder().xx. .setUsage(AudioAttributes.USAGE_ASSISTANCE_SONIFICATION)
        //[note] new SoundPool player with (AudioAttributes.USAGE_ASSISTANCE_SONIFICATION)
        // AudioPolicyHelper.h:: audio_attributes_to_stream_type()
            case AUDIO_USAGE_ASSISTANCE_SONIFICATION:
                return AUDIO_STREAM_SYSTEM;
        //[note] from above, the sound effect’s stream type is AUDIO_STREAM_SYSTEM.
        //[note] from adjust volume at launcher, the sound effect’s volume is AUDIO_STREAM_MUSIC
        
        int sampleId = mSoundPool.load(filePath, 0);
        //[note] load sound effect files from: 
            /system/media/audio/ui/Effect_Tick.ogg, 
                KeypressStandard.ogg, KeypressSpacebar.ogg, 
                KeypressDelete.ogg, KeypressReturn.ogg, 
                KeypressInvalid.ogg

    [note 1]
    AudioService.java:: AudioHandler ::onLoadSoundEffects()
    -> loadTouchSoundAssets();
    -> ***.java:: loadTouchSoundAssets()
        loadTouchSoundAssetDefaults()
        // [note] initial SOUND_EFFECT_FILES_MAP[]
        parser = mContext.getResources().getXml(com.android.internal.R.xml.audio_assets);
        //[note] load xml file: ./frameworks/base/core/res/res/xml/audio_assets.xml and init SOUND_EFFECT_FILES[]
            <audio_assets version="1.0">
            <group name="touch_sounds">
                <asset id="FX_KEY_CLICK" file="Effect_Tick.ogg"/>
                <asset id="FX_FOCUS_NAVIGATION_UP" file="Effect_Tick.ogg"/>
                <asset id="FX_FOCUS_NAVIGATION_DOWN" file="Effect_Tick.ogg"/>
                <asset id="FX_FOCUS_NAVIGATION_LEFT" file="Effect_Tick.ogg"/>
                <asset id="FX_FOCUS_NAVIGATION_RIGHT" file="Effect_Tick.ogg"/>
                <asset id="FX_KEYPRESS_STANDARD" file="KeypressStandard.ogg"/>
                <asset id="FX_KEYPRESS_SPACEBAR" file="KeypressSpacebar.ogg"/>
                <asset id="FX_KEYPRESS_DELETE" file="KeypressDelete.ogg"/>
                <asset id="FX_KEYPRESS_RETURN" file="KeypressReturn.ogg"/>
                <asset id="FX_KEYPRESS_INVALID" file="KeypressInvalid.ogg"/>
            </group>
            </audio_assets>

### Touch sound playback flow (when move the focus at the launcher):
    AudioService.java:: playSoundEffect(effectType)
    //[note] effectType = audio_assets.xml 裡 id value:
    // FX_KEY_CLICK, FX_FOCUS_NAVIGATION_UP, FX_FOCUS_NAVIGATION_DOWN …
    -> ***.java:: playSoundEffectVolume()
        sendMsg(mAudioHandler, MSG_PLAY_SOUND_EFFECT,)
    -> AudioService.java:: AudioHandler:: handleMessage()
        case MSG_PLAY_SOUND_EFFECT:
            onPlaySoundEffect()
    -> ***.java:: AudioHandler:: onPlaySoundEffect()
        onLoadSoundEffects();
        //[note] Check mSoundPool is created or not
        mSoundPool.play();

    [note 1]
    AudioService.java:: AudioHandler:: onPlaySoundEffect()
        mSoundPool.play();
    -> SoundPool.cpp:: SoundPool::play()
    -> ***.cpp:: SoundChannel::play()
        newTrack = new AudioTrack();
        oldTrack = mAudioTrack;
        mAudioTrack = newTrack;
        newTrack->setVolume();
        mAudioTrack->start();

### The touch sound at Headphone is more quite than it at Speaker. How to modify it?
    Modify from audio_policy_volumes.xml:
    [note 1] the default xxx_configuration.xml files are under: xxx/frameworks/av/services/audiopolicy/config/
    
    Original:
    <volume stream="AUDIO_STREAM_SYSTEM" deviceCategory="DEVICE_CATEGORY_HEADSET">
        <point>1,-3000</point>
        <point>33,-2600</point>
        <point>66,-2200</point>
        <point>100,-1800</point>
    </volume>
    
    changed to:
    <volume stream="AUDIO_STREAM_SYSTEM" deviceCategory="DEVICE_CATEGORY_HEADSET">
        <point>1,-2400</point>
        <point>33,-1600</point>
        <point>66,-800</point>
        <point>100,0</point>
    </volume>

