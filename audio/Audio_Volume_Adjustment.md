# Audio Volume Adjustment

1.  android java code's 音量設置, 首先調用到AudioManager.java
2.  b.	有兩種方法可以設置音量: setStreamVolume 和 adjustStreamVolume
3.  setStreamVolume：傳入index直接設置音量值.
4.  adjustStreamVolume：傳入direction，根據direction和獲取到的步長設置音量. setStreamVolume方法與adjustStreamVolume其實是殊途同歸

### AudioManager.java:: setStreamVolume()
    ->AudioService.java:: setStreamVolume()
    -> ***.java::onSetStreamVolume()  (Note: ***.java means file name is same as above)
    -> ***.java:: setStreamVolumeInt ()
    -> (1) streamState.setIndex 
          /* save volume index into VolumeStreamState*/
       (2) sendMsg(, MSG_SET_DEVICE_VOLUME,)
          /* post msg to handler to set volume */
    -> ***.java:: handleMessage()
        case MSG_SET_DEVICE_VOLUME:
          setDeviceVolume()
    -> ***.java:: setDeviceVolume()
    -> (1) streamState.applyDeviceVolume_syncVSS(device);
        /* apply volume */
       (2) sendMsg(, MSG_PERSIST_VOLUME,)
        /* post a persist msg to save current volume index */

### [note 1]
    -> AudioService.java:: sendMsg(, MSG_PERSIST_VOLUME,)
      -> ***.java:: handleMessage()
          case MSG_PERSIST_VOLUME:
            persistVolume()
      -> ***.java:: persistVolume()
      -> call System.putIntForUser() to save volume index to database (settings.db)
### [note 2]
    -> AudioService.java:: applyDeviceVolume_syncVSS() will call into JNI API setStreamVolumeIndex()
    -> android_media_AudioSystem.cpp:: android_media_AudioSystem_setStreamVolumeIndex()
    -> AudioSystem.cpp:: AudioSystem::setStreamVolumeIndex()
    -> const sp<IAudioPolicyService>& aps = AudioSystem::get_audio_policy_service();
        aps->setStreamVolumeIndex(stream, index, device);
    -> ***.cpp:: AudioSystem::get_audio_policy_service()
    -> gAudioPolicyService = interface_cast<IAudioPolicyService>(binder);
      interface_cast<IAudioPolicyService>(binder);
      from IInterface.h: 
        will return (new Bp##INTERFACE(obj);)
        will return BpAudioPolicyService class
      /* So from audio policy service對外export給client呼叫的Bpxxx interface */
      so aps->setStreamVolumeIndex(stream, index, device);
      equal to IAudioPolicyService.cpp::BpAudioPolicyService:: setStreamVolumeIndex()
      -> remote()->transact(SET_STREAM_VOLUME, data, &reply);
      remote()是通過繼承關係BpAudioPolicyService -> BpInterface -> BpRefBase,在類BpRefBase中定義的:
      inline  IBinder*  remote()  { return mRemote; }
      IBinder* const    mRemote; = = > mRemote也是const的，只賦值一次
  
  ![3-2-1](/audio/res/3-2-1.png)
  
    mRemote is initial from parameter o of BpRefBase’s constructor.
    mRemote’s 實體是IAudioPolicyService, 而IAudioPolicyService是由:
    In AudioSystem::get_audio_policy_service()
    -> sp<IServiceManager> sm = defaultServiceManager();
    -> IServiceManager.cpp ::ProcessState::self()->getContextObject(NULL));
    -> ProcessState.cpp:: ProcessState::getContextObject()
    -> ProcessState.cpp:: ProcessState::getStrongProxyForHandle()
    -> b = BpBinder::create(handle);
    From BpBinder.h:: class BpBinder : public IBinder
    
    So remote()->transact(SET_STREAM_VOLUME, data, &reply); will go 
    -> BpBinder.cpp:: BpBinder::transact()
    -> IPCThreadState::self()->transact()
    
    IPCThreadState就是真正交換通訊的地方了. Client side透過audio policy service (aps) export出來的interface向 aps傳送cmd, 而audio policy service則透過joinThreadPool() 處理client傳遞的 cmd:
    main_audioserver.cpp::main()
    -> main_audioserver.cpp ::IPCThreadState::self()->joinThreadPool();
      -> IPCThreadState.cpp:: IPCThreadState::getAndExecuteCommand()
      ->***.cpp: IPCThreadState::executeCommand
      -> case BR_TRANSACTION:
      -> error = reinterpret_cast<BBinder*>(tr.cookie)->transact()

    From AudioPolicyService.h’s aps declaration:
    
  ![3-2-2](/audio/res/3-2-2.png)
  
    public BnAudioPolicyService,
    -> class BnAudioPolicyService : public BnInterface<IAudioPolicyService>
    -> class BnInterface : public INTERFACE, public BBinder
    根據繼承關係, 處理 IPC command will go:
    AudioPolicyService.cpp:: AudioPolicyService::onTransact()
      -> return BnAudioPolicyService::onTransact(code, data, reply, flags);
      -> IAudioPolicyService.cpp:: BnAudioPolicyService::onTransact()
      -> case SET_STREAM_VOLUME:
         reply->writeInt32(static_cast <uint32_t>(setStreamVolumeIndex()))
      -> will call AudioPolicyService:: setStreamVolumeIndex()
      -> AudioPolicyInterfaceImpl.cpp:: AudioPolicyService:: setStreamVolumeIndex()
