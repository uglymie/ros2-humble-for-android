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

## 编译过程

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
4. 克隆ros2和ros2 java源码
   
   ```
   mkdir -p $HOME/ros2_android_ws/src
   cd $HOME/ros2_android_ws
   curl https://raw.githubusercontent.com/ros2-java/ros2_java/main/ros2_java_android.repos | vcs import src
   ```
   
   这里的版本是ros2 galactic，想要换成最新的长期支持版本humble也可以，需要将ros2_java_android.repos里面的galactic字样修改为humble，其中Fast DDS的版本也可以更新为最新的版本。
   整理了一份humble的repos:
   
   ```
   curl https://raw.githubusercontent.com/uglymie/ros2-humble-for-android/main/ros2_java_android.repos | vcs import src
   ```
5. 设置编译配置
   
   ```
   export PYTHON3_EXEC="$( which python3 )"
   export PYTHON3_LIBRARY="$( ${PYTHON3_EXEC} -c 'import os.path; from distutils import sysconfig; print(os.path.realpath(os.path.join(sysconfig.get_config_var("LIBPL"), sysconfig.get_config_var("LDLIBRARY"))))' )"
   export PYTHON3_INCLUDE_DIR="$( ${PYTHON3_EXEC} -c 'from distutils import sysconfig; print(sysconfig.get_config_var("INCLUDEPY"))' )"
   export ANDROID_ABI=armeabi-v7a
   export ANDROID_NATIVE_API_LEVEL=android-29
   export ANDROID_TOOLCHAIN_NAME=arm-linux-androideabi-clang
   ```
   
   其中ANDROID相关选项可以更换，如
   
   ```
   export PYTHON3_EXEC="$( which python3 )"
   export PYTHON3_LIBRARY="$( ${PYTHON3_EXEC} -c 'import os.path; from distutils import sysconfig; print(os.path.realpath(os.path.join(sysconfig.get_config_var("LIBPL"), sysconfig.get_config_var("LDLIBRARY"))))' )"
   export PYTHON3_INCLUDE_DIR="$( ${PYTHON3_EXEC} -c 'from distutils import sysconfig; print(sysconfig.get_config_var("INCLUDEPY"))' )"
   export ANDROID_ABI=x86_64
   export ANDROID_NATIVE_API_LEVEL=android-21
   export ANDROID_TOOLCHAIN_NAME=x86_64-clang
   ```
6. 编译
   
   ```
   colcon build \
      --packages-ignore cyclonedds rcl_logging_log4cxx rcl_logging_spdlog rosidl_generator_py rclandroid ros2_talker_android ros2_listener_android \
      --cmake-args \
      -DPYTHON_EXECUTABLE=${PYTHON3_EXEC} \
      -DPYTHON_LIBRARY=${PYTHON3_LIBRARY} \
      -DPYTHON_INCLUDE_DIR=${PYTHON3_INCLUDE_DIR} \
      -DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE} \
      -DANDROID=ON \
      -DANDROID_FUNCTION_LEVEL_LINKING=OFF \
      -DANDROID_NATIVE_API_LEVEL=${ANDROID_TARGET} \
      -DANDROID_TOOLCHAIN_NAME=${ANDROID_TOOLCHAIN_NAME} \
      -DANDROID_STL=c++_shared \
      -DANDROID_ABI=${ANDROID_ABI} \
      -DANDROID_NDK=${ANDROID_NDK} \
      -DTHIRDPARTY=ON  \
      -DCOMPILE_EXAMPLES=OFF \
      -DCMAKE_FIND_ROOT_PATH="${PWD}/install" \
      -DBUILD_TESTING=OFF \
      -DRCL_LOGGING_IMPLEMENTATION=rcl_logging_noop \
      -DTHIRDPARTY_android-ifaddrs=FORCE
   ```
   
   此编译命令经过多次问题排查更改，已经可以避免大多数错误。但是仍然可能出现其他错误，比如找不到jni.h和jni_md.h
   
   ```
   fatal error: jni.h: No such file or directory
   ```
   
   解决方法: 可以添加全局搜索路径到 .bashrc 或者配置文件 /etc/profile 中，其中java-11-openjdk-amd64为当前系统已经安装的jdk版本，CPATH适用于所有语言。
   
   ```
   export CPATH=/usr/lib/jvm/java-11-openjdk-amd64/include:$CPATH
   export CPATH=/usr/lib/jvm/java-11-openjdk-amd64/include/linux:$CPATH
   ```
   
   编译到Fast-DDS时可能出现找不到asio和tinyxml2的头文件，可以在其目录下的CMakeList.txt文件里添加包含头文件路径
   ```
   include_directories(thirdparty/asio/asio/include)
   include_directories(thirdparty/tinyxml2)
   ```
   如果后续遇到其他问题再补充


## 打包过程




