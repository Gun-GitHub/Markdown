## 当使用 `sudo` 执行指令时，是否覆盖用户原本的环境变量取决于 `sudo` 的配置和使用方式：

1. **默认行为**  
   - `sudo` 默认会保留一部分环境变量，如 `HOME`、`USER`、`LOGNAME`，但会清除一些敏感变量，如 `LD_LIBRARY_PATH`、`PYTHONPATH`，以增强安全性。  
   - 例如，直接执行：
     ```sh
     sudo env
     ```
     
     你会发现一些环境变量被修改或移除。
2. **使用 `-E` 选项（保留用户环境变量）**  
   - `sudo -E` 允许保留当前用户的环境变量。例如：
     ```sh
     sudo -E echo $PATH
     ```
   - 但是否生效取决于 `/etc/sudoers` 中的 `env_keep` 规则，某些变量可能仍会被过滤。
3. **使用 `env` 指定环境变量**  
   - 你可以使用 `env` 命令显式传递变量，例如：
     ```sh
     sudo env PATH=$PATH some_command
     ```
4. **切换到 root 用户环境**  
   - 使用 `sudo -i` 或 `sudo su` 切换到 root 时，会加载 root 用户的环境，而不是当前用户的环境。例如：
     ```sh
     sudo -i
     env
     ```
   - 这样会使用 `/root/.bashrc` 或 `/root/.profile` 进行环境初始化。
5. **影响 `sudo` 行为的配置 (`/etc/sudoers`)**  
   - `Defaults env_reset`：重置环境变量（默认开启）。  
   - `Defaults env_keep += "VAR_NAME"`：保留特定变量。  
   - `Defaults secure_path="/sbin:/bin:/usr/sbin:/usr/bin"`：在 `sudo` 执行时，`PATH` 可能被重置为此值。

### 结论：

- **默认情况下，`sudo` 会覆盖部分环境变量**，但会保留基本变量（如 `HOME`、`USER`）。
- **使用 `sudo -E` 可以尝试保留环境变量**，但具体效果取决于 `/etc/sudoers` 规则。
- **如果你需要完整的 root 环境，请使用 `sudo -i` 或 `sudo su`**。

特别是在 slurm 执行指令的时候特别明显:

```sh
wjx1@cn1:/root$ srun -c 1 --mem=1GB /usr/bin/sudo /usr/bin/env
slurmstepd: error: couldn't chdir to `/root': Permission denied: going to /tmp instead
slurmstepd: error: couldn't chdir to `/root': Permission denied: going to /tmp instead
LANGUAGE=zh_CN:zh
LANG=zh_CN.UTF-8
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:
TERM=xterm
DISPLAY=localhost:11.0
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
MAIL=/var/mail/root
LOGNAME=root
USER=root
HOME=/root
SHELL=/bin/bash
SUDO_COMMAND=/usr/bin/env
SUDO_USER=wjx1
SUDO_UID=1514
SUDO_GID=500

