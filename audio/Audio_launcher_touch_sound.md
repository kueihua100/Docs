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

