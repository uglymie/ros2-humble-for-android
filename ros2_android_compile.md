# 在Ubuntu22.04下编译ros2 humble的Android版本

## 在终端配置网络代理

1. 由于该项目需要在GitHub上下载大量ros2相关的源码包，国内访问GitHub受限，虽然网上提供了很多访问GitHub的方法，但是速度和稳定性都有所欠缺，所以最好还是直接使用网络代理的方法。
2. 打开.bashrc文件，在文件末尾输入
   
   ```
   export http_proxy='http://localhost:12345'
   export https_proxy='http://localhost:12345'
   ```
   
   其中的12345是代理的端口号，需要依据自己的端口号对应修改，保存退出后source一下就可以了
3. 测试
   在终端输入
   
   ```
   wget www.google.com
   ```
   
   返回以下结果即可
   
   ```
   --2022-11-11 13:00:50--  http://www.google.com/
   正在解析主机 localhost (localhost)... 127.0.0.1
   正在连接 localhost (localhost)|127.0.0.1|:12345... 已连接。
   已发出 Proxy 请求，正在等待回应... 200 OK
   长度： 未指定 [text/html]
   正在保存至: “index.html”
   
   index.html              [ <=>                ]  14.91K  --.-KB/s    用时 0s  
   
   2021-11-16 13:00:51 (116 MB/s) - “index.html” 已保存 [15270]
   ```

## 配置全局网络代理

也可以在网络设置中配置全局代理方式

打开设置-->网络-->网络代理,方法选择手动,填写代理,最后点击应用到整个系统

## 配置Android开发环境

1. 配置Android SDK
   SDK可以直接下载Android studio进行配置，在初次启动Android studio时会提示用户安装必要的sdk和其他模块，一般Ubuntu下会安装在用户目录下。
2. 配置Android NDK
   可以直接前往[NDK官网](https://developer.android.google.cn/ndk/downloads/)下载，下载后可以和SDK放在同一目录，至于DNK的版本选最新的就可以。
3. 设置环境变量
   在.bashrc文件最后一排设置SDK和NDK的环境变量，如下
   ```
   export ANDROID_HOEM=~/Android/Sdk
   export PATH=$PATH:$ANDROID_HOEM/tools:$ANDROID_HOEM/platform-tool
   export ANDROID_NDK=~/Android/ndk-r25b
   ```

