## RESTful API 入门
为了方便**webapp**前后端间通信, **RESTFul API**成为统一API设计风格.

#### 重要规则:
在 **url** 只能用名词描述. 所有请求动作必须使用 **Http Method**

#### Http Method:
    GET（SELECT）：从服务器取出资源（一项或多项)
    POST（CREATE）：在服务器新建一个资源
    PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）
    PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）
    DELETE（DELETE）：从服务器删除资源




## HTTP vs WS
前后端建立长期实时通信时 *(如后端推送警报)*, 需要使用WebSocket, 这是采用ws进行通信.

![](https://raw.githubusercontent.com/6eyu/Study-Backend/master/images/bg2017051503.jpg)
