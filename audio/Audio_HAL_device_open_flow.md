# Audio HAL device open flow:
### Open HAL device:
    AudioPolicyManager.cpp:: AudioPolicyManager::AudioPolicyManager() {
        ...
        loadConfig()
        //[note] call deserializeAudioPolicyXmlConfig() to load config file audio_policy_configuration.xml
      
        initialize()
    }
    -> ***.cpp:: AudioPolicyManager::initialize() {
        ...
        for (const auto& hwModule : mHwModulesAll) {
          mpClientInterface->loadHwModule(hwModule->getName());
        }
    }
    -> AudioPolicyClientImpl.cpp:: AudioPolicyService:: AudioPolicyClient::loadHwModule()
    -> AudioFlinger.cpp::AudioFlinger::loadHwModule()
    -> ***.cpp:: AudioFlinger::loadHwModule_l()
        mDevicesFactoryHal->openDevice()
    -> DevicesFactoryHalHybrid.cpp:: DevicesFactoryHalHybrid::openDevice()
    -> DevicesFactoryHalHidl.cpp:: DevicesFactoryHalHidl::openDevice()
    -> DevicesFactoryAll.cpp:: BpHwDevicesFactory::openDevice()
      //[note-TBC]
        xxx/out/soong/.intermediates/hardware/interfaces/audio/4.0/android.hardware.audio@4.0_genc++/gen/android/hardware/audio/4.0
        
    ===> Through Binder IPC <===
    
    -> DevicesFactory.cpp
    -> DevicesFactory.impl.h:: DevicesFactory::openDevice()
    -> ***.h:: DevicesFactory::loadAudioInterface() {
        ...
        hw_get_module_by_class();
        audio_hw_device_open();
        //check HW version
        if ((*dev)->common.version < AUDIO_DEVICE_API_VERSION_MIN) {
          audio_hw_device_close();
        }
    }
