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
    
    -> hardware.c:: hw_get_module_by_class(class_id, inst, xx)
        //[note 1] class_id=audio
        //[note 2] inst= primary, r_submix, usb, a2dp.
        The module name of [primary, submix, usb, a2dp] are from audio_policy_configuration.xml's "module" field.
        //[note 3] How to search audio HAL libraries:
            a) lib_name = class_id.inst = audio.promary, audio.r_submix, audio.usb, audio.a2dp
            b) Above audio module’s default libraries are: 
                audio.primary.default.so, 
                audio.r_submix.default.so, 
                audio.usb.default.so, 
                audio.a2dp.default.so
            c) 根據 property key, 會去尋找是否有 device/platform 相應的libraries, 不然就會load default libraries. 
            例如 audio.r_submix.default.so and audio.usb.default.so. 
