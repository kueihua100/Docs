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
    
     ////////////////////////////////////////////////////////////////////////////////////
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

    [note 1]
    AudioService.java:: VolumeStreamState::VolumeStreamState()
        AudioSystem.initStreamVolume();
    -> android_media_AudioSystem.cpp:: android_media_AudioSystem_initStreamVolume()
    -> AudioSystem.cpp:: AudioSystem::initStreamVolume()
    -> IAudioPolicyService.cpp:: initStreamVolume()
        remote()->transact(INIT_STREAM_VOLUME, data, &reply);
    -> xxx.cpp:: onTransact()
        case INIT_STREAM_VOLUME:
            reply->writeInt32(static_cast <uint32_t>(initStreamVolume(stream, indexMin,indexMax)));
    -> AudioPolicyInterfaceImpl.cpp:: AudioPolicyService::initStreamVolume()
        mAudioPolicyManager->initStreamVolume()
    -> AudioPolicyManager.cpp:: AudioPolicyManager::initStreamVolume()
    -> VolumeCurve.h:: VolumeCurvesCollection:: initStreamVolume()
        editValueAt(stream).setVolumeIndexMin(indexMin);
        editValueAt(stream).setVolumeIndexMax(indexMax);

### Audio volume curve initial flow:
    AudioPolicyManager.cpp:: AudioPolicyManager::AudioPolicyManager()
        ...
        loadConfig();
        initialize();

    [note 1]
    AudioPolicyManager.cpp:: AudioPolicyManager::loadConfig()
    -> xxx.cpp:: deserializeAudioPolicyXmlConfig()
        serializer.deserialize()
    -> Serializer.cpp:: PolicySerializer::deserialize()
        // read volume data from audio_policy_volumes.xml and default_volume_tables.xml
        VolumeTraits::Collection volumes;
        deserializeCollection<VolumeTraits>(doc, cur, volumes, &config);
    -> xxx.cpp:: VolumeTraits::deserialize()
        element->add(CurvePoint(point[0], point[1]));
        // element is VolumeCurvesCollection
    -> VolumeCurve.h:: VolumeCurvesCollection::add(const sp<VolumeCurve> &volumeCurve)
        return editCurvesFor(streamType).add(volumeCurve);
    -> xxx.h:: VolumeCurvesForStream:: add(const sp<VolumeCurve> &volumeCurve)
    -> Serializer.cpp:: PolicySerializer::deserialize()
        config.setVolumes(volumes);
    -> AudioPolicyConfig.h:: AudioPolicyConfig:: setVolumes()
        *mVolumeCurves = volumes;

    [note 2]
    AudioPolicyManager.cpp:: AudioPolicyManager::initialize()
        mVolumeCurves->initializeVolumeCurves()
        //** IVolumeCurvesCollection.h:: virtual void initializeVolumeCurves() {} is empty function, so do nothing
        //** mVolumeCurves was initialed at [Note 1] step.
