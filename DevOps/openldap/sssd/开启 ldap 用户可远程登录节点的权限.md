# 开启 ldap 用户可远程登录节点的权限

## 安装sssd

### sssd 介绍

sssd服务是一个守护进程，该进程可以用来访问多种验证服务器，如LDAP，Kerberos等，并提供授权。SSSD是 介于本地用户和数据存储之间的进程，本地客户端首先连接SSSD，再由SSSD联系外部资源提供者(一台远程服务器)

1. 避免了本地每个客户端程序对认证服务器大量连接，所有本地程序仅联系SSSD，由SSSD连接认证服务器或SSSD缓存，有效的降低了负载。
2. 允许离线授权。SSSD可以缓存远程服务器的用户认证身份，这允许在远程认证服务器宕机是，继续成功授权用户访问必要的资源。

### 集成sssd

1. 安装 sssd:

```sh
yum -y install openldap-clients sssd authconfig nss-pam-ldapd
```

2. 配置 sssd

```sh
[sssd]
config_file_version = 2
services = nss,pam,autofs, sudo
domains = funsine.com
debug_level = 9

[nss]
homedir_substring = /home

[pam]

[domain/funsine.com]
id_provider = ldap
auth_provider = ldap
#autofs_provider = ldap
#chpass_provider = ldap
ldap_uri = ldap://192.168.100.30:389
ldap_search_base = dc=funsine,dc=com
ldap_id_use_start_tls = False
ldap_tls_cacertdir = /etc/openldap/certs
ldap_default_bind_dn = cn=admin,dc=funsine,dc=com
ldap_default_authtok_type = password
ldap_default_authtok = admin@123
cache_credentials = True
ldap_tls_reqcert = never
entry_cache_timeout = 600
#ldap_network_timeout = 3
#ldap_connection_expire_timeout = 60
sudoers_base ou=sudo,dc=funsine,dc=com
sudo_provider = ldap

[sudo]
```

3. 执行如下命令启用sssd服务

```sh
authconfig --enablesssd --enablesssdauth --enablelocauthorize --update
```

4. 修改 sssd 配置文件的权限

```sh
chmod 600 /etc/sssd/sssd.conf
```

5. 启动sssd服务并加入系统自启动

```sh
systemctl start sssd
systemctl enable sssd
systemctl status sssd
```

## OpenLDAP与SSH集成

1. 修改配置文件/etc/ssh/sshd_config，使ssh通过pam认证账户

```sh
#PasswordAuthentication yes
#PermitEmptyPasswords no
PasswordAuthentication yes


# and ChallengeResponseAuthentication to 'no'.
# WARNING: 'UsePAM no' is not supported in Red Hat Enterprise Linux and may cause several
# problems.
UsePAM yes
```

2. 修改配置文件/etc/pam.d/sshd，以确认调用pam认证文件

```sh
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open env_params
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    include      password-auth
session    include      postlogin
session    required      pam_mkhomedir.so
# Used with polkit to reauthorize users in remote sessions
-session   optional     pam_reauthorize.so prepare
```

3. 修改配置文件 /etc/pam.d/password-auth, 把 sssd 相关的服务配置进去

```sh
#%PAM-1.0
# This file is auto-generated.
# User changes will be destroyed the next time authconfig is run.
auth        required      pam_env.so
auth        required      pam_faildelay.so delay=2000000
auth        [default=1 ignore=ignore success=ok] pam_succeed_if.so uid >= 1000 quiet
auth        [default=1 ignore=ignore success=ok] pam_localuser.so
auth        sufficient    pam_unix.so nullok try_first_pass
auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
auth        sufficient    pam_sss.so forward_pass
auth        required      pam_deny.so

account     required      pam_unix.so
account     sufficient    pam_localuser.so
account     sufficient    pam_succeed_if.so uid < 1000 quiet
account     [default=bad success=ok user_unknown=ignore] pam_sss.so
account     required      pam_permit.so

password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
password    sufficient    pam_sss.so use_authtok


password    required      pam_deny.so

session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
-session     optional      pam_systemd.so
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so
session     optional      pam_sss.so
```

4. 修改/etc/pam.d/system-auth配置文件

```sh
#%PAM-1.0
# This file is auto-generated.
# User changes will be destroyed the next time authconfig is run.
auth        required      pam_env.so
auth        required      pam_faildelay.so delay=2000000
auth        [default=1 ignore=ignore success=ok] pam_succeed_if.so uid >= 1000 quiet
auth        [default=1 ignore=ignore success=ok] pam_localuser.so
auth        sufficient    pam_unix.so nullok try_first_pass
auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
auth        sufficient    pam_sss.so forward_pass
auth        required      pam_deny.so

account     required      pam_unix.so
account     sufficient    pam_localuser.so
account     sufficient    pam_succeed_if.so uid < 1000 quiet
account     [default=bad success=ok user_unknown=ignore] pam_sss.so
account     required      pam_permit.so

password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
password    sufficient    pam_sss.so use_authtok
password    required      pam_deny.so

session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
-session     optional      pam_systemd.so
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so
session     optional      pam_sss.so
```

5. 重启sshd服务

```sh
systemctl restart sshd
```

## sssd.conf 一些配置字段的解释

### entry_cache_timeout

这是最常用的设置，定义了缓存条目在被认为过期之前可以存储多久。值以秒为单位。例如，设置为3600表示缓存条目保持有效期为一小时。

### cache_credentials

布尔值参数，用于确定是否缓存用户的凭证。在网络或LDAP服务不可用时，如果这个选项被设置为True，用户仍然可以使用缓存的凭证进行本地登录。

### account_cache_expiration

这是一个较新的参数，可以用来设置用户帐户信息的具体缓存过期时间。如果配置了这个参数，它会覆盖entry_cache_timeout对用户帐户信息的设置。
