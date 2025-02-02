﻿## 一、RabbitMQ的基本认识
### RabbitMQ是什么？有什么特点?
RabbitMQ 是一个消息中间件：**`它接受、存储、转发消息，并不处理消息`** 。RabbitMQ服务器是用Erlang语言编写的。

也可能直接问什么是消息队列？**`消息队列就是一个使用队列来通信的组件`**。

### AMQP是什么?
AMQP（Advanced Message Queuing Protocol，**`高级消息队列协议`**）是一个 **`进程间 传递 异步消息 的 网络协议`** 。
![在这里插入图片描述](https://img-blog.csdnimg.cn/9e1b187dcb824cb89244cab241f695cb.png?x-oss-process=image,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_18,color_FFFFFF,t_70,g_se,x_16)


RabbitMQ就是 **`AMQP 协议的 Erlang 的实现`** (当然 RabbitMQ 还支持 STOMP2、 MQTT3 等协议 ) AMQP 的模型架构 和 RabbitMQ 的模型架构是一样的，**`生产者将消息发送给交换器，交换器和队列绑定`** 。

RabbitMQ 中的 **`交换器、交换器类型、队列、绑定、路由键等`** 都是遵循的 AMQP 协议中相 应的概念。目前 RabbitMQ 最新版本默认支持的是 AMQP 0-9-1。
### AMQP协议3层？
- **`Module Layer`**：协议最高层，主要定义了一些客户端调用的命令，客户端可以用这些命令实现自己的业务逻辑。

- **`Session Layer`**：中间层，主要负责客户端命令发送给服务器，再将服务端应答返回客户端，提供可靠性同步机制和错误处理。

- **`TransportLayer`**：最底层，主要传输二进制数据流，提供帧的处理、信道服用、错误检测和数据表示等。

### AMQP模型的几大组件？
- 交换器 (Exchange)：消息代理服务器中用于把消息路由到队列的组件。

- 队列 (Queue)：用来存储消息的数据结构，位于硬盘或内存中。

- 绑定 (Binding)：一套规则，告知交换器消息应该将消息投递给哪个队列。

### 怎么理解生产者Producer、消费者Consumer?
生产者

- 消息生产者，就是投递消息的一方。
- 消息一般包含两个部分：消息体（payload)和标签(Label)。

