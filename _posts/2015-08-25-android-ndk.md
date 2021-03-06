---
layout: post
title: Android NDK 开发详解
category: 技术
---

*参考网址：*[Native Development Kit (NDK)](https://www3.ntu.edu.sg/home/ehchua/programming/android/Android_NDK.html "Markdown")

####一、 Installing the Native Development Kit (NDK)

1. Setting up all the necessary tools for Android programming, such as JDK, Eclipse, Android SDK, Eclipse ADT (Read **the 2nd method** of [Android 5.0 开发环境搭建](http://lsclone.github.io/blog/%E6%8A%80%E6%9C%AF/2015/08/20/android5.html "Markdown")); and (for Windows Users) Cygwin (Read "How to install Cygwin" and "GCC and Make").

2. Download the Android NDK from **http://developer.android.com/tools/sdk/ndk/index.html** (e.g., android-ndk-r8-windows.zip).

3. Unzip the downloaded zip file into a directory of your choice (e.g., d:\myproject). The NDK will be unzipped as d:\myproject\android-ndk-r8. I shall denote the installed directory as **NDK_HOME**.

4. Include the NDK installed directory in the **PATH** environment variable.

5. Eclipse -> Windows -> Preferences -> Android -> NDK, **NDK Location**: NDK installed directory(e.g., d:\myproject\android-ndk-r8)

####二、Writing a Hello-world Android NDK Program (NDK实例)

**Step 0: Read the Documentation**

Read "Android NDK" @ [http://developer.android.com/tools/sdk/ndk/index.html](Read "Android NDK" @ http://developer.android.com/tools/sdk/ndk/index.html "Markdown").

**Step 1: Write an Android JNI program**

Create an Android project called "AndroidHelloJNI", with application name "Hello JNI" and package "com.mytest". Create an activity called "JNIActivity" with Layout name "activity_jni" and Title "Hello JNI".

Replaced the "JNIActivity.java" as follows:

    package com.mytest;
     
    import android.os.Bundle;
    import android.app.Activity;
    import android.widget.TextView;
     
    public class JNIActivity extends Activity {
     
       static {
          System.loadLibrary("myjni"); // "myjni.dll" in Windows, "libmyjni.so" in Unixes
       }
     
       // A native method that returns a Java String to be displayed on the
       // TextView
       public native String getMessage();
     
       @Override
       public void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          // Create a TextView.
          TextView textView = new TextView(this);
          // Retrieve the text from native method getMessage()
          textView.setText(getMessage());
          setContentView(textView);
       }
    }

**Step 2: Generating C/C++ Header File using "javah" Utility**

Create a folder "jni" under the project's root (right-click on the project ⇒ New ⇒ Folder). Create a sub-folder "include" under "jni" for storing the header files.

Run "javah" utility (**from a CMD shell**) to create C/C++ header called "HelloJNI.h":

```
> javah --help
......
// Change directory to <project-root>/jni/include
> javah -classpath ../../bin/classes;<ANDROID_SDK_HOME>\platforms\android-<xx>\android.jar 
  -o HelloJNI.h com.mytest.JNIActivity
```

参考实例(**cmd 命令行模式**)

    > javah -classpath ../../bin/classes;C:\adt-bundle-windows-x86_64-20140321\sdk\platforms\android-19\android.jar 
      -o HelloJNI.h com.mytest.JNIActivity

* -classpath: in our case, we need the JNIActivity.class which is kept in "<project-root>\bin\classes"; and its superclass Android.app.Activity.class which is kept in android.jar under the Android SDK.
* android.jar contains **android api classes** ([classes in android.jar](http://www.java2s.com/Code/Jar/a/Downloadandroidjar.htm "Markdown"))
* -o: to set the output filename.
* You need to use the fully-qualified name "com.mytest.JNIActivity".

**Step 3: C Implementation - HelloJNI.c**

Create the following C program called "HelloJNI.c" under the "jni" directory (right-click on the "jni" folder ⇒ New ⇒ File):

    #include <jni.h>
    #include "include/HelloJNI.h"
     
    JNIEXPORT jstring JNICALL Java_com_mytest_JNIActivity_getMessage
              (JNIEnv *env, jobject thisObj) {
       return (*env)->NewStringUTF(env, "Hello from native code!");
    }

**Step 4: Create an Android makefile - Android.mk**

Create an Android makefile called "Android.mk" under the "jni" directory (right-click on "jni" folder ⇒ New ⇒ File), as follows:

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_MODULE    := myjni
LOCAL_SRC_FILES := HelloJNI.c

include $(BUILD_SHARED_LIBRARY)
```

In the above makefile, "myjni" is the name of our shared library (used in System.loadLibrary()), and "HelloJNI.c" is the source file.

**Step 5: Build NDK**

**Start a CMD shell**, change directory to the project's root directory, and run "ndk-build" script provided by Android NDK (the Android NDK installed directory shall be in the PATH).

```
// Change directory to <project-root>
> ndk-build
Compile thumb : myjni <= HelloJNI.c
SharedLibrary  : libmyjni.so
Install        : libmyjni.so => libs/armeabi/libmyjni.so
```

you can also build NDK with **cygwin** :

```
// open cygwin and change directory to <project-root>
> $NDK_HOME/ndk-build
```

NOTES:

* Use "ndk-build --help" to display the command-line options.
* Use "ndk-build V=1" to display the build messages.
* Use "ndk-build -B" to perform a force re-built.

**Step 6: Run the Android App**

Run the android app, via "Run As" ⇒ "Android Application". You shall see the message from the native program appears on the screen.

Check the "LogCat" panel to confirm that the shared library "libmyjni.so" is loaded.

    ...: Trying to load lib /data/data/com.example.androidhellojni/lib/libmyjni.so ...
    ...: Added shared lib /data/data/com.example.androidhellojni/lib/libmyjni.so ...

**Step 7: print log**

```
#include <android/log.h>
#define LOGD(...) ((void)__android_log_print(ANDROID_LOG_DEBUG, "native-tag", __VA_ARGS__))
#define LOGI(...) ((void)__android_log_print(ANDROID_LOG_INFO,  "native-tag", __VA_ARGS__))
#define LOGW(...) ((void)__android_log_print(ANDROID_LOG_WARN,  "native-tag", __VA_ARGS__))
#define LOGE(...) ((void)__android_log_print(ANDROID_LOG_ERROR, "native-tag", __VA_ARGS__))

...
LOGD("%s: func failed", __FUNCTION__);
...
```

**Android.mk** add:

```
LOCAL_LDLIBS    := -llog
```

**Step 8: Android.mk**

头文件包含路径： 

    LOCAL_C_INCLUDES 	:= ./
    LOCAL_C_INCLUDES 	+= ../

链接外部库文件：

    LOCAL_LDFLAGS  		:= -L./
    LOCAL_LDLIBS 		:= -lhello  #eg,. libhello.a

**链接外部库文件的推荐方式：**

链接外部静态库：

```
LOCAL_PATH			    := $(call my-dir)

include $(CLEAR_VARS)
LOCAL_MODULE            := math
LOCAL_SRC_FILES         := libmath.a
#LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)
include $(PREBUILT_STATIC_LIBRARY)

include $(CLEAR_VARS)

LOCAL_C_INCLUDES 	    := ./include/

LOCAL_MODULE    	    := hello  #eg,. libhello.so
LOCAL_SRC_FILES 	    := hello.c

LOCAL_STATIC_LIBRARIES	:= math
LOCAL_LDLIBS 		    := -llog

include $(BUILD_SHARED_LIBRARY)
```

链接外部动态库：

```
LOCAL_PATH			    := $(call my-dir)

include $(CLEAR_VARS)
LOCAL_MODULE            := math
LOCAL_SRC_FILES         := libmath.so
#LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)
include $(PREBUILT_STATIC_LIBRARY)

include $(CLEAR_VARS)

LOCAL_C_INCLUDES 	    := ./include/

LOCAL_MODULE    	    := hello  #eg,. libhello.so
LOCAL_SRC_FILES 	    := hello.c

LOCAL_SHARED_LIBRARIES	:= math
LOCAL_LDLIBS 		    := -llog

include $(BUILD_SHARED_LIBRARY)
```

判断语句写法：

```
ifeq ($(TARGET_ARCH_ABI), armeabi-v7a)
	common_CFLAGS += -march=armv7-a -mfloat-abi=softfp -mfpu=neon
	common_LDFLAGS += -Wl,--fix-cortex-a8
else
...
endif
```

JNI层调用OpenGL ES需要连接库文件：

```
LOCAL_LDLIBS := -ljnigraphics -lGLESv1_CM -lGLESv2
```

* [Android.mk小结](http://blog.csdn.net/crazyman2010/article/details/40401545 "android.mk")
* [编写Android.mk中的LOCAL_SRC_FILES的终极技巧](http://blog.csdn.net/fu_zk/article/details/12836431 "android.mk")
* [Android.mk 中的 LOCAL_SRC_FILES, LOCAL_C_INCLUDES](http://blog.163.com/caty_nuaa/blog/static/90390720144269528857/?COLLCC=3225355915& "android.mk")
* [Android.mk语法说明（android ndk开发）](http://www.linuxidc.com/Linux/2011-09/42638p2.htm "android.mk")

**Step 9: Application.mk**

[armeabi和armeabi-v7a 以及x86](http://blog.sina.com.cn/s/blog_95c607dd0102uxau.html "markdown")
```
APP_ABI := armeabi armeabi-v7a arm64-v8a x86
通常只编译armeabi-v7a即可
```

[怎么在android app中使用STL库](http://www.myexception.cn/android/1704478.html "markdown")
```
APP_STL := stlport_static 推荐
or APP_STL := gnustl_static
```

* 通用例子

```
APP_PLATFORM := android-19
NDK_TOOLCHAIN_VERSION := 4.8
APP_ABI := armeabi-v7a
APP_STL := gnustl_static
APP_OPTIM := debug
#APP_OPTIM := release
APP_CPPFLAGS := -O3 -fno-rtti -fno-exceptions -finline-functions -funswitch-loops -fgcse-after-reload -pipe -flto -mfpu=neon
#APP_CPPFLAGS := -O0 -g -frtti -fexceptions -finline-functions -funswitch-loops -fgcse-after-reload -pipe
```

*参考网址*：

* [Application.mk帮助文档](https://developer.android.com/ndk/guides/application_mk.html "Markdown")
* [Android NDK开发指南（一） Application.mk文件](http://www.cnblogs.com/yaozhongxiao/archive/2012/03/06/2381586.html "Markdown")
* [ndk 添加c++11支持](http://jingyan.baidu.com/article/b87fe19ebd51fa52183568f7.html "Markdown")
* [Android NDK的C++11标准支持](http://www.tuicool.com/articles/NFjEby "Markdown")
* [Application.mk简介](http://blog.csdn.net/wang_shaner/article/details/41479721 "Markdown")

**Step 10: ndk-gdb debug**

*参考网址*：

* [Android ndk-gdb 调试](http://blog.csdn.net/kaiqiangzhang001/article/details/21108857 "Markdown")
* [android 通过gdbserver 调试c++](http://blog.csdn.net/liuhongxiangm/article/details/32085175 "Markdown")

====================================================================

**MORE REFERENCES & RESOURCES**

1. [Windows环境下Android NDK环境搭建](http://blog.csdn.net/pengchua/article/details/7582949 "Markdown")

2. [How to install Cygwin](https://www3.ntu.edu.sg/home/ehchua/programming/howto/Cygwin_HowTo.html "Markdown")

3. Cygwin Setup - Choose Download Site(s) add : http://mirrors.sohu.com

4. [Cygwin官方下载页面](https://cygwin.com/install.html "Markdown")

5. [Android.mk | Android Developer -- google官网NDK之Android.mk(Makefile)说明文档](https://developer.android.com/ndk/guides/android_mk.html#mdv "Markdown") 

6. **java与native之间的参数传递参考：** [Android JNI 开发详解](http://lsclone.github.io/blog/%E6%8A%80%E6%9C%AF/2015/08/25/android-jni.html "android jni")

7. [Android NDK开发（2）----- JNI多线程](http://biancheng.dnbcw.info/shouji/388426.html "Markdown")


====================================================================

====================================================================

Android Studio NDK开发

Using Android Studio 2.2 or higher with the Android plugin for Gradle version 2.2.0 or higher, you can add C and C++ code to your app by compiling it into a native library that Gradle can package with your APK. Your Java code can then call functions in your native library through the Java Native Interface (JNI)

Android Studio's default build tool for native libraries is CMake. Android Studio also supports ndk-build due to the large number of existing projects that use the build toolkit to compile their native code. If you want to import an existing ndk-build library into your Android Studio project, see the section about how to configure Gradle to link to your native library. However, if you are creating a new native library, you should use CMake.

**Step 1: Download the NDK and Build Tools**

To compile and debug native code for your app, you need the following components:

> The Android Native Development Kit (NDK): a toolset that allows you to use C and C++ code with Android, and provides platform libraries that allow you to manage native activities and access physical device components, such as sensors and touch input.

> CMake: an external build tool that works alongside Gradle to build your native library. You do not need this component if you only plan to use ndk-build.

> LLDB: the debugger Android Studio uses to debug native code.

You can install these components using the **SDK Manager**:

> From an open project, select Tools > Android > SDK Manager from the menu bar.

> Click the SDK Tools tab.

> Check the boxes next to LLDB, CMake, and NDK

**Step 2: Create a New Project with C/C++ Support**

Creating a new project with support for native code is similar to creating any other Android Studio project, but there are a few additional steps:

> In the Configure your new project section of the wizard, check the Include C++ Support checkbox.

> Click Next.

> Complete all other fields and the next few sections of the wizard as normal.

> In the Customize C++ Support section of the wizard, you can customize your project with the following options:

>> **C++ Standard**: use the drop-down list to select which standardization of C++ you want to use. Selecting Toolchain Default uses the default CMake setting.

>> **Exceptions Support**: check this box if you want to enable support for C++ exception handling. If enabled, Android Studio adds the -fexceptions flag to cppFlags in your module-level build.gradle file, which Gradle passes to CMake.

>> **Runtime Type Information Support**: check this box if you want support for RTTI. If enabled, Android Studio adds the -frtti flag to cppFlags in your module-level build.gradle file, which Gradle passes to CMake.

> Click Finish.

* The cpp group is where you can find all the native source files, headers, and prebuilt libraries that are a part of your project. For new projects, Android Studio creates a sample C++ source file, **native-lib.cpp**, and places it in the src/main/cpp/ directory of your app module. This sample code provides a simple C++ function, stringFromJNI(), that returns the string “Hello from C++”. You can learn how to add additional source files to your project in the section about how to Create new native source files.

* The External Build Files group is where you can find build scripts for CMake or ndk-build. Similar to how **build.gradle** files tell Gradle how to build your app, CMake and ndk-build require a build script to know how to build your native library. For new projects, Android Studio creates a CMake build script, **CMakeLists.txt**, and places it in your module’s root directory. You can learn more about the contents of this build script in the section about how to Create a Cmake Build Script.

**Step 3: Create a CMake build script (CMakeLists.txt)**

You can now configure your build script by adding CMake commands. To instruct CMake to create a native library from native source code, add the **cmake_minimum_required()** and **add_library()** commands to your build script:

```
# Sets the minimum version of CMake required to build your native library.
# This ensures that a certain set of CMake features is available to
# your build.

cmake_minimum_required(VERSION 3.4.1)

# Specifies a library name, specifies whether the library is STATIC or
# SHARED, and provides relative paths to the source code. You can
# define multiple libraries by adding multiple add.library() commands,
# and CMake builds them for you. When you build your app, Gradle
# automatically packages shared libraries with your APK.

add_library( # Specifies the name of the library.
             native-lib

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             src/main/cpp/native-lib.cpp
	     src/main/cpp/other.cpp
	     ...)
```

When you add a source file or library to your CMake build script using add_library(), Android Studio also shows associated header files in the Project view after you sync your project. However, in order for CMake to locate your header files during compile time, you need to add the **include_directories()** command to your CMake build script and specify the path to your headers:

```
add_library(...)

# Specifies a path to native header files.
include_directories(src/main/cpp/include/)

```

The convention CMake uses to name the file of your library is as follows:

> lib*library-name*.so

For example, if you specify "native-lib" as the name of your shared library in the build script, CMake creates a file named libnative-lib.so. However, when loading this library in your Java code, use the name you specified in the CMake build script:

```
static {
    System.loadLibrary(“native-lib”);
}
```

Note: If you rename or remove a library in your CMake build script, you need to clean your project before Gradle applies the changes or removes the older version of the library from your APK. To clean your project, select **Build > Clean Project** from the menu bar.

Android Studio automatically adds the source files and headers to the cpp group in the Project pane. By using multiple **add_library()** commands, you can define additional libraries for CMake to build from other source files.

**Step 4: Add NDK APIs(library)**

Add the **find_library()** command to your CMake build script to locate an NDK library and store its path as a variable. You use this variable to refer to the NDK library in other parts of the build script. The following sample locates the **Android-specific log support library** and stores its path in **log-lib**:

```
find_library( # Defines the name of the path variable that stores the
              # location of the NDK library.
              log-lib

              # Specifies the name of the NDK library that
              # CMake needs to locate.
              log )
	      
find_library( z-lib z )

find_library( jnigraphics-lib jnigraphics)

find_library( glesv1-lib GLESv1_CM )

find_library( glesv2-lib GLESv2 )

find_library( egl-lib  EGL)
```

In order for your native library to call functions in the log library, you need to link the libraries using the **target_link_libraries()** command in your CMake build script:

```
find_library(...)

# Links your native library against one or more other native libraries.
target_link_libraries( # Specifies the target library.
                       native-lib

                       # Links the log library to the target library.
                       ${log-lib}

                       ${z-lib}

                       ${jnigraphics-lib}

                       ${glesv1-lib}

                       ${glesv2-lib}

                       ${egl-lib})
```

The NDK also includes some libraries as source code that you need to build and link to your native library. You can compile the source code into a native library by using the **add_library()** command in your CMake build script. To provide a path to your local NDK library, you can use the **ANDROID_NDK** path variable, which Android Studio automatically defines for you.

The following command tells CMake to build **android_native_app_glue.c**, which manages NativeActivity lifecycle events and touch input, into a static library and links it to **native-lib**:

```
add_library( app-glue
             STATIC
             ${ANDROID_NDK}/sources/android/native_app_glue/android_native_app_glue.c )

# You need to link static libraries against your shared native library.
target_link_libraries( native-lib app-glue ${log-lib} )
```

**Step 5: Add other prebuilt libraries**

Adding a prebuilt library is similar to specifying another native library for CMake to build. However, because the library is already built, you need to use the **IMPORTED** flag to tell CMake that you only want to import the library into your project:

```
add_library( imported-lib
             SHARED
             IMPORTED )
```

You then need to specify the path to the library using the **set_target_properties()** command as shown below.

Some libraries provide separate packages for specific CPU architectures, or **Application Binary Interfaces (ABI)**, and organize them into separate directories. This approach helps libraries take advantage of certain CPU architectures while allowing you to use only the versions of the library you want. To add multiple ABI versions of a library to your CMake build script, without having to write multiple commands for each version of the library, you can use the **ANDROID_ABI** path variable. This variable uses a list of the default ABIs that the NDK supports, or a filtered list of ABIs you manually configure Gradle to use. For example:

```
add_library(...)
set_target_properties( # Specifies the target library.
                       imported-lib

                       # Specifies the parameter you want to define.
                       PROPERTIES IMPORTED_LOCATION

                       # Provides the path to the library you want to import.
                       imported-lib/src/${ANDROID_ABI}/libimported-lib.so )
```

For CMake to locate your header files during compile time, you need to use the **include_directories()** command and include the path to your header files:

```
include_directories( imported-lib/include/ )
```

> Note: If you want to package a prebuilt library that is not a build-time dependency—for example, when adding a prebuilt library that is a dependency of imported-lib, you do not need perform the following instructions to link the library.

To link the prebuilt library to your own native library, add it to the target_link_libraries() command in your CMake build script:

```
target_link_libraries( native-lib imported-lib ${log-lib} )
```

**Step 6: Manually configure Gradle**

You can specify optional arguments and flags for CMake or ndk-build by configuring another **externalNativeBuild** block within the **defaultConfig** block of your **module-level build.gradle** file. Similar to other properties in the **defaultConfig** block, you can override these properties for each product flavor in your build configuration.

By default, Gradle builds your native library into separate .so files for the ABIs the NDK supports and packages them all into your APK. If you want Gradle to build and package only certain **ABI** configurations of your native libraries, you can specify them with the **ndk.abiFilters** flag in your **module-level build.gradle** file.

```
android {
    ...
    defaultConfig {
    	...
        externalNativeBuild {
            cmake {
                cppFlags "-std=c++11 -fexceptions -D_ANDROID -D_OPENGLES"
            }
        }
        ndk {
            // Specifies the ABI configurations of your native
            // libraries Gradle should build and package with your APK.
            abiFilters 'x86', 'armeabi-v7a'
        }
    }
    buildTypes {...}
    externalNativeBuild {
        cmake {
	    // Provides a relative path to your CMake build script.
            path "CMakeLists.txt"
        }
    }
}
```

*参考网址*：

* [Add C and C++ Code to Your Project](https://developer.android.com/studio/projects/add-native-code.html#existing-project "ndk")
* [AndroidStudio支持新的NDK的操作使用](http://www.cnblogs.com/zhuyuliang/p/5007016.html "ndk")
* [Android Studio使用新的Gradle构建工具配置NDK环境](http://blog.csdn.net/sbsujjbcy/article/details/48469569 "ndk")
* [最详细Android Studio + NDK范例](http://bbs.51cto.com/thread-1316339-1-1.html "ndk")
