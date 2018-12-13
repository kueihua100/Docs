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
    If need debug, follow the "How to debug CTS" part.
# How to GTS and debug GTS?
    # adb connect -s 172.22.56.203:5555
    # cd android-gts/tools
    # ./gts-tradefed 
    # run gts -s 172.22.56.203:5555 -m GtsGmscoreHostTestCases -t com.google.android.gts.audio.AudioHostTest#testMixByUidCapturing
    # run gts -s 172.22.56.203:5555 -m GtsGmscoreHostTestCases -t com.google.android.gts.audio.AudioHostTest#testTwoChannelCapturing
    If need debug, follow the "How to debug CTS" part.
    
# Debug skill:
#### 1. 千軍萬馬
    每個老闆都有千軍萬馬可以調度, 你也可以有~ 但是只有千軍與萬馬. XDD
        千軍就是在你的API 進入點, 打印enter
        萬馬就是在你的API 離開點, 打印exit
        
    千軍萬馬最適用 HAL debugging, 千軍萬馬沒有成對, 應該就要看看hang在哪? 
    or 在哪一個條件return了, 有做好error handling嗎? and return value對嗎?
    
#### 2. 空城計
    有一些問題是透過千軍萬馬, 不容易找到問題, 這通常是程式run 一陣子 後的random hang.
    或者是 某一個測項 單獨測 OK, 整個測 NG.
    此時需要開始進行跳過某個 API 的空城計作法, 如果跳過就OK, 就是該 API 有問題.
    
#### 3. 偷天換日
    有些問題在certification program會遇到assertion, some assert macro has timeout value to check the state is changed or not?
    So maybe 是你的底層太晚回應造成, 若懷疑是如此, 採用 偷天換日 作法, 可以加條件在第一次呼叫時, 無條件return, 若如此OK了, 就可以正名問題在哪~
    但重點不是用 hack 作法, 而是要修正自己的問題~
