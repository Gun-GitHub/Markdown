修改文件 /etc/xrdp/startwm.sh：

```sh
vim /etc/xrdp/startwm.sh
```

添加以下内容：

```sh
nohup /usr/local/src/fs-screen/FsScreen-1.0.0-linux-x86_64-setup.AppImage --no-sandbox &
```

整体看起来是这样的：

```sh
#!/bin/sh
# xrdp X session start script (c) 2015, 2017, 2021 mirabilos
# published under The MirOS Licence

# Rely on /etc/pam.d/xrdp-sesman using pam_env to load both
# /etc/environment and /etc/default/locale to initialise the
# locale and the user environment properly.
#unset DBUS_SESSION_BUS_ADDRESS
#unset XDG_RUNTIME_DIR
nohup /usr/local/src/fs-screen/FsScreen-1.0.0-linux-x86_64-setup.AppImage --no-sandbox &

if test -r /etc/profile; then
        . /etc/profile
fi

if test -r ~/.profile; then
        . ~/.profile
fi

#export GNOME_SHELL_SESSION_MODE=ubuntu
#export XDG_CURRENT_DESKTOP=ubuntu:GNOME
test -x /etc/X11/Xsession && exec /etc/X11/Xsession
exec /bin/sh /etc/X11/Xsession
#if [ -f "$HOME/.xsession" ]; then
#  . "$HOME/.xsession"
#elif [ -f "$HOME/.xinitrc" ]; then
#  . "$HOME/.xinitrc"
#else
#  exec /etc/X11/Xsession
#fi

```

---

linux 操作系统注销指令

```sh
loginctl terminate-session $(loginctl show-user 用户名 -p Sessions --value)
```