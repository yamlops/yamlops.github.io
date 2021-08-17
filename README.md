# yamlops.github.io
## Dockerfile中指令: 

- **FROM** : 多阶段构建镜像 需要多用在生产中

- **RUN**指令 : 是 docker build 的时候运行的命令，可以定义多个RUN，每一个RUN都会运行

- **CMD**指令 : 是 docker run 的时候运行的命令，不可以定义多个，只会运行最后的一个CMD

- **ENTRYPOINT**指令 : 是docker run 的时候运行的命令，如果有ENTRYPOINT和CMD同时存在，CMD ["<param1>","<param2>"]就成为了为ENTRYPOINT传参数的指令了.ENTRYPOINT无法被覆盖。

- **ARG**指令 : 是docker build的时候定义的参数

- **ENV**指令 : 是docker run 的时候定义的环境变量

- **ADD**指令 : 是将Dockerfile所在的工作目录中文件复制到镜像里 例如：ADD demo-test.py /usr/local/bin/

- **EXPOSE** : 是定义需要暴露的端口 例如： EXPOSE 80 8080

- **VOLUME** : 是逻辑卷存储。例如：VOLUME ["/app/data"]

## K8S：

	kubelet: 是API Server 的客户端，与API Server随时通信，接收API Server 和 Scheduler 分发的由自己负责执行的任务 
	
		CNI: 容器网络接口     <-->   Network Plugins      \
		CSI: 容器存储接口	    <-->   Storage Device      --》 Pod(Containers|Storage|Network)
		CRI: 容器运行时接口	<-->   Container Runtime    /
	
	控制器: 负责编排应用
	
		无状态应用: 通常使用 Deployment
		有状态应用: 通常使用 Statefulset
	
	Service : 为Pod组织网络的
	Ingress : 接入集群外部流量的


	网络: 
	
		节点网络 : 集群当中各个节点通过物理网卡进行通信的叫节点网络
		Pod网络 : 一个或多个节点上可以有多个Pod，而Pod之间通信的网络叫Pod网络。Pod之间的网络通信功能K8S没有直接支持，而是委托第三方网络插件来实现的。通过第三方插件实现通信的网络叫做Pod网络。插件有flannel calico canal（前两个的结合）
		service网络 : 可以称之为虚拟网络，从本质上讲他并不存，只是出现在iptables 和 ipvs规则当中出现，用于Pod之间通信、以及Pod与外部客户端之间通信时作为流量的管理、整形、地址转换等功能的组件当中使用的。
	
	K8S的三种管理方式:
	
		命令式命令
		命令式配置文件
		声明式配置文件

- OCI（Open Container Initiative，开放容器标准）

- CRI 机制，即容器运行时接口（Container Runtime Interface）

## CGroups

![image-20201115213713866](/Users/newcgtn/Library/Application Support/typora-user-images/image-20201115213713866.png)



## Docker :

### 容器的几种状态：

* **Up： 表示正在运行。可以docker stop ID **

* **Created：被创建之后由于启动参数错误而没有启动成功的。**

- **Exited：已经退出(stop)的容器。这种容器可以删除，也可以docker start ID 。**

- **paused：被暂停的容器状态**



**docker ps 只查看正在运行中状态的容器**

**docker ps -a 查看所有状态的容器**

**docker ps -a -q 查看所有容器，并把容器ID列出来。 **

**docker pause  容器ID ：暂停某个容器。**

**docker unpause  容器ID：取消暂停。**

**docker rm 容器 ： 删除容器。慎重使用。**

**docker rm	-f  容器：强制删除容器。正在运行的容器也能删除。 **

**docker rm -v 容器: 删除容器时跟着删除关联的卷(volumes)**

### 端口映射

```bash
root@hyrila:~# docker run -d -it -p 8080:80 nginx
```

#### 容器多端口映射

```bash
root@hyrila:~# docker run -d -it -p 10080:80 -p 10443:443 nginx
```

