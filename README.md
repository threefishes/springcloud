@@@THREEFISHES
%%springcloud.
spring-boot项目构建运行.
1. In terminal, use maven to build package to jar.<br/>
2. Use java -jar tf.jar,run the application.<br/>
3. Request http://127.0.0.1 to view web function.<br/>
------------------------------------------------------------------------------------------------------
部署到外部tomcat并以war形式，需要将tomcat设置为private，但是由于idea BUG问题，需要在本地运行时，将此注释掉（在 Intellij Idea 15 中使用maven时，所有 scope 为 provided 的依赖都是不会被加入到 classpath 中的，目前该bug尚未被修复(bug report)。如果你的web应用是部署到容器中的，那么这个bug不会影响使用，因为web应用中provided的依赖在容器运行时会被提供。如果你做Spring Boot开发，有带provided的依赖时，直接在IDE中运行项目会导致ClassNotFound异常。<br/>
解决方案有二：<br/>
使用spring-boot:run这个 maven goal 运行程序。但这样会失去 Idea 的 debug功能，不推荐。点击IDE右侧的Maven Projects, 找到spring-boot:run，右键选择 debug 运行）.<br/>
<!--scope>provided</scope-->
其他解决方案参见<br/>
http://blog.csdn.net/sonycong/article/details/70173354.<br/>
<br/>
docker.
假设我们应用是www,目录位置在/app/deploy/www.
对于war文件，docker下的tomcat没自动解压出文件，手动解压后可以使用（unzip tf.war）.
docker run --privileged=true -v /app/deploy/www:/usr/local/tomcat/webapps/tf  -p 8099:8080 tomcat:9

-------------------------------------------------------------------------------
docker部署
1. 192.168.111.178 安装docker,启动systemctl start docker.
如果启动多个dcoker，需要手动指定相关参数.
注册docker账号.
在服务器端登录dockerdocker login<br/>
开启docker远程API，修改docker配置文件<br/>
#vi /usr/lib/systemd/system/docker.service<br/>
进入编辑模式后，将ExecStart这一行后面加上 -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock ，改完后如下所示:<br/>
        ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock<br/>
这里就写4个0，你可别改成自己的ip哦，保存后退出，重新加载配置文件<br/>
$systemctl daemon-reload<br/>
启动docker<br/>
$systemctl start docker<br/>
输入#netstat -anp|grep 2375 显示docker正在监听2375端口，输入#curl 127.0.0.1:2375/info  显示一大堆信息，证明远程api就弄好了
<br/>
2. 项目里面创建Dockerfile
3. pom.xml增加docker插件相关信息
<configuration>
    <imageName>threefishes/${project.artifactId}</imageName>
    <dockerDirectory>src/main/docker</dockerDirectory>
    <resources>
        <resource>
            <targetPath>/</targetPath>
            <directory>${project.build.directory}</directory>
            <include>${project.build.finalName}.jar</include>
        </resource>
    </resources>
</configuration>
4. 执行如下命令中的一个
$maven打包并构建镜像
mvn clean package -DskipTests=true docker:build
$maven打包构建镜像并push到仓库
mvn clean package docker:build -DpushImage
$maven打包构建镜像，将指定tag的镜像push到仓库，该命令需使用
$<imageTags><imageTag>...</imageTag></imageTags>标签
mvn clean package docker:build -DpushImageTag
5. 成功上传后，执行docker run -p 80:80 -t threefishes/tf，启动项目
6. 访问 http://192.168.111.178/find

---------------------------------------------------------------------------------------------------------------
docker2
安装Docker
yum install docker
本文使用的系统是centos7,ubuntu使用以下命令
sudo apt-get update
sudo apt-get install docker-engine
如果报了以下错误，是因为yum被其它进程使用了

Another app is currently holding the yum lock; waiting for it to exit...
  The other application is: PackageKit
    Memory :  12 M RSS (924 MB VSZ)
    Started: Mon Jan  2 17:22:13 2017 - 1 day(s) 1:06:13 ago
    State  : Sleeping, pid: 16208

查看正在yum使用的进程
ps -ef|grep yum
kill掉它即可

