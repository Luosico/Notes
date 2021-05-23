# <font size="70">Docker</font>
## 1、Docker

**Docker是一种运行与Linux和Windows上的软件，用于创建、管理和编排容器**



## 2、Docker引擎

<img src="https://s1.ax1x.com/2020/06/16/NkF77V.md.jpg" style="zoom: 80%;" />



	构成：Docker客户端(Docker Client)、Docker守护进程(Docker daemon)、containerd、runc
	
	容器运行时与Docker deamon是解耦的，称为“无守护进程的容器”，所以对Docker Deamon的维护和升级工作不会影响运行中的容器
	
	runc：唯一作用是创建容器，与操作系统内核原语通信，实质上是一个轻量级的、针对Libcontainer进行了包装的命令行交互工具。容器进程作为runc的子进程启动，启动完毕，runc将会退出
	
	containerd：管理容器的生命周期，以daemon（守护进程）的方式运行
	shim：实现无daemon的容器不可或缺的工具，每次创建容器它都会fork一个新的runc实例，容器创建完毕，对应的runc进程就会退出，相关联的containerd-shim进程就会成为容器的父进程
		  shim的部分职责如下：
		  	- 保持所有STDIN和STDOUT流是开启状态，从而当daemon重启的时候，容器不会因为管道（pipe）的关闭而终止
		  	- 将容器的退出状态反馈给daemon



##  3、常用命令

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
				--name <name>	指定容器名字
				--flag		挂载卷
				-restart 		<重启策略>	设置重启策略，包括always、unless-stopped、on-failed
				-d				在后台启动容器
				-p	port1:port2	将主机的port1端口映射到容器内的port2端口
				-it				将当前shell切换到容器终端，按Ctrl-PQ组合键可以在退出容器的同时还保持容器运行，输入exit退出shell也会导致容器退出
				--privileged		使容器内的root具有真正的root权限，否则容器内的root只是普通用户权限
			start			启动已经存在的容器，被stop的
			exec			创建新的bash连接到运行中的容器，此时该bash输入exit将不会导致容器终止 
							docker exec -it xxx bash
			stop			手动停止容器，可用start再次启动容器
			pause			暂停容器内的所有进程
			unpause			恢复容器内的所有进程
			rm				删除容器
			inspect			显示容器的配置细节和运行时信息
			...				bash使用docker container查看



## 4、Dockerfile

**用来构建镜像的文本文件**

#### a、用途

- 对当前应用的描述
- 知道Docker完成应用的容器化（创建一个包含当前应用的镜像）

#### b、示例( Node.js )

```
from alpine
lable maintainer="nigelpoulton@hotmail.com"
run apk add --update node.js nodejs-npm
copy . /src
workdir /src
run npm install
expose 8080
entrypoint ["node","./app.js"]
```

- **from**       指定的镜像会作为基础镜像层，并且必须是第一条指令
- **lable**       为镜像指定标签，每个标签都是key-value形式，可通过标签添加自定义元数据，这里指定当前镜像的维护者为“nigelpoulton@hotmail.com”
- **run**          在from指定的基础镜像层之上，运行指定的命令，新建一个镜像层（每条run指令对应一条）来存储这些安装内容, **一般用来安装软件包**
- **copy**        从上下文目录（主机中的目录）中复制文件或者目录到容器里指定路径，会新建镜像层
- **workdir**  指定工作目录，会在构建镜像的每一层中都存在，该目录必须已经存在
- **expose**    只是让使用明白使用的端口，如果想使得容器与主机的端口有映射关系，必须在**容器启动**的时候加上 **-P 参数**指定端口映射关系
- **entrypoint**  启动时的默认命令，不会被docker run 的命令行参数指定的指令所覆盖，有多条时仅最后一条生效
- **cmd**               容器启动时执行的命令，会被docker run 的命令行参数指定的指令所覆盖，有多条时仅最后一条生效
- **env**                 设置环境变量， env  key1=value1  key2=value2
- **volume**          定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷;可实现挂载功能，可以将本地文件夹或者其他容器中的文件夹挂在到这个容器中



## 5、Image 镜像

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



## 6、容器

**容器是镜像的运行时实例，共享主机的操作系统/内核**

#### 重启策略

- always	除非通过stop明确停止，否则会一直尝试重启；当deamon重启的时候，停止的容器（即使使用stop）也会重启

- unless-stopped    处于stop停止，不会在deamon重启的时候被重启

- on-failed     在退出容器且返回值不是0的时候重启；就算被stop，在deamon重启的时候，也会被重启

  ```
  docker container run --name test -it --restart always alpine sh
  ```

#### 端口

