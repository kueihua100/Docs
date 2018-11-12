# Audio launcher touch sound (key tone)
1. Android 開到桌面 (launcher), 上下左有移動, 應該會有touch sound (key tone)
2. UI touch sounds are put under: 
###
    /xxx/pie/frameworks/base/data/sounds (Oreo and Pie both). 
    But at android Pie, launcher touch sound “Effect_Tick.ogg” was removed. 
    Needs to be added by vendor and put under /system/media/audio/ui at target board.
    
