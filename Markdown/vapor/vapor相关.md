1.安装vapor

[官方连接](https://github.com/NCAR/VAPOR/releases/tag/v3.9.1)

安装包下载连接

```sh
wget https://github.com/NCAR/VAPOR/releases/download/v3.9.1/VAPOR3-3.9.1-Ubuntu22.sh
```

进入安装包所在文件夹

```sh
cd /path/to/VAPOR3-3.9.1-Ubuntu22.sh
```

修改文件,添加可执行权限,然后执行

```sh
chmod 755 /path/to/VAPOR3-3.9.1-Ubuntu22.sh && /path/to/VAPOR3-3.9.1-Ubuntu22.sh
```

然后输入两次 y 确定安装和安装路径,然后安装 pt5 的相关lib 库

```sh
sudo apt install qt5*
```

Qt 5.10 使用 renameat2 系统调用，该调用仅在内核 3.15 之后可用,5.10以后的qt 需要内核3.15才能调,然后用strip那个命令 去掉一部分就好了

```sh
sudo strip --remove-section=.note.ABI-tag /path/to/VAPOR3-3.9.2-Linux/lib/libQt5Core.so.5
```

  然后暴露一下libQt5Gui.so.5 所在的路径就可以了

```sh
export LD_LIBRARY_PATH=/path/to/VAPOR3-3.9.2-Linux/lib 
```

- 2.vapor的一些指令
  
  使用 vapor 的指令结构

```sh
vapor [options] [session.vs3] [data files...]
```

       **vapor 指令参数极其相关参数**

|  OPTION                                   |NUM_ARGS                                       |                         DEFAULT|                           中文翻译|
|--|:--:|--|--|
|-ftype|1|Specify file format of datasets passed in command line. Valid options are: auto vdc wrf mpas dcp ugrid cf bov.defoult:auto|      指定命令行中传递的数据集的文件格式。有效选项有： auto,vdc,wrf,mpas,dcp,ugrid,cf,  bov.默认:auto|
|-help|0|Print this message and exit.defoult: false|打印此消息并退出。默认: false|
|-output|1|Output image file when using -render. This will be suffixed with the timestep number. Supports tiff, jpg.defoult:vapor.tiff|       使用-render时输出图像文件。这将带有时间步数的后缀。支持 tiff、jpg.默认:vapor.tiff|
|-render|0|Render a given session file and exit .defoult:false,render:Barb,Contour,Flow,lmage,isoSurface,Model,Particle,Slice,TwoDData,Volume,WireFrame|       渲染给定的会话文件并退出。默认: false,render有以下类型:Barb,Contour,Flow,lmage,isoSurface,Model,Particle,Slice,TwoDData,Volume,WireFrame|
|-resolution|1|Output resolution when using -render.defoult:1920x1080|使用 -render 时的输出分辨率。默认:1920x1080|
|-timesteps|1|Timesteps to render when using -render. Defaults to all timesteps. defoult:0:0|       使用 -render 时渲染的时间步长。默认为所有时间步长。默认:0:0|

        比如使用 vapor 打开一个 wrf 文件,指令如下:

```sh
./vapor -ftype wrf /var/hpc-root/zj/Downloads/wrfout_d01_2005-06-04_06_05_00.unknow
```
