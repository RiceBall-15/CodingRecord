## ActiveMQ学习

1. #### 消息中间件应用场景

   ```text
   异步处理
   应用解耦
   流量削峰
   ```

   ##### 异步处理：

   场景说明：用户注册，需要执行三个业务逻辑，分别为写入用户表，发注册邮件以及注册短信。

   >串行方式

   将注册信息写入数据库成功后，发送注册邮件，在发送注册短信。以上三个任务全部完成后，返回给客户端。

   ![image-20200520112836330](C:\Users\LiYuanZhuo\AppData\Roaming\Typora\typora-user-images\image-20200520112836330.png)

   >并行方式

   将注册信息写入数据库成功后，发送注册邮件的同时，发送注册短信。以上三个任务完成后，返回给客户端。与串行的差别是，并行的方式可以提高处理的时间。

   ![image-20200520113840571](C:\Users\LiYuanZhuo\AppData\Roaming\Typora\typora-user-images\image-20200520113840571.png)

   >异步处理

   引入消息中间件，将部分的业务逻辑，进行异步处理。改造后的架构如下：

![image-20200520114008891](C:\Users\LiYuanZhuo\AppData\Roaming\Typora\typora-user-images\image-20200520114008891.png)

##### 		应用解耦

​		场景说明：用户下单后，订单系统需要通知库存系统。

​		传统做法是，订单系统调用库存系统的接口，如下图：

​	![image-20200520133709376](C:\Users\LiYuanZhuo\AppData\Roaming\Typora\typora-user-images\image-20200520133709376.png)

​		传统模式的缺点：假如库存系统无法访问，则订单减库存将失败，从而导致订单失败，订单系统与库存系统耦合，引入应用消息队列后，如下图：

![image-20200520133857017](C:\Users\LiYuanZhuo\AppData\Roaming\Typora\typora-user-images\image-20200520133857017.png)

​		

​		订单系统：用户下单后，订单系统完成持久化处理，将消息写入消息队列，返回用户订单下单成功

​		库存系统：订阅下单的消息，采用拉、推的方式，获取下单信息，库存系统根据下单信息，进行库存操作

​		假如：再下单时库存系统不能正常使用，也不影响正常下单，实现了订单系统与酷讯系统的应用解耦。



##### 		流量削峰

​		流量削峰也是消息队列中的常用场景，一般应用在秒杀或者团购活动中使用，

​		通过加入消息队列完成如下功能：

​			a、可以控制活动人数

​			b、可以缓解短时间内高流量压垮应用

![image-20200520134852935](C:\Users\LiYuanZhuo\AppData\Roaming\Typora\typora-user-images\image-20200520134852935.png)

​			用户的请求，服务器接收后，首先写入消息队列。加入消息队列长度超过最大数量，则直接抛弃用户请求或者跳转到错误页面。秒杀业务根据消息队列中的请求信息，在做后续处理

#### 	常见的消息中间件产品对比

![image-20200520135557714](C:\Users\LiYuanZhuo\AppData\Roaming\Typora\typora-user-images\image-20200520135557714.png)





#### 	JMS消息模型

​	消息中间件一般有两种传递模式：点对点模式（P2P）和发布-订阅模式（Pub/Sub）。

​		1、P2P 点对点模型（Queue队列模型）

​		2、Pub/Sub 发布/订阅模型（Topic主题模型）

##### 		点对点模式

![image-20200520140415832](C:\Users\LiYuanZhuo\AppData\Roaming\Typora\typora-user-images\image-20200520140415832.png)

#### JMS消息组成

1. ##### TextMessage

   >发送

   ```java
   @Test
       public void sendTextMessage(){
           //参数一：队列的名称或者主题名称
           //参数二：消息内存
           jmsTemplate.send("springboot_queue", new MessageCreator() {
               @Override
               public Message createMessage(Session session) throws JMSException {
                   TextMessage textMessage = session.createTextMessage("test message");
                   textMessage.setJMSMessageID("10000");
                   textMessage.setJMSPriority(7);
                   textMessage.setJMSCorrelationID("22222");
                   return textMessage;
               }
           });
       }
   ```

   >接收

   ```java
       @JmsListener(destination = "springboot_queue")
       public void receiveTextMessage(Message message){
           if (message instanceof TextMessage){
              TextMessage textMessage = (TextMessage) message;
               try {
                   System.out.println("接收消息：" + textMessage.getText());
               } catch (JMSException e) {
                   e.printStackTrace();
               }
           }
       }
   ```

2. ##### MapMessage

   ```java
       @Test
       public void sendMapMessage(){
           //参数一：队列的名称或者主题名称
           //参数二：消息内存
           jmsTemplate.send("springboot_queue", new MessageCreator() {
               @Override
               public Message createMessage(Session session) throws JMSException {
                   MapMessage mapMessage = session.createMapMessage();
                   mapMessage.setString("name","张三");
                   mapMessage.setInt("age",20);
                   return mapMessage;
               }
           });
       }
   ```

   

   ```java
       @JmsListener(destination = "springboot_queue")
       public void receiveMapMessage(Message message){
           if (message instanceof MapMessage){
               MapMessage mapMessage = (MapMessage) message;
               try {
                   System.out.println("名称：" + mapMessage.getString("name"));
                   System.out.println("年龄：" + mapMessage.getString("age"));
               } catch (JMSException e) {
                   e.printStackTrace();
               }
           }
       }
   ```