kill -9 16208
安装完成，查看安装是否成功

docker info        #查看docker的情况
docker --version   #查看docker的版本
启动Docker服务
service docker start
启动Docker的hello-world

从Docker Hub下载一个hello-world镜像
docker pull hello-world
运行hello-world镜像

docker run hello-word
输出以下信息
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

至此，我们已成功运行起第一个Docker容器

tomcat运行环境<br/>
1、搜索Docker Hub里的tomcat镜像<br/>
docker search tomcat<br/>
部分搜索结果如下<br/>
NAME                        DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED<br/>
tomcat                      Apache Tomcat is an open source implementa...   1132      [OK]<br/>
dordoka/tomcat              Ubuntu 14.04, Oracle JDK 8 and Tomcat 8 ba...   29                   [OK]<br/>
cloudesire/tomcat           Tomcat server, 6/7/8                            12                   [OK]<br/>
davidcaste/alpine-tomcat    Apache Tomcat 7/8 using Oracle Java 7/8 wi...   11                   [OK]<br/>
andreptb/tomcat             Debian Jessie based image with Apache Tomc...   6                    [OK]<br/>
可以看到，星数最高的是官方的tomcat，有关官方tomcat的镜像可以访问<br/>
https://hub.docker.com/r/library/tomcat/<br/>
这里写图片描述<br/>
上面 “7.0.73-jre7, 7.0-jre7, 7-jre7, 7.0.73, 7.0, 7”等等 是这个tomcat库支持的tag（标签），这里我们选用的是 “7” 这个标签<br/>
2、拉取Docker Hub里的镜像<br/>
docker pull tomcat:7<br/>
3、拉取完成后查看本地的镜像<br/>
docker images #所有镜像<br/>
docker image tomcat:7  #查看REPOSITORY为tomcat:7的镜像<br/>
4、运行tomcat镜像<br/>
docker run tomcat:7<br/>
可以访问 http://ip:8080 确认容器的tomcat已启动成功<br/>
使用以下命令来查看正在运行的容器<br/>
docker ps<br/>
若端口被占用，可以指定容器和主机的映射端口<br/>
docker run -p 8081:8080 tomcat:7<br/>
启动后，访问地址是http://ip:8081<br/>
5、运行我们的web应用<br/>
假设我们应用是www,目录位置在/app/deploy/www<br/>
docker run --privileged=true -v /app/deploy/www:/usr/local/tomcat/webapps/www  -p 8081:8080 tomcat:7<br/>
-v /app/deploy/www:/usr/local/tomcat/webapps/www 是把/app/deploy/www的目录挂载至容器的/usr/local/tomcat/webapps/www。<br/>
–privileged=true是授予docker挂载的权限
至此，已成功把web应用部署在Docker容器运行

常用命令

%%查看所有镜像<br/>
docker images<br/>
%%正在运行容器<br/>
docker ps<br/>
%%查看docker容器<br/>
docker ps -a<br/>
%%启动tomcat:7镜像<br/>
docker run -p 8080:8080 tomcat:7<br/>
%%以后台守护进程的方式启动<br/>
docker run -d tomcat:7<br/>
%%停止一个容器<br/>
docker stop b840db1d182b<br/>
%%进入一个容器<br/>
docker attach d48b21a7e439<br/>
%%进入正在运行容器并以命令行交互<br/>
docker exec -it e9410ee182bd /bin/sh<br/>
%%以交互的方式运行<br/>
docker run -i -t -p 8081:8080 tomcat:7 /bin/bash<br/>
<br/>
-----------------------------------------------------------
RabbitMQ

添加用户<br/>
%%rabbitmqctl add_user cnrmall cnrmall<br/>
给用户添加权限<br/>
%%rabbitmqctl set_permissions cnrmall ".*" ".*" ".*"<br/>
设置用户为admin标签<br/>
%%Rabbitmqctl set_user_tags cnrmall administrator<br/>
启动rabbitmq服务<br/>
$$systemctl start rabbitmq-server<br/>
允许远程登录及网页登录<br/>
$$rabbitmq-plugins enable rabbitmq_management<br/>
网页访问地址:http://192.168.111.178:15672<br/>