##### 启动容器时网络协议默认是TCP协议，如果指定协议的话 “/tcp /udp” "/" 后跟上协议即可；如下示例：

```bash
root@hyrila:~# docker run -d -it -p 10080:80/tcp  -p 10443:443/tcp  -p 10053:53/udp nginx
```

##### 查找所有容器并删除

```bash
root@hyrila:~# docker rm -fv `docker ps -a -q`
```



#### 进入容器 docker exec

```bash
root@hyrila:~# docker exec -it f2805e476e01  sh
```

#### 查看容器信息 docker inspect

```bash
root@hyrila:~# docker inspect -f "{{.State.Pid}}" f2805e476e01  # 这个是查看容器的PID
```

- 这个命令 **-f** 选项后面要跟 **"{{}}"** ,括号内以"."开头，跟上顶级层信息，逐层往后写。比如上面的例子，查看容器PID，PID信息属于顶级State 里的子信息。所以写成"{{.State.Pid}}"

- 查看容器PID的另一种方式 **docker top**

  ```bash
  root@hyrila:~# docker top f2805e476e01UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMDroot                6145                6125                0                   17:34               pts/0               00:00:00            /bin/sh
  ```

  

![image-20201129175124992](/Users/newcgtn/Library/Application Support/typora-user-images/image-20201129175124992.png)



### 查看和删除 Exited 状态的容器

```bash
root@hyrila:~# docker ps -a -f status=exitedroot@hyrila:~# docker ps -aq -f status=exitedroot@hyrila:~# docker rm -fv `docker ps -aq -f status=exited`
```



![image-20201202182011917](/Users/newcgtn/Library/Application Support/typora-user-images/image-20201202182011917.png)



## 制作镜像

#### 1. 基于正在运行的容器制作镜像

> **就是启动一个容器，在容器内安装命令，程序，服务等之后，再提交为一个镜像。这个方式用的很少。优点制作起来方便，缺点是后期想改动的话需要再次跑起来，改完后再提交为容器**

- 下面我启动了centos容器，进去到里面安装了nginx， 并测试把页面修改了一下。然后根据这个容器我生成了新的镜像。

```bash
root@hyrila:~# docker commit -a "WXG IN BJ WorkSpaces" -m "YUM INSTALL NGINX FROM BASE IMAGE CENTOS:7.7.1908" eb8996ca6ad2 centos-ngix:1.01.201204
```

#### 根据容器提交生成镜像

![image-20201204111358822](/Users/newcgtn/Library/Application Support/typora-user-images/image-20201204111358822.png)

#### 查看生成的镜像并用来启动容器

![image-20201204112735061](/Users/newcgtn/Library/Application Support/typora-user-images/image-20201204112735061.png)

- 检查下效果

  ![image-20201204112946382](/Users/newcgtn/Library/Application Support/typora-user-images/image-20201204112946382.png)

  

#### 2. 基于Dockerfile制作镜像

> **基础镜像==父镜像**

##### EXPOSE ： 指令白话概念

- 在Dockerfile中的 **`EXPOSE`**指令指出的端口都是该容器启动后容器内的端口。**并不是宿主机的端口**。

##### ADD ： 将宿主机上的文件或目录添加到镜像里面去

- **`ADD`** 会对 **`tar.gz`**的包自动解压，但是相同功能的**`COPY`**却不会。

##### ENV ：给容器指定全局变量。

#### 3. Nginx 源码编译安装的Dockerfile实战

