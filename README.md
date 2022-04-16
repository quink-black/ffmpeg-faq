# FFmpeg FAQ

## 编译相关

### 怎样使用相同的参数，重新运行configure命令

* 背景：更新FFmpeg代码之后，有时候需要重新执行configure，否则可能出现编译错误

* 方法：

  * 自动的方法

    ```shell
    make config
    ```

  * 手动的方法

    有时候可能并不是要重新运行configure，而是想查看下当前用了什么配置，可以在build目录的config.h中查找：

    ```shell
    grep FFMPEG_CONFIGURATION config.h
    ```

### 怎样“不”自动查找第三方库？

* 背景：configure时FFmpeg会自动查找第三方库，编译出来的FFmpeg如果拷贝到其他机器去使用，可能因为各个机器安装的第三方库有差异，导致缺少第三方**动态库**而无法运行

* 方法：

  ```shell
  ./configure --disable-autodetect
  ```

### pkg-config相关

#### pkg-config的作用有哪些？

1. 和手动指定cflags/ldflags相比，可以简化操作

2. 减少错误

   * 使用静态链接时，pkg-config会给出需要链接的其他库，例如：

     ```
     $ pkg-config --static --libs libavformat
     -lavformat -lXv -lX11 -lXext -lSDL2 -lzmq -lwebp -lssh -lpulse -lopenmpt -ldrm -lbs2b -lbluray -lass -lgnutls -lSDL2 -lcrystalhd -lvdpau -lX11 -lva -lva-x11 -lX11 -lva -lva-drm -lva -lxcb -lxcb-shm -lxcb-xfixes -lxcb-shape -lcdio_paranoia -lcdio_cdda -lcdio -lsndio -ljack -lasound -lSDL2 -lGL -lopenal -lxml2 -lzvbi -lzmq -lxvidcore -lx265 -lx264 -lwebpmux -lwebp -lwavpack -lvpx -lm -lvpx -lm -lvpx -lm -lvpx -lm -lvorbisenc -lvorbis -ltwolame -ltheoraenc -ltheoradec -logg -lspeex -lssh -lsoxr -lsnappy -lm -lshine -lrubberband -lrsvg-2 -lm -lgio-2.0 -lgdk_pixbuf-2.0 -lgobject-2.0 -lglib-2.0 -lcairo -lpulse -lopus -lopenmpt -lopenjp2 -lopencv_core -lopencv_imgproc -lmysofa -lmp3lame -lgsm -lgme -lstdc++ -lfribidi -lfreetype -lfontconfig -lfreetype -lflite_cmu_time_awb -lflite_cmu_us_awb -lflite_cmu_us_kal -lflite_cmu_us_kal16 -lflite_cmu_us_rms -lflite_cmu_us_slt -lflite_usenglish -lflite_cmulex -lflite -ldrm -ldc1394 -lcaca -lbs2b -lbluray -lass -lraw1394 -lavc1394 -lrom1394 -liec61883 -lgnutls -lchromaprint -lm -ldl -llzma -lbz2 -lz -pthread -lavcodec -lXv -lX11 -lXext -lSDL2 -lzmq -lwebp -lssh -lpulse -lopenmpt -ldrm -lbs2b -lbluray -lass -lgnutls -lSDL2 -lcrystalhd -lvdpau -lX11 -lva -lva-x11 -lX11 -lva -lva-drm -lva -lxcb -lxcb-shm -lxcb-xfixes -lxcb-shape -lcdio_paranoia -lcdio_cdda -lcdio -lsndio -ljack -lasound -lSDL2 -lGL -lopenal -lxml2 -lzvbi -lzmq -lxvidcore -lx265 -lx264 -lwebpmux -lwebp -lwavpack -lvpx -lm -lvpx -lm -lvpx -lm -lvpx -lm -lvorbisenc -lvorbis -ltwolame -ltheoraenc -ltheoradec -logg -lspeex -lssh -lsoxr -lsnappy -lm -lshine -lrubberband -lrsvg-2 -lm -lgio-2.0 -lgdk_pixbuf-2.0 -lgobject-2.0 -lglib-2.0 -lcairo -lpulse -lopus -lopenmpt -lopenjp2 -lopencv_core -lopencv_imgproc -lmysofa -lmp3lame -lgsm -lgme -lstdc++ -lfribidi -lfreetype -lfontconfig -lfreetype -lflite_cmu_time_awb -lflite_cmu_us_awb -lflite_cmu_us_kal -lflite_cmu_us_kal16 -lflite_cmu_us_rms -lflite_cmu_us_slt -lflite_usenglish -lflite_cmulex -lflite -ldrm -ldc1394 -lcaca -lbs2b -lbluray -lass -lraw1394 -lavc1394 -lrom1394 -liec61883 -lgnutls -lchromaprint -lm -ldl -llzma -lbz2 -lz -pthread -lswresample -lm -lsoxr -lavutil -ldrm -lm
     ```

   * 解决链接顺序问题。链接时越基础的、被其他库依赖的库应放在后面，如avcodec在avformat之后，avutil在avcodec之后。虽然还有其他解决链接顺序的手段，pkg-config是比较简单的一种，让FFmpeg自己来告诉你正确的链接顺序。

