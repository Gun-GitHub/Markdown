### Usage: 

```
    xpra 

    xpra start [DISPLAY]

    xpra start-desktop [DISPLAY]

    xpra start-monitor [DISPLAY]

    xpra shadow [DISPLAY]

    xpra shadow-screen [DISPLAY]

    xpra upgrade [DISPLAY]

    xpra upgrade-desktop [DISPLAY]

    xpra upgrade-monitor [DISPLAY]

    xpra upgrade-shadow [DISPLAY]

    xpra attach [DISPLAY]

    xpra detach [DISPLAY]

    xpra info [DISPLAY]

    xpra id [DISPLAY]

    xpra version [DISPLAY]

    xpra stop [DISPLAY]

    xpra exit [DISPLAY]

    xpra clean [DISPLAY1] [DISPLAY2]..

    xpra clean-sockets [DISPLAY]

    xpra clean-displays [DISPLAY]

    xpra screenshot filename [DISPLAY]

    xpra control DISPLAY command [arg1] [arg2]..

    xpra print DISPLAY filename

    xpra shell [DISPLAY]

    xpra showconfig

    xpra list

    xpra list-sessions

    xpra list-windows

    xpra sessions

    xpra launcher

    xpra gui

    xpra start-gui

    xpra bug-report

    xpra toolbox

    xpra displays

    xpra docs

    xpra html5

    xpra autostart

    xpra encoding

    xpra example

    xpra path-info

    xpra list-mdns

    xpra mdns-gui
```

## Options:

####   --version             

```text
显示程序的版本号并退出
```

####   -h, --help            

```text
显示此帮助消息并退出
```

####   --tray=yes|no         

```text
启用 Xpra 自己的系统托盘菜单。Default: yes
```

####   --delay-tray=yes|no   

```text
在显示系统托盘之前等待第一个事件。Default: no
```

## Server Options:

####     这些选项仅在启动或升级服务器时相关。

####     --start=CMD         

```text
在服务中调起程序(可能重复调起)。Default:none.
```

####     --start-late=CMD

```text
服务初始化完成后在中调起的程序（可能会重复）。Default:none.
```

####     --start-child=CMD   

```text
在服务中调起程序, 可以连带选项 exit-with-children (可能重复运行多个命令)。Default: none.
```

####     --start-child-late=CMD

```text
服务初始化完成后在中调起的程序,可以连带选项 exit-with-children (可能重复运行多个命令)。Default: none.
```

####     --start-after-connect=START_AFTER_CONNECT

```text
当第一个客户端连接服务之后,服务调起程序(可能会重复)。Default: none.
```

####     --start-child-after-connect=START_CHILD_AFTER_CONNECT

```text
在第一次客户端连接服务的时候,服务启动程序,可以连带选项 exit-with-children(可能重复运行多个命令).Default: none.
```

####     --start-on-connect=START_ON_CONNECT

```text
服务每次客户连接的时候启动程序(可能重复).Default: none.
```

####     --start-child-on-connect=START_CHILD_ON_CONNECT

```text
服务每次客户连接的时候启动程序,可以连带选项 exit-with-children(可能重复).Default: none.
```

####     --start-on-last-client-exit=START_ON_LAST_CLIENT_EXIT

```sh
服务会在每次客户端断开连接并且没有其他的客户端离开的时候启动程序(可能重复).Default: none.
```

####     --start-child-on-last-client-exit=START_CHILD_ON_LAST_CLIENT_EXIT

```sh
服务会在每次客户端断开连接并且没有其他的客户端离开的时候启动程序,可以连带选项 exit-with-children(可能重复). Default: none.
```

####     --exec-wrapper=CMD  

```sh
用于执行命令的包装器。Default: none.
```

####     --terminate-children=yes|no

```sh
停止服务时停止所有的子命令. Default: False
```

####     --exit-with-children=yes|no

```sh
停止服务当 --start-child 的程序退出时
```

####     --exit-with-windows=yes|no

```sh
当最后一个窗口消失时终止服务器
```

####     --start-new-commands=yes|no

```sh
允许客户端在服务器上执行新命令。Default: yes.
```

####     --start-via-proxy=yes|no|auto

```sh
通过系统代理服务器启动服务器。Default:False.
```