3. ##### ObjectMessage

   ```java
       @Test
       public void sendObjectMessage(){
           //参数一：队列的名称或者主题名称
           //参数二：消息内存
           jmsTemplate.send("springboot_queue", new MessageCreator() {
               @Override
               public Message createMessage(Session session) throws JMSException {
                   //实体类一定要实现 Serializable
                   User user = new User("小明","1234");
                   ObjectMessage objectMessage = session.createObjectMessage(user);
                   return objectMessage;
               }
           });
       }
   ```

   ```java
       @JmsListener(destination = "springboot_queue")
       public void receiveObjectMessage(Message message){
           if (message instanceof ObjectMessage){
               ObjectMessage objectMessage = (ObjectMessage) message;
               try {
                   User user = (User) objectMessage.getObject();
                   System.out.println("名称：" + user.getUsername());
                   System.out.println("密码：" + user.getPwd());
               } catch (JMSException e) {
                   e.printStackTrace();
               }
           }
       }
   ```

   

4. ##### BytesMessage

   ```java
       @Test
       public void sendBytesMessage(){
           //参数一：队列的名称或者主题名称
           //参数二：消息内存
           jmsTemplate.send("springboot_queue", new MessageCreator() {
               @Override
               public Message createMessage(Session session) throws JMSException {
   
                   BytesMessage bytesMessage = session.createBytesMessage();
                   //读取文件
                   File file = new File("");
                   try {
                       //构建文件输入流
                       FileInputStream inputStream = new FileInputStream(file);
                       //把文件读写入到缓存数组里
                       byte[] buffer = new byte[Math.toIntExact(file.length())];
                       inputStream.read(buffer);
                       //把缓存数据组写入到bytesMessage中
                       bytesMessage.writeBytes(buffer);
                   }catch (Exception e) {
                       e.printStackTrace();
                   }
                   return bytesMessage;
               }
           });
       }
   
   ```

   

   ```java
       @JmsListener(destination = "springboot_queue")
       public void receiveByteMessage(Message message){
           if (message instanceof BytesMessage){
               BytesMessage bytesMessage = (BytesMessage) message;
               try {
                   //设计缓存数组
                   byte[] bytes = new byte[(int)bytesMessage.getBodyLength()];
                   //把字节消息的内容写入到缓存数组
                   bytesMessage.readBytes(bytes);
                   //构建文件输出流
                   FileOutputStream outputStream = new FileOutputStream("");
                   //把数据写出本地盘
                   outputStream.write(bytes);
               } catch (Exception e) {
                   e.printStackTrace();
               }
           }
       }
   ```

5. ##### StreamMessage

   ```java
       @Test
       public void sendStreamMessage(){
        //参数一：队列的名称或者主题名称
           //参数二：消息内存
           jmsTemplate.send("springboot_queue", new MessageCreator() {
               @Override
            public Message createMessage(Session session) throws JMSException {
                   StreamMessage streamMessage = session.createStreamMessage();
                   streamMessage.writeString("你好");
                   streamMessage.writeInt(20);
                   return streamMessage;
               }
           });
       
   ```
   
   ```java
       @JmsListener(destination = "springboot_queue")
       public void receiveStreamMessage(Message message){
           if (message instanceof StreamMessage){
               StreamMessage streamMessage = (StreamMessage) message;
               try {
                   System.out.println(streamMessage.readString());
                   System.out.println(streamMessage.readInt());
               } catch (Exception e) {
                   e.printStackTrace();
               }
           }
       }
   ```
   
   

#### 消息持久化

ActiveMQ提供了以下三种的消息存储方式：

1. Memory 消息存储-基于内存的消息存储。

2. 基于日志消息存储方式，KahaDB是ActiveMQ的默认日志存储方式，它提供了容量的提升和恢复能力。

3. 基于JDBC的消息存储方式-数据存储于数据库中。

    ActiveMQ持久化机制流程图

   ![image-20200521140738091](C:\Users\LiYuanZhuo\AppData\Roaming\Typora\typora-user-images\image-20200521140738091.png)

![image-20200521140757364](C:\Users\LiYuanZhuo\AppData\Roaming\Typora\typora-user-images\image-20200521140757364.png)

#### 消息事务

​	消息事务，是保证消息传递原子性的一个重要特征，和JDBC的事务特征类似。

​	生产者、消费者与消费服务器直接都支持事务性；

​	ActiveMQ的事务主要偏向在生产者的应用。

###### 	ActiveMQ消息事务流程图

![image-20200521143010433](C:\Users\LiYuanZhuo\AppData\Roaming\Typora\typora-user-images\image-20200521143010433.png)

#### 如何防止消费方消息重复消费

1. 如果消费方是做数据库操作，那么可以吧消息的ID作为表的唯一主键，这样在重试的情况下，会触发主键冲突，从而避免数据出现脏数据
2. 如果消费方不是做数据库操作，那么可以借助第三方的应用，例如redis，来记录消费记录。每次消息被消费完成时候，把当前消息的ID作为key存入redis，每次消费前，先到redis查询有没有该消息的消费记录。

#### 如何防止消息丢失

1. 在消息生产者和消费者使用事务
2. 在消费方采用手动消息确认（ACK）
3. 消息持久化，例如JDBC或日志