```

```sh
wjx1@cn1:/root$ srun -c 1 --mem=1GB /usr/bin/env
SHELL=/bin/bash
LANGUAGE=zh_CN:zh
PWD=/root
LOGNAME=wjx1
XDG_SESSION_TYPE=tty
MOTD_SHOWN=pam
HOME=/var/hpc-root/wjx1
LANG=zh_CN.UTF-8
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:
SSH_CONNECTION=192.168.0.141 59879 192.168.1.203 22
LESSCLOSE=/usr/bin/lesspipe %s %s
XDG_SESSION_CLASS=user
TERM=xterm
LESSOPEN=| /usr/bin/lesspipe %s
USER=wjx1
DISPLAY=localhost:11.0
SHLVL=2
XDG_SESSION_ID=1160
XDG_RUNTIME_DIR=/run/user/0
SSH_CLIENT=192.168.0.141 59879 22
XDG_DATA_DIRS=/usr/local/share:/usr/share:/var/lib/snapd/desktop
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/0/bus
MAIL=/var/mail/wjx1
SSH_TTY=/dev/pts/1
_=/usr/local/bin/srun
SLURM_CONF=/var/spool/slurmd/conf-cache/slurm.conf
SLURM_PRIO_PROCESS=0
SRUN_DEBUG=3
SLURM_UMASK=0022
SLURM_CLUSTER_NAME=fs_dev
SLURM_SUBMIT_DIR=/root
SLURM_SUBMIT_HOST=cn1
SLURM_JOB_NAME=env
SLURM_JOB_CPUS_PER_NODE=1
SLURM_MEM_PER_NODE=1024
SLURM_NTASKS=1
SLURM_NPROCS=1
SLURM_CPUS_PER_TASK=1
SLURM_DISTRIBUTION=cyclic,pack
SLURMD_DEBUG=2
SLURM_JOB_ID=476
SLURM_JOBID=476
SLURM_STEP_ID=0
SLURM_STEPID=0
SLURM_NNODES=1
SLURM_JOB_NUM_NODES=1
SLURM_NODELIST=cn1
SLURM_JOB_PARTITION=normal
SLURM_TASKS_PER_NODE=1
SLURM_JOB_UID=1514
SLURM_JOB_USER=wjx1
SLURM_JOB_GID=500
SLURM_JOB_GROUP=slurmuser
SLURM_JOB_ACCOUNT=wjx1
SLURM_JOB_QOS=wjx1
SLURM_WORKING_CLUSTER=fs_dev:gpu1:6817:9984:109
SLURM_JOB_NODELIST=cn1
SLURM_STEP_NODELIST=cn1
SLURM_STEP_NUM_NODES=1
SLURM_STEP_NUM_TASKS=1
SLURM_STEP_TASKS_PER_NODE=1
SLURM_STEP_LAUNCHER_PORT=45735
SLURM_SRUN_COMM_PORT=45735
SLURM_SRUN_COMM_HOST=192.168.1.203
SLURM_TOPOLOGY_ADDR=cn1
SLURM_TOPOLOGY_ADDR_PATTERN=node
SLURM_CPUS_ON_NODE=1
SLURM_CPU_BIND=quiet,mask_cpu:0x0000000001
SLURM_CPU_BIND_LIST=0x0000000001
SLURM_CPU_BIND_TYPE=mask_cpu:
SLURM_CPU_BIND_VERBOSE=quiet
SLURM_JOB_END_TIME=1772869291
SLURM_JOB_START_TIME=1741333291
SLURM_TASK_PID=418630
SLURM_NODEID=0
SLURM_PROCID=0
SLURM_LOCALID=0
SLURM_LAUNCH_NODE_IPADDR=192.168.1.203
SLURM_GTIDS=0
SLURMD_NODENAME=cn1
TMPDIR=/tmp
slurmstepd: error: couldn't chdir to `/root': Permission denied: going to /tmp instead
slurmstepd: error: couldn't chdir to `/root': Permission denied: going to /tmp instead
```

## 设置 /etc/sudoers 使得 sudo 可以在执行某些指令的时候可以保留用户的所有环境变量

要让 `sudo` 在执行某些指令时保留用户的**所有环境变量**，需要修改 `/etc/sudoers` 配置，具体步骤如下：

### 方案1: 仅对特定用户保留所有环境变量

如果你不希望全局修改，而是**仅对特定用户**（如 `your_user`）生效，可以在 `sudoers` 里添加：

```sh
Defaults:your_user !env_reset
Defaults:your_user env_keep += "*"
```

这样，`your_user` 在执行 `sudo` 时可以保留所有环境变量，而其他用户仍然使用默认行为。

### 方案2: 仅对特定命令保留所有环境变量

如果你只想对**某些命令**保留环境变量（如 `python`），可以这样设置：

```sh
Cmnd_Alias PRESERVE_ENV_CMDS = /usr/bin/python, /usr/bin/env
Defaults!PRESERVE_ENV_CMDS !env_reset
Defaults!PRESERVE_ENV_CMDS env_keep += "*"
```

这样，执行以下命令时会保留所有环境变量：

```sh
sudo python
sudo env
```

但其他 `sudo` 命令仍会重置环境。

### **注意事项**

1. **安全风险**：
   - `Defaults env_keep += "*"` 可能允许用户保留 `LD_LIBRARY_PATH`、`PYTHONPATH` 等变量，可能会带来安全风险（比如加载不受信任的共享库）。
   - 如果是生产环境，建议仅允许某些特定变量，如：
     ```sh
     Defaults env_keep += "PATH HOME USER LANG"
     ```
2. **`secure_path` 影响 `PATH`**：
   - `sudo` 默认会使用 `secure_path` 配置：
     ```sh
     Defaults secure_path="/sbin:/bin:/usr/sbin:/usr/bin"
     ```
   - 这会覆盖 `PATH`，如果希望保留原 `PATH`，需要禁用它：
     ```sh
     Defaults !secure_path
     ```
3. **修改 `sudoers` 必须使用 `visudo`**：
   - 直接编辑 `/etc/sudoers` 可能导致语法错误，建议用 `visudo` 进行修改。

### **总结**

- **全局生效**：`Defaults !env_reset` 和 `Defaults env_keep += "*"`  
- **特定用户生效**：`Defaults:your_user !env_reset`  
- **特定命令生效**：`Defaults!PRESERVE_ENV_CMDS !env_reset`

如果是开发环境，可以直接使用 `Defaults env_keep += "*"`，但生产环境建议精细化控制环境变量。

## 演示修改结果:

```sh
Cmnd_Alias PRESERVE_ENV_CMDS = /var/hpc-root/.system/slurm/slurm-luanch-code-server-app.sh, /usr/bin/env
Defaults!PRESERVE_ENV_CMDS !env_reset
Defaults!PRESERVE_ENV_CMDS env_keep += "*"
%slurmuser ALL=(ALL) NOPASSWD: /var/hpc-root/.system/slurm/slurm-luanch-code-server-app.sh
%slurmuser ALL=(ALL) NOPASSWD: /usr/bin/env
```