####     --proxy-start-sessions=yes|no

```sh
允许代理服务器根据需要启动新会话。Default: yes.
```

####     --dbus-launch=CMD   

```sh
 在dbus启动上下文中启动会话，保留为空以关闭。Default: 'dbus-launch --sh-syntax --close-stderr'.
```

####     --source=SOURCE      

```sh
要加载到服务环境变量的配置文件。Default:'/etc/profile'.
```

####     --source-start=SOURCE_START

```sh
脚本加载环境变量给启动指令.Default: none.
```

####     --start-env=START_ENV

```text
定义被用于'start-child'和'start'的环境变量,能被多次定义. Default: 'UBUNTU_MENUPROXY=', 'QT_X11_NO_NATIVE_MENUBAR=1','MWNOCAPTURE=true', 'MWNO_RIT=true', 'MWWM=allwm', 'GDK_BACKEND=x11', 'QT_QPA_PLATFORM=xcb', 'QT_SCALE_FACTOR=1', 'GTK_CSD=0'.
```

####     --systemd-run=yes|no|auto

```text
使用 systemd-run 包装服务器启动命令。Default:auto.
```

####     --systemd-run-args=ARGS

```text
传递给 systemd-run 的命令行参数。Default: ''.
```

####     --http-scripts=off|all|SCRIPTS

```text
启用内置 Web 服务器脚本。Default: 'all'.
```

####     --html=on|off|[HOST:]PORT

```text
启用 Web 服务器和 html5 客户端。 Default:'auto'.
```

####     --uid=UID           

```text
当服务器由 root 启动时要更改为的用户 ID。 Default: 0.
```

####     --gid=GID

```text
当服务器由 root 启动时要更改为的组 ID。 Default: 0.
```

####     --daemon=yes|no

```text
当作为服务运行时守护进程化 (default: yes)
```

####     --chdir=DIR         

```text
修改当前工作路径 (default: no)
```

####     --pidfile=PIDFILE   

```text
将进程id 写到这个文件中去(default:'${XPRA_SESSION_DIR}/server.pid')
```

####     --log-dir=LOG_DIR 

```text
放置日志文件的目录
```

####     --log-file=LOG_FILE

```text
守护进程化时，日志消息将发送到这里。Default: 'server.log'如果相对文件名指定为相对于 --log-dir，则 '$DISPLAY' 的值将替换为实际使用的显示
```

####     --attach=yes|no|auto

```text
服务启动后立即连接客户端(default: auto)
```

####     --file-transfer=yes|no|ask

```text
支持文件传输。Default: yes.
```

####     --open-files=yes|no|ask

```text
自动打开上传的文件(有潜在危险). Default: 'auto'.
```

####     --open-url=yes|no|ask

```text
自动打开url连接(有潜在风险)Default: 'auto'.
```

####     --printing=yes|no|ask

```text
支持打印。Default: yes.
```

####     --file-size-limit=SIZE

```text
最大文件传输的大小. Default: '1G'.
```

####     --lpadmin=COMMAND   

```text
指定要使用的 lpadmin 命令。 Default:'/usr/sbin/lpadmin'./usr/sbin/lpadmin 是用于管理打印机和打印队列的命令行工具，通常在 Unix/Linux 系统中使用。通过 lpadmin 命令，可以添加、删除、修改打印机或打印队列，以及配置打印机的参数设置。一般情况下，只有系统管理员或具有特定权限的用户可以使用 lpadmin 命令来管理打印机。常见的用法包括添加新的打印机、设置默认打印机、修改打印机属性等操作。
```

####     --lpinfo=COMMAND    

```text
指定要使用的 lpinfo 命令。Default:'/usr/sbin/lpinfo'./usr/sbin/lpinfo 是一个用于查询打印机信息的命令行工具，通常在 Unix/Linux 系统中使用。通过 lpinfo 命令，可以列出系统中已安装的打印机及其相关属性，如打印机名称、位置、支持的打印格式等。这个命令对于了解系统中可用的打印机和打印服务的配置信息非常有用。可以通过查询打印机列表及其属性来进行打印机管理和配置工作。
```

####     --exit-with-client=yes|no

```text
当最后一个客户端断开连接的时候关闭服务.Default: no
```

