# Linux-Docker-Notes
### Introduction
通过下载安装CentOS,Linux，以及在Linux上安装docker，基于docker compose 实现多容器的部署和管理的过程，有很多收获，记录在此。
### 安装VirtualBox
VMware适合需要高性能虚拟化体验的专业人士和企业用户‌，VirtualBox适合个人用户和开发者使用，VirtualBox免费开源，所以选用VirtualBox安装虚拟机。

安装VirtualBox时，默认的安装路径是C盘，如果想更改路径需要进行以下操作：

**win + R** 输入**cmd**打开控制台，以安装到D盘为例，依次运行以下代码：

~~~
icacls D:\VirtualBox /reset /t /c
icacls D:\VirtualBox /inheritance:d /t /c
icacls D:\VirtualBox /grant *S-1-5-32-545:(OI)(CI)(RX) /t /c
icacls D:\VirtualBox /deny *S-1-5-32-545:(DE,WD,AD,WEA,WA) /t /c
icacls D:\VirtualBox /grant *S-1-5-11:(OI)(CI)(RX) /t /c
icacls D:\VirtualBox /deny *S-1-5-11:(DE,WD,AD,WEA,WA) /t /c
~~~

**注意**：虽然安装路径变为D盘，但C盘中依然有VirtualBox,卸载重装时必须将C盘的残留文件删除，才能正常安装使用。
### 安装CentOS7.9
创建一个磁盘，名为CentOS的虚拟机。注意，放在合适的位置
https://mirrors.aliyun.com/centos/7.9.2009/isos/x86_64/ <br>
下载CentOS-7-x86_64-DVD-2207-02.iso <br>
在虚拟机的光驱，引入下载的CentOS镜像 <br>
运行虚拟机，进入centos安装模式<br>
打开网络功能，默认的自动分配IP<br>
安装位置，无需分区<br>
安装基础服务器版，后续软件包可以再添加<br>
安装过程中，设置root账号密码，创建一个普通操作权限的用户/密码<br>

root用户是CentOS系统中的超级用户，拥有系统的最高权限。root用户可以执行任何操作，包括对系统文件和配置的修改，安装和卸载软件等。root用户可以进入任何目录，对任何文件都有读权限，包括访问敏感文件如/etc和/var等。此外，root用户可以使用系统命令sbin目录下的程序，进行硬件监控和数据获取等操作，不受系统软硬件限制，如磁盘空间和内存使用等‌12。<br>

普通用户则拥有有限的权限。普通用户通常由系统管理员创建，用于执行普通任务和日常操作。他们只能访问自己的用户目录和其他被授权的文件和目录。普通用户不能修改系统级配置和安装软件，只能使用/bin和/usr/bin目录下的命令。由于普通用户权限受限，他们的操作对系统的影响较小，有助于保护系统免受用户错误、恶意软件和安全漏洞的影响‌。<br>

