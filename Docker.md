# <center>Docker</center>
---
### Docker引擎
	构成：Docker客户端(Docker Client)、Docker守护进程(Docker daemon)、containerd、runc

###  常用命令
	service docker start	启动docker
				   stop		停止docker
				   restart	重启docker
	systemctl enable docker 开机自启动

	docker version			查看docker版本信息
	docker system info		显示系统范围信息

	docker image 
			ls				列举镜像
			build			从Dockerfile构建一个image
			history			显示一个image的历史信息
			import			从tarball导入内容以创建文件系统镜像
			inspect			显示一个或多个镜像的详细信息，包括镜像的分层信息
			load			从tar archiver或STDIN加载镜像
			prune			移除全部悬虚镜像，加上-a移除全部没有被容器使用的镜像

			pull			Pull an image or a repository from a registry
			push			Push an image or a repository to a registry
			rm				移除一个或多个镜像
			save			Save one or more images to a tar archive (streamed to STDOUT by default)
			tag				Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

	docker search			从Docker Hub搜索镜像，包括官方的和非官方的，默认只返回25行

	docker container
			ls				列出全部处于运行中状态的容器
			ls -a			列出全部的容器，包括停止状态的
			run <image> <app>	启动容器，只在第一次运行时使用，将镜像放到容器中
				-name <name>	指定容器名字
				-restart <重启策略>	设置重启策略，包括always、unless-stopped、on-failed
				-d				在后台启动容器
				-p	port1:port2	将主机的port1端口映射到容器内的port2端口
			start			启动已经存在的容器，被stop的
			exec			创建新的bash连接到容器，该bash输入exit将不会导致容器终止
			stop			手动停止容器
			pause			暂停容器内的所有进程
			unpause			恢复容器内的所有进程
			rm				删除容器
			inspect			显示容器的配置细节和运行时信息
			...				bash使用docker container查看
	
	