####     --idle-timeout=IDLE_TIMEOUT

```text
客户端空闲的时候断开客户端连接(设置0为禁用).Default: 0 seconds
```

####     --server-idle-timeout=SERVER_IDLE_TIMEOUT

```text
退出服务当服务端空闲的时候.(设置0为禁用). Default: 0 seconds
```

####     --use-display=yes|no|auto

```text
使用现有的显示，而不是使用 xvfb 命令启动显示。Default: yes
```

####     --xvfb=CMD          

```sh
如何运行无显示服务器
Default: 
  Xvfb +extension GLX 
       +extension Composite 
       -screen 0 8192x4096x24+32 
       -nolisten tcp 
       -noreset 
       -auth $XAUTHORITY
       -dpi 96x96'

```

####     --displayfd=FD      

```text
xpra 服务器会将显示编号作为换行符终止的字符串写回到该文件描述符上。
```

####     --fake-xinerama=path|auto|no

```text
为会话设置虚拟 xinerama 支持。 你可以指定 libfakeXinerama.so 库的路径，或者一个布尔值。Default: yes.
```

```text
Xinerama 是一个由 X.Org Foundation 开发的 X 服务器扩展，用于支持多个物理显示器的单个 X 屏幕。它允许用户在多个显示器上创建一个连续的虚拟桌面，并使窗口管理器能够管理跨越多个显示器的窗口。这使得用户能够更自由地在多个显示器之间移动窗口，并在它们之间进行工作。 Xinerama 最初是为了解决多显示器设置中的窗口管理问题而设计的。
```

#### --resize-display=yes|no|widthxheight[@HZ]

```text
是否应调整服务器显示大小以匹配客户端分辨率。默认值：是。Default: yes.
```

####     --bind=SOCKET       

```text
监听Unix域套接字上的连接。您可以多次指定此选项以不同的方式监听。Default: auto
```

####     --bind-tcp=[HOST]:[PORT]

```text
侦听 TCP 上的连接。使用--tcp-auth鉴权. 您可以使用不同的主机和端口组合多次指定此选项。
```

####     --bind-ws=[HOST]:[PORT]

```text
监听 WebSocket 上的连接。使用 --ws-auth 鉴权。您可以多次指定此选项，使用不同的主机和端口组合。
```

####     --bind-wss=[HOST]:[PORT]

```text
监听通过 HTTPS / wss（安全 WebSocket）进行的连接。使用 --wss-auth 来保障安全性。您可以多次指定此选项，使用不同的主机和端口组合。
```

####     --bind-ssl=[HOST]:PORT

```text
监听 SSL 连接。使用 --ssl-auth 来确保安全性。您可以多次指定此选项，使用不同的主机和端口组合
```

####     --bind-ssh=[HOST]:PORT

```text
监听使用 SSH 传输的连接。使用 --ssh-auth 来确保安全性。您可以多次指定此选项，使用不同的主机和端口组合。
```

####     --bind-rfb=[HOST]:PORT

```text
监听 RFB 连接。使用 --rfb-auth 来确保安全性。您可以多次指定此选项，使用不同的主机和端口组合。
```

####     --bind-quic=[HOST]:PORT

```text
监听 QUIC HTTP/3 或 WebTransport 连接。使用 --quic-auth 来确保安全性。您可以多次指定此选项，使用不同的主机和端口组合。
```

####     --bind-vsock=[CID]:[PORT]

```text
监听 VSOCK 连接。您可以多次指定此选项，使用不同的 CID 和端口组合。
```

####     --mdns=yes|no       

```text
通过 mDNS 发布会话信息。Default:yes.
```

####     --dbus-proxy=yes|no

```text
转发来自客户端的 DBus 调用。 Default: yes.
```

####     --dbus-control=yes|no

```text
允许通过其 DBus 接口控制服务器。Default: yes.
```

##   Server Controlled Features:

  ###       这些选项可以在客户端或服务器上指定，但服务器的设置将优先于客户端的设置。

####     --bandwidth-limit=BANDWIDTH_LIMIT

```text
限制使用的带宽。该值以每秒比特数指定，使用值 '0' 可以禁用限制。默认值为 'auto'。
```

