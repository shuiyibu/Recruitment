# 获取JDK源码
[OpenJDK™ Source Releases](https://jdk7.java.net/source.html)

# 构建编译环境

[Command line tools for Xcode](https://developer.apple.com)

[Apache Ant](http://ant.apache.org/srcdownload.cgi)

```
brew install ant
```
sudo ln -s /usr/bin/llvm-g++ /Applications/Xcode.app/Contents/Developer/usr/bin/llvm-g++

# 进行编译
```
/Library/Java/JavaVirtualMachines/jdk1.8.0_71.jdk/Contents/Home
```

```
#
export LANG=C

#
export ALT_BOOTDIR=/Library/Java/JavaVirtualMachines/jdk1.8.0_71.jdk/Contents/Home

#
export ALLOW_DOWNLOAD=true

#
export HOTSPOT_BUILD_JOBS=6
export ALT_PARALLEL_COMPILE_JOBS=6

#
export SKIP_COMPARE_IMAGES=true


#
export USE_PRECOMPILED_HEADER=true

#
export BUILD_LANGTOOLS=true
export BUILD_HOTSPOT=true
export BUILD_JDK=true

#
BUILD_DEPLOY=false

#
BUILD_INSTALL=false

#
export ALT_OUTPUTDIR=/Users/langdylan/Lion/JVM/jdkBuild/openjdk_7u40/build

#
unset JAVA_HOME
unset CLASSPATH

make 2>&1 | tee $ALT_OUTPUTDIR/build.log

```
