# 009-CentOS 7安装Docker



### 一、yum换源（云服务器跳过此步）

```bash
cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

yum clean all
yum makecache

yum repolist
```





### 二、移除旧版本docker

```bash
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```





### 三、安装docker-ce

```bash
# 安装依赖
yum install -y yum-utils
# 换源
yum-config-manager --add-repo   http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 安装docker-ce
yum install docker-ce -y
```

### 四、配置镜像加速

打开华为云控制台，搜索“容器镜像服务SWR”后点击进入。（所有云服务平台都有该服务）

左侧选择镜像资源 --> 镜像中心，然后在右上角点击镜像加速器，在打开的对话框中复制镜像加速地址。

```
{
    "registry-mirrors": [ "https://你的镜像加速ID.mirror.swr.myhuaweicloud.com" ]
}
```

回到服务器中，配置镜像加速文件

```sh
vim /etc/docker/daemon.json
```

将华为云的镜像加速地址粘贴到该文件内，保存后退出。

### 五、启动docker

```bash
systemctl start docker   # 启动docker
systemctl enable docker # 开机自启
```

若启动失败则清空上面配置的daemon.json文件后启动，然后成功启动后重新配置daemon.json文件，再重启。