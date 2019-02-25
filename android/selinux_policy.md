# SELinux Policy

#### 1. Where to make?
    type "make" at android root, eg. oreo/pie folder

#### 2. What files to update?
    pie/out/target/product/xxx/vendor/etc/selinux/*  -> /vendor/etc/selinux/*
    pie/out/target/product/xxx/system/etc/selinux/*  -> /system/etc/selinux/*
    
#### 3. How to fix "neverallow" compile error?
    a. Check from whitelist form pie/system/sepolicy/
    b. add your property key with whitelist 
    c. add set_prop(xxxx, white_list_key) or get_prop(xxx. white_list_key) to your_service.te