3. 版本检查

   configure里可能通过pkg-config去检查第三方库的版本。手动调用pkg-config检查库版本的方式：

   ```
   $ pkg-config --exists --print-errors 'srt > 1.4.5'
   Requested 'srt > 1.4.5' but version of srt is 1.4.3
   ```

虽然pkg-config的结果最终还是作用于cflags/ldflags等等，但它代表着软件开发的一种思想：通过增加一层抽象来解耦。

#### pkg-config pc文件不在标准路径下怎么办？

```
export PKG_CONFIG_PATH=$path_to_pkg_config_pc_dir
```

#### pc文件里的路径和实际路径不对应怎么办？

* 背景：在一台机器上编译release一个库，生成.pc文件，里面包含了当时的install路径。把release的库拿到其他机器上去用，路径不匹配

* 方法：

  > --define-prefix                         try to override the value of prefix for each .pc file found with a guesstimated value based on the location of the .pc file

  ```
  pkg-config --cflags --define-prefix libfoo
  ```

  具体到FFmpeg，可以

  ```
  configure --pkg-config-flags="--define-prefix"
  ```

  注意直接传给FFmpeg configure有风险，会影响所有库的pkg-config执行。特定库需要修改pc文件中的路径，可以把`pkg-config --define-prefix --cflags libfoo` `pkg-config --define-prefix --libs libfoo`的结果传给FFmpeg configure。

### 如何编译无依赖的二进制文件

1- 保证所有的依赖库都有静态库(.a) 的版本，否则会出现链接错误，包括c++ 的静态库。
如何编译或安装依赖库的静态版本需要自行根据依赖库编译说明或所在平台的安装说明。

```sh
# Centos 安装c++ 静态库
sudo yum install libstdc++-static
```

2- configure 指定静态编译相关选项

```sh
  --extra-cflags="-static" \
  --extra-ldflags="-static" \        # 静态链接二进制 ffmpeg
  --pkg-config-flags="--static" \    # pkgconf 使用静态库
```

3. 确认是否是无依赖的二进制

```sh
ldd ffmpeg
# 如果输出 not a dynamic executable 则表示二进制无依赖
```

### 如何编译 `doc/example` 目录下的例子

```sh
make -j8 examples
```

## avutil相关

### av_freep

#### av_freep的作用

* 防止use after free
* 防止double free

简要说明：

1. 如果不清楚这两个概念，建议重学C语言；

2. 如果不清楚什么场合该用av_free，什么场合用av_freep，建议使用av_freep()；