####     --bandwidth-detection=BANDWIDTH_DETECTION

```text
自动检测运行时的带宽限制。 Default: 'False'.
```

####     --readonly=yes|no   

```text
禁用来自客户端的键盘输入和鼠标事件。  Default: no.
```

####     --clipboard=yes|no|clipboard-type

```text
启用剪贴板支持。Default: yes.
```

####     --clipboard-direction=to-server|to-client|both

```text
剪贴板同步的方向。 Default: both.
```

####     --notifications=yes|no

```text
转发系统通知。Default: yes.
```

####     --system-tray=yes|no

```text
转发系统托盘图标。Default: yes.
```

####     --cursors=yes|no    

```text
转发自定义应用程序鼠标指针。Default: yes.
```

####     --bell=yes|no       

```text
转发系统响铃。Default: yes.
```

####     --webcam=WEBCAM     

```text
摄像头转发，可以用于指定设备。Default: auto.
```

####     --mousewheel=MOUSEWHEEL

```text
鼠标滚轮前进，可用于禁用设备（“no”）或反转某些轴（“invert-all”、“invert-x”、“invert-y”、“invert-z”）。Default: on.
```

####     --input-devices=APINAME

```text
用于输入设备的API。Default: auto.
```

####     --xsettings=auto|yes|no

```text
Xsettings 同步。Default: yes.
```

####     --mmap=yes|no|mmap-filename

```text
使用内存映射传输进行本地连接。Default: yes.
```

####     --sharing=yes|no    

```text
允许多个客户端连接到同一会话。Default: auto.
```

####     --lock=yes|no       

```text
防止会话被新客户端接管。Default: auto.
```

####     --remote-logging=no|send|receive|both

```text
将客户端的所有日志输出转发到服务器。Default: yes.
```

##   Audio Options:

###     这些选项可以在客户端或服务器上指定，但服务器的设置将优先于客户端的设置。

####     --audio=yes|no      

```text
Enable or disable all audio support. Default: yes.
```

####     --pulseaudio=yes|no|auto

```text
为会话启动 PulseAudio 服务器。Default:auto.
```

####     --pulseaudio-command=PULSEAUDIO_COMMAND

```sh
启动 PulseAudio 服务器的命令。
  Default: 
    pulseaudio \
    --start \
    -n \
    --daemonize=false \
    --system=false \
    --exit-idle-time=-1 \
    --load=module-suspend-on-idle \
    --load=module-null-sink sink_name=Xpra-Speaker sink_properties=device.description="Xpra Speaker" \
    --load=module-null-sink sink_name=Xpra-Microphone sink_properties=device.description="Xpra Microphone" \
    --load=module-remap-source source_name=Xpra-Mic-Source source_properties=device.description="Xpra Mic Source" master=Xpra-Microphone.monitor channels=1 \
    --load=module-native-protocol-unix socket=$XPRA_PULSE_SERVER \
    --load=module-dbus-protocol \
    --load=module-x11-publish \
    --log-level=2 \
    --log-target=stderr \
    --enable-memfd=no

```

####     --pulseaudio-configure-commands=PULSEAUDIO_CONFIGURE_COMMANDS

```sh
配置 PulseAudio 服务器所使用的命令。
Default:
  [
    'pactl set-default-sink Xpra-Speaker',
    'pactl set-default-source Xpra-Mic-Source'
  ]


```

####     --speaker=on|off|disabled

```sh
将音频输出转发到客户端。Default: on.
```

####     --speaker-codec=SPEAKER_CODEC

```sh
指定用于转发扬声器音频输出的编解码器。此参数可以多次指定，指定编解码器的顺序定义了首选编解码器顺序。使用特殊值 'help' 可以获取选项列表。当未指定时，允许使用所有可用的编解码器，并使用第一个编解码器。
```

####     --microphone=on|off|disabled

```sh
将音频输入转发到服务器。Default: off.
```

####     --microphone-codec=MICROPHONE_CODEC

```sh
指定用于转发麦克风音频输出的编解码器。此参数可以多次指定，指定编解码器的顺序定义了首选编解码器顺序。使用特殊值 'help' 可以获取选项列表。当未指定时，允许使用所有可用的编解码器，并使用第一个编解码器。
```