```
docker container run -d --name webserver -p 80:8080 ...
```

**将Docker主机的80端口映射到容器内的8080端口**

Docker image inspect ...查看镜像，其中的**Cmd**展示了容器 **将会执行的命令（默认命令）或应用**



## 7、持久化数据

### 1、卷

##### 步骤

- 创建卷
- 创建容器
- 将卷挂载到容器的某个目录下

```
# 创建卷
docker volume create myvol
# 挂载到容器中
docker container run --mount myvol,target=/vol #若卷存在就使用，不存在则会自动创建卷并使用
```

### 2、挂载目录或文件

```
docker container run -v /home/test:/home/test1
```



## 8、Docker Compose

**在 Docker 节点上，以单引擎模式（Single-Engine Mode）进行多容器应用的部署和管理**

### 1、步骤

	1. 编写 `Dockerfile` 来定义应用的运行环境，让它能到处复制
	2. 使用YAML文件定义组成应用的服务，默认使用文件名`docker-comopose.yml`，也可以使用`-f`参数指定具体文件
	3. 运行命令`docker-compose up`启动compose来运行你的应用

```yml
version: "3.8" //指定Compose file格式版本，对应与Docker Engine release版本
services:  //定义不同的应用服务

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        max_replicas_per_node: 1
        constraints:
          - "node.role==manager"

  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - "5000:80"
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure

  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - "5001:80"
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints:
          - "node.role==manager"

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints:
          - "node.role==manager"

networks:
  frontend:
  backend:

volumes:
  db-data:
```



### 2、docker-compose.yml

​	服务定义包含应用于该服务启动的每个容器的配置，就像将命令行参数传递给`docker run`一样。 同样，网络和卷定义类似于`ocker network create`和`docker volume create`

​	与docker run一样，默认情况下会使用Dockerfile中指定的选项，例如`CMD`，`EXPOSE`，`VOLUME`，`ENV` , 无需在docker-compose.yml中再次指定它们。

#### build

​	构建时应用的配置选项，指定构建上下文路径

```yaml
version: "3.8"
services:
  webapp:
    build: ./dir
```

​	或作为一个对象，该对象具有 context 和指定的 dockerfile 文件以及 args 参数值

```yaml
version: "3.8"
services:
  webapp:
    build:
      context: ./dir  #指定Dockerfile文件所在的路径，可以是本机文件路径或者URL
      dockerfile: Dockerfile-alternate   #指定context指定的目录下的Dockerfile的名称
      args:   #Dockerfile 在 build 过程中需要的参数 (等同于 docker container build --build-arg 的作用),可以在Dockerfile中使用该参数值
        buildno: 1
        -buildno=1
       lables
```

#### 参数定义

