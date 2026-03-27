- ascend310 推理卡测试
  
  ## 使用官方的推理镜像
  1. 注册[https://ascendhub.huawei.com/](https://ascendhub.huawei.com/)账号并登录，鼠标悬停用户名出现菜单，电机镜像下载凭证，设置镜像下载凭证。下载凭证一次设置只有24小时有效期，超出时间需要重新设置。
     
      ![](https://secure2.wostatic.cn/static/gGxEZtcC94aK2hC6A6p5Hv/image.png?auth_key=1708398352-5caZAnyisW6QYXVyBQaxeg-0-8fe30b059d0986e5476dbb314bb990ea)
  2. 在ascendhub上找到推理镜像，infer-modelzoo。查看对应的镜像版本，注意镜像版本与芯片有关。可选择23.0.RC2-mxvision版本。
     
      ![](https://secure2.wostatic.cn/static/qzajsqnH2iu78d8sfhK48/image.png?auth_key=1708398352-ust7r8YgrMbepuN7E6x7tD-0-601f913e046b694eb9862cc81b8f2868)
  3. 安装23.0.RC2-mxvision镜像对应的cann套件，前往[ascend资源下载页面](https://www.hiascend.com/developer/download/community/result?module=cann&cann=6.3.RC2.alpha005)选择**对应版本的CANN**，下载推理框架(注意架构和芯片的差异)，此处选择红框所示。nnrt是推理框架，nnae是训练框架，toolkit同时包含训练和推理框架；选择aarch架构镜像。
     
      ![](https://secure2.wostatic.cn/static/biTMxEjaSvJdC4cTXKhYXb/image.png?auth_key=1708398352-bLvvvvTfVnELUaJfi4Unap-0-e02345b064becae9b062a1679591e73c)
     
      下载到推理服务器上后解压缩，有两重压缩。全部解压缩后，运行命令安装nnrt`./Ascend-cann-nnrt_6.3.RC2_linux-aarch64.run --install --install-path=/usr/local/Ascend`。
     
      并将nnrt自带的脚本添加环境变量

```sh
echo "source /usr/local/Ascend/nnrt/set_env.sh" >> /etc/profile
source /etc/profile
```

4. 拉取推理镜像
到镜像详情页中点击目标版本镜像，点击立即下载，会出现镜像下载指令
   
    `docker login -u user_name ascendhub.huawei.com`
   
    注意：一定要先注册用户名并生成下载凭证，否则子命令会报http 500 错误。登录时会需要输入密码，这个密码就是**下载凭证（而不是ascendhub注册时的密码)**
   
    登录后使用命令拉取镜像`docker pull [ascendhub.huawei.com/public-ascendhub/infer-modelzoo:23.0.RC2-mxvision](http://ascendhub.huawei.com/public-ascendhub/infer-modelzoo:23.0.RC2-mxvision)`
   
    ![](https://secure2.wostatic.cn/static/jiGrvbm1QT4CV6fUuFjmx2/image.png?auth_key=1708398352-ebqX4WwDGn4Z6s3HSMeiFB-0-df9907cbe63a2bfb9393cbf7307af1f3)
5. 启动推理测试demo

```sh
docker run -it --rm \
--device=/dev/davinci0 \
--device=/dev/davinci_manager \
--device=/dev/devmm_svm \
--device=/dev/hisi_hdc \
-v /usr/local/dcmi:/usr/local/dcmi \
-v /var/log/npu:/var/log/npu \
-v /usr/local/Ascend/driver:/usr/local/Ascend/driver \
-v /usr/slog:/usr/slog \
-v /usr/local/bin/npu-smi:/usr/local/bin/npu-smi \
-v /usr/local/Ascend/driver/lib64/:/usr/local/Ascend/driver/lib64/ \
-v /usr/local/Ascend/driver/tools/:/usr/local/Ascend/driver/tools/ \
-v /usr/local/Ascend/add-ons/:/usr/local/Ascend/add-ons/ \
-v /data:/data \
ascendhub.huawei.com/public-ascendhub/infer-modelzoo:23.0.RC2-mxvision \
bash test_model.sh
```

```
  出现如下结果即证明推理成功

  ![](https://secure2.wostatic.cn/static/4C5CeJPbFNMq8JL2DVF6Ne/8a52100b7b9acc3b6ccba2d149e3c025.png?auth_key=1708398352-6yspSNzfHiY26r7B2GzFD7-0-526a3841cf5329aa732d294d57324802)
```

  https://www.hiascend.com/document/detail/zh/CANNCommunityEdition/700alpha002/infacldevg/aclcppdevg/aclcppdevg_000004.html

  https://www.hiascend.com/document/detail/zh/canncommercial/70RC1/inferapplicationdev/aclpythondevg/aclpythondevg_01_0146.html
