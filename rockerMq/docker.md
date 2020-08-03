# Docker 安装 RocketMQ及简单使用

## 检索镜像

```
ocker search rocketmq
```

显示以下信息：

```
NAME                               DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
rocketmqinc/rocketmq               Image repository for Apache RocketMQ            35                                      
styletang/rocketmq-console-ng      rocketmq-console-ng                             27                                      
foxiswho/rocketmq                  rocketmq                                        23                                      
laoyumi/rocketmq                                                                   10                                      [OK]
apacherocketmq/rocketmq            Docker Image for Apache RocketMQ                8                                       
xlxwhy/rocketmq                    alibaba's rocketmq                              4                                       
rocketmqinc/rocketmq-namesrv       Customized RocketMQ Name Server Image for Ro…   2                                       
huanwei/rocketmq-broker                                                            2                                       
rocketmqinc/rocketmq-broker        Customized RocketMQ Broker Image for RocketM…   1                                       
apacherocketmq/rocketmq-operator   RocketMQ Operator is to manage RocketMQ serv…   1                                       
pangliang/rocketmq-console-ng                                                      1                                       
2019liurui/rocketmq-namesrv        RocketMQ name service image for RocketMQ-Ope…   1                                       
2019liurui/rocketmq-broker         RocketMQ broker image for RocketMQ-Operator     1                                       
coder4/rocketmq                    rocketmq                                        1                                       [OK]
pengzu/rocketmq-console-ng         web console for rocketmq ,this code is from …   0                                       
huanwei/rocketmq                                                                   0                                       
huanwei/rocketmq-broker-k8s                                                        0                                       
rocketmqinc/rocketmq-operator      The Kubernetes operator for RocketMQ            0                                       
king019/rocketmq                   rocketmq                                        0                                       
slpcat/rocketmq-console-ng                                                         0                                       
2019liurui/rocketmq-operator       Kubernetes Operator for RocketMQ !              0                                       
huanwei/rocketmq-operator                                                          0                                       
fengzt/rocketmq-broker             apache rocketmq 4.2.0 broker server(官方文档…       0                                       
fengzt/rocketmq-nameserver         apache rocketmq 4.2.0 nameserver                0                                       
slpcat/rocketmq                                                                    0
 
```

## 检索具体版本

我就随便选一个吧，如foxiswho/rocketmq，以下是一个查看当前镜像所有的版本shell命令：

```
curl https://registry.hub.docker.com/v1/repositories/foxiswho/rocketmq/tags | tr -d '[\[\]" ]' | tr '}' '\n' | awk -F: -v image='foxiswho/rocketmq' '{if(NR!=NF && $3 != ""){printf("%s:%s\n",image,$3)}}'
```

显示以下信息：

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   934    0   934    0     0    310      0 --:--:--  0:00:03 --:--:--   310
foxiswho/rocketmq:4.7.0
foxiswho/rocketmq:base
foxiswho/rocketmq:base-4.3.2
foxiswho/rocketmq:base-4.4.0
foxiswho/rocketmq:base-4.5.0
foxiswho/rocketmq:base-4.5.1
foxiswho/rocketmq:base-4.5.2
foxiswho/rocketmq:base-4.6.1
foxiswho/rocketmq:broker
foxiswho/rocketmq:broker-4.3.2
foxiswho/rocketmq:broker-4.4.0
foxiswho/rocketmq:broker-4.5.0
foxiswho/rocketmq:broker-4.5.1
foxiswho/rocketmq:broker-4.5.2
foxiswho/rocketmq:broker-4.6.1
foxiswho/rocketmq:broker-4.7.0
foxiswho/rocketmq:server
foxiswho/rocketmq:server-4.3.2
foxiswho/rocketmq:server-4.4.0
foxiswho/rocketmq:server-4.5.0
foxiswho/rocketmq:server-4.5.1
foxiswho/rocketmq:server-4.5.2
foxiswho/rocketmq:server-4.6.1
foxiswho/rocketmq:server-4.7.0
```


## 启动NameServer

`docker run -d -p 9876:9876 --name rmqserver  foxiswho/rocketmq:server-4.5.1`

## 启动broker

```
docker run -d -p 10911:10911 -p 10909:10909 --name rmqbroker --link rmqserver:namesrv -e "NAMESRV_ADDR=namesrv:9876" -e "JAVA_OPTS=-Duser.home=/opt" -e "JAVA_OPT_EXT=-server -Xms128m -Xmx128m" foxiswho/rocketmq:broker-4.5.1
```

Broker容器中默认的配置文件的路径为：

`/etc/rocketmq/broker.conf`

也可以通过-v参数指定本机的配置文件：

```
docker run -d -p 10911:10911 -p 10909:10909 --name rmqbroker --link rmqserver:namesrv -e "NAMESRV_ADDR=namesrv:9876" -e "JAVA_OPTS=-Duser.home=/opt" -e "JAVA_OPT_EXT=-server -Xms128m -Xmx128m" -v /conf/broker.conf:/etc/rocketmq/broker.conf foxiswho/rocketmq:broker-4.5.1
```

- 注意事项：
    
    broker.conf的配置内容如下：
    ```
    brokerClusterName = DefaultCluster
    brokerName = broker-a
    brokerId = 0
    deleteWhen = 04
    fileReservedTime = 48
    brokerRole = ASYNC_MASTER
    flushDiskType = ASYNC_FLUSH
    ```

    在最后一行后面新增

    `brokerIP1 = 192.168.3.27`

    ip改成你的linux宿主机的ip

## 安装 rocketmq console

如果一切正常，NameServer和Broker一会儿就会安装好，为了管理上的方便，rocketmq console也是必不可少的工具了，通过上面查询的方式找到需要启动的版本，启动方式如下：

```
docker run -d --name rmqconsole -p 8180:8080 --link rmqserver:namesrv\
 -e "JAVA_OPTS=-Drocketmq.namesrv.addr=namesrv:9876\
 -Dcom.rocketmq.sendMessageWithVIPChannel=false"\
 -t styletang/rocketmq-console-ng