```yaml
version           # 指定 compose 文件的版本
    services          # 定义所有的 service 信息, services 下面的第一级别的 key 既是一个 service 的名称

        build                 # 指定包含构建上下文的路径, 或作为一个对象，该对象具有 context 和指定的 dockerfile 文件以及 args 参数值
            context               # context: 指定 Dockerfile 文件所在的路径
            dockerfile            # dockerfile: 指定 context 指定的目录下面的 Dockerfile 的名称(默认为 Dockerfile)
            args                  # args: Dockerfile 在 build 过程中需要的参数 (等同于 docker container build --build-arg 的作用)
            cache_from            # v3.2中新增的参数, 指定缓存的镜像列表 (等同于 docker container build --cache_from 的作用)
            labels                # v3.3中新增的参数, 设置镜像的元数据 (等同于 docker container build --labels 的作用)
            shm_size              # v3.5中新增的参数, 设置容器 /dev/shm 分区的大小 (等同于 docker container build --shm-size 的作用)

        command               # 覆盖容器启动后默认执行的命令, 支持 shell 格式和 [] 格式

        configs               # 不知道怎么用

        cgroup_parent         # 不知道怎么用

        container_name        # 指定容器的名称 (等同于 docker run --name 的作用)

        credential_spec       # 不知道怎么用

        deploy                # v3 版本以上, 指定与部署和运行服务相关的配置, deploy 部分是 docker stack 使用的, docker stack 依赖 docker swarm
            endpoint_mode         # v3.3 版本中新增的功能, 指定服务暴露的方式
                vip                   # Docker 为该服务分配了一个虚拟 IP(VIP), 作为客户端的访问服务的地址
                dnsrr                 # DNS轮询, Docker 为该服务设置 DNS 条目, 使得服务名称的 DNS 查询返回一个 IP 地址列表, 客户端直接访问其中的一个地址
            labels                # 指定服务的标签，这些标签仅在服务上设置
            mode                  # 指定 deploy 的模式
                global                # 每个集群节点都只有一个容器
                replicated            # 用户可以指定集群中容器的数量(默认)
            placement             # 不知道怎么用
            replicas              # deploy 的 mode 为 replicated 时, 指定容器副本的数量
            resources             # 资源限制
                limits                # 设置容器的资源限制
                    cpus: "0.5"           # 设置该容器最多只能使用 50% 的 CPU 
                    memory: 50M           # 设置该容器最多只能使用 50M 的内存空间 
                reservations          # 设置为容器预留的系统资源(随时可用)
                    cpus: "0.2"           # 为该容器保留 20% 的 CPU
                    memory: 20M           # 为该容器保留 20M 的内存空间
            restart_policy        # 定义容器重启策略, 用于代替 restart 参数
                condition             # 定义容器重启策略(接受三个参数)
                    none                  # 不尝试重启
                    on-failure            # 只有当容器内部应用程序出现问题才会重启
                    any                   # 无论如何都会尝试重启(默认)
                delay                 # 尝试重启的间隔时间(默认为 0s)
                max_attempts          # 尝试重启次数(默认一直尝试重启)
                window                # 检查重启是否成功之前的等待时间(即如果容器启动了, 隔多少秒之后去检测容器是否正常, 默认 0s)
            update_config         # 用于配置滚动更新配置
                parallelism           # 一次性更新的容器数量
                delay                 # 更新一组容器之间的间隔时间
                failure_action        # 定义更新失败的策略
                    continue              # 继续更新
                    rollback              # 回滚更新
                    pause                 # 暂停更新(默认)
                monitor               # 每次更新后的持续时间以监视更新是否失败(单位: ns|us|ms|s|m|h) (默认为0)
                max_failure_ratio     # 回滚期间容忍的失败率(默认值为0)
                order                 # v3.4 版本中新增的参数, 回滚期间的操作顺序
                    stop-first            #旧任务在启动新任务之前停止(默认)
                    start-first           #首先启动新任务, 并且正在运行的任务暂时重叠
            rollback_config       # v3.7 版本中新增的参数, 用于定义在 update_config 更新失败的回滚策略
                parallelism           # 一次回滚的容器数, 如果设置为0, 则所有容器同时回滚
                delay                 # 每个组回滚之间的时间间隔(默认为0)
                failure_action        # 定义回滚失败的策略
                    continue              # 继续回滚
                    pause                 # 暂停回滚
                monitor               # 每次回滚任务后的持续时间以监视失败(单位: ns|us|ms|s|m|h) (默认为0)
                max_failure_ratio     # 回滚期间容忍的失败率(默认值0)
                order                 # 回滚期间的操作顺序
                    stop-first            # 旧任务在启动新任务之前停止(默认)
                    start-first           # 首先启动新任务, 并且正在运行的任务暂时重叠

            注意：
                支持 docker-compose up 和 docker-compose run 但不支持 docker stack deploy 的子选项
                security_opt  container_name  devices  tmpfs  stop_signal  links    cgroup_parent
                network_mode  external_links  restart  build  userns_mode  sysctls

        devices               # 指定设备映射列表 (等同于 docker run --device 的作用)

        depends_on            # 定义容器启动顺序 (此选项解决了容器之间的依赖关系， 此选项在 v3 版本中 使用 swarm 部署时将忽略该选项)
            示例：
                docker-compose up 以依赖顺序启动服务，下面例子中 redis 和 db 服务在 web 启动前启动
                默认情况下使用 docker-compose up web 这样的方式启动 web 服务时，也会启动 redis 和 db 两个服务，因为在配置文件中定义了依赖关系

                version: '3'
                services:
                    web:
                        build: .
                        depends_on:
                            - db      
                            - redis  
                    redis:
                        image: redis
                    db:
                        image: postgres                             

        dns                   # 设置 DNS 地址(等同于 docker run --dns 的作用)

        dns_search            # 设置 DNS 搜索域(等同于 docker run --dns-search 的作用)

        tmpfs                 # v2 版本以上, 挂载目录到容器中, 作为容器的临时文件系统(等同于 docker run --tmpfs 的作用, 在使用 swarm 部署时将忽略该选项)

        entrypoint            # 覆盖容器的默认 entrypoint 指令 (等同于 docker run --entrypoint 的作用)

        env_file              # 从指定文件中读取变量设置为容器中的环境变量, 可以是单个值或者一个文件列表, 如果多个文件中的变量重名则后面的变量覆盖前面的变量, environment 的值覆盖 env_file 的值
            文件格式：
                RACK_ENV=development 

        environment           # 设置环境变量， environment 的值可以覆盖 env_file 的值 (等同于 docker run --env 的作用)

        expose                # 暴露端口, 但是不能和宿主机建立映射关系, 类似于 Dockerfile 的 EXPOSE 指令

        external_links        # 连接不在 docker-compose.yml 中定义的容器或者不在 compose 管理的容器(docker run 启动的容器, 在 v3 版本中使用 swarm 部署时将忽略该选项)

        extra_hosts           # 添加 host 记录到容器中的 /etc/hosts 中 (等同于 docker run --add-host 的作用)

        healthcheck           # v2.1 以上版本, 定义容器健康状态检查, 类似于 Dockerfile 的 HEALTHCHECK 指令
            test                  # 检查容器检查状态的命令, 该选项必须是一个字符串或者列表, 第一项必须是 NONE, CMD 或 CMD-SHELL, 如果其是一个字符串则相当于 CMD-SHELL 加该字符串
                NONE                  # 禁用容器的健康状态检测
                CMD                   # test: ["CMD", "curl", "-f", "http://localhost"]
                CMD-SHELL             # test: ["CMD-SHELL", "curl -f http://localhost || exit 1"] 或者　test: curl -f https://localhost || exit 1
            interval: 1m30s       # 每次检查之间的间隔时间
            timeout: 10s          # 运行命令的超时时间
            retries: 3            # 重试次数
            start_period: 40s     # v3.4 以上新增的选项, 定义容器启动时间间隔
            disable: true         # true 或 false, 表示是否禁用健康状态检测和　test: NONE 相同

        image                 # 指定 docker 镜像, 可以是远程仓库镜像、本地镜像

        init                  # v3.7 中新增的参数, true 或 false 表示是否在容器中运行一个 init, 它接收信号并传递给进程

        isolation             # 隔离容器技术, 在 Linux 中仅支持 default 值

        labels                # 使用 Docker 标签将元数据添加到容器, 与 Dockerfile 中的 LABELS 类似

        links                 # 链接到其它服务中的容器, 该选项是 docker 历史遗留的选项, 目前已被用户自定义网络名称空间取代, 最终有可能被废弃 (在使用 swarm 部署时将忽略该选项)

        logging               # 设置容器日志服务
            driver                # 指定日志记录驱动程序, 默认 json-file (等同于 docker run --log-driver 的作用)
            options               # 指定日志的相关参数 (等同于 docker run --log-opt 的作用)
                max-size              # 设置单个日志文件的大小, 当到达这个值后会进行日志滚动操作
                max-file              # 日志文件保留的数量

        network_mode          # 指定网络模式 (等同于 docker run --net 的作用, 在使用 swarm 部署时将忽略该选项)         

        networks              # 将容器加入指定网络 (等同于 docker network connect 的作用), networks 可以位于 compose 文件顶级键和 services 键的二级键
            aliases               # 同一网络上的容器可以使用服务名称或别名连接到其中一个服务的容器
            ipv4_address      # IP V4 格式
            ipv6_address      # IP V6 格式

            示例:
                version: '3.7'
                services: 
                    test: 
                        image: nginx:1.14-alpine
                        container_name: mynginx
                        command: ifconfig
                        networks: 
                            app_net:                                # 调用下面 networks 定义的 app_net 网络
                            ipv4_address: 172.16.238.10
                networks:
                    app_net:
                        driver: bridge
                        ipam:
                            driver: default
                            config:
                                - subnet: 172.16.238.0/24

        pid: 'host'           # 共享宿主机的 进程空间(PID)

        ports                 # 建立宿主机和容器之间的端口映射关系, ports 支持两种语法格式
            SHORT 语法格式示例:
                - "3000"                            # 暴露容器的 3000 端口, 宿主机的端口由 docker 随机映射一个没有被占用的端口
                - "3000-3005"                       # 暴露容器的 3000 到 3005 端口, 宿主机的端口由 docker 随机映射没有被占用的端口
                - "8000:8000"                       # 容器的 8000 端口和宿主机的 8000 端口建立映射关系
                - "9090-9091:8080-8081"
                - "127.0.0.1:8001:8001"             # 指定映射宿主机的指定地址的
                - "127.0.0.1:5000-5010:5000-5010"   
                - "6060:6060/udp"                   # 指定协议

            LONG 语法格式示例:(v3.2 新增的语法格式)
                ports:
                    - target: 80                    # 容器端口
                      published: 8080               # 宿主机端口
                      protocol: tcp                 # 协议类型
                      mode: host                    # host 在每个节点上发布主机端口,  ingress 对于群模式端口进行负载均衡

        secrets               # 不知道怎么用

        security_opt          # 为每个容器覆盖默认的标签 (在使用 swarm 部署时将忽略该选项)

        stop_grace_period     # 指定在发送了 SIGTERM 信号之后, 容器等待多少秒之后退出(默认 10s)

        stop_signal           # 指定停止容器发送的信号 (默认为 SIGTERM 相当于 kill PID; SIGKILL 相当于 kill -9 PID; 在使用 swarm 部署时将忽略该选项)

        sysctls               # 设置容器中的内核参数 (在使用 swarm 部署时将忽略该选项)

        ulimits               # 设置容器的 limit

        userns_mode           # 如果Docker守护程序配置了用户名称空间, 则禁用此服务的用户名称空间 (在使用 swarm 部署时将忽略该选项)

        volumes               # 定义容器和宿主机的卷映射关系, 其和 networks 一样可以位于 services 键的二级键和 compose 顶级键, 如果需要跨服务间使用则在顶级键定义, 在 services 中引用
            SHORT 语法格式示例:
                volumes:
                    - /var/lib/mysql                # 映射容器内的 /var/lib/mysql 到宿主机的一个随机目录中
                    - /opt/data:/var/lib/mysql      # 映射容器内的 /var/lib/mysql 到宿主机的 /opt/data
                    - ./cache:/tmp/cache            # 映射容器内的 /var/lib/mysql 到宿主机 compose 文件所在的位置
                    - ~/configs:/etc/configs/:ro    # 映射容器宿主机的目录到容器中去, 权限只读
                    - datavolume:/var/lib/mysql     # datavolume 为 volumes 顶级键定义的目录, 在此处直接调用

            LONG 语法格式示例:(v3.2 新增的语法格式)
                version: "3.2"
                services:
                    web:
                        image: nginx:alpine
                        ports:
                            - "80:80"
                        volumes:
                            - type: volume                  # mount 的类型, 必须是 bind、volume 或 tmpfs
                                source: mydata              # 宿主机目录
                                target: /data               # 容器目录
                                volume:                     # 配置额外的选项, 其 key 必须和 type 的值相同
                                    nocopy: true                # volume 额外的选项, 在创建卷时禁用从容器复制数据
                            - type: bind                    # volume 模式只指定容器路径即可, 宿主机路径随机生成; bind 需要指定容器和数据机的映射路径
                                source: ./static
                                target: /opt/app/static
                                read_only: true             # 设置文件系统为只读文件系统
                volumes:
                    mydata:                                 # 定义在 volume, 可在所有服务中调用

        restart               # 定义容器重启策略(在使用 swarm 部署时将忽略该选项, 在 swarm 使用 restart_policy 代替 restart)
            no                    # 禁止自动重启容器(默认)
            always                # 无论如何容器都会重启
            on-failure            # 当出现 on-failure 报错时, 容器重新启动

        其他选项：
            domainname, hostname, ipc, mac_address, privileged, read_only, shm_size, stdin_open, tty, user, working_dir
            上面这些选项都只接受单个值和 docker run 的对应参数类似

        对于值为时间的可接受的值：
            2.5s
            10s
            1m30s
            2h32m
            5h34m56s
            时间单位: us, ms, s, m， h
        对于值为大小的可接受的值：
            2b
            1024kb
            2048k
            300m
            1gb
            单位: b, k, m, g 或者 kb, mb, gb
    networks          # 定义 networks 信息
        driver                # 指定网络模式, 大多数情况下, 它 bridge 于单个主机和 overlay Swarm 上
            bridge                # Docker 默认使用 bridge 连接单个主机上的网络
            overlay               # overlay 驱动程序创建一个跨多个节点命名的网络
            host                  # 共享主机网络名称空间(等同于 docker run --net=host)
            none                  # 等同于 docker run --net=none
        driver_opts           # v3.2以上版本, 传递给驱动程序的参数, 这些参数取决于驱动程序
        attachable            # driver 为 overlay 时使用, 如果设置为 true 则除了服务之外，独立容器也可以附加到该网络; 如果独立容器连接到该网络，则它可以与其他 Docker 守护进程连接到的该网络的服务和独立容器进行通信
        ipam                  # 自定义 IPAM 配置. 这是一个具有多个属性的对象, 每个属性都是可选的
            driver                # IPAM 驱动程序, bridge 或者 default
            config                # 配置项
                subnet                # CIDR格式的子网，表示该网络的网段
        external              # 外部网络, 如果设置为 true 则 docker-compose up 不会尝试创建它, 如果它不存在则引发错误
        name                  # v3.5 以上版本, 为此网络设置名称
```