####     --audio-source=AUDIO_SOURCE

```sh
指定用于捕获音频流的音频系统（使用 'help' 获取选项）。
```

####     --av-sync=AV_SYNC  

```sh
尝试同步音频和视频。默认值：是。 Default: yes.
```

##   Encoding and Compression Options:

###       这些选项由客户端用于指定所需的图片和网络数据压缩。它们也可以在服务器上作为默认设置来指定。

####     --encodings=ENCODINGS

```sh
指定允许的编码方式。 Default: all.
```

####     --encoding=ENCODING

```sh
要使用的图像压缩算法，请指定 'help' 以获取选项列表。Default: auto.
```

####     --video-encoders=VIDEO_ENCODERS

```sh
指定要启用的视频编码器，请指定 'help' 以获取所有选项的列表。
```

####     --proxy-video-encoders=PROXY_VIDEO_ENCODERS

```sh
在运行代理服务器时，指定要启用的视频编码器，请指定 'help' 以获取所有选项的列表。
```

####     --csc-modules=CSC_MODULES

```sh
指定要启用的颜色空间转换模块，请指定 'help' 以获取所有选项的列表。Default: all.
```

####     --video-decoders=VIDEO_DECODERS

```sh
指定要启用的视频解码器，请指定 'help' 以获取所有选项的列表。
```

####     --video-scaling=SCALING

```sh
应该使用多少自动视频降低分辨率，从 1（很少）到 100（积极），0 为禁用。Default: auto.
```

####     --min-quality=MIN-LEVEL

```sh
设置自动质量设置中允许的最低编码质量，从 1 到 100，0 为未设置。Default: 1.
```

####     --quality=LEVEL    

```sh
使用固定的图像压缩质量 - 仅适用于有损编码，从 1 到 100，0 表示使用自动设置。Default: 0.
```

####     --min-speed=SPEED   

```sh
 设置自动速度设置中允许的最低编码速度，从 1 到 100，0 表示未设置。Default: 1.
```

####     --speed=SPEED      

```sh
使用给定的编码速度进行图像压缩，从 1 到 100，0 表示使用自动设置。Default: 0.
```

####     --auto-refresh-delay=DELAY

```sh
自动进行无损刷新之前的空闲延迟，以秒为单位。0.0 表示禁用。Default: 0.15.
```

####     --compressors=COMPRESSORS

```sh
要启用的数据包压缩器Default: none, lz4, zlib, brotli.
```

####     --packet-encoders=PACKET_ENCODERS

```sh
要启用的数据包编码器。Default: rencodeplus,rencode, bencode, yaml.
```

####     -z LEVEL, --compression_level=LEVEL

```sh
在压缩数据包数据时使用的压缩程度。通常情况下，您不需要使用此选项，默认值应该足够好，图片数据会单独进行压缩（参见 --encoding）。0 表示禁用压缩，9 表示最大（最慢）压缩。默认值为 1。
```

##   Client Features Options:

###       这些选项控制影响外观或键盘的客户端功能。

####     --reconnect=yes|no  

```sh
重新连接到服务器. Default: yes.
```

####     --opengl=(yes|no|auto)[:backends]

```sh
使用 OpenGL 加速渲染。Default: probe.
```

####     --splash=yes|no|auto

```sh
在加载客户端时显示启动画面。 Default: auto.
```

####     --headerbar=auto|no|force

```sh
为装饰窗口添加带有菜单的标题栏。Default: auto.
```

####     --windows=yes|no    

```sh
转发窗口Default: yes.
```

####     --session-name=SESSION_NAME

```sh
此会话的名称，可用于通知、菜单等。 Default: 'Xpra'.
```

####     --min-size=MIN_SIZE

```sh
普通装饰窗口的最小大小，即 100x20。Default: none.
```

####     --max-size=MAX_SIZE

```sh
普通窗口的最大大小，即 800x600。 Default: none.
```

####     --refresh-rate=VREFRESH

```sh
要使用的垂直刷新率，即每秒的目标帧数。此值可以以绝对形式指定："50"，也可以以检测到的值的百分比形式指定："50%"。Default: 'auto'.
```

####     --desktop-scaling=SCALING

