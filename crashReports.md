
# crashReports

##### crash日志文件的获取

当app运行崩溃时，系统会自动生成crash文件并保存在你的手机中。这里我们可以以同步的方式来获取到手机上的crash文件。具体步骤如下：

1. 把你的手机连接到电脑
2. 打开itunes,选择本电脑，点击同步![Mou icon](https://github.com/hmxxxhhh/images/blob/master/crash1.png?raw=true)
3. 进入文件路径~/Library/Logs/CrashReporter/MobileDevice/6s/即可看到同步的crash文件。


##### app文件 dSYM文件的获取

1. 打开organizer,选择你所打包的app，右击show in finder ![Mou icon](https://github.com/hmxxxhhh/images/blob/master/crash2.png?raw=true)
2. 右击显示包类容，dSYM文件在/dSYMs文件下（有时候该文件夹下有多个文件）如下：![Mou icon](https://github.com/hmxxxhhh/images/blob/master/crash3.png?raw=true)其中前两个文件分别为armv7和arm64下的，最后一个我认为是两者合并后的。
3. app文件![Mou icon](https://github.com/hmxxxhhh/images/blob/master/crash4.png?raw=true)

##### symbolicatecrash工具

网上看了好多，但是发现在我的电脑都找不到，在Xcode7.2下，其路径为：
/Applications/Xcode.app/Contents/SharedFrameworks/DTDeviceKitBase.framework/Versions/A/Resources/symbolicatecrash

##### 符号化

未符号化之前的代码,我们是无法解读的：  

![Mou icon](https://github.com/hmxxxhhh/images/blob/master/crash5.png?raw=true)  

下面介绍如何符号化：  

1. 在把以上的crash日志文件、app文件、dSYM文件、symbolicatecrash工具拷贝到同一文件夹下：![Mou icon](https://github.com/hmxxxhhh/images/blob/master/crash6.png?raw=true) (crash.log可以先忽略)
2. 检查日志文件、app文件、dSYM文件的UUID是否一致，   dwarfdump --uuid xxx.app/xxx (xxx工程名)，查看xx.app.dSYM文件的uuid的方法，在terminal中输入命令：dwarfdump --uuid xxx.app.dSYM (xxx工程名)，而.crash的uuid位于，crash日志中的Binary Images:中的第一行尖括号内。如：armv7<8bdeaf1a0b233ac199728c2a0ebb4165>
3. 执行DEVELOPER_DIR="/Applications/XCode.app/Contents/Developer"
4. 执行./symbolicatecrash -v crashText-2016-05-11-135410.crash 1CB2ED07-0083-3423-8001-D2A276CA404E.dSYM/ >crash.log  

打开crash.log文件如下：  
![Mou icon](https://github.com/hmxxxhhh/images/blob/master/crash7.png?raw=true)

此时你发现有些部分可能并没有符号化完全，这时我们可以根据地址解析内容输入以下命令：  
 
 	atos -arch arm64 -o crashText.app/crashText -l 0x100000000 0x100006b3c 
 
 结果如下：  
 ![Mou icon](https://github.com/hmxxxhhh/images/blob/master/crash8.png?raw=true)








