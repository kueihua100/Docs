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
