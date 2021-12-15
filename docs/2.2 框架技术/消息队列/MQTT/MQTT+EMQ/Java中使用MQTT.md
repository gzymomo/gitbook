# 一、Java中使用MQTT

## 1.1 添加依赖

```xml
<dependencies>
   <dependency>
        <groupId>org.eclipse.paho</groupId>
        <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
        <version>1.2.0</version>
   </dependency>
</dependencies>
```



## 1.2 配置

```java
String HOST = "tcp://ip:1883"  # 服务器地址
String userName = ""  # 用户名
String password = ""  # 密码
String clientId = ""  # 
int qos = 1|2|0	# 消息的级别
String topic = ""  # 发布的主题
String count = ""  # 发布的内容  
```

## 1.3 发布端

```java
MemoryPersistence memoryPersistence = new MemoryPersistence();
        try {
            MqttClient client = new MqttClient("","",memoryPersistence);
            MqttConnectOptions connOpts = new MqttConnectOptions();
            connOpts.setCleanSession(false);
            connOpts.setUserName("");
            connOpts.setPassword("".toCharArray());
            client.connect(connOpts);
            // 设置回调函数，当订阅到信息时调用此方法
            client.setCallback(new MqttCallback() {
                @Override
                public void connectionLost(Throwable throwable) {
                    // 连接失败时调用
                }

                @Override
                public void messageArrived(String s, MqttMessage mqttMessage) throws Exception {
                    // 订阅成功，并接受信息时调用
                    mqttMessage.getPayload(); // 获取消息内容
                }

                @Override
                public void deliveryComplete(IMqttDeliveryToken iMqttDeliveryToken) {

                }
            });
            
            client.subscribe("xxx");
            
            client.disconnect();
            client.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
```

## 1.4 接收端

```bash
MemoryPersistence memoryPersistence = new MemoryPersistence();
        try {
            MqttClient client = new MqttClient("","",memoryPersistence);
            MqttConnectOptions connOpts = new MqttConnectOptions();
            connOpts.setCleanSession(false);
            connOpts.setUserName("");
            connOpts.setPassword("".toCharArray());
            client.connect(connOpts);
            // 设置回调函数，当订阅到信息时调用此方法
            client.setCallback(new MqttCallback() {
                @Override
                public void connectionLost(Throwable throwable) {
                    // 连接失败时调用
                }

                @Override
                public void messageArrived(String s, MqttMessage mqttMessage) throws Exception {
                    // 订阅成功，并接受信息时调用
                    mqttMessage.getPayload(); // 获取消息内容
                }

                @Override
                public void deliveryComplete(IMqttDeliveryToken iMqttDeliveryToken) {

                }
            });
            
            client.subscribe("xxx");
            
            client.disconnect();
            client.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
```

# 二、Demo2

## 2.1 封装Mqtt

```java
public class MyMqtt {
    private String host = "tcp://localhost:61613";
    private String userName = "admin";
    private String passWord = "password";
    private MqttClient client;
    private String id;
    private static MyMqtt instance; // = new MyMqtt();
    private MqttTopic mqttTopic;
    private String myTopic = "Topics/htjs/serverToPhone";
    private MqttMessage message;
    public MyMqtt(String id) {
        this(id, null, false);
    }
    
    public MyMqtt(String id, MqttCallback callback, boolean cleanSession){
        try {
             //id应该保持唯一性
            client = new MqttClient(host, id, new MemoryPersistence());
            MqttConnectOptions options = new MqttConnectOptions();
            options.setCleanSession(cleanSession);
            options.setUserName(userName);
            options.setPassword(passWord.toCharArray());
            options.setConnectionTimeout(10);
            options.setKeepAliveInterval(20);
            if (callback == null) {
                client.setCallback(new MqttCallback() {
    
                    @Override
                    public void connectionLost(Throwable arg0) {
                        // TODO 自动生成的方法存根
                        System.out.println(id + " connectionLost " + arg0);
                    }
    
                    @Override
                    public void deliveryComplete(IMqttDeliveryToken arg0) {
                        // TODO 自动生成的方法存根
                        System.out.println(id + " deliveryComplete " + arg0);
                    }
    
                    @Override
                    public void messageArrived(String arg0, MqttMessage arg1) throws Exception {
                        // TODO 自动生成的方法存根
                        System.out.println(id + " messageArrived: " + arg1.toString());
                    }
                });
            } else {
                client.setCallback(callback);
            }
            client.connect(options);
        } catch (MqttException e) {
            // TODO 自动生成的 catch 块
            e.printStackTrace();
        }
    }
    
    public void sendMessage(String msg) {
        sendMessage(myTopic, msg);
    }
    
    public void sendMessage(String topic, String msg){
        try {
            message = new MqttMessage();
            message.setQos(1);
            message.setRetained(true);
            message.setPayload(msg.getBytes());
            mqttTopic = client.getTopic(topic);
            MqttDeliveryToken token = mqttTopic.publish(message);//发布主题
            token.waitForCompletion();
        } catch (MqttPersistenceException e) {
            // TODO 自动生成的 catch 块
            e.printStackTrace();
        } catch (MqttException e) {
            // TODO 自动生成的 catch 块
            e.printStackTrace();
        }
    }
    
    public void subscribe(String[] topicFilters, int[] qos) {
        try {
            client.subscribe(topicFilters, qos);
        } catch (MqttException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }// 订阅主题

    }
    
}
```

## 2.2 调用

Sever端发布主题：

```java
MyMqtt myMqtt = new MyMqtt("Sever");
myMqtt.sendMessage("发你信息");
```

Client端订阅主题：

```java
private MqttClient client;
private MqttConnectOptions options;
private static String[] myTopics = { "Topics/htjs/phoneToServer", "Topics/htjs/serverToPhone" };
private static int[] myQos = { 2, 2 };

public static void main(String[] args) {
    System.out.println("client start...");
    MyMqtt myMqtt = new MyMqtt("client");
    myMqtt.subscribe(myTopics, myQos);
}
```

