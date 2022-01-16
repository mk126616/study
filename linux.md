# **常规命令**

查看ip：ifconfig

操作服务：service 服务名 status/start/stop

防火墙：firewalld  

安装：yum install 名称

查找应用：yum search

设置服务自动重启：systemctl enable 服务名.service

登录mysql : mysql -u root -p

退出mysql：exit

登录psql：psql -U postgres

退出psql:  \q

安装基础命令： apt-get update   apt-get install <命令名称>

文本批量替换：文本底行模式下 %S/<原始值>/<目标值>

远程复制命令：scp  local_file  remote_username@remote_ip:remote_folder  

设置服务开启自动启动：systemctl enable docker（以docker为例）

磁盘挂载：mount为临时挂载，永久挂载需要vim etc/fstab修改系统文件增加如下数据，再进行mount挂载（取消挂载为umount 磁盘分区）

```txt
/dev/sda4                                 /var/lib/docker         ext2    defaults        1 1
```



 

# **遇到的错误解决方案**

1. 问题：docker容器启动WARNING: IPv4 forwarding is disabled. Networking will not work.

解决：vi /usr/lib/sysctl.d/00-system.conf 添加如下代码： net.ipv4.ip_forward=1 重启network服务 systemctl restart network





# **Docker**

## 1. **docker build创建镜像**

参数： -d ：给docker镜像添加标记

 

示例：docker build -d tomcat:v1 . ：创建名为toncat的镜像，tag为v1 ，“.”为上下文路径，docker在docker引擎中构建镜像，需要把上下文路径上所有文件打包发送给引擎。

## 2. **DockerFile文件创建镜像**

命令：COPY:从宿主机拷贝文件 到镜像内的路径中，路径不存在时会新建

ADD:同为拷贝文件，但是当目标文件为tar包压缩格式为gzip, bzip2 以及 xz 时会自动解压（ADD jdk1.8.0_151 /usr/java/jdk）

CMD:指定容器启动时的操作，可以执行一个可执行文件或者shell命令（CMD /opt/tomcat/bin/catalina.sh run

）

RUN:构建镜像时，执行的shell命令或者可执行文件

FROM:指定基础镜像（FROM centos:6）

MAINTAINER：指定镜像的的所有人（MAINTAINER mk [<1974698751@qq.com>）](mailto:<1974698751@qq.com>）)

ENV:定义环境变量（ENV JAVA_HOME=/usr/java/jdk）

VOLUME：挂载目录,容器启动时自动挂载，也可以在创建容器时指定

WORKDIR：指定工作目录，目录必须用命令提前创建

 

EXPOSE：端口映射，容器启动时自动映射，也可以在创建容器时指定

 

 

 

## 3. **保存生成的镜像**

Docker save -o 文件名 已经过的load镜像名称 ；-o表示输出到文件中

## 4. **使用已有容器创建镜像导入镜像**

docker export -o ./tomcat_jdk_v2.tar.gz tomcat:v2 ；-o表示输出到文件

docker import tomcat.tar.gz tomcat:v2 ；v2是镜像的tag

## **5.****docker commit :****从容器创建一个新的镜像**

## **6.加载镜像文件到本地镜像仓库**

docker load -i 镜像文件，-i表示从文件输入

 

## 7. **使用镜像创建容器**

docker run --name tomcat -p 8080:8080 -d --restart=aways tomcat_jdk:v1；-d代表后台运行容器，用户退出容器会在后台运行，不会停止

## 8. **docker-compose.yml**

可以定义多个容器，docker-compose up -d 使用yml文件创建容器

## 9. **已存在容器更新设置**

docker update --restart=always <container>

# **Linux安装常用工具**

apt-get update ##跟新

//vi

apt install vim

//weget

apt install weget

//yum

apt install yum

//ifconfig

apt install net-tools

//ping

apt install iputils-ping



 