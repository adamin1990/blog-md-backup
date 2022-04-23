# Redmi Note 9 5g（M2007J22C）刷TWRP,Magisk，LSPosed

## step 1 解锁手机  
- [小米官方解锁工具及教程](https://www.miui.com/unlock/download.html)
## step 2 刷Android10官方rom（可选）  
 
如果MIUI版本是12.5.1（Android 11），可以选择降低到V12.5.8.0.RJECNXM（Android 10）,我在Android 11上用Xposed模块执行shell操作文件的时候会遇到权限问题，所以选择降到Android10，这个是可选步骤，不需要可以直接跳过这一步。  

- 下载官方刷机工具，[MiFlash 最新版(2018.5.28) ](http://bigota.d.miui.com/tools/MiFlash2018-5-28-0.zip)
- 下载Android 10 刷机包线刷版到[12.5.8.0.RJECNXM](https://web.vip.miui.com/page/info/mio/mio/detail?postId=23527587&app_version=dev.20051)  -
- 关机状态长按power键和音量下键进入fastboot模式 
- 打开MiFlash刷机，注意右下角模式选择flash_all,这样刷完还是解锁状态，具体看官方给的文档  

## step 3 刷第三方recovery TWRP 

- 进入fastboot模式
- 执行命令 fastboot flash recovery xxxx.img 刷twrp 
- 执行命令 astboot --disable-verity --disable-verification flash vbmeta vbmeta.img 去除加密验证 

## step 4 刷magisk 

- 第三步刷完，执行命令fastboot reboot recovery 进去twrp
- 点高级进入side load，滑动解锁 
- 执行命令adb sideload magisk.zip 

## step 5 安装Riru，Lspoded 
- 重启后打开magisk ，在模块里分别安装Riru，Lspoded

TWRP,vbmeta,Magisk，Lspoded下载地址：链接：https://pan.baidu.com/s/1_kYqHhHJ6rLrMsFYmXFhxw 
提取码：1111 
-