```sh
要缩放客户端桌面的比例。此值可以以绝对像素形式指定："WIDTHxHEIGHT"，也可以以分数形式指定："3/2"，或者只是一个小数："1.5"。您还可以单独指定每个维度："2x1.5"。Default: 'on'.
```

####     --desktop-fullscreen=DESKTOP_FULLSCREEN

```sh
如果窗口来自桌面或阴影服务器，则将其设置为全屏，并按比例缩放以适应屏幕。Default:：'False'。
```

####     --border=BORDER     

```sh
绘制在 xpra 窗口内部的边框，以区分它们与本地窗口。格式为：color[,size]。Default:：'auto,5:off'。
```

####     --title=TITLE       

```sh
作为窗口标题显示的文本，可以使用远程元数据变量。默认值为：'@title@ on @hostinfo@'。
```

####     --window-close=WINDOW_CLOSE

```sh
当客户端关闭窗口时采取的操作。有效选项为：'forward'（转发）、'ignore'（忽略）、'disconnect'（断开连接）。Default：'auto'。
```

####     --window-icon=WINDOW_ICON

```sh
默认图像的路径，该图像将用于所有窗口（应用程序可以覆盖此设置）。
```

####     --tray-icon=TRAY_ICON

```sh
将用作的图标的图像的路径系统路径或docker
```

####     --shortcut-modifiers=SHORTCUT_MODIFIERS

```sh
键盘快捷键所需的默认修饰键集合。Default auto.
```

####     --key-shortcut=KEY_SHORTCUT

```sh
定义触发特定操作的键盘快捷键。如果未定义快捷键，则默认为：
    Control+Menu:toggle_keyboard_grab
    Control+Menu:toggle_keyboard_grab
    Shift+Menu:toggle_pointer_grab
    Shift+F11:toggle_fullscreen  
    #+F1:show_menu
    #+F2:show_start_new_command  
    #+F3:show_bug_report
    #+F4:quit  
    #+F5:show_window_info  
    #+F6:show_shortcuts
    #+F7:show_docs  
    #+F8:toggle_keyboard_grab
    #+F9:toggle_pointer_grab  
    #+F10:magic_key
    #+F11:show_session_info  
    #+F12:toggle_debug
    #+plus:scaleup  
    #+minus:scaledown
    #+underscore:scaledown  
    #+KP_Add:scaleup
    #+KP_Subtract:scaledown  
    #+KP_Multiply:scalereset
    #+bar:scalereset  
    #+question:scalingoff

```

####     --keyboard-sync=yes|no

```sh
同步键盘状态.Default: yes.
```

####     --keyboard-raw=yes|no

```sh
发送原始键盘按键码。Default: no.
```

####     --keyboard-layout=LAYOUT

```sh
要使用的键盘布局。Default: none.
```

####     --keyboard-layouts=LAYOUTS

```sh
要启用的键盘布局。Default: none.
```

####     --keyboard-variant=VARIANT

```sh
要使用的键盘布局变体。Default: none.
```

####     --keyboard-variants=VARIANTS

```sh
要启用的键盘布局变体。 Default: none.
```

####     --keyboard-options=OPTIONS

```sh
要使用的键盘布局选项。Default: none.
```

##   SSL Options:

###       这些选项适用于客户端和服务器。请参考手册页以获取详细信息。

### --ssl=SSL

```sh
是否在 TCP 套接字上启用 SSL，以及用于什么目的（需要 'ssl-cert'）。默认值为 'yes'。
```

### --ssl-key=SSL_KEY

```sh
要使用的密钥文件。默认值为 ''。
```

### --ssl-key-password=SSL_KEY_PASSWORD

```sh
用于解密密钥文件的密码。默认值为 ''。
```

### --ssl-cert=SSL_CERT

```sh
要使用的证书文件。默认值为 ''。
```

### --ssl-protocol=SSL_PROTOCOL

```sh
指定要使用的 SSL 协议版本。默认值为 'TLSv1_2'。
```

### --ssl-ca-certs=SSL_CA_CERTS

```sh
ca_certs 文件包含一组连接的“认证机构”证书，或者您可以将其设置为包含 CA 文件的目录。默认值为 'default'。
```

### --ssl-ca-data=SSL_CA_DATA

