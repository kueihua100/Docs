# How to VTS and debug VTS?
#### Setup adb and aapt
    # sudo apt install adb
    # sudo apt install aapt
    
    [note] 
    if the installed adb version is older than pie/out/host/linux-x86/bin/adb
    you can have a copy in your bin_path and add this bin_path to your PATH environment~
    
#### How to build VTS code:  
    # source build/envsetup.sh
    # lunch <productName>
    # make vts
    VTS testsuit is under: out/host/linux-x86/vts/android-vts.zip
  
#### How to run VTS?  
    # adb connect -s 172.22.54.176:5555
    # cd android-vts/tools
    # ./vts-tradefed
    # list m <== list VTS module
    # run vts -s 172.22.54.176:5555 -m VtsHalAudioEffectV4_0Target
    # run vts -l debug -s 172.22.54.176:5555 -m VtsHalAudioEffectV4_0Target
    # run vts -s 172.22.54.176:5555 -m VtsHalAudioV4_0Target -t FloatAccessorPrimaryHidlTest.MasterVolumeTest
    
#### How to debug?
    1. Fist read the html report under "results" folder
    2. Check device_logcat_test_xxxx.tgz under "logs" folder
    3. Check host_log_xxxx.tgz under "logs" folder
    
# How to CTS and debug CTS?
#### How to run CTS? 
Download CTS testsuit from [HERE](https://source.android.com/compatibility/cts/downloads)  

    # adb connect -s 172.22.56.203:5555
    # cd android-cts/tools
    # ./cts-tradefed
    # run cts -s 172.22.56.203:5555 -m CtsNativeMediaAAudioTestCases
    
#### How to debug CTS? 
    1. Fist read the html report under "results" folder
    2. Check device_logcat_test_xxxx.tgz under "logs" folder
    3. Check host_log_xxxx.tgz under "logs" folder
    4. De-compile android-cts\testcases\xxxx_apk to JAVA code according the test module.
       And check the JAVA code.
    
# How to CTS Verifier and debug CTS verifier?
# How to GTS and debug GTS?
