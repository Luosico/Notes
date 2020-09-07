# <font size="70">Docker</font>
### 1、Docker

**Docker是一种运行与Linux和Windows上的软件，用于创建、管理和编排容器**

### 2、Docker引擎

<img src="https://s1.ax1x.com/2020/06/16/NkF77V.md.jpg" style="zoom: 80%;" />



	构成：Docker客户端(Docker Client)、Docker守护进程(Docker daemon)、containerd、runc
	
	容器运行时与Docker deamon是解耦的，称为“无守护进程的容器”，所以对Docker Deamon的维护和升级工作不会影响运行中的容器
	
	runc：唯一作用是创建容器，与操作系统内核原语通信，实质上是一个轻量级的、针对Libcontainer进行了包装的命令行交互工具。容器进程作为runc的子进程启动，启动完毕，runc将会退出
	
	containerd：管理容器的生命周期，以daemon（守护进程）的方式运行
	shim：实现无daemon的容器不可或缺的工具，每次创建容器它都会fork一个新的runc实例，容器创建完毕，对应的runc进程就会退出，相关联的containerd-shim进程就会成为容器的父进程
		  shim的部分职责如下：
		  	- 保持所有STDIN和STDOUT流是开启状态，从而当daemon重启的时候，容器不会因为管道（pipe）的关闭而终止
		  	- 将容器的退出状态反馈给daemon



###  3、常用命令

	service docker start	启动docker
				   stop		停止docker
				   restart	重启docker
	systemctl enable docker 开机自启动
	
	docker version			查看docker版本信息
	docker system info		显示系统范围信息
	
	docker image 
			ls				列举镜像
			build			从Dockerfile构建一个image
				--squash	创建一个合并的镜像
			history			显示一个image的历史信息
			import			从tarball导入内容以创建文件系统镜像
			inspect			显示一个或多个镜像的详细信息，包括镜像的分层信息
			load			从tar archiver或STDIN加载镜像
			prune			移除全部悬虚镜像，加上-a移除全部没有被容器使用的镜像
	
			pull			Pull an image or a repository from a registry
			push			Push an image or a repository to a registry
							镜像名必须为：用户名/仓库名:tag  若仓库没有会新建仓库  
							docker push luosico/tomcat:test
			rm				移除一个或多个镜像
			save			Save one or more images to a tar archive (streamed to STDOUT by default)
			tag				Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
	
	docker tag source_image[:tag] target_image[:tag]	给镜像打标签
	
	docker search			从Docker Hub搜索镜像，包括官方的和非官方的，默认只返回25行
			--limit			限制返回行数
	
	docker container
							容器会在shell退出后终止，除非设置后台运行
			ls				列出全部处于运行中状态的容器
			ls -a			列出全部的容器，包括停止状态的
			run <options> <image>:<tag> <app>	启动容器，只在第一次运行时使用，将镜像放到容器中
				-name <name>	指定容器名字
				--flag		挂在卷
				-restart 		<重启策略>	设置重启策略，包括always、unless-stopped、on-failed
				-d				在后台启动容器
				-p	port1:port2	将主机的port1端口映射到容器内的port2端口
				-it				将当前shell切换到容器终端，按Ctrl-PQ组合键可以在退出容器的同时还保持容器运行
			start			启动已经存在的容器，被stop的
			exec			创建新的bash连接到运行中的容器，此时该bash输入exit将不会导致容器终止
			stop			手动停止容器，可用start再次启动容器
			pause			暂停容器内的所有进程
			unpause			恢复容器内的所有进程
			rm				删除容器
			inspect			显示容器的配置细节和运行时信息
			...				bash使用docker container查看

### 4、Dockerfile

- 用来构建镜像的文本文件

### 5、Image 镜像

   [<img src="https://s1.ax1x.com/2020/09/07/wKt4gS.jpg" alt="wKt4gS.jpg" style="zoom: 25%;" />](https://imgchr.com/i/wKt4gS)

**Docker镜像由一些松耦合的只读镜像层组成**，所以说镜像由一个或多个镜像层组成

#### 查看镜像层

```
docker image inspect ubuntu:latest
```

#### 基础镜像层

所有的Docker镜像都起始于一个基础镜像，当进行修改或增加新的内容时，就会在当前的镜像层之上，创建新的镜像层

**同一个文件，上层的镜像层会覆盖掉底层镜像层**

Docker通过**存储引擎（新版本采用快照机制）**的方式来实现镜像层堆栈，并保证多镜像层对外展示为统一的文件系统

#### 共享镜像层

多个镜像之间可以共享镜像层，所以在下载镜像的时候，对于重复的镜像层不会再重新下载

#### 镜像与镜像层

**镜像**本身就是配置对象，其中包含了镜像层的列表以及一些元数据信息

**镜像层**才是实际数据存储的地方，比如文件，并且镜像层之间是**完全独立的**

镜像与镜像层都有唯一的**内容散列值**标识，由于镜像在传输过程中会被压缩，导致散列值改变，每个镜像层同时会有一个**分发散列值**

#### 多架构镜像

解决同一个镜像适应多个平台的问题（Linux、Windows等）

**Manifest列表**：指某个镜像标签支持的架构列表，Docker会根据当前当前平台和架构选择正确的镜像版本

### 6、容器

**容器是镜像的运行时实例，共享主机的操作系统/内核**

