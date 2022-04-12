

# 前后分离项目记录

## 1.建立数据库表

```mysql
create table ums_user(
id bigint primary key auto_increment,
name varchar (20) not null comment '用户姓名',
phone varchar(11) not null comment '手机号',
email varchar(64) not null comment '用户邮箱',
icon varchar(255) not null comment '用户头像',
active tinyint default 1 not null comment '状态'
)engine=innodb comment '用户表';
```

![image-20220324203530636](前后分离.assets/image-20220324203530636.png)



## 2.vue脚手架

```xml
1.安装 node 和 git ，node需要安装长期维护班

2.win + r cmd 回车

3.查看npm的版本号 （npm 是前端的项目构建工，类型java中的maven）
#npm-v

4.如果不是6版本，尽量降级到6版本

5.安装cnpm
#npm install -g cnpm

6.安装vue脚手架 cnpm
#cnpm install -g @vue/cli@3.10.0

7.安装后校验
#vue-V

--------------------------------以上步骤每台电脑只安装一次------------------------------

1.将目录切入到要创建的目录中

2.创建项目 # vue create xxx   xxx是自己的项目名
? Please pick a preset: 选择预设
default (babel, eslint) 默认值
Manually select features 手动选择功能 --选这个 

3. 选择以下四个
? Please pick a preset: Manually select features
? Check the features needed for your project: (Press <space> to select, <a> to toggle all, <i> to invert selection)
(*) Babel
( ) TypeScript
( ) Progressive Web App (PWA) Support
(*) Router
(*) Vuex 
(*) CSS Pre-processors
( ) Linter / Formatter  ---这个千万别选 否则会有格式检查
( ) Unit Testing
( ) E2E Testing

4. 选择 y
? Please pick a preset: Manually select features
? Check the features needed for your project: Babel, Router, Vuex, CSS Pre-processors 
? Use history mode for router? (Requires proper server setup for index fallback in production) (Y/n) 直接回车

5. Less  ----选这个
Sass/SCSS (with dart-sass)
Sass/SCSS (with node-sass)
Less  ----选这个
Stylus

6. 这两个选哪个都行直接回车
? Where do you prefer placing config for Babel, PostCSS, ESLint, etc.? (Use arrow keys)
In dedicated config files
In package.json

7. 直接回车
? Save this as a preset for future projects? (y/N)

8. 有的话选择npm

9.切入到项目目录
#cd 项目名

10.添加axios
#vue add axios

11.添加element ui
#vue add element

12.是否全部引入
? How do you want to import Element? (Use arrow keys)
> Fully import ----图省事选这个
  Import on demand

13.是否添加css预编译
? Do you wish to overwrite Element's SCSS variables? (y/N) 直接回车 是N 上面已经引入过了

```

## 3.搭建后端微服务架构

（父子项目 都是maven项目）

### 3.1父项目(parent)

左边是cloud版本，右边是boot版本 --- 2.2，2.3  ==springboot官网==

<img src="前后分离.assets/image-20220408125104988.png" alt="image-20220408125104988" style="zoom:50%;" />

==导入阿里巴巴cloud==

<img src="前后分离.assets/image-20220408125749776.png" alt="image-20220408125749776" style="zoom:50%;" />

### 3.2父项目的pom文件

```xml
<packaging>pom</packaging> 打包方式是pom 

    <properties>
        <spring-boot.version>2.6.6</spring-boot.version> 1  1和2要看上面的图对应版本
        <spring-cloud.version>2021.0.1</spring-cloud.version> 2 
        <spring-cloud-alibaba.version>2021.0.1.0</spring-cloud-alibaba.version> 
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

        </dependencies>
    </dependencyManagement>
```

## 4.子项目创建

==admin项目创建==

* pom导入web依赖 版本号不用写，因为在父项目中已经进行了项目
* 自己手写写启动类和resource里的东西
* 如果想用undertow 可以在web包里exclusion掉tomcat 

```xml
pom依赖

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

==product项目创建==



```xml
pom依赖

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```



```xml
minio

docker run --name lxl-minio -d -p 9000:9000 -p 9001:9001 -v /mnt/data:/data -e MINIO_ROOT_USER=admin -e MINIO_ROOT_PASSWORD=123456789 quay.io/minio/minio server /data --console-address ":9001"
```

## 5.注册中心

* 安装注册中心及配置中心（nacos）
* 采用docker容器运行

```xml
docker 命令
MODE=standalone 是配置单机版，集群和单机的写代码都一样