```

然后通过如下命令检查一下启动情况：

`docker ps|grep rocketmq`

结果如下：

```
614a3d0ed22b        styletang/rocketmq-console-ng        "sh -c 'java $JAVA_O…"   2 hours ago         Up 2 hours          0.0.0.0:8180->8080/tcp                                                                                                                                       rmqconsole
a76891fbfd35        foxiswho/rocketmq:broker-4.5.1       "/bin/sh -c 'cd ${RO…"   2 hours ago         Up 16 minutes       0.0.0.0:10909->10909/tcp, 0.0.0.0:10911->10911/tcp                                                                                                           rmqbroker
01d29f8f68c8        foxiswho/rocketmq:server-4.5.1       "/bin/sh -c 'cd ${RO…"   2 hours ago         Up 2 hours          0.0.0.0:9876->9876/tcp 
```

- 其他docker命令：
    
    重启网络
   
    `systemctl restart network`

    停止、删除所有docker容器

    ```
    docker stop $(docker ps -a -q)  #停止所有docker容器
    docker rm $(docker ps -a -q)  #删除所有
    ```

## 简单Java示例

maven依赖

```xml
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client</artifactId>
    <version>4.5.0</version>
</dependency>
```

生产者
```java
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.common.RemotingHelper;

public class Producer {
	public static void main(String[] args) throws MQClientException, InterruptedException {

		// 需要一个producer group名字作为构造方法的参数，这里为producer1
		DefaultMQProducer producer = new DefaultMQProducer("producer1");

		// 设置NameServer地址,此处应改为实际NameServer地址，多个地址之间用；分隔
		// NameServer的地址必须有，但是也可以通过环境变量的方式设置，不一定非得写死在代码里
		producer.setNamesrvAddr("192.168.3.27:9876");
		producer.setVipChannelEnabled(false);

		// 为避免程序启动的时候报错，添加此代码，可以让rocketMq自动创建topickey
		producer.setCreateTopicKey("AUTO_CREATE_TOPIC_KEY");
		producer.start();

		for (int i = 0; i < 10; i++) {
			try {
				// topic 主题名称
				// pull 临时值 在消费者消费的时候 可以根据msg类型进行消费
				// body 内容
				Message message = new Message("producer-topic", "msg", ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET));
				SendResult sendResult = producer.send(message);

				System.out.println("发送的消息ID:" + sendResult.getMsgId() + "--- 发送消息的状态：" + sendResult.getSendStatus());
			} catch (Exception e) {
				e.printStackTrace();
				Thread.sleep(1000);
			}
		}
		producer.shutdown();
	}
}

```

消费者
```java
import java.util.List;
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.consumer.ConsumeFromWhere;
import org.apache.rocketmq.common.message.MessageExt;

public class Consumer {
	private static final String ADDR = "192.168.3.27:9876";

	public static void main(String[] args) throws MQClientException {
		// 设置消费者组
		DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer-group");

		consumer.setVipChannelEnabled(false);
		consumer.setNamesrvAddr(ADDR);
		// 设置消费者端消息拉取策略，表示从哪里开始消费
		consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);

		// 设置消费者拉取消息的策略，*表示消费该topic下的所有消息，也可以指定tag进行消息过滤
		consumer.subscribe("producer-topic", "msg");

		// 消费者端启动消息监听，一旦生产者发送消息被监听到，就打印消息，和rabbitmq中的handlerDelivery类似
		consumer.registerMessageListener(new MessageListenerConcurrently() {

			@Override
			public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
				for (MessageExt messageExt : msgs) {
					String topic = messageExt.getTopic();
					String tag = messageExt.getTags();
					String msg = new String(messageExt.getBody());
					System.out.println("*********************************");
					System.out.println("消费响应：msgId : " + messageExt.getMsgId() + ",  msgBody : " + msg + ", tag:" + tag + ", topic:" + topic);
					System.out.println("*********************************");
				}

				return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
			}
		});

		// 调用start()方法启动consumer
		consumer.start();
		System.out.println("Consumer Started....");
	}
}

```

