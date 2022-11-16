使用 fvm 管理flutter 版本

```
 config     设置FVM的配置信息
 flutter    代理flutter命令 
 install    安装flutter SDK版本
 list       列出已安装的flutter SDK版本
 releases   列表flutter已经发布SDK版本。
 remove     删除flutter SDK版本
 use        你想使用的flutter SDK版本
 version    当前安装fvm版本
```



### 创建项目

> fvm flutter create 项目名称

### 启动空安全

> cd 项目名称
>
> dart migrate --apply-changes

### web的两种不同的打包方式 

> flutter run -d chrome --web-renderer html
> flutter build web --web-renderer canvaskit

### 桌面端开发

> flutter config --enable-windows-desktop
>
> flutter config --enable-macos-desktop
>
> flutter config --enable-linux-desktop

想要确保桌面 **已成功启用**，可以列出可用的设备

> ```
> flutter devices
> ```







## Flutter 重新创建指定语言的android/ios目录
```sh
1, 移除android目录，重新创建指定语言的android目录

### 进入工程目录，删除android目录

rm -rf android

### 重新创建java语言的android目录

flutter create -a java .

### 重新创建kotlin语言的android目录

flutter create -a kotlin .

2. 移除ios目录，重新创建指定语言的ios目录

### 进入工程目录，删除ios

rm -rf ios 

### 重新创建指定swift语言的ios目录

flutter create -i swift .

### 重新创建指定objective-c 语言的ios目录

flutter create -i objc .  
```





