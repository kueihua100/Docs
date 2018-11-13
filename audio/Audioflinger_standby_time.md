# Audioflinger standby time:

    AudioFlinger.cpp::AudioFlinger::onFirstRef()
    -> if (property_get("ro.audio.flinger_standbytime_ms", val_str, NULL) >= 0)
    -> mStandbyTimeInNsecs = milliseconds(int_val);
