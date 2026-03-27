# Ubuntu 18.04 出现GLIBC_2.28 not found的解决方法(亲测有效)

https://blog.csdn.net/Youning_Yim/article/details/129343107

```sh
RUN echo "deb http://security.debian.org/debian-security buster/updates main" >> /etc/apt/sources.list \
    && apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 112695A0E562B32A 54404762BBB6E853 \
    && apt list --upgradable \
    && apt update \
    && apt install libc6-dev -y
```