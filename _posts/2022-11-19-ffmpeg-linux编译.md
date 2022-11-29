---
layout: post
title: ffmpeg linux 编译
date: 2022-11-18
Author: aixz
categories:
tags: [笔记, 音视频]
comments: true
---

1.linux服务器安装NDK ndk下载 低于18的版本或者修改configure配置

2.下载[ffmpeg](https://ffmpeg.org/releases/ffmpeg-snapshot.tar.bz2)源码解压并在ffmpeg文件夹中创建 build_android.sh,脚本如下,NDK地址替换为本地实际路径,并且检查确保toolchain路径正确.

```shell
#!/bin/bash
NDK=/root/android-ndk-r25b
TOOLCHAIN_ROOT_DIR=linux-x86_64
TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/$TOOLCHAIN_ROOT_DIR
API=29
#要编译的ffmpeg内容方法
function build_android {
    echo "Compiling FFmpeg for $CPU"
    ./configure \
    --prefix=$PREFIX \
    --disable-neon \
    --enable-gpl \
    --enable-shared \
    --enable-jni \
    --disable-static \
    --disable-doc \
    --disable-ffmpeg \
    --disable-ffplay \
    --disable-ffprobe \
    --disable-doc \
    --disable-symver \
    --disable-vulkan \
    --disable-stripping \
    --cross-prefix=$CROSS_PREFIX \
    --target-os=android \
    --arch=$ARCH \
    --cpu=$CPU \
    --cc=$CC
    --cxx=$CXX
    --enable-cross-compile \
    --sysroot=$SYSROOT \
    --extra-cflags="-Os -fpic $OPTIMIZE_CFLAGS" \
    --extra-ldflags="$ADDI_LDFLAGS" \
    $ADDITIONAL_CONFIGURE_FLAG
    make clean
    make
    make install
    echo "The Compilation of FFmpeg for $CPU is completed"
}
#armv8-a
ARCH=arm64
CPU=armv8-a
CC=$TOOLCHAIN/bin/aarch64-linux-android$API-clang
CXX=$TOOLCHAIN/bin/aarch64-linux-android$API-clang++
SYSROOT=$NDK/toolchains/llvm/prebuilt/$TOOLCHAIN_ROOT_DIR/sysroot
CROSS_PREFIX=$TOOLCHAIN/bin/aarch64-linux-android-
PREFIX=$(pwd)/android/$CPU
OPTIMIZE_CFLAGS="-march=$CPU"
build_android
#armv7-a
ARCH=arm
CPU=armv7-a
CC=$TOOLCHAIN/bin/armv7a-linux-androideabi$API-clang
CXX=$TOOLCHAIN/bin/armv7a-linux-androideabi$API-clang++
SYSROOT=$NDK/toolchains/llvm/prebuilt/$TOOLCHAIN_ROOT_DIR/sysroot
CROSS_PREFIX=$TOOLCHAIN/bin/arm-linux-androideabi-
PREFIX=$(pwd)/android/$CPU
OPTIMIZE_CFLAGS="-mfloat-abi=softfp -mfpu=vfp -marm - march=$CPU "
build_android
#x86
ARCH=x86
CPU=x86
CC=$TOOLCHAIN/bin/i686-linux-android$API-clang
CXX=$TOOLCHAIN/bin/i686-linux-android$API-clang++
SYSROOT=$NDK/toolchains/llvm/prebuilt/$TOOLCHAIN_ROOT_DIR/sysroot
CROSS_PREFIX=$TOOLCHAIN/bin/i686-linux-android-
PREFIX=$(pwd)/android/$CPU
OPTIMIZE_CFLAGS="-march=i686 -mtune=intel -mssse3 - mfpmath=sse -m32"
build_android
#x86_64
ARCH=x86_64
CPU=x86-64
CC=$TOOLCHAIN/bin/x86_64-linux-android$API-clang
CXX=$TOOLCHAIN/bin/x86_64-linux-android$API-clang++
SYSROOT=$NDK/toolchains/llvm/prebuilt/$TOOLCHAIN_ROOT_DIR/sysroot
CROSS_PREFIX=$TOOLCHAIN/bin/x86_64-linux-android-
PREFIX=$(pwd)/android/$CPU
OPTIMIZE_CFLAGS="-march=$CPU -msse4.2 -mpopcnt -m64 - mtune=intel"
build_android
```



```shell
#遇到的问题
1. ./configure: line 995: /android-ndk-r19c/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64/bin/arm-linux-androideabi-gcc: No such file or directory
C compiler test failed.
>从NDK19之后arm-linux-androideabi-gcc 被移除,所以要么NDK下载低版本要么在 ffmpeg/configure 中修改参数:
	1.CMDLINE_SET 新增cross_prefix_clang
	2. cc_default="${cross_prefix}${cc_default}" -> cc_default="${cross_prefix_clang}${cc_default}"
	3. cxx_default="${cross_prefix}${cxx_default}" -> cxx_default="${cross_prefix_clang}${cxx_default}"
	
也可以直接指定
CC=$TOOLCHAIN/bin/x86_64-linux-android$API-clang
CXX=$TOOLCHAIN/bin/x86_64-linux-android$API-clang++
CROSS_PREFIX=$TOOLCHAIN/bin/x86_64-linux-android-


2./root/android-ndk-r25b/toolchains/llvm/prebuilt/linux-x86_64/sysroot/usr/include/vulkan/vulkan.h:89:10: fatal error: 'vulkan_beta.h' file not found
#include "vulkan_beta.h"
         ^~~~~~~~~~~~~~~
>加入 --disable-vulkan \

3./ffbuild/pkgconfig_generate.sh:行21: libavfilter/libavfilter.version: 没有那个文件或目录
>make clean all -> make clean

4.STRIP	install-libavdevice-shared
strip: Unable to recognise the format of the input file `/root/ffmpeg/android/armv8-a/lib/libavdevice.so'
make: *** [ffbuild/library.mak:120：install-libavdevice-shared] 错误 1
> --disable-stripping \
ndk 25 已经没有strip文件了

4. /..//bin/ld: unrecognised emulation mode: armelf_linux_eabi
Supported emulations: elf_x86_64 elf32_x86_64 elf_i386 elf_iamcu i386linux elf_l1om elf_k1om i386pep i386pe


```



依赖顺序

```shell
# libraries, in any order
avcodec_deps="avutil"
avcodec_suggest="libm stdatomic"
avdevice_deps="avformat avcodec avutil"
avdevice_suggest="libm stdatomic"
avfilter_deps="avutil"
avfilter_suggest="libm stdatomic"
avformat_deps="avcodec avutil"
avformat_suggest="libm network zlib stdatomic"
avutil_suggest="clock_gettime ffnvcodec libm libdrm libmfx opencl user32 vaapi vulkan videotoolbox corefoundation corevideo coremedia bcrypt stdatomic"
postproc_deps="avutil gpl"
postproc_suggest="libm stdatomic"
swresample_deps="avutil"
swresample_suggest="libm libsoxr stdatomic"
swscale_deps="avutil"
swscale_suggest="libm stdatomic"

avcodec_extralibs="pthreads_extralibs iconv_extralibs dxva2_extralibs lcms2_extralibs"
avfilter_extralibs="pthreads_extralibs"
avutil_extralibs="d3d11va_extralibs nanosleep_extralibs pthreads_extralibs vaapi_drm_extralibs vaapi_x11_extralibs vdpau_x11_extralibs"

```

