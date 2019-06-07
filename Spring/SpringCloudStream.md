### Spring Cloud Stream & RabbitMQ [原文](https://www.e4developer.com/2018/01/28/setting-up-rabbitmq-with-spring-cloud-stream/)
**使用 docker 运行 Rabbitmq**  <br> '-d' 代表后台运行
```
docker run -d --hostname my-rabbit --name some-rabbit -p 15672:15672  -p 5672:5672 rabbitmq:3-management
```
成功运行后 可登录[RabbitMQ dashboard](http://localhost:15672), username & password = guest & guest.

**A Stream Rabbit dependency**
```
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
    </dependency>
```

#### Comsumer App (消息 接收者)
- *main class*
```
@EnableBinding(Sink.class)
@SpringBootApplication
public class FoodOrderConsumerApplication {
 
    public static void main(String[] args) {
        SpringApplication.run(FoodOrderConsumerApplication.class, args);
    }
 
    @StreamListener(target = Sink.INPUT)
    public void processCheapMeals(String meal){
        System.out.println("This was a great meal!: "+meal);
    }
}
```
- *application.properties* (同 application.yml)
```
server.port=0 #设置 0 为了同时运行多个实例
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
 
spring.cloud.stream.bindings.input.destination=foodOrders
spring.cloud.stream.bindings.input.group=foodOrdersIntakeGroup
```

#### Publisher App (消息 发送者)
- *model class*
```
@JsonIgnoreProperties(ignoreUnknown = true)
public class FoodOrder {
 
    private String restaurant;
    private String customerAddress;
    private String orderDescription;
 
    public FoodOrder(){
 
    }
 
    public FoodOrder(String restaurant, String customerAddress, String orderDescription) {
        this.restaurant = restaurant;
        this.customerAddress = customerAddress;
        this.orderDescription = orderDescription;
    }
 
    public void setRestaurant(String restaurant) {
        this.restaurant = restaurant;
    }
 
    public void setCustomerAddress(String customerAddress) {
        this.customerAddress = customerAddress;
    }
 
    public void setOrderDescription(String orderDescription) {
        this.orderDescription = orderDescription;
    }
 
    public String getRestaurant() {
        return restaurant;
    }
 
    public String getCustomerAddress() {
        return customerAddress;
    }
 
    public String getOrderDescription() {
        return orderDescription;
    }
 
    @Override
    public String toString() {
        return "FoodOrder{" +
                "restaurant='" + restaurant + '\'' +
                ", customerAddress='" + customerAddress + '\'' +
                ", orderDescription='" + orderDescription + '\'' +
                '}';
    }
}
```
- *interface 获取 MessageChannel 对象 来发送消息*
```
public interface FoodOrderSource {
 
    @Output("foodOrdersChannel")
    MessageChannel foodOrders();
 
}
```
- *为了在controller中使用上面的接口, 需要一个class实例化接口. 以下是正确用法*
```
@EnableBinding(FoodOrderSource.class)
public class FoodOrderPublisher {
}
```
- *controller*
```
@RestController
public class FoodOrderController {
 
    @Autowired
    FoodOrderSource foodOrderSource;
 
    @RequestMapping("/order")
    @ResponseBody
    public String orderFood(@RequestBody FoodOrder foodOrder){
        foodOrderSource.foodOrders().send(MessageBuilder.withPayload(foodOrder).build());
        System.out.println(foodOrder.toString());
        return "food ordered!";
    }
}
```
- *application.properties* (同 application.yml)
```
server.port=8080
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
 
spring.cloud.stream.bindings.foodOrdersChannel.destination=foodOrders
spring.cloud.stream.default.contentType=application/json
```