```sh
PEM 或 DER 编码的证书数据，可选择转换为十六进制。默认值为 ''。
```

### --ssl-ciphers=SSL_CIPHERS

```sh
设置可用的密码，它应该是 OpenSSL 密码列表格式中的字符串。默认值为 'DEFAULT'。
```

### --ssl-client-verify-mode=SSL_CLIENT_VERIFY_MODE

```sh
是否尝试验证客户端的证书以及如果验证失败时的行为方式。默认值为 'optional'。
```

### --ssl-server-verify-mode=SSL_SERVER_VERIFY_MODE

```sh
是否尝试验证服务器的证书以及如果验证失败时的行为方式。默认值为 'required'。
```

### --ssl-verify-flags=SSL_VERIFY_FLAGS

```sh
用于证书验证操作的标志。默认值为 'X509_STRICT'。
```

### --ssl-check-hostname=yes|no

```sh
是否匹配对等方证书的主机名或接受任何主机，有风险。默认值为 'yes'。
```

### --ssl-server-hostname=hostname

```sh
要匹配的服务器主机名。默认值为 ''。
```

### --ssl-options=options

```sh
在此上下文中启用的一组 SSL 选项。默认值为 'ALL,NO_COMPRESSION'。
```

##  Advanced Options:

###       这些选项适用于客户端和服务器。请参考手册页以获取详细信息。

### --env=ENV

```sh
定义将应用于此进程和所有子进程的环境变量，可以多次指定。默认值为 'NO_AT_BRIDGE=1'。
```

### --challenge-handlers=CHALLENGE_HANDLERS

```sh
用于处理服务器身份验证挑战的处理程序。默认为所有。
```

### --password-file=PASSWORD_FILE

```sh
包含连接所需密码的文件（用于保护 TCP 模式）。默认值为无。
```

### --forward-xdg-open=FORWARD_XDG_OPEN

```sh
拦截对 xdg-open 的调用并将其转发到客户端。默认值为 'none'。
```

### --open-command=OPEN_COMMAND

```sh
用于打开文件和 URL 的命令。默认值为 '/usr/bin/xdg-open'。
```

### --modal-windows=MODAL_WINDOWS

```sh
尊重模态窗口。默认值为 'False'。
```

### --input-method=INPUT_METHOD

```sh
要为使用 start 或 start-child 启动的客户端应用程序配置的 X11 输入方法（默认值为 'auto'，选项为 auto、none、keep、xim、IBus、SCIM、uim）。
```

### --dpi=DPI

```sh
客户端应用程序应尝试遵循的 '每英寸点数' 值，范围为 10 到 1000，或自动设置为 0。默认值为 auto。
```

### --pixel-depth=PIXEL_DEPTH

```sh
在启动服务器时使用的虚拟帧缓冲区的 '每像素位数'（8、16、24 或 30），或在启动客户端时用于渲染的值。默认值为 0（自动）。
```

### --sync-xvfb=SYNC_XVFB

```sh
用于 X11 无缝服务器的虚拟帧缓冲区的同步频率（0 禁用）。默认值为 0。
```

### --client-socket-dirs=CLIENT_SOCKET_DIRS

```sh
客户端创建其控制套接字的目录。默认值为 '/run/user/$UID/xpra/clients'。
```

### --socket-dirs=SOCKET_DIRS

```sh
查找套接字文件的目录。默认值为 '/run/user/$UID/xpra'、'/run/xpra'、'~/.xpra'。
```

### --socket-dir=SOCKET_DIR

```sh
放置/查找套接字文件的目录。默认值为 '$XPRA_SOCKET_DIR' 或 'socket-dirs' 中的第一个有效目录。
```

### --system-proxy-socket=SYSTEM_PROXY_SOCKET

```sh
用于联系系统范围代理服务器的套接字路径。默认值为 '/run/xpra/system'。
```

### --sessions-dir=SESSIONS_DIR

```sh
放置/查找会话文件的目录。默认值为 '$XDG_RUNTIME_DIR/xpra'。
```

### --ssh-upgrade=SSH_UPGRADE

```sh
将 TCP 套接字升级为处理 SSH 连接。默认值为 'True'。
```

### --rfb-upgrade=RFB_UPGRADE

