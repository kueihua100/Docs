# Audio Volume Initial:
    a. At boot up stage, AudioService will call readSettings(), applyAllVolumes()
    b. Adjust volume will call persistVolume()

### Audio init stream volume flow:
    AudioService.java::
     /** Maximum volume index values for audio streams */
      protected static int[] MAX_STREAM_VOLUME = new int[] {
          5,  // STREAM_VOICE_CALL
          7,  // STREAM_SYSTEM
          7,  // STREAM_RING
          15, // STREAM_MUSIC
          7,  // STREAM_ALARM
          7,  // STREAM_NOTIFICATION
          15, // STREAM_BLUETOOTH_SCO
          7,  // STREAM_SYSTEM_ENFORCED
          15, // STREAM_DTMF
          15, // STREAM_TTS
          15  // STREAM_ACCESSIBILITY
      };

      /** Minimum volume index values for audio streams */
      protected static int[] MIN_STREAM_VOLUME = new int[] {
          1,  // STREAM_VOICE_CALL
          0,  // STREAM_SYSTEM
          0,  // STREAM_RING
          0,  // STREAM_MUSIC
          1,  // STREAM_ALARM
          0,  // STREAM_NOTIFICATION
          0,  // STREAM_BLUETOOTH_SCO
          0,  // STREAM_SYSTEM_ENFORCED
          0,  // STREAM_DTMF
          0,  // STREAM_TTS
          1   // STREAM_ACCESSIBILITY
      };

    AudioService.java::AudioService()
        ...
        createStreamStates()
    -> AudioService.java::AudioService()::createStreamStates()
        for () {
        new VolumeStreamState(name, type);
        }
        checkAllFixedVolumeDevices();
        checkAllAliasStreamVolumes();
        checkMuteAffectedStreams();
        updateDefaultVolumes();
    -> AudioService.java:: VolumeStreamState::VolumeStreamState()
        AudioSystem.initStreamVolume();
        readSettings();

        
      
