
# Ubuntu 安装 Emby 并挂载网盘远程播放指南

## 准备条件

1. Ubuntu 24.04 LTS 服务器  
2. Emby 安装包  
3. OpenList 安装包  
4. alist-strm 安装包  

---

## 一、环境准备

### ✅ 建议使用 root 用户操作

```bash
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

### 依赖说明

| 包名 | 说明 |
|------|------|
| `apt-transport-https` | 支持 apt 使用 HTTPS 协议 |
| `ca-certificates` | 安装根证书，验证 HTTPS 安全连接 |
| `curl` | 命令行网络工具，下载脚本常用 |
| `software-properties-common` | 提供 `add-apt-repository` 命令支持添加源 |

---

## 二、创建必要目录

> 📁 路径结构树形图
> 以下为安装与挂载路径的树形结构展示：
```
/root
├── docker
│   ├── openlist
│   │   └── data              # OpenList 配置文件
│   ├── emby
│   │   └── config            # Emby 配置文件
│   └── strm
│       ├── config            # alist-strm 配置文件
│       └── data              # alist-strm 数据目录
├── download                  # 安装包与镜像存放目录
└── video
    ├── music                 # 音乐文件挂载目录
    └── strm                  # .strm 流媒体挂载目录
```

```bash
mkdir -p \
/root/docker/openlist \
/root/docker/openlist/data \
/root/docker/emby \
/root/docker/emby/config \
/root/docker/strm \
/root/docker/strm/config \
/root/docker/strm/data \
/root/download \
/root/video \
/root/video/music \
/root/video/strm
```

---

## 三、安装 Docker

1. 前往 [Docker Ubuntu 软件包目录](https://download.docker.com/linux/ubuntu/dists/noble/pool/stable/amd64/) 下载以下 `.deb` 安装包：

> 💡 提示：按住 Ctrl（Windows）或 Cmd（macOS）点击链接，可在新标签页中打开。

   - `docker-ce`
   - `docker-ce-cli`
   - `containerd.io`
   - `docker-buildx-plugin`
   - `docker-compose-plugin`

2. 使用 SFTP 工具将 `.deb` 包上传到 `/root/download/`

3. 安装 Docker：

```bash
cd /root/download
sudo dpkg -i *.deb
```
> 如果出现依赖缺失，运行以下命令修复：
```
sudo apt-get install -f
```

4. 验证安装：

```bash
sudo systemctl status docker
docker -v
docker images
```
> 查询docker版本号码
> 
> docker -v
> 
> 查看docker中有什么镜像
> 
> docker images

---

## 四、下载并导入镜像（推荐在网络好的电脑完成）

> 这次教程的环境是 Ubuntu 24.04 LTS 服务器，架构是 amd64。在这种环境下，如果你用正常网络下载 Docker 镜像并导出，要注意一点：如果你用的是 ARM 电脑，默认拉下来的是 ARM 架构的镜像。要想在 amd64 服务器上部署，就得强制指定架构，强制拉取 amd64 版本的镜像，这样才能保证镜像能正常运行。

## 项目资源整理

###  Emby

- **Docker 镜像地址**：  
  [https://hub.docker.com/r/amilys/embyserver](https://hub.docker.com/r/amilys/embyserver)

---

###  alist-strm

- **GitHub 地址**：  
  [https://github.com/tefuirZ/alist-strm](https://github.com/tefuirZ/alist-strm)

- **Docker 镜像地址**：  
  [https://hub.docker.com/r/itefuir/alist-strm](https://hub.docker.com/r/itefuir/alist-strm)

---

###  OpenList

- **GitHub 地址**：  
  [https://github.com/OpenListTeam/openlist](https://github.com/OpenListTeam/openlist)

- **Docker 镜像地址**：  
  [https://hub.docker.com/r/openlistteam/openlist](https://hub.docker.com/r/openlistteam/openlist)



### 1. Emby 镜像

```bash
docker pull --platform=linux/amd64 amilys/embyserver:latest
docker inspect --format='{{.Architecture}}' amilys/embyserver:latest
docker save -o amilys-embyserver-amd64.tar amilys/embyserver:latest
```

### 2. Alist-strm 镜像

```bash
docker pull --platform=linux/amd64 itefuir/alist-strm:latest
docker inspect --format='{{.Architecture}}' itefuir/alist-strm:latest
docker save -o itefuir-alist-strm-amd64.tar itefuir/alist-strm:latest
```

### 3. OpenList 镜像

```bash
docker pull --platform=linux/amd64 openlistteam/openlist:latest
docker inspect --format='{{.Architecture}}' openlistteam/openlist:latest
docker save -o openlist-amd64.tar openlistteam/openlist:latest
```

### 3. 1 OpenList 镜像现在还没有latest（使用 beta 版本）

```bash
docker pull --platform=linux/amd64 openlistteam/openlist:beta
docker inspect --format='{{.Architecture}}' openlistteam/openlist:beta
docker save -o openlist-amd64.tar openlistteam/openlist:beta
```

### 4 如果需要将镜像导出为 .tar 文件并保存到指定路径，可以使用如下命令：

```bash
docker save -o /root/download/openlist-amd64.tar openlistteam/openlist:latest
```
> 使用 docker save 命令时，指定保存的路径和文件名（如 /路径/路径/文件名.tar），后面跟上要导出的镜像及版本（项目名称:版本号），即可将镜像导出到指定位置。

---

## 五、镜像导入服务器

> 使用 SFTP 工具将 `.tar ` 包上传到 `/root/download/`
> 
> 进入镜像存放目录
> 
> 镜像导入服务器
> 
> 并且导入镜像到服务器

```bash
cd /root/download
docker load -i amilys-embyserver-amd64.tar
docker load -i itefuir-alist-strm-amd64.tar
docker load -i openlist-amd64.tar
docker images
```

---

## 六、获取设备 UID / GID（用于容器权限）

```bash
id
```
>  示例输出：
>  uid=0(root) gid=0(root) groups=0(root)

---

## 七、运行容器

> 安装docker镜像到容器 命令介绍
> 
> docker run -d \
> 
> （docker 运行 创建容器）
> 
>   --name emby \
>   
> （创建容器名字为 emby） 
> 
>   -p 8096:8096 \
>   
>   (-p 内外端口映射设置)
>   
> （容器外部端口：容器内部端口）
> 
>   -p 8920:8920 \
>   
> （容器外部端口：容器内部端口） 
> 
>   -v /root/docker/emby/config/:/config \
>   
> （-v内外路径设置）
> 
> （外部路径：容器内部路径）
> 
>   -v /root/video/:/video \
>   
> （外部路径：容器内部路径）
> 
>   -e UID=0 \
>   
> （之前获取设备id的 UID号码）
> 
>   -e GID=0 \
>   
> （之前获取设备id的 GID号码）
> 
>  -e TZ=Asia/Shanghai \
>  
> （设置容器时区为亚洲上海时区） 
> 
> amilys/embyserver:latest
> 
> （使用（镜像名字）：版本）

### 1. 运行 Emby 容器

```bash
docker run -d \
  --name emby \
  -p 8096:8096 \
  -p 8920:8920 \
  -v /root/docker/emby/config/:/config \
  -v /root/video/:/video \
  -e UID=0 \
  -e GID=0 \
  -e TZ=Asia/Shanghai \
  amilys/embyserver:latest
