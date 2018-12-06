# How to VTS and debug VTS?
#### Setup adb and aapt
    # sudo apt install adb
    # sudo apt install aapt
    
    [note] 
    if the installed adb version is older than pie/out/host/linux-x86/bin/adb
    you can have a copy in your bin_path and add this bin_path to your PATH environment~
    
#### How to build VTS code:  
VTS google info page from [HERE](https://source.android.com/compatibility/vts/systems)  

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
#### How to build CTS code:  
CTS google info page from [HERE](https://source.android.com/compatibility/cts/development)  

    # source build/envsetup.sh
    # lunch <productName>
    # make cts
    cTS testsuit is under: out/host/linux-x86/cts/android-cts.zip
    [note] cts source is under xxx/pie/cts/xxx
    
#### How to run CTS? 
Download CTS testsuit from [HERE](https://source.android.com/compatibility/cts/downloads)  

    # adb connect -s 172.22.56.203:5555
    # cd android-cts/tools
    # ./cts-tradefed
    # run cts -s 172.22.56.203:5555 -m CtsNativeMediaAAudioTestCases
    # run cts -s 172.22.56.203:5555 -m CtsNativeMediaAAudioTestCases -t android.nativemedia.aaudio.AAudioTests#SPM_AAudioInputStreamCallbackTest_testRecording_SHARED__0__DEFAULT
    
#### How to debug CTS? 
    1. Fist read the html report under "results" folder
    2. Check device_logcat_test_xxxx.tgz under "logs" folder
    3. Check host_log_xxxx.tgz under "logs" folder
    4. De-compile android-cts\testcases\xxxx_apk to JAVA code according the test module.
       And check the JAVA code.

#### How to read the code from CTS source?
    Use CtsNativeMediaAAudioTestCases as an example.
    From test_result.xml: there is a test case named:
    SPM_AAudioInputStreamCallbackTest_testRecording_SHARED__0__LOW_LATENCY
    
    1. From xxx/pie/cts//tests/tests/nativemedia/aaudio/jni/test_aaudio_callback.cpp
    2. There are 2 TEST_P in test_aaudio_callback.cpp:
        TEST_P(AAudioInputStreamCallbackTest, testRecording)
        TEST_P(AAudioOutputStreamCallbackTest, testPlayback)
    3. There are 2 INSTANTIATE_TEST_CASE_P in test_aaudio_callback.cpp:
        INSTANTIATE_TEST_CASE_P(SPM, AAudioInputStreamCallbackTest, ...)
        INSTANTIATE_TEST_CASE_P(SPM, AAudioOutputStreamCallbackTest, ...)
    combine above 2 and 3 to form test cases.
[note 1] TEST_P and INSTANTIATE_TEST_CASE_P macro can refer to [HERE](https://github.com/abseil/googletest/blob/master/googletest/docs/advanced.md)
# How to CTS Verifier and debug CTS verifier?
Download  CTS Verifier APK and CTS Media Files from [HERE](https://source.android.com/compatibility/cts/verifier)

    # adb install -r -g CtsVerifier.apk
    Then open CtsVerifier APK from target and follow the UI to do test~
    
# How to GTS and debug GTS?
    # adb connect -s 172.22.56.203:5555
    # cd android-gts/tools
    # ./gts-tradefed 
    # run gts -s 172.22.56.203:5555 -m GtsGmscoreHostTestCases -t com.google.android.gts.audio.AudioHostTest#testMixByUidCapturing
    # run gts -s 172.22.56.203:5555 -m GtsGmscoreHostTestCases -t com.google.android.gts.audio.AudioHostTest#testTwoChannelCapturing
