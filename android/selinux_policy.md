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
