### 编译AOSP源码

注意环境变量的设置：

```bash
source ./build/envsetup.sh
# lunch sdk_phone_x86_64
lunch sdk_phone_x86
make -j32
```

编译完成后需要用的重要的镜像与文件:

```
system-qemu.img		userdata-qemu.img		vendor-qemu.img
ramdisk-qemu.img	encryptionkey.img		VerifiedBootParams.textproto
data(目录)			kernel-ranchu
```

### Windows设置

默认需要安装AS，然后安装AS SDK。先通过`AVD Manager`创建对应API的设备。然后在创建的`avd`设备目录中设置好需要启动的目录。例如`C:\Users\Administrator\.android\avd\AOSP-12_1.avd\config.ini`里面有设备启动的相关配置，如果需要配置`image.sysdir.1`启动镜像的位置，可以修改该项。同时也可以将上一步骤的镜像复制到`image.sysdir.1`指定的目录(需要去除-qemu字段)，注意，如果复制了`system.img`需要将`VerifiedBootParams.textproto`一并复制过去，否则在`init`阶段会出现avb错误。下面是启动命令：

```bash
 C:\Users\Administrator\AppData\Local\Android\Sdk> .\emulator\emulator.exe -avd AOSP-12  -system Z:\workspace\gphone\AndroidS\AOSP\out\target\product\emulator_x86_64\system-qemu.img -kernel  Z:\workspace\gphone\AndroidS\AOSP\out\target\product\emulator_x86_64\kernel-ranchu   -show-kernel  -encryption-key  Z:\workspace\gphone\AndroidS\AOSP\out\target\product\emulator_x86_64\encryptionkey.img -ramdisk Z:\workspace\gphone\AndroidS\AOSP\out\target\product\emulator_x86_64\ramdisk-qemu.img -vendor  Z:\workspace\gphone\AndroidS\AOSP\out\target\product\emulator_x86_64\vendor-qemu.img -data  Z:\workspace\gphone\AndroidS\AOSP\out\target\product\emulator_x86_64\userdata-qemu.img -cache Z:\workspace\gphone\AndroidS\AOSP\out\target\product\emulator_x86_64\cache.img
```

windows环境下，其实所有配置都是按照`config.ini`进行启动，所以可以直接通过如下方式启动：

```bash
 C:\Users\Administrator\AppData\Local\Android\Sdk> .\emulator\emulator.exe -avd AOSP-12 
```

启动界面如下：

![image-20220105102050326](D:\system\hexo\myblog\wdxx\source\images\image-20220105102050326.png)

![image-20220105102117679](D:\system\hexo\myblog\wdxx\source\images\image-20220105102117679.png)

### 参考

1. https://source.android.com/setup/create/avd
