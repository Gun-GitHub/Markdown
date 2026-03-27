# ldap 修改 windows AD 域密码必须开启 ssl 即 LDAPS

# 配置启用基于 SSL 的 LDAP(LDAPS)

## 安装证书服务 AD CS

默认情况下，LDAP 通信未加密。这使得恶意用户能够使用网络监控软件查看传输中的数据包。这就是许多企业安全策略通常要求组织加密所有 LDAP 通信的原因。为了减少这种形式的数据泄露，微软提供了一个选项：您可以启用通过安全套接字层 (SSL)/传输层安全性 (TLS) 的 LDAP，也称为 LDAPS。利用 LDAPS，您可以提高整个网络的安全性

安装Active Directory证书服务

![截图](1c02665d1056843c0c58b3248280129b.png)

勾选第一个证书颁发机构

![截图](3cbc753621325ca010920281f8694de6.png)

然后一直下一步，直到安装完成即可

![截图](d7a08d05c9ab0715208f31240578329c.png)

## 配置ADCS

![截图](1f6f2cb4ceac7d83cbd25bfb282df677.png)

![截图](721ee3138b665a4f49db024a41fc2f4a.png)

选择证书颁发机构

![截图](35ba8ba705cfdf6c36f9f8af67ed3cbb.png)

选择企业

![截图](d2ff743bbfd029b5b7bc8471aae43d02.png)

选择根

![截图](074506907423c5423d6fd00237f4df69.png)

创建新的私钥

![截图](af4b981425d7768ddc4631ffdcdd1681.png)

下一步

![截图](5b6393f477965020cdc392d72f416502.png)

![截图](d8295dc69c29bad17f176efc60626e27.png)

![截图](411af277499f8daa208114bcd2fd3080.png)

![截图](20d5df14bfae484224879a3c21d7fa82.png)

![截图](81bdaba3535ffbce713f6693e08c3144.png)

如下配置完成

![截图](ebcab71878e19b9ed11baeabddc774f7.png)

## 证书配置

打开AD CS，选择证书颁发机构

![截图](d37fad46c2e20d908d03737e5612523b.png)

选择证书模板，右键管理

![截图](360905c11b31d6a907b1f1205f1ab63c.png)

选择Kerberos身份验证，右键 复制模板

![截图](90b2ca2d18fdcbad3a0df121a309ba70.png)

然后会有一个Kerberos身份验证的副本，右键更改名称，更改为LDAPS

![截图](1694759f81f5df1cae8fe16380d6604a.png)

选择LDAPS，右键属性

![截图](05e3b9b498812ae04131cf5a9d16f5b8.png)

![截图](05e3b9b498812ae04131cf5a9d16f5b8.png)

设置模板属性，请求处理——>允许导出私钥(O)

![截图](25cd7f6b7ef00f577499e36556253bad.png)

创建证书模板

![截图](32cc76f56bbabd7fd6cac088b2e185d5.png)

选择LDAPS，确定

![截图](b989bfa2704053e12c17838537153583.png)

![截图](a9ac7c3e7cf056f2de57a17a3963b35c.png)

![截图](a05c490da7b28dd5013bad5543d5ed69.png)

![截图](eacf25f2640e60ada0d2d0553b3f6283.png)

![截图](84d504edfc0f2a82bc46df2694edc6f9.png)

![截图](c3feb80d86026ff84c70096f2f29b449.png)

![截图](7aa623eeb8ca7a8a33d98fc19c93cfb9.png)

![截图](4d5613b5ce660b1139eceab7de327670.png)

![截图](8fc7abb23726ba34deee956147da4cda.png)

![截图](b0013f404203a77e5fe31e25bcc0a81b.png)

![截图](c691b4e8a3b9c670c55c24ff7a18ff1b.png)

![截图](1c8002da6c7afb25aa21a12f6089de3f.png)

[参考文档](https://blog.csdn.net/hwe_govern/article/details/131231300)