切换用户方面，普通用户可以通过输入su命令并输入root密码来切换到root用户。相反，root用户可以通过输入su 用户名来切换回普通用户。使用sudo命令也可以让普通用户以root身份执行特定命令，但不会直接切换到root用户。注意，输入密码时终端不显示密码字符‌<br>
安装完毕，光驱卸载iso，重启<br>
查看网络是否正常，ping通百度<br>
停止操作快捷键 **Ctrl + C**<br>
Linux一些快捷键及其功能:
- Tab键‌：这是最基本的自动补全快捷键。输入命令或文件名的前几个字母时，按下Tab键，系统会自动补全或列出匹配的选项，只需选择正确的选项即可。如果按一次Tab键没有唯一匹配的选项，可以连续按两次Tab键来列出所有可能的选项。
- Ctrl+A‌：将光标移动到命令行的开始位置。
- Ctrl+E‌：将光标移动到命令行的末尾。
- Ctrl+U‌：删除从光标位置到行首的所有内容。
- Ctrl+K‌：删除从光标位置到行尾的所有内容。
- Ctrl+W‌：删除光标前的单词。
- Ctrl+Y‌：将之前删除的内容贴回命令行。
- Ctrl+L‌：清屏，类似于clear命令。
- Ctrl+D‌：关闭终端会话。
- Ctrl+C‌：终止当前命令。
- Ctrl+R‌：在命令历史中搜索命令，类似于history命令的搜索功能。
- 上下键切换命令
### VB网络地址转换(NAT)模式端口映射
桥接和NAT的区别：<br>
桥接模式（Bridged Networking）是一种网络连接方式，它将不同的网络设备连接在一起，使它们可以互相通信。在桥接模式下，虚拟机被视为局域网中的一台独立主机，能够访问局域网中的任何设备。用户需要手动为虚拟机配置IP地址、子网掩码，并确保虚拟机与宿主机器处于同一网段，这样才能实现虚拟机和宿主机器之间的通信。桥接模式的优点是灵活性高，虚拟机可以直接提供网络服务，但需要确保局域网内有足够的IP地址供虚拟机使用‌。<br>
NAT模式（Network Address Translation）则是一种网络地址转换技术，通过转换网络地址实现内部网络与外部网络之间的通信。在NAT模式下，虚拟机的TCP/IP配置信息由宿主机的DHCP服务器提供，用户无法手动修改这些设置。这种模式的优点是设置简单，不需要额外的IP地址，但缺点是虚拟机不能作为服务器提供服务，因为没有自己的公网IP地址‌。<br>
**鉴于需登录的校园网络特点，网络模式使用默认网络地址转换(NAT)模式**<br>
从虚拟机切出鼠标/键盘的控制的快捷键: **右侧Ctrl键**
### 安装Bitvise SSH Client，连接虚拟服务器安装
在VB网络设置中，创建一个宿主机到虚拟机的ssh端口映射，将宿主机10022端口映射到虚拟机22端口。<br>
为什么通过ssh连接服务器，而不直接在虚拟机中操作？<br>
1. 安全性‌：SSH使用公钥加密来验证远程计算机和用户身份，确保数据传输的安全性。它提供了数据完整性、加密和验证，防止未经授权的访问‌。
2. ‌灵活性‌：通过SSH连接，用户可以在任何有网络连接的地方访问虚拟机，不受物理位置的限制。这在使用多台设备或需要在不同地点工作时非常方便‌。
3. 资源利用‌：通过SSH连接，可以充分利用主机的计算资源和网络连接，而不需要在每台设备上都安装和配置虚拟机环境‌。
4. 多任务处理‌：用户可以在主机上同时运行多个任务和应用程序，而不需要在虚拟机中单独管理每个任务，提高了工作效率‌。 

使用SSH连接服务器时，需要注意以下几点：<br>
1. ‌网络配置‌：确保虚拟机的网络设置正确，包括IP地址、子网掩码、网关等配置正确‌。
2. 防火墙设置‌：确保虚拟机的防火墙允许SSH连接，或者添加规则允许特定的端口和IP地址进行通信‌。
3. SSH服务状态‌：检查虚拟机上的SSH服务是否已经启动，如果没有启动需要手动启动服务‌。
4. 认证方式‌：确保SSH客户端连接时使用的认证方式（如密码、证书等）与虚拟机上的配置一致‌。

如果能够正确进入服务器，则可以在VB中为虚拟机创建一个系统快照作为基础镜像， 后续操作出问题，可以回滚到当前版本。也可直接删虚拟机重来。
### Linux基本命令
- cd 目录名 ： 进入目录
- cd /home ：进入home目录
- cd .. : 返回上级目录
- cd : 返回根目录
- mkdir 目录名 ： 创建目录
- mkdir -p 目录名 ： 创建多级目录
- touch 文件名 ： 创建文件
- cat 文件名 ： 查看文件
- rm -r 目录名 ： 删除目录
- rm 文件名 ： 删除文件
- mv 旧名 新名 ： 重命名
- 查看系统版本 ： cat /etc/\*release*
- 查看系统内核 ： uname -a
- cpu/内存占用 ： top
- 磁盘占用 ： df -h 或 lsblk
- vi 文件名 ： （创建）进入编辑文件
- i ：插入模式
- Esc:wq : 保存退出
- Esc:q : 不保存强制退出
- 查看IP地址 ： ip addr（windows:查看IP地址:ipconfig 查看所有运行端口：netstat -ano 查看被占用端口对应的 PID：netstat -aon|findstr "8081"）

