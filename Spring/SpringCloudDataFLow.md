# Spring Cloud Data FLow 入门
A ***cloud-native*** programming and operating model for composable data microservices.

开发者能够为 ***common use cases*** 来创建和编排 data pipelines 例如: 数据摄取，实时分析和数据导入/导出. [见原文](https://www.baeldung.com/spring-cloud-data-flow-stream-processing)


### Apps分类:
>**Source**: 数据的来源 (可以是API接口, 接收用户数据) 
<br> **Processor**: 处理数据后再发出 (对数据进行加工)
<br>**Sink**: 存储数据 (接收来自 source 或 processor 的数据, 并将数据写入持久层)

### 管道通信:
- Apache Kafka
- RabbitMQ

### 运行: 
- Docker compose 配置文件运行 [见详情](https://springcloud.cc/spring-cloud-dataflow.html#getting-started-deploying-spring-cloud-dataflow-docker)
- jar 手动运行 [见详情](https://springcloud.cc/spring-cloud-dataflow.html#getting-started-deploying-spring-cloud-dataflow)

> 成功运行后可以登录[Spring Cloud Data Flow Dashboard](http://localhost:9393/dashboard) 
<br>**注意** 手动配置可能需要自行安装运行 RabbitMQ

### 自定义 Stream Apps (使用Rabbitmq)
**A Stream Rabbit dependency**
```
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
    </dependency>
```
#### 创建 source application:
- *main class*
```
    @EnableBinding(Source.class) //与stream 绑定 该程序为 source
    @SpringBootApplication
    public class SpringDataFlowTimeSourceApplication {
        
        public static void main(String[] args) {
            SpringApplication.run(
            SpringDataFlowTimeSourceApplication.class, args);
        }
    }
```
- *生产数据 component:*
```
    @Bean
    @InboundChannelAdapter(
        value = Source.OUTPUT,  //输出
        poller = @Poller(fixedDelay = "10000", maxMessagesPerPoll = "1") //每 10 秒发送 1 次 poller
    )
    public MessageSource<Long> timeMessageSource() {
        return () -> MessageBuilder.withPayload(new Date().getTime()).build();
    }
```
#### 创建 processer application
- *main class*
````
    @EnableBinding(Processor.class) //与stream 绑定该程序为 processer
    @SpringBootApplication
    public class SpringDataFlowTimeProcessorApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(
            SpringDataFlowTimeProcessorApplication.class, args);
        }
    }
````
- *接收发送数据 component:* 
```
    @Transformer(inputChannel = Processor.INPUT, 
                outputChannel = Processor.OUTPUT)
    public Object transform(Long timestamp) {
    
        DateFormat dateFormat = new SimpleDateFormat("yyyy/MM/dd hh:mm:yy");
        String date = dateFormat.format(timestamp);
        return date;
    }
```
#### 创建 sink application
- *main class*
```
    @EnableBinding(Sink.class)
    @SpringBootApplication
    public class SpringDataFlowLoggingSinkApplication {
    
        public static void main(String[] args) {
        SpringApplication.run(
            SpringDataFlowLoggingSinkApplication.class, args);
        }
    }
```
- *接收数据 简单打印出来:*
```
    @StreamListener(Sink.INPUT)
    public void loggerSink(String date) {
        logger.info("Received: " + date);
    }
```
### 注册 部署 Apps 
程序创建好后 可打包成 .jar 文件, 上传 maven 后通过 uri 注册. 本地测试可以用files路径(如第一条uri)
 
* 注册和部署,可以使用 [dashboard](http://localhost:9393/dashboard) 进行可视化操作

**在Data Flow Shell中操作,需要先运行 Data Flow Shell .jar 程序**

- **注册**
```
app register --name time-source --type source --uri file:///home/xxxx/xxxx/chat/target/spring-data-flow-time-source:jar:0.0.1-SNAPSHOT
 
app register --name time-processor --type processor --uri maven://org.baeldung.spring.cloud:spring-data-flow-time-processor:jar:0.0.1-SNAPSHOT
 
app register --name logging-sink --type sink --uri maven://org.baeldung.spring.cloud:spring-data-flow-logging-sink:jar:0.0.1-SNAPSHOT
```
- **创建&部署 pipeline**

```
stream create --name time-to-log --definition 'time-source | time-processor | logging-sink'
```
then
```
stream deploy --name time-to-log
```