```

> 浏览器访问：`http://服务器IP:8096`，首次设置语言、账号和密码。

---

### 2. 运行 OpenList 容器

```bash
docker run -d \
  --name=openlist \
  -v /root/docker/openlist/data/:/opt/openlist/data \
  -p 5244:5244 \
  -e PUID=0 \
  -e PGID=0 \
  -e UMASK=022 \
  openlistteam/openlist:beta
```

> 查看日志获取初始密码：

```bash
docker logs openlist
```

> 示例密码：`Successfully created the admin user... password is: 0p93Lr9V`  
> 示例密码：`password is: 0p93Lr9V`    这里 0p93Lr9V 就是初始密码 包含大小写 请复制
> 浏览器访问：`http://服务器IP:5244`，账号：`admin`，密码：日志中的初始密码。

---

### 3. 运行 alist-strm 容器

```bash
docker run -d \
  --name alist-strm-container \
  -p 5000:5000 \
  -v /root/docker/strm/config/:/config \
  -v /root/docker/strm/data/:/data \
  -v /root/video/:/video \
  itefuir/alist-strm:latest
```

> 浏览器访问：`http://服务器IP:5000`，注册账号即可使用。

---

## 八、容器管理命令

- 停止容器：

```bash
docker stop emby
docker stop openlist
docker stop alist-strm-container
```

- 启动容器：

```bash
docker start emby
docker start openlist
docker start alist-strm-container
```

- 设置自动重启：

```bash
docker update --restart=always emby openlist alist-strm-container
```

---

## 九、升级容器镜像（以 alist-strm 为例）

### 1. 导入新镜像

```bash
docker load -i /root/download/itefuir-alist-strm-amd64.tar
```

### 2. 停止旧容器

```bash
docker stop alist-strm-container
```

### 3. 备份配置

```bash
cp -r /root/docker/strm/config/ /root/docker/strm/config_backup_$(date +%s)
```

### 4. 删除旧容器

```bash
docker rm alist-strm-container
```

### 5. 重建容器（保持原端口配置）

```bash
docker run -d \
  --name alist-strm-container \
  -p 5000:5000 \
  -v /root/docker/strm/config/:/config \
  -v /root/docker/strm/data/:/data \
  -v /root/video/:/video \
  itefuir/alist-strm:latest
```

---

## 十、查看容器运行状态

```bash
docker ps -a
```

---