在Linux中，查看目录中的文件可以使用ls命令。以下是一些常用的ls命令选项：
- ls：列出当前目录中的文件和子目录。
- ls -a：列出所有文件和子目录，包括隐藏文件（文件名以.开头的文件）。
- ls -l：以长格式列出文件详细信息（包括权限、所有者、大小和修改日期）。
- ls -lh：以人类可读的格式（如用K、M、G等表示大小）显示长格式列表。
- ls -la：结合了-a和-l的功能，即列出所有文件（包括隐藏文件）并以长格式显示。
- ls -d */：只列出所有子目录。
- ls -S：根据文件大小排序。
- ls -t：根据修改时间排序。
- ls -R：递归地列出所有子目录中的文件。
### Linux系统中目录的内容详解
[csdn参考](https://blog.csdn.net/mathlxj/article/details/106340541) <br>
补充：/usr/libexec/ 目录通常包含一些不应由用户直接运行的系统服务和工具，而是由其他系统进程调用。
### 安装Docker
Docker是一个开源的应用容器引擎，它基于操作系统层级的虚拟化技术，将软件与其依赖项打包为容器。这些容器可以在任何安装了Docker引擎的服务器上运行，包括流行的‌Linux机器和‌Windows机器。<br>
它与传统的虚拟化方法有显著的不同。Docker 容器不是完整的操作系统，而是对应用及其依赖的封装。<br>
主要的区别在于：
1. Docker 利用操作系统的内核，而传统虚拟化方法（如 VMware 或 KVM）创建了一个完整的操作系统堆栈。
2. Docker 容器的启动速度快，占用的空间小，而传统虚拟机通常需要更多资源来启动，并且占用更多的磁盘空间。
3. Docker 容器是轻量级的，它们可以在相同的操作系统内运行，并且彼此隔离。而传统虚拟机在各自的操作系统内运行，并且通常需要额外的管理程序来进行隔离。<br>

Docker 的主要优势包括：
- 快速部署和启动应用
- 更高的计算密度（在同一硬件上可以运行更多的容器）
- 简化了配置和管理应用的生命周期
- 更高的系统安全性（通过隔离机制）
- 更高的资源利用率（容器在需要时才启动）

[Docker命令大全（参考csdn）](https://blog.csdn.net/Aaaaaaatwl/article/details/140149394)

安装教程 https://docs.docker.com/engine/install/centos/ <br>
CentOs 9 安装docker：
<li>卸载旧版本</li>

```
sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
<li>安装 yum-utils，它提供了 yum-config-manager，用于管理 yum 软件源</li>

```
yum install -y yum-utils
```
<li>添加 Docker CE 软件源 （※为了加快速度，此处配置了国内的阿里镜像源）</li>

```
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
<li>安装 Docker CE</li>

```
yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
<li>启动 Docker</li>

```
systemctl start docker
```
sudo yum install -y yum-utils 报错问题 https://blog.csdn.net/weixin_43490087/article/details/141924032<br>
获取密钥失败问题 多运行几次命令<br>
hello world运行失败或超时 基于vi修改配置文件 配置加速地址：
~~~
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
        "https://ustc-edu-cn.mirror.aliyuncs.com/",
        "https://ccr.ccs.tencentyun.com/",
        "https://docker.m.daocloud.io/"
    ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
systemctl status docker
~~~
### Docker Compose
docker已默认集成docker compose。
[教程](https://www.runoob.com/docker/docker-compose.html?spm=a2c6h.13046898.publish-article.66.78fc6ffaIUcpyO)<br>
Orchestration System（编排系统）是一种用于协调和管理多个服务、应用程序或系统之间的交互和工作流的技术。它在云计算、微服务架构和容器化环境中尤为重要，目的是提高自动化、效率和可管理性。以下是一些关键方面：

1. 功能与目的
   自动化：编排系统可以自动化重复的任务，如部署、配置和监控，减少人工干预。
   协调工作流：通过定义任务的执行顺序和依赖关系，确保各个组件能有效协同工作。
   资源管理：优化资源的使用，确保负载均衡和高可用性。
2. 应用场景
   容器编排：例如，Kubernetes 是一种流行的容器编排工具，用于管理容器的部署、扩展和操作。
   云服务管理：编排系统能够管理多个云服务提供商之间的资源和服务，简化云架构的管理。
   工作流自动化：在数据处理、软件开发等领域，编排系统可以自动化复杂的工作流程，提高效率。
3. 关键技术
   API集成：通过API与各种服务和应用进行集成，便于数据交换和操作。
   事件驱动：基于事件的架构，使得系统能够响应不同事件并自动触发相应的任务。
   监控与反馈：实时监控系统状态，及时调整资源和工作流。
4. 优点
   提高效率：减少人工操作，提高任务执行的速度和准确性。
   灵活性：能够快速适应变化的需求和技术环境。
   可扩展性：支持大规模系统的管理，便于添加新服务或组件。
5. 挑战
   复杂性：随着系统规模的扩大，编排的复杂性也会增加。
   故障排除：在分布式系统中，定位和解决故障可能变得更加困难。
总之，Orchestration System 是现代IT基础设施中不可或缺的一部分，特别是在云计算和微服务的背景下，其重要性日益凸显。<br>

需要Docker Compose的原因是因为它极大地简化了多容器应用的部署和管理过程。Docker Compose的主要用途包括简化多容器应用的部署和管理、快速搭建开发环境、调试和测试应用，以及管理生产环境。通过在一个文件中定义多个容器及其依赖关系和配置参数，Docker Compose帮助用户避免了在本地安装和配置各种依赖的繁琐过程，提高了开发效率。此外，它还允许用户通过简单的命令启动、停止和重建整个应用环境，进一步简化了多容器应用的部署和管理流程。Docker Compose适合于单主机或多主机（但非集群化）环境下的应用部署，适用于开发、测试及小型生产环境。尽管Docker本身已经足够强大，能够用来部署任何应用程序，但当面对由多个容器组成的服务时，如一个web应用和它的数据库，Docker Compose能够方便地定义、运行这些容器，并确保它们之间正确地连接，这一点在开发和测试环境中尤为重要，但也可以应用于生产环境，尤其是需要快速部署和扩展服务时‌.<br>

Kubernetes (K8s) 和 Docker Compose 都是容器编排工具，但它们有不同的应用场景和优势。

Kubernetes:
- 适用于大规模和复杂的集群管理。
- 提供了更多的高级功能，如服务发现、负载均衡、存储管理等。
- 设计理念是Declarative（声明式），你可以通过配置文件来描述集群的期望状态。
- 适用于多节点的集群和生产环境。

Docker Compose:
- 适用于单节点上的容器编排。
- 简单易用，主要用于开发和测试环境。
- 可以通过Dockerfile定义容器的构建过程，通过docker-compose.yml定义服务和网络。
- 用于管理容器的集合，而非整个分布式系统。
- docker已默认集成docker compose。

编写Docker Compose文件的主要特点是使用YAML格式来定义和运行由多个容器构成的应用。<br>
主要特点包括：
- 使用版本化的配置格式。
- 使用服务、网络和卷的抽象概念。
- 可以同时启动多个容器作为服务。
- 支持环境变量和参数化配置。
- 易于扩展和维护

YAML是一种直观的数据序列化格式，旨在以人类可读的方式表达数据，同时便于与脚本语言交互。YAML文件通常用于配置文件、数据交换和描述性数据的表示，其设计目标包括人类容易阅读、适用于不同程序间的数据交换、适合描述程序所使用的数据结构，特别是脚本语言。YAML文件以数据为核心，比传统的XML方式更加简洁，文件的扩展名可以使用.yml或.yaml。‌
YAML的基本语法包括以下几点：
- 大小写敏感‌：YAML对大小写敏感。
- 缩进表示层级关系‌：使用缩进（通常是两个或四个空格）来表示数据的层级关系，同一层级的元素左侧对齐。不允许使用Tab键进行缩进。
- 注释‌：使用#符号进行单行注释，从该字符到行尾的内容都会被解析器忽略。
- ‌标量‌：单个的、不可再分的值，如字符串、数值、布尔值或null。
- 序列‌：一组按次序排列的值，也称为列表或数组。
- 映射‌：键值对的集合，也称为字典或哈希表。

YAML文件可以由一个或多个文档组成，文档间使用---（三个横线）作为分隔符，每个文档也可以使用...（三个点号）作为结束符（可选）。YAML能够描述比JSON更加复杂的结构，例如“关系锚点”可以表示数据引用（如重复数据的引用）。
#### MySQL
创建mysql服务目录，在win编写docker compose脚本，拉取mysql:8镜像；挂载宿主机相对目录下./data/到容器数据存储区；声明时区；声明默认root密码；映射linux宿主机与容器端口。
复制到服务器执行。能通过win宿主机的idea database客户端连接到容器的mysql数据库。<br>
#### Tomcat
idea创建一个基于java:21 + tomcat:10的maven web项目，仅包含测试主页；编译构建打包，获取war文件。<br>
支撑java:21 + tomcat:10的tomcat镜像拉取 : **tomcat:10.1-jdk21** <br>
创建tomcat服务目录及资源目录；编写编排脚本，映射端口，时区。<br>
挂载宿主资源目录到容器tomcat默认工作目录。复制脚本/war包运行，宿主机通过IP映射端口访问。<br>
**win浏览器访问127.0.0.1:18080/tomcat-mysql-test-hemy-1.0-SNAPSHOT/**
### MySQL + Tomcat
创建web-project服务目录，编写脚本整合mysql+tomcat2个子服务；<br>
tomcat需访问mysql，在docker-compose脚本中tomcat部分声明依赖mysql。<br>
扩展以上maven web项目配置；添加JDBC依赖/数据源配置，数据源地址是jdbc:mysql://mysql:3306/mysql_database<br>
**服务内Tomcat子服务的容器需要访问子服务MySQL，属于同一网络，因此只需要通过3306端口就可以访问MySQL。**<br>
初始化数据库,重新打包部署到服务器，创建容器测试。<br>
命令查看tomcat容器内输出，是否启动正常。<br>
编写tomcat依赖mysql服务健康监测情况启动配置，重新创建容器测试。





