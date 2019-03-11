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
    
    //set max volume index of STREAM_MUSIC
    AudioService.java::AudioService()
        ...
        int maxMusicVolume = SystemProperties.getInt("ro.config.media_vol_steps", -1);
        if (maxMusicVolume != -1) {
            MAX_STREAM_VOLUME[AudioSystem.STREAM_MUSIC] = maxMusicVolume;
        }
        
    //set default volume index of STREAM_MUSIC if no previous saved index (eg. first installation)
    AudioService.java::AudioService()
        ...
        int defaultMusicVolume = SystemProperties.getInt("ro.config.media_vol_default", -1);
        if (defaultMusicVolume != -1 &&
                defaultMusicVolume <= MAX_STREAM_VOLUME[AudioSystem.STREAM_MUSIC]) {
            AudioSystem.DEFAULT_STREAM_VOLUME[AudioSystem.STREAM_MUSIC] = defaultMusicVolume;
        } else {
            if (isPlatformTelevision()) {
                AudioSystem.DEFAULT_STREAM_VOLUME[AudioSystem.STREAM_MUSIC] =
                        MAX_STREAM_VOLUME[AudioSystem.STREAM_MUSIC] / 4;
            } else {
                AudioSystem.DEFAULT_STREAM_VOLUME[AudioSystem.STREAM_MUSIC] =
                        MAX_STREAM_VOLUME[AudioSystem.STREAM_MUSIC] / 3;                   
            }
        }
    //set max/default volume index of STREAM_ALARM/STREAM_SYSTEM at AudioService.java::AudioService().
    
    +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
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

    [note 3] default_volume_tables.xml:
        <reference name="DEFAULT_MEDIA_VOLUME_CURVE">
        <!-- Default Media reference Volume Curve -->
            <point>1,-5800</point>
            <point>20,-4000</point>
            <point>60,-1700</point>
            <point>100,0</point>
        </reference>
        
        ** you can change the volume curve by your own, for example above can be changed into below: **
        ** and -5800, -4000, ... are db values. **
        
        <reference name="DEFAULT_MEDIA_VOLUME_CURVE">
        <!-- Default Media reference Volume Curve -->
            <point>1,-7000</point>
            <point>25,-2400</point>
            <point>50,-1200</point>
            <point>75,-300</point>
            <point>100,0</point>
        </reference>

    [note 4] If you want to not modified the PCM data (always max volume index), 
             from default_volume_tables.xml, to use below curve:
        <reference name="FULL_SCALE_VOLUME_CURVE">
        <!-- Full Scale reference Volume Curve -->
            <point>0,0</point>
            <point>100,0</point>
        </reference>
            
