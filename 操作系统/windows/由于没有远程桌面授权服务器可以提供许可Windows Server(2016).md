由于没有远程桌面授权服务器可以提供许可证，远程会话连接已断开。请跟服务器管理员联系。原因是服务器安装了远程桌面服务RemoteApp,这个是需要授权的。但是微软官方给予了120天免授权使用，超过120天还没有可用授权就会出现远程会话被中断，可修改注册表来延长使用期限，此方法win2008和win2012适用
<img src="410141d75fd65737770fd8b7b2c90564.png" alt="截图" style="zoom:50%;" />

1. 首先进入安装了远程桌面服务RemoteApp的服务器

2. 按 win+R 键 打开运行，输入regedit 然后按 确定

![截图](d66613551ad56b9aca9ef244c61ec6cd.png)

3. 然后就打开了注册表编辑器

![截图](3e76f21210f0e275f12b4cdd6575ccbd.png)

4. 然后进入 HKEY_LOCAL_MACHINE \ SYSTEM \ CurrentControlSet \ Control \ Terminal Server \ RCM \ GracePeriod ，选中 GracePeriod 然后右键 点击 权限

![截图](38fbea5ea64f86ab946b2456dfc22020.png)

5. 选择Administrators , 把完全控制勾上，点击确定后。

![截图](911831e3d9c4bdd171b966dd4938961d.png)

如果修改成功了直接执行后面的7

如果执行失败了，就会出现以下信息

![截图](5caa6a717ba159b8531da393a6f95520.png)

6. administrator权限赋予
回到GracePeriod，右键权限，点击高级

![截图](a13d59a5991a0889fd5bef131783fb54.png)

点击更改所有者

![截图](80b0af98c7899172bc2140e4756f975d.png)

注意：这里选择的是Administrators，看上图中的图标。
更改所有者后在审核里进行添加

![截图](d783c53615ec7bea8a474b5afe78fcf1.png)

选择Administrators为主体后，在此处添加完全控制权限

![截图](044e75ba65e704f66dcb58bc2da68440.png)

选择Administrators , 然后把 完全控制 勾上 ， 再按确定

![截图](f6d6f4eed3876a593c05dded5c2fa5b2.png)

7.选中注册表中的 REG_BINARY，然后右键修改名字

![截图](c980ae592bfd663baa3bdf70dc62440e.png)

9、然后关掉注册表编辑器，重启服务器
10、重启服务器后就不会被中断了，此方法可以继续使用120天，等到120天到了再按此方法进行操作就可以再继续使用了
