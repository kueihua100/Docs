# SELinux Policy

#### 1. Where to make?
    type "make" at android root, eg. oreo/pie folder

#### 2. What files to update?
    pie/out/target/product/xxx/vendor/etc/selinux/*  -> /vendor/etc/selinux/*
    pie/out/target/product/xxx/system/etc/selinux/*  -> /system/etc/selinux/*
    
#### 3. How to fix "neverallow" compile error?
    a. Check from whitelist form pie/system/sepolicy/
        eg. exported_system_prop
    b. add your property key with whitelist at xxx_contexts:
        aaa.bbb.ccc.       u:object_r:exported_system_prop:s0
    c. add set_prop(xxxx, white_list_key) or get_prop(xxx. white_list_key) to your_service.te
        set_prop(your_service_A, exported_system_prop)
        get_prop(your_service_B, exported_system_prop)
        
#### How to fix below running error?
    init: Could not start service 'vendor.tuner-hal-1-0' as part of class 'hal': File /vendor/bin/hw/android.hardware.tv.sampleTuner @1.0-service(labeled "u:object_r:vendor_file:s0") 
        has incorrect label or no domain transition from u:r:init:s0 to another SELinux domain defined. 
        Have you configured your service correctly? https://source.android.com/security/selinux/device-policy#label_new_services_and_address_denials
        [note] see "Label new services and address denials" part from https://source.android.com/security/selinux/device-policy?hl=en
        [add]
            a) hal_sampleTuner 
            b} to include "hal_sampleTuner" into file_contexts

    android.hardwar: type=1400 audit(0.0:355): avc: denied { read } for name="u:object_r:hwservicemanager_prop:s0" dev="tmpfs" ino=1926 scontext=u:r:hal_sampleTuner :s0 tcontext=u:object_r:hwservicemanager_prop:s0 tclass=file permissive=0
    [add in hal_sampleTuner] allow hal_sampleTuner  hwservicemanager_prop:file { read };

    android.hardwar: type=1400 audit(0.0:243): avc: denied { open } for path="/dev/__properties__/u:object_r:hwservicemanager_prop:s0" dev="tmpfs" ino=8291 scontext=u:r:hal_sampleTuner :s0 tcontext=u:object_r:hwservicemanager_prop:s0 tclass=file permissive=0
    [add in hal_sampleTuner] allow hal_sampleTuner  hwservicemanager_prop:file { open read };

    android.hardwar: type=1400 audit(0.0:249): avc: denied { getattr } for path="/dev/__properties__/u:object_r:hwservicemanager_prop:s0" dev="tmpfs" ino=6504 scontext=u:r:hal_sampleTuner :s0 tcontext=u:object_r:hwservicemanager_prop:s0 tclass=file permissive=0    
    [add in hal_sampleTuner] allow hal_sampleTuner  hwservicemanager_prop:file { getattr open read };

    android.hardwar: type=1400 audit(0.0:276): avc: denied { map } for path="/dev/__properties__/u:object_r:hwservicemanager_prop:s0" dev="tmpfs" ino=8086 scontext=u:r:hal_sampleTuner :s0 tcontext=u:object_r:hwservicemanager_prop:s0 tclass=file permissive=0
    [add in hal_sampleTuner] allow hal_sampleTuner  hwservicemanager_prop:file { getattr open read map };