```sh
在此延迟（以秒为单位）后将 TCP 套接字升级为发送 RFB 握手（默认值为 5）。
```

### -d FILTER1,FILTER2,..., --debug=FILTER1,FILTER2,...

```sh
启用调试的类别列表（也可以使用 "all" 或 "help"，默认值为 ''）。
```

### --ssh=CMD

```sh
运行 ssh 的方式。默认值为 'auto'。
```

### --exit-ssh=yes|no|auto

```sh
断开连接时是否终止 SSH。默认值为 True。
```

### --username=USERNAME

```sh
客户端提供的用于身份验证的用户名。默认值为 ''。
```

### --auth=AUTH

```sh
要使用的身份验证模块（默认值为 none）。
```

### --tcp-auth=TCP_AUTH

```sh
要用于 TCP 套接字的身份验证模块 - 不建议使用，请使用每个套接字的语法（默认值为 none）。
```

### --ws-auth=WS_AUTH

```sh
要用于 Websockets 的身份验证模块 - 不建议使用，请使用每个套接字的语法（默认值为 none）。
```

### --wss-auth=WSS_AUTH

```sh
要用于安全 Websockets 的身份验证模块 - 不建议使用，请使用每个套接字的语法（默认值为 none）。
```

### --ssl-auth=SSL_AUTH

```sh
要用于 SSL 套接字的身份验证模块 - 不建议使用，请使用每个套接字的语法（默认值为 none）。
```

### --ssh-auth=SSH_AUTH

```sh
要用于 SSH 套接字的身份验证模块 - 不建议使用，请使用每个套接字的语法（默认值为 none）。
```

### --rfb-auth=RFB_AUTH

```sh
要用于 RFB 套接字的身份验证模块 - 不建议使用，请使用每个套接字的语法（默认值为 none）。
```

### --quic-auth=QUIC_AUTH

```sh
要用于 QUIC 套接字的身份验证模块 - 不建议使用，请使用每个套接字的语法（默认值为 none）。
```

### --vsock-auth=VSOCK_AUTH

```sh
要用于 vsock 套接字的身份验证模块 - 不建议使用（默认值为 'none'）。
```

### --min-port=MIN_PORT

```sh
创建 TCP 套接字时允许的最小端口号（默认值为 '1024'）。
```

### --mmap-group=MMAP_GROUP

```sh
在与客户端创建 mmap 文件时，将 mmap 文件的组权限设置为此组，使用特殊值 'auto' 可以使用与我们连接的服务器套接字文件的所有者相同的值（默认值为 'auto'）。
```

### --socket-permissions=SOCKET_PERMISSIONS

```sh
在创建服务器 Unix 域套接字时使用的文件访问模式（默认值为 '600'）。
```

### --pings=yes|no

```sh
发送 ping 数据包的频率（以秒为单位，使用零禁用）。默认值为 5。
```

### --clipboard-filter-file=CLIPBOARD_FILTER_FILE

```sh
包含必须被过滤掉的剪贴板内容的正则表达式的文件名。
```

### --local-clipboard=SELECTION

```sh
在使用翻译剪贴板时要同步的本地剪贴板选择的名称（默认值为 CLIPBOARD）。
```

### --remote-clipboard=SELECTION

```sh
在使用翻译剪贴板时要同步的远程剪贴板选择的名称（默认值为 CLIPBOARD）。
```

### --remote-xpra=CMD

```sh
在远程主机上运行 xpra 的方式（默认值为 xpra 或 $XDG_RUNTIME_DIR/xpra/run-xpra 或 /usr/local/bin/xpra 或 ~/.xpra/run-xpra 或 Xpra_cmd.exe）。
```

### --encryption=ALGO

```sh
指定要使用的加密密码，指定 'help' 以获取选项列表（默认值为 None）。
```

### --encryption-keyfile=FILE

```sh
指定包含加密密钥的文件（默认值为 ''）。
```

### --tcp-encryption=ALGO

```sh
指定要用于 TCP 套接字的加密密码，指定 'help' 以获取选项列表（默认值为 None）。
```

### --tcp-encryption-keyfile=FILE

```sh
指定用于 TCP 套接字的加密密钥的文件（默认值为 ''）。
```