3. 如果清楚什么场合适用哪一个，更佳；
4. 如果能从代码结构设计、生命周期管理上，减少用av_freep()的场合，最佳。

#### av_freep的代码实现为何如此奇怪？

```c
void av_freep(void *arg)
{
    void *val;

    memcpy(&val, arg, sizeof(val));
    memcpy(arg, &(void *){ NULL }, sizeof(val));
    av_free(val);
}
```

1. 参数为什么是`void *`，而不是`void **`?

   因为任意指针可以与`void *`来回转换，而任意指针强转为`void **`是非法的，例如

   ```c
       int *p = malloc(sizeof(*p) * 123);
       void **q = &p;
   ```

   > warning: incompatible pointer types initializing 'void **' with an expression of type 'int **' [-Wincompatible-pointer-types]
   
   **`void **`只能是`void*`的指针**。
   
2. 为什么有两个`memcpy`，为什么不用下面的简单形式：

   ```c
   void av_freep(void *arg)
   {
       void **ptr = (void **)arg;
       av_free(*ptr);
       *ptr = NULL;
   }
   ```
   只有当arg实际为`void **`转成的`void *`时，`void **ptr = (void **)arg;`才是合法的；即不能把其他类型如`int **`转成
   
   `void *`，再把`void *`转成`void **`。背后的规则是**strict aliasing rule **[wiki aliasing](https://en.wikipedia.org/wiki/Aliasing_(computing))。
   
   两个memcpy的实现形式，**没有假设`arg`是`void **`，只假设`arg`是任意指针的指针**。
   
   当前的实现解决了aliasing violations，但它仍有一个标准之外的假设：运行的CPU架构所有指针类型用了相同的表达形式。过去存在不同指针类型用不同表达形式的CPU架构，见[comp.lang.c FAQ list · Question 5.17](http://c-faq.com/null/machexamp.html)。
   
3. 关于strict aliasing

   strict aliasing可以让编译器做激进的优化，但对违反strict aliasing rule的代码会造成特别隐蔽的bug。所以，写代码严格遵守规则，而开启`-fstrict-aliasing`要谨慎。

   深入分析讨论见：[What is the Strict Aliasing Rule and Why do we care?](https://gist.github.com/shafik/848ae25ee209f698763cffee272a58f8)
## avcodec编解码相关

## avformat相关

### H.265编码MP4封装，macOS/iOS上无法播放

例如：

```
ffmpeg -i ~/Movies/hevc-6fps.mp4 -c copy /tmp/test.mp4
```

用QuickTime player打开/tmp/test.mp4报错。

方法：加上`-tag:v hvc1`

```
ffmpeg -i ~/Movies/hevc-6fps.mp4 -c copy -tag:v hvc1 /tmp/test.mp4
```

原因：

Apple要求mp4中的codec tag必须是hvc1:

```
                        [stsd: Sample Description Box]
                            position = 1184036
                            size = 2269
                            version = 0
                            flags = 0x000000
                            entry_count = 1
                            [hvc1: Visual Description]
                                position = 1184052
                                size = 2253
```

[apple developer](https://developer.apple.com/documentation/http_live_streaming/http_live_streaming_hls_authoring_specification_for_apple_devices?language=objc)

> 1.10. You SHOULD use video formats in which the parameter sets are stored in the sample descriptions, rather than the samples. (That is, use `'avc1'`, `'hvc1'`, or `'dvh1'` rather than `'avc3'`, `'hev1'`, or `'dvhe'`.)

关于参数集in-band/out-of-band的差异，见

[H.264参数集处理](https://gist.github.com/quink-black/6828ebf722f6a4d35fbc5c5bc2dbaf42)



### 如何获取视频最后一帧

```sh
ffmpeg -sseof -0.5 -i foo.mp4 -an -update 1 bar.png
```

* `-sseof`，seek参考点为结尾
* `-update 1`，不断覆盖同一个图片文件

## Filter相关

