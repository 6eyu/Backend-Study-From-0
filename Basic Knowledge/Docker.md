# Docker (简单入门)

详细文档请看[官网](https://docs.docker.com/get-started/)或[中文](https://yeasy.gitbooks.io/docker_practice/introduction/what.html)
- **Image**         镜像: 用来创建容器(Container). 可以存储到 Docker Registry 的仓库中
- **Containers**    容器: 将镜像(Image)实例化并运行.
- **Services**      通过 compose.yml 文件配置,部署在 Stacks 上, 实例化并运行指定的 containers.
- **Stacks**        承载 Services.
- **Swarms**        多个 Docker Servers 连接组成网络成为更大的 Docker Server.

### 创建 Image(镜像)
**Step 1**: 配置 Dockerfile
    
    # Use an official Python runtime as a parent image
    FROM openjdk:11

    # 复制打包好的jar
    ADD /target/discovery-0.1.jar discovery.jar

    # Make port 8761 available to the world outside this container
    EXPOSE 8761

    # 运行指定 jar 
    ENTRYPOINT ["java", "-jar", "discovery.jar"]

**Step 2**: Build 镜像

在 Dockerfile 所在文件夹中 运行以下命令: ***注意-命令行最后的 "." 号***

    docker build --tag=friendlyhello .

可在 name 后加 tag 如下: ***注意-命令行最后的 "." 号***
    
    --tag=friendlyhello:v0.0.1 .

查看已创建的 Images

    $ docker image ls

### 通过 Image 创建 Container
只需一道命令行:

    docker run -p 4000:8761 friendlyhello

**注**: 4000 是外部端口, 8761 容器端口. 

这里将容器端口8761映射到外部端口4000, 我们可以通过 localhost:4000 来访问该容器.


### Services 和 Stack
**Step 1:** 配置 docker-compose.yml 

    version: "3"
    services:
    eureka1:
        image: eiz-discovery:dev1
        networks:
        springcloud-overlay:
            aliases:
            - eureka
        ports:
        - "8761:8761"
        environment:
        - ADDITIONAL_EUREKA_SERVER_LIST=http://eureka2:8761/eureka/,http://eureka3:8761/eureka/

    eureka2:
        image: eiz-discovery:dev1
        networks:
        springcloud-overlay:
            aliases:
            - eureka
        ports:
        - "8762:8761"
        environment:
        - ADDITIONAL_EUREKA_SERVER_LIST=http://eureka1:8761/eureka/,http://eureka3:8761/eureka/

    eureka3:
        image: eiz-discovery:dev1
        networks:
        springcloud-overlay:
            aliases:
            - eureka
        ports:
        - "8763:8761"
        environment:
        - ADDITIONAL_EUREKA_SERVER_LIST=http://eureka2:8761/eureka/,http://eureka1:8761/eureka/
        
    networks:
    springcloud-overlay:
        external:
        name: springcloud-overlay

以上配置了3个 eureka 服务互相注册, 统一命名为 eureka

环境变量在程序的 application.yml 中设置如下:

     defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/,${ADDITIONAL_EUREKA_SERVER_LIST}

**Step 2:** 启动 swarm

    docker swarm init
 如果不运行 swarm, 会出现 error 信息: “this node is not a swarm manager.” 

 **Step 3:** 部署  docker-compose.yml

    docker stack deploy -c docker-compose.yml getstartedlab

查看 services:

    docker service ls

关闭程序:

    docker stack rm getstartedlab

关闭 swarm:

    docker swarm leave --force