消费者
- 消费消息，也就是接收消息的一方。
- 消费者连接到RabbitMQ服务器，并订阅到队列上。消费消息时只消费消息体，丢弃标签。
### RabbitMQ的组件？（很重要）
![在这里插入图片描述](https://img-blog.csdnimg.cn/5d3c07fd3ae8405289ab2731b9488534.png?x-oss-process=image,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_20,color_FFFFFF,t_70,g_se,x_16)
- Connection：publisher／consumer 和 broker 之间的 TCP 连接
- Channel：如果每一次访问 RabbitMQ 都建立一个 Connection，在消息量大的时候建立 TCP Connection 的开销将是巨大的，效率也较低。Channel 是在 connection 内部建立的逻辑连接。Channel 作为轻量级的Connection 极大减少了操作系统建立 TCP connection 的开销
- Broker：接收和分发消息的应用，**`RabbitMQ Server 就是 Message Broker`**
- **`Virtual host`** ：出于多租户和安全因素设计的，**`把 AMQP 的基本组件划分到一个虚拟的分组中`**，类似于网络中的 namespace 概念。当多个不同的用户使用同一个 RabbitMQ server 提供的服务时，可以划分出多个 vhost，每个用户在自己的 vhost 创建 exchange／queue 等
- Exchange：message 到达 broker 的第一站，根据分发规则，匹配查询表中的 **`routing key`**，分发消息到 queue 中去。**`常用的类型有：direct (point-to-point), topic (publish-subscribe) and fanout (multicast)`**
- Queue：消息最终被送到这里等待 consumer 取走
- Binding：**`通过绑定将交换器和队列关联起来，一般会指定一个routingKey,这样RabbitMq就知道如何正确路由消息到队列了`**。

### 为什么需要消息队列？（重要）
**`异步处理、服务解耦、流量控制（削峰）`**
### 生产者发送消息/消费者消费消息的过程？
> 生产者发送消息过程

1.Producer先连接到Broker,建立连接Connection,开启一个信道(Channel)。

2.Producer声明一个交换机并设置好相关属性。

3.Producer声明一个队列并设置好相关属性。

4.Producer通过路由键将交换器和队列绑定起来。

5.Producer发送消息到Broker,其中包含路由键、交换器等信息。

6.相应的交换器根据接收到的路由键查找匹配的队列。

7.如果找到，将消息存入对应的队列，如果没有找到，会根据生产者的配置丢弃或者退回给生产者。

8.关闭信道。

9.管理连接。
> 消费者接收消息过程

1.Consumer先连接到Broker,建立连接Connection,开启一个信道(Channel)。

2.向Broker请求消费响应的队列中消息，可能会设置响应的回调函数。

3.等待Broker回应并投递相应队列中的消息，接收消息。

4.消费者确认收到的消息,ack。

5.RabbitMq从队列中删除已经确定的消息。

6.关闭信道。

7.关闭连接。

## 二、消息应答策略（防止生队列到消费者消息丢失）
### 消费者的消息应答策略有哪些？MQ如何将消息可靠投递到消费者？（防止队列到消费者的消息丢失）消息被拒绝后会怎么样？
消息应答就是：消费者在接收到消息并且处理该消息之后，告诉 rabbitmq 它已经处理了，rabbitmq 可以把该消息删除了。**`防止队列到消费者的消息丢失`** 。

1. MQ将消息push给Client（或Client来pull消息）
2. Client得到消息并做完业务逻辑
3. Client发送Ack消息给MQ，通知MQ删除该消息，**`此处有可能因为网络问题导致Ack失败，那么Client会重复消息，这里就引出消费幂等的问题；`**
4. MQ将已消费的消息删除。
> <font size=4 color=blue>**自动应答，默认策略，autoAck = true**</font>

- 如果配置的是自动应答，那么消息发送后立即被认为已经传送成功，如果程序这时候突然宕机，那么消息<font color=red>**直接丢失**</font>，这是很不安全的。
- 不论乎消费者对消息处理是否成功，都会告诉队列删除消息。
> <font size=4 color=blue>**手动应答：设置手动应答，autoAck = false**</font>
- 确认应答：channel.basicAck(用于肯定确认)RabbitMQ 已知道该消息并且成功的处理消息，可以将其丢弃
- 拒绝应答一：**`channel.basicNack()`**   (用于否定确认)，三个参数
- 拒绝应答二：**`channel.basicReject()`**  (用于否定确认)，两个参数

```java
channel.basicConsume(QUEUE_NAME,false,
	(consumerTag, message) -> {
	    // 接收到的消息
	    String receviedMessage = new String(message.getBody());
	    System.out.println("消费者接收到的消息："+receviedMessage);
	    /**
	     * 确认应答
	     *  1.哪个消息
	     *  2.应答一个消息还是应答一个消息
	     *      true: 把当前消息和之前的消息一起应答
	     *      false: 只应答当前消息
	     */
	    // channel.basicAck(message.getEnvelope().getDeliveryTag(),false);
	    /**
	     * 拒绝 确认受到应答
	     *  1.哪个消息
	     *  2.应答一个消息还是应答一个消息
	     *      true: 把当前消息和之前的消息一起拒绝
	     *      false: 只拒绝当前消息
	     *  3.是否重新入队
	     *      true: 消息重新入队
	     *      false: 丢弃消息 或者 进入死信队列（死信队列需要配置）
	     */
	    // channel.basicNack(message.getEnvelope().getDeliveryTag(),false,false);
	    /**
	     * 拒绝 确认受到应答
	     *  1.哪个消息
	     *  2.是否重新入队
	     *      true: 消息重新入队
	     *      false: 丢弃消息 或者 进入死信队列（死信队列需要配置）
	     */
	    channel.basicReject(message.getEnvelope().getDeliveryTag(),false);
	},
	(consumerTag) -> {
	    System.out.println("消费失败");
	});
```
### 消费者 未应答的消息 会怎么样？（自动重新入队，会有幂等性问题）
如果消费者由于某些原因失去连接(其通道已关闭，连接已关闭或 TCP 连接丢失)，导致消息未发送 ACK 确认，**`RabbitMQ 将了解到消息未完全处理，并将对其重新排队。`** 

如果此时其他消费者可以处理，它将很快将其重新分发给另一个消费者。这样，即使某个消费者偶尔死亡，也可以确保不会丢失任何消息。

但会存在幂等性的问题。
### 如何选择用 自动应答 还是 手动应答 ？（重要）
这两种方式实在 **`高吞吐量`** 和 **`数据传输安全性`** 之间做权衡。
### 什么是不公平分发？怎么实现？有什么作用？
RabbitMQ 分发消息采用的轮训分发，但是在某种场景下这种策略并不是很好，比方说有两个消费者在处理任务，其中有个消费者 1 处理任务的速度非常快，而另外一个消费者 2 处理速度却很慢。这就导致一个很忙，一个很闲。
> 设置预取值：**`该值定义通道上允许的未确认消息的最大数量。`**

```java
// 设置非公平，并传入预取值
int prefetchCount = 2;
channel.basicQos(prefetchCount);
```
## 三、持久化 （防止宕机造成队列中消息丢失） 
###  什么是RabbitMQ持久化？（重要）
消息都在内存中，服务崩溃后会丢失，需要将 **`队列`** 和 **`消息`** 都标记为 **`持久化`**。

队列用来存储消息，先持久化队列，再持久化消息。
### 持久化能保证大数据不丢失吗？（重要）
不能，加入在写入磁盘时宕机了，则数据会丢失。**`发布确认机制可以做到`**。

## 四、发布确认（防止生产者到交换机消息丢失）
### 什么是发布确认？生产者如何将消息可靠投递到MQ？
生产者将信道 **`设置成 confirm 模式`** ，一旦信道进入 confirm 模式，所有在该信道上面发布的消息都将会被 **`指派一个唯一的 ID(从 1 开始)`** ，一旦消息被投递到所有匹配的队列之后， **`broker 就会发送一个确认给生产者`** ( **`包含消息的唯一 ID`** )，这就使得生产者知道消息已经正确到达目的队列了。

如果消息和队列都设置了<font color=red>**持久化**</font>的，那么 **`确认消息会在将消息写入磁盘之后发出`**  。
### 如果宕机了客户端没收到确认消息怎么办？
此处有可能因为网络问题导致Ack消息无法发送到Client，那么Client在等待超时后，会重传消息。
### 发布确认的策略有哪些？（三种、重要）

**`单个确认发布、批量确认发布、异步确认发布`**

> <font size=4 color=blue>**失败单个确认发布、批量确认发布 是阻塞的**</font>

> <font size=4 color=blue>**异步发布确认 是非阻塞的**</font>

- 将消息同时发送到map和队列中
- 收到消息，回调删除map中的消息，如果是multiple，则同时删除该消息之前的消息
- 未受到消息，从map中取出消息重新发送
![在这里插入图片描述](https://img-blog.csdnimg.cn/51653ecebe844f4fb47ea40dc18ce134.png?x-oss-process=image,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_20,color_FFFFFF,t_70,g_se,x_16)
### 三种策略对比 （重要）
- 单独发布消息：同步等待确认，简单，但吞吐量非常有限。
- 批量发布消息：批量同步等待确认，简单，合理的吞吐量。一旦出现问题但很难推断出是那条消息出现了问题，只能对这一批从新发送。造成重复消费。
- 异步处理：**`最佳性能和资源使用，在出现错误的情况下可以很好地控制`**，但是实现起来稍微困难。
### 交换器无法根据自身类型和路由键找到符合条件队列时会怎么处理？
#### 处理方式一：消息回退（mandatory 参数）
通过设置 mandatory 参数可以在当消息传递过程中不可达目的地时将消息 **`返回给生产者`** 。
```java
/**
* true：交换机无法将消息进行路由时，会将该消息返回给生产者
* false：如果发现消息无法进行路由，则直接丢弃
*/
rabbitTemplate.setMandatory(true);
```
有了 mandatory 参数和回退消息，我们 **`就获得了对无法投递消息的感知能力，有机会在生产者的消息无法被投递时发现并处理。`**
#### 处理方式二：备份交换机
在 RabbitMQ 中，有一种备份交换机的机制存在，可以很好的应对这个问题。
- 当交换机接收到一条不可路由消息时，将会把这条消息转发到备份交换机中，由备份交换机来进行转发和处理，**`通常备份交换机的类型为 Fanout`** ；
- 这样就能把所有消息都投递到与其绑定的队列中，然后我们在备份交换机下绑定一个队列，这样所有那些原交换机无法被路由的消息，就会都进入这个队列了。

当然，我们还可以建立一个报警队列，用独立的消费者来进行监测和报警。
> <font size=4 color=blue>**这里为什么不用死信交换机？**</font>

**`为队列设置死信交换机来存储那些处理失败的消息，可是这些不可路由消息根本没有机会进入到队列，因此无法使用死信队列来保存消息。`**
#### 消息回退 和 备份交换机 那个优先级更高？（备份交换机优先级更高）
当 消息回退 和 备份交换机 同时存在，那么消息是回退给生产者，还是发送到备份交换机呢？
- **`备份交换机优先级更高`**
## 五、交换机（exchange）
### 什么是交换机？
message 到达 broker 的第一站，根据分发规则，匹配查询表中的 **`routing key`** ，分发消息到 queue 中去。

生产者只能将消息发送到交换机(exchange)
- 一方面它接收来自生产者的消息；
- 另一方面将它们推入队列。

![在这里插入图片描述](https://img-blog.csdnimg.cn/0789f06a9e294c5fbb87f3312e0ec5af.png?x-oss-process=image,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_14,color_FFFFFF,t_70,g_se,x_16)

### 交换机有那些类型？（重要）
总共有以下类型：**`直接(direct), 主题(topic), 扇出(fanout) ,标题(headers) `** 。标题(headers)  用的少。
-  扇出(fanout)：**`它是将接收到的所有消息广播到它知道的所有队列中。`**
- 直接(direct)：**`消息只去到它绑定的routingKey 队列中去`** 。
- 主题(topic)：在 direct 上进一步细分。发送到类型是 topic 交换机的消息的 routing_key 不能随意写，必须满足一定的要求，它必须是一个单词列表，以点号分隔开。
- 标题(headers)：通过 Key-value 匹配queues 。也是忽略routingKey的一种路由方式。
> <font size=4 color=blue>**topic对routingKey书写的要求**</font>

发送到类型是 topic 交换机的消息的 routing_key 不能随意写，必须满足一定的要求，它必须是一个单词列表，以点号分隔开。这些单词可以是任意单词，比如说："stock.usd.nyse", "nyse.vmw", "quick.orange.rabbit".这种类型的。当然这个单词列表最多不能超过 255 个字节。

在这个规则列表中，其中有两个替换符是大家需要注意的：
- *(星号) 可以代替一个单词；
- #(井号) 可以替代零个或多个单词。
## 六、死信队列（解决没有消费者消费的消息）
### 什么是死信？
**`由于特定的原因导致 queue 中的某些消息无法被消费，这样的消息如果没有后续的处理，就变成了死信，有死信自然就有了死信队列。`**
![在这里插入图片描述](https://img-blog.csdnimg.cn/3e229c77e4b54f87949dfe9d0e007f27.png?x-oss-process=image,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_20,color_FFFFFF,t_70,g_se,x_16)
### 产生死信的原因有哪些?（重要）
- **`消息 TTL（Time To Live） 过期`**：消息再队列中过期，**`区分队列TTL`**
- **`队列达到最大长度`**，无法再添加数据到 mq 中
- **`消息被拒绝`** (basic.reject 或basic.nack)并且 requeue=false
## 七、延迟队列（希望消息在指定时间被处理）
### 什么是延迟队列？
延时队列内部是有序的，最重要的特性就体现在它的延时属性上，延时队列中的元素是希望在指定时间到了以后或之前取出和处理。

简单来说，延时队列就是 **`用来存放需要在指定时间被处理的元素的队列`** 。
### 应用场景有哪些？
1. 订单在十分钟之内未支付则自动取消。
2. 新创建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒。
3. 用户注册成功后，如果三天内没有登陆则进行短信提醒。
4. 用户发起退款，如果三天内没有得到处理则通知相关运营人员。
5. 预定会议后，需要在预定的时间点前十分钟通知各个与会人员参加会议。
### 它和定时任务的区别，有什么优点？
定时任务会使数据库的压力瞬时增大

定时任务一般用于数据量少，对时间不是严格限制的场景。
### 延迟队列如何实现？（重要）
#### 设置队列TTL
- 队列TTL：如果设置了队列TTL，**`则队列中的消息过期后直接被 丢弃 或 加入死信队列，不用发送给消费之。`**
- 问题：**`每增加一个新的时间需求，就要新增一个队列`**
#### 设置消息TTL
- 消息TTL：如果设置了消息TTL，如果消息过期后还在队列中，不会被 丢弃 或 加入死信队列。**`而是在发送给消费者后，由消费者判断是否过期，如果过期就 丢弃 或 加入死信队列。`**
- 问题： 消息可能并不会按时“死亡“ ，**`因为 RabbitMQ 只会检查第一个消息是否过期，如果过期则丢到死信队列`**， 如果第一个消息的延时时长很长，而第二个消息的延时时长很短，第二个消息并不会优先得到执行 。
#### Rabbitmq 插件实现延迟队列（解决队列TTL 和 消息TTL 缺点）

在这里新增了一个队列delayed.queue,一个自定义交换机 delayed.exchange，绑定关系如下:
![在这里插入图片描述](https://img-blog.csdnimg.cn/34ef39d9a22e4fc59423f670843a8c65.png)
在我们自定义的交换机中，这是一种新的交换类型，**`该类型消息支持延迟投递机制`** ， 消息传递后并不会立即投递到目标队列中，而是 **`存储在 mnesia(一个分布式数据系统)表中`** ，**`当达到投递时间时，才投递到目标队列中`**。
## 八、幂等性
#### 什么是 幂等性？
**`用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生了副作用。`**
- 比如支付，不能因为点击多次导致付款多次
#### 如何解决 消息重复消费  ？（两种方法）
消费者在消费 MQ 中的消息时，MQ 已把消息发送给消费者， **`消费者在给MQ 返回 ack 时网络中断， 故 MQ 未收到确认信息`** ，该条消息会重新发给其他的消费者，或者在网络重连后再次发送给该消费者，但实际上该消费者已成功消费了该条消息，造成消费者消费了重复的消息。

**`MQ 消费者的幂等性的解决一般 使用全局 ID  或者 写个唯一标识，每次消费消息时用该 id 先判断该消息是否已消费过。`**

在海量订单生成的业务高峰期，生产端有可能就会重复发生了消息，这时候消费端就要实现幂等性， 这就意味着我们的消息永远不会被消费多次，即使我们收到了一样的消息。业界主流的幂等性有两种操作:
-  **`唯一 ID + 指纹码机制，利用数据库主键去重`** 。
-  **`利用 redis 的原子性去实现`** 。

> <font size=4 color=blue>**唯一ID + 指纹码机制**</font>

指纹码:我们的一些规则或者时间戳加别的服务给到的唯一信息码,它并不一定是我们系统生成的，基本都是由我们的业务规则拼接而来，但是一定要保证唯一性。**`将唯一ID + 指纹码拼接好的值作为数据库主键`**，就可以进行去重了。**`即在消费消息前呢，先去数据库查询这条消息的指纹码标识是否存在，没有就执行insert操作，如果有就代表已经被消费了，就不需要管了。`**
- 优势：实现简单就一个拼接，然后查询判断是否重复；
- 劣势：**`在高并发时`**，如果是单个数据库就会有写入性能瓶颈。当然也可以采用分库分表提升性能，但也不是我们最推荐的方式。
> <font size=4 color=blue>**Redis 原子性**</font>

利用 redis 执行 **`setnx 命令`**，天然具有幂等性。从而实现不重复消费。setnx：key存在，就不执行。

## 九、什么是优先级队列？使用场景？
在我们系统中有一个订单催付的场景，我们的客户在天猫下的订单,淘宝会及时将订单推送给我们，如果在用户设定的时间内未付款那么就会给用户推送一条短信提醒，很简单的一个功能对吧，但是，tmall 商家对我们来说，肯定是要分大客户和小客户的对吧，比如像苹果，小米这样大商家一年起码能给我们创造很大的利润，所以理应当然，他们的订单必须得到优先处理。

要让队列实现优先级需要做的事情有如下事情：
- **`队列需要设置为优先级队列`**
- **`消息需要设置消息的优先级`**
- 消费者需要等待消息已经发送到队列中才去消费因为，这样才有机会对消息进行排序。
## 十、什么是惰性队列？使用场景？
- 队列具备两种模式：default 和 lazy。默认的为 default 模式，lazy 模式即为惰性队列的模式。
- RabbitMQ 从 3.6.0 版本开始引入了惰性队列的概念。
- **`惰性队列会尽可能的将消息存入磁盘中，而在消费者消费到相应的消息时才会被加载到内存中。`**
- **`它的一个重要的设计目标是能够支持更长的队列，即支持更多的消息存储。`**
- 但是 I/0 开销大
## 十一、集群 TODO
RabbitMQ 有三种模式：**`单机模式、普通集群模式、镜像集群模式`**
### 集群部署流程？TODO
### 普通集群？（没啥用，单机模式也是）
### 什么是镜像集群？如何保证RabbitMQ消息队列的高可用?
镜像集群是RabbitMQ高可用的一种模式，跟普通集群模式不一样的是，你创建的queue，无论 **`元数据(元数据指RabbitMQ的配置数据)还是queue里的消息都会存在于多个实例上`** ，然后每次你写消息到queue的时候，都会自动把消息到多个实例的queue里进行消息同步。

这样在单节点失效的情况下，整个集群仍旧可以提供服务。但是由于数据需要在多个节点复制，在增加可用性的同时，系统的吞吐量会有所下降。

在实现机制上，mirror queue内部实现了一套选举算法，有一个master和多个slave。
## 十二、补充
### 在使用 MQ 消息队列时，如何确保消息不丢失？
分三个方面回答：
- 如何知道有消息丢失？
- 哪些环节可能丢消息？
- 如何确保消息不丢失？

![在这里插入图片描述](https://img-blog.csdnimg.cn/bbd4bc33d2b1452f9060a100acdff506.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_20,color_FFFFFF,t_70,g_se,x_16)
- 消息生产阶段： 从消息被生产出来，然后提交给 MQ 的过程中，只要能正常收到 MQ Broker 的 ack 确认响应，就表示发送成功，所以只要处理好返回值和异常，这个阶段是不会出现消息丢失的。

- 消息存储阶段： 这个阶段一般会直接交给 MQ 消息中间件来保证，但是你要了解它的原理，比如 Broker 会做副本，保证一条消息至少同步两个节点再返回 ack。

- 消息消费阶段： 消费端从 Broker 上拉取消息，只要消费端在收到消息后，不立即发送消费确认给 Broker，而是等到执行完业务逻辑后，再发送消费确认，也能保证消息的不丢失。

### RabbitMQ 生产者发消息非常快，消费者消费不过来怎么办？
- 首先有一个问题：这会导致消息队列溢出吗？会
- 当生产者速度过高导致RabbitMQ队列堆积大量消息流控时，RabbitMQ将阻塞生产者连接。主观上消费者速度应该至少保持不变，但实际观察发现，生产者和消费者的速度均受影响，且不平稳。
	- 为什么消费速度降低了，因为CPU已经被霸占了。
- 当队列堆积大量的消息时，消费速度不稳定，那就想办法提高消费者利用率吧。
	- 消费者利用率代表了RabbitMQ能够立即deliver message给消费者的能力。
	- 可以通过增加消费者数量，提高Prefetch count，使用批量Ack方式提高消费者利用率。这样消费速度就会比较稳定。
- 为了防止生产者使用过快的速度生产消息，RabbitMQ实现了一种称为 **`信用流的内部机制`** ，RabbitMQ内部的各种系统将使用该机制来 **`限制生产者`**，不至于让消费者望尘莫及。
### 消息重传机制？TODO
