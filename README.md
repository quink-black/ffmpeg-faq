# FFmpeg FAQ

[toc]

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



## avcodec编解码相关



## avformat相关

#### H.265编码MP4封装，macOS/iOS上无法播放

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

## Filter相关