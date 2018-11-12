# Headphone volume will be reset after AC off->on
### [Method 1] Add a property key at under audio makefile (xxx_audio.mk)
    PRODUCT_SYSTEM_DEFAULT_PROPERTIES += audio.safemedia.bypass=true
    [note 1] After Pie, audio service only can access system property key.
    
    [note 2]
    AudioService.java::onSystemReady()
    -> sendMsg(xx, SystemProperties.getBoolean("audio.safemedia.bypass", false) ?
                        0 : SAFE_VOLUME_CONFIGURE_TIMEOUT_MS);
    [note 3]
    AudioService.java::onConfigureSafeVolume()
    -> boolean safeMediaVolumeBypass =
               SystemProperties.getBoolean("audio.safemedia.bypass", false);
               
### [Method 2] 
