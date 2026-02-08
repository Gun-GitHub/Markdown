### 相关文档参照链接:

	[如何利用 JuiceFS 的性能工具做分析和调优](https://juicefs.com/zh-cn/blog/usage-tips/a-case-with-profile-and-stats)

### 启动指令类似为:

```sh
  juicefs mount -d --entry-cache 300 --attr-cache 300 localhost /mnt/jfs
```

### 启动指令更新为:

```sh
 /usr/local/bin/juicefs mount -d --free-space-ratio 0.5 --cache-dir /data/jfscache --entry-cache 300 --attr-cache 300 --writeback redis://:****@redis-master.redis:6379/1 /var/hpc-root
```

### 注意:

```sh
  --cache-dir /data/jfscache
```

	==指定缓存目录 尽量不要在系统分区==