docker run --name lxl-nacos -d -p 8848:8848 -e MODE=standalone nacos/nacos-server:2.0.2
```

## 10.设置虚拟机的ip

```xml

vi /etc/sysconfig/network-scripts/ifcfg-ens32

按下i进入编辑模式 
第二行dhcp是动态分配ip ---> 改为 static
最后一行ONBOOT 表示网络是否随服务器重启 ----> 选为yes
在最下面添加IPADDR
==========================================
IPADDR=192.168.175.131 ---->这是自定义的固定ip
NETMASK=255.255.255.0 ----> 这是子网掩码
GATEWAY=192.167.175.2 ----> 这可能是子网ip
==========================================
esc :wq

重启网络
systemctl restart network

查看网络配置
ifconfig
```



![image-20220411235209775](前后分离.assets/image-20220411235209775.png)

* 看清网段是175

![image-20220411235530103](前后分离.assets/image-20220411235530103.png)

![image-20220411235640167](前后分离.assets/image-20220411235640167.png)

* 端口映射

![image-20220411235737218](前后分离.assets/image-20220411235737218.png)

* ipconfig 查询配置的ip是否生效

![image-20220412000618783](前后分离.assets/image-20220412000618783.png)

## 11.设置虚拟机的DNS服务器

```xml
vi /etc/resolv.conf

编辑
nameserver 223.5.5.5  这是阿里的dns

保存并退出

测试
ping www.baidu.com

以后就可以用xshell了
```

![image-20220412001438921](前后分离.assets/image-20220412001438921.png)

![image-20220412001426858](前后分离.assets/image-20220412001426858.png)

## 12.虚拟机的使用技巧

```xml
date查看服务器时间是否准确

下载时间同步工具
yum -y install ntpdate

执行命令
ntpdate time.windows.com
```

![image-20220412001832533](前后分离.assets/image-20220412001832533.png)

## 13.docker的安装

```xml
1.打开阿里云的容器镜像服务下的镜像加速器

2.按照文档安装 - 安装命令
curl -fsSL https://get.docker.com | bash -s docker

3.配置镜像加速器
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xzp9x8vn.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

4.启动docker
systemctl start docker 

5.查看docker的版本号
docker version
```

![image-20220412002445284](前后分离.assets/image-20220412002445284.png)

## 14.使用docker安装开发环境

```xml
docker 命令
1.MODE=standalone 是配置单机版，集群和单机的写代码都一样

安装nacos
1.docker run --name lxl-nacos -d -p 8848:8848 -e MODE=standalone nacos/nacos-server:2.0.2

安装mysql
1.docker run --name lxl-mysql -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root mysql:5.6

安装tomcat8
1.docker search tomcat8 ---搜索tomcat镜像
2.inovatrend/tomcat8-java8 --这是版本号
3.完整命令 docker run --name lxl-tomcat -p8080:8080 -d inovatrend/tomcat8-java8
```

## 15.配置远程部署骚操作

```
1.配置docker中的tomcat的配置文件
docker logs lxl-tomcat  --- 查看tomcat启动日志 目的是为了查看tomcat的位置
下图中/opt/tomcat中

2.进去容器内部
docker exec -it lxl-tomcat /bin/bash

3.cd /opt/tomcat   ---> ls

4.配置文件在conf下
  cd conf/
  
5.修改tomcat-users.xml的文件
  vi tomcat-users.xml
  
6.翻到最下面
 role标签只留前两个 下面的role也得删除对应的
 然后设置个好记的密码 保存并推出
 
7.exit 退出小房间到大房间

8.改了配置文件必须重启
  docker restart lxl-tomcat
```

![image-20220412134249070](前后分离.assets/image-20220412134249070.png)

![image-20220412134641995](前后分离.assets/image-20220412134641995.png)

* 修改前

![image-20220412134859407](前后分离.assets/image-20220412134859407.png)

* 修改后

![image-20220412135053739](前后分离.assets/image-20220412135053739.png)

* 在pom文件中写以下代码

![image-20220412140146201](前后分离.assets/image-20220412140146201.png)

* 点这个就相当于直接部署

![image-20220412140129023](前后分离.assets/image-20220412140129023.png)

# n.docker安装rabbitmq

* 安装rabbitmq

```xml
docker run --name lxl-rabbitmq -d -p 5672:5672 -p 15672:15672 -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=123456789 rabbitmq:3-management
```

* 导包

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-amqp -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
    <version>2.6.3</version>
</dependency>
```

