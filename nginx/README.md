# nginx镜像使用说明
nginx镜像定制化分为四个阶段：
1. 定制0：直接使用官方nginx镜像。
2. 定制1：基于官方nginx镜像的Dockerfile简单改造。
3. 定制2：基于纯净linux镜像的手动commit构建。
4. 定制3：基于纯净linux镜像的。

# 对比结果
1. 前期不喜欢定制0，是因为与我定制化安装思路不同，比如用的基础系统镜像不同、安装位置、配置文件位置/内容等都是默认值，未经过定制化。定制1也只是对定制0的简单改进，区别不大。
2. 而后采用定制2、定制3，希望能实现纯手动安装过程的高可定制化效果，实践之后发现实用性不大：
    1. 通常nginx手动安装时，能够定制日志路径、指定运行用户、修改nginx.conf、修改各服务器配置文件、添加系统服务并开机启动；
    2. 上述定制化分析：
        1. 在定制0(官方nginx镜像)中，通过目录挂载可以将日志挂载到主机任意目录，同样能够挂载nginx.conf和服务器配置文件，并且配置文件中指定的root目录也是通过挂载本地主机目录实现定制的。
        2. 此外，对镜像本身的修改，比如指定运行用户等，可以通过定制1的方式局部修改；
        3. 并且系统服务与开机启动由docker运行机制来完成；
3. 经过上述分析对比，发现平时手动安装的高可定制化优势，在官方nginx镜像中都是能够包含得住。
4. 此外，还有个严重的区别，就是官方镜像时经过严格优化的，镜像体积100MB左右，自己按常规过程定制化安装达到了400MB+，如果所有linux软件部署都如此做，耗费磁盘也是个槽点。体积过大主要是由于update和安装构建工具过多导致的。常规软件部署时，这些基础设施是公用的，并不存在什么问题，但是docker的每个镜像都占用一份就很不合理了，如果去做优化，也肯定是达不到官方的程度，因为思路根本不同，如果仍然去优化，费时费力不说，Dockerfile可能无限接近官方版本了，那还不如直接使用定制0/1，直接满足所有需求。

# 最佳实践
1.	加速器：省去修改源的步骤。
2.	使用定制0。
3.	日志目录、配置文件、server目录、www目录：通过挂载实现灵活配置。
4.	开机启动：重启后容器仍然存在，通过启动参数restart，确保容器被拉起。

```shell
mkdir -p /starlab/gitRepos
git clone git@github.com:bi-kai/starlab-docker-images.git
mkdir -p /starlab/docker
cp -r /starlab/gitRepos/starlab-docker-images/nginx /starlab/docker
cd /starlab/docker/nginx

docker pull nginx

docker run -d --name nginx_service -p 81:80 \
-m 512m --memory-swap 1G --restart=always \
-v /starlab/docker/nginx/logs:/var/log/nginx \
-v /starlab/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /starlab/docker/nginx/conf/vhosts:/etc/nginx/conf.d \
-v /starlab/docker/nginx/www:/usr/share/nginx/html/:ro \
nginx

```

上述覆盖中，本地服务器配置目录为vhosts，镜像中服务器配置目录为conf.d，通过映射仍保持conf.d文件夹，在本地nginx.conf中引用的仍然是conf.d文件夹。

-EOF-
