## Docker And Swarm Mode

#### 前言

&nbsp;虽然工作中一直在用Docker和Docker Swarm，但是总感觉有点陌生，总想自己亲手来写写和配置Docker 容器相关的事情，这篇文章主要是参考了[Los Techies](https://lostechies.com/gabrielschenker/2016/08/26/containers-an-index/) 中关于Docker 和 Docker Swarm 的一些内容，然后自己做一个尝试和练习。

&nbsp;本文主要讲的就是使用VirtualBox 来建立 Docker node ，然后组合成 Docker Swarm。

#### 环境配置

因为之前一直是在Linux的环境下操作的，这次是想在家里的Windows上用Docker来试试的。由于Hyper-V啦，Virtualization等各类问题，也可能是对Docker 的不熟悉，折腾了两个晚上之后放弃了，最后决定在Mac上进行操作。

Docker 及其版本:

> 1. Docker：Docker version 17.12.0-ce, build c97c6d6
> 2. Docker-machine: docker-machine version 0.13.0, build 9ba6da9
> 3. Docker-compose:docker-compose version 1.18.0, build 8dd22a9

然后还需要安装[Virtualbox](https://www.virtualbox.org) 来模拟多个服务器，根据自己机器的环境安装对应的版本。

#### 创建节点

```shell
for N in 1 2 3 4 5;
do docker-machine create --driver virtualbox node$N;
done
```

可能需要花费一点时间，然后使用

`docker-machine ls`

> 接下来就会看到下面的这张图。
>
> ![](https://ws1.sinaimg.cn/large/006tNc79gy1fpkt7lz47qj314e06m0uj.jpg)

如果我们要进入某个node，则需要执行`docker-machine ssh node1`

> ![](https://ws3.sinaimg.cn/large/006tNc79gy1fpkt9q97puj31420fmdho.jpg)

之前一直卡在docker-compose上面，因为我在Windows上一直无法安装，使用`docker pull docker/compose:1.8.0` 没想到这个坑，这个是之前的，目前只要安装好docker 的Mac 或者Windows客户端就自动安装好了 docker-machine 和docke-compose

#### 开始Swarmkit

&nbsp;做好上面的准备之后，我们已经可以开始创建Docker Swarm 了，Docker ,理解Swarm clusters：

> A swarm is a group of machines that are running Docker and joined into a cluster.. After that has happened, you continue to run the Docker commands you’re used to, but now they are executed on a cluster by a **swarm manager**. The machines in a swarm can be physical or virtual. After joining a swarm, they are referred to as **nodes**.

接下来我们就创建一个Swarm,

1. 首先进行node1 `docker-machine ssh node1`

2. 执行 `docker swarm init --advertise-addr [node1-ip-address]` 比如我的node1 的ip地址是`192.168.99.100`，则命令为 `docker swarm init --advertise-addr 192.168.99.100` 执行完之后就会看到控制台：

   > ```shell
   > $ docker swarm init --advertise-addr 192.168.99.100 
   > Swarm initialized: current node (ddec2zwjaes2kpkmnshkmpstq) is now a manager.
   >
   > To add a worker to this swarm, run the following command:
   >     docker swarm join --token SWMTKN-1-4kxfjieyyjdab91c9sxunj75ivke51y4geoucu7g4cvl7g2p1u-7m00eooi3obknk2cjx8zexkea 192.168.99.100:2377
   >     
   >
   > To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
   >
   > ```

3. 执行完这个命令后，node1 就成为了leader，你可以依次进入每个容器然后执行 `docker swarm join --token SWMTKN-1-4kxfjieyyjdab91c9sxunj75ivke51y4geoucu7g4cvl7g2p1u-7m00eooi3obknk2cjx8zexkea 192.168.99.100:2377` 加入到Swarm中。

4. 执行完上述命令后，则我们有一个5个node 的Swarm了。一个node1 leader 节点，然后4个worker 节点。

   > In a production environment we should have at least 3 master nodes for high availability since master nodes store all the information about the swarm and its state.


5. 我们可以通过 `docker node promote node2 node3 ` 来把worker 节点提升为leader节点

   > ![](https://ws3.sinaimg.cn/large/006tNc79gy1fpktthvyuej3146076407.jpg)

关于这个Reachable是什么意思呢？当我们现在的node2 节点由于某种原因挂了，或者停止了，那么在node1和node3中就会自动选择一个当作leader，然后等node2恢复后，就变成了Reachable状态了。

使用 `docker-machine stop node1` 来停止节点

#### Service

&nbsp;有了上面关于的那些基础设施后，接下来可以了解Service了，在了解早起爱你，我们先来说说Node,个人感觉node就像一台服务器一样，然后加入到Swarm里面，一堆的node然后就可以组建成一个Swarm。

> A node is an instance of Docker Engine participating in a swarm.

在Swarm里面，node 有两种形式，一种是 worker，另一种是manager。woker的权限就少的有点可怜了，连`docker service ls` 都看不了。

&nbsp;说起Service服务，首先它是跑在node上面的，我觉得吧它就像比如一个应用，比如登陆站点服务。

> A service is the definition of the tasks to execute on the worker nodes. It is the central structure of the swarm system and the primary root of user interaction with the swarm.

> ![](https://ws2.sinaimg.cn/large/006tKfTcgy1fplx6bnzhej31jk06qwfu.jpg)

> Services are really just “containers in production.” A service only runs one image, but it codifies the way that image runs—what ports it should use, how many replicas of the container should run so the service has the capacity it needs, and so on. Scaling a service changes the number of container instances running that piece of software, assigning more computing resources to the service in the process.

使用 `docker-compose.yml` 来定义，运行和配置服务

> `docker rmi $(docker images -q -f dangling=true)`

#### Docker使用过程中的一些命令

`docker ps` / `docker container ls` 查看当前容器列表

`docker image ls` / `docker images` 查看当前镜像列表

`docker build -t imageName .` 建立镜像命令

`docker run -p 4000:80 imageName` 运行容器命令

`docker run -d -p 4000:80 imageName` 后台运行容器命令

`docker container stop containerId` 停止容器命令

`docker swarm init ` 初始化swarm

`docker stack deploy -c docker-compose.yml appName` 建立名为appName 的App Stack，若调整了docker-compose.yml ，也可用该命令进行站点的更新

`docker service ls` 查看当前运行的服务的信息

`docker service ps servicename` 列出当前服务所运行的节点及其状态

`docker stack rm appName` 下线名为appName 的app 

`docker swarm leave --force` 离开Swarm

`docker-machine ls` 列出当前的虚拟机信息

`docker-machine ssh machine_name` 进入名为 machine_name 的虚拟机中，如果无法进行，则使用`docker-machine --native-ssh ssh machine_name`

`docker node ls` 查看当前Swarm的节点信息，需要在Swarm 中执行该命令，否则报错

> 这两个命令和起来很有用，相当于附加到当前的Shell控制台上，可以不进入虚拟机，然后直接操作该虚拟机。
>
> `docker-machine env <machine>` 
>
> `eval $(docker-machine env <machine>)`
>
> get the command to configure your shell to talk to `<machine>`. 
>
> eval $(docker-machine env -u) 取消绑定



> 当镜像是存储在私人Registry上时，这需要 login 然后deploy时加上`--with-registry-auth` 命令
>
> `docker login registry.example.com`
>
> `docker stack deploy --with-registry-auth -c docker-compose.yml appName` 

> ![](https://ws1.sinaimg.cn/large/006tNc79gy1fpr749p9kij31kw0zsjy9.jpg)







