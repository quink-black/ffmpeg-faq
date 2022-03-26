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



## avcodec编解码相关



## avformat相关



## Filter相关