```dockerfile
root@hyrila:/opt/dockerfile/nginx-src# pwd/opt/dockerfile/nginx-srcroot@hyrila:/opt/dockerfile/nginx-src# lsDockerfile  nginx-1.18.0.tar.gzroot@hyrila:/opt/dockerfile/nginx-src# root@hyrila:/opt/dockerfile/nginx-src# cat Dockerfile ## 用于测试Dockerfile 构建nginx Web静态网站镜像#FROM centos:7.7.1908LABEL maintainer="Wang Xiaoguang < imongol@aliyun.com > or < aiops@qq.com >" # LABEL 是作者信息RUN yum -y install tree vim telnet net-tools \    && echo "It is a test file." > /root/file.txt \    && yum -y install epel-release \     && yum -y install wget curl lrzsz gcc gcc-c++ \     && yum -y install pcre pcre-devel zlib zliv-devel \    && yum -y install make openssl openssl-devel unzipADD nginx-1.18.0.tar.gz /usr/local/srcRUN cd /usr/local/src/nginx-1.18.0 \    && ./configure --prefix=/apps/nginx \      && make && make install \    && rm -rf nginx-1.18.0 \    && ln -s /apps/nginx/sbin/nginx /usr/bin/nginx \    && cd /usr/local/src/ && rm -rf nginx-1.18.0 \    && echo "<h1> Website From Nginx Source Dockerfile Images. </h1>" > /apps/nginx/html/index.htmlEXPOSE 80 443 # 容器启动后暴露的端口CMD ["nginx", "-g", "daemon off;"]root@hyrila:/opt/dockerfile/nginx-src#
```

#### 4. 制作centos基础系统镜像



#### 5. 制作JDK镜像

![image-20210103143821670](/Users/newcgtn/Library/Application Support/typora-user-images/image-20210103143821670.png)

#### 6. 分层镜像

![image-20210121144231091](/Users/newcgtn/Library/Application Support/typora-user-images/image-20210121144231091.png)

### 7. Dockerfile 优化减少image的大小

![image-20210128164117877](/Users/newcgtn/Library/Application Support/typora-user-images/image-20210128164117877.png)

- **大小对比图**

![image-20210128163726321](/Users/newcgtn/Library/Application Support/typora-user-images/image-20210128163726321.png)

### 小架构实验

![image-20210131135702091](/Users/newcgtn/Library/Application Support/typora-user-images/image-20210131135702091.png)



## Pod的创建过程

**`User Use kubectl Client CMD greate a pod to API Server `** **`-->`** **`API Server Write to etcd DB`**  **`-->`**  **`Scheduler Watch ETCD `** **`-->`** **`Scheduler See There is Have a New Pod`** **`-->`** **`bind pod`** **`-->`** **`Scheduler Write bind pod messages to ETCD`** **`-->`** **`kubelet client watch API Server `** **`-->`** **`kubelet client see the new pod and than bound pod(Get the pod message and than deploy pod on node)`** **`-->`** **`kubelet run pod on node `** **`-->`** **`kubelet Update Pod status to API Server `** **`-->`** **`API Server write pod status to ETCD`**



###  kubectl api-versions  : 查看k8s支持的群组和版本信息

### kubectl explain Kind_Name(or Kind_Name.ResourceNname...) : 查看资源清单及配置字段



### rc : 发行后选版本



### Pod 安全上下文

![image-20210815123055607](/Users/newcgtn/Library/Application Support/typora-user-images/image-20210815123055607.png)



### 拉取镜像策略

![image-20210815124825471](/Users/newcgtn/Library/Application Support/typora-user-images/image-20210815124825471.png)



强制删除一个pod
![image-20210815130445602](/Users/newcgtn/Library/Application Support/typora-user-images/image-20210815130445602.png)

### Pod 安全上下文允许设置的安全的sysctl内核参数有三个

![image-20210815133720145](/Users/newcgtn/Library/Application Support/typora-user-images/image-20210815133720145.png)



- **如果要配置非安全的内核参数的话，需要修改kubelet配置文件，将允许的内核参数加进去。**

```bash
vim /etc/default/kubelet# /etc/default/kubelet 内容KUBELET_EXTRA_ARGS='--allowed-unsafe-sysctls=参数1,参数2,...' #意思就是允许这些不安全的内核参数在Pod 安全上下文里去设定。# 注意⚠️：每个node 节点都要配置然后重启kubelet哟 
```

### Pod 健康探测

![image-20210817110433666](/Users/newcgtn/Library/Application Support/typora-user-images/image-20210817110433666.png)



