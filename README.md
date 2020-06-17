# Docker 命令
### 1.私有仓库
> docker run -d -p 5000:5000 -v /program/docker/data/registry:/tmp/registry registry

### 2.Redis
#### redis服务端
##### 正常启动 
> docker run -p 6379:6379 -d redis:latest redis-server
##### 参数启动 
> docker run -p 6379:6379 -v $PWD/data:/data -d redis:latest redis-server --appendonly yes
##### 命令说明：
> -p 6379:6379 : 将容器的6379端口映射到主机的6379端口<br/>
> -v $PWD/data:/data : 将主机中当前目录下的data挂载到容器的/data<br/>
> --appendonly yes : 在容器执行redis-server启动命令，并打开redis持久化配置

#### redis客户端
> docker exec -ti d0b86 redis-cli <br/>
> docker exec -ti d0b86 redis-cli -h localhost -p 6379 <br/>
> docker exec -ti d0b86 redis-cli -h 127.0.0.1 -p 6379 <br/>
> docker exec -ti d0b86 redis-cli -h 172.17.0.3 -p 6379 <br/>
> docker exec -it redis_s redis-cli <br/>
> docker exec -it redis_s redis-cli -h 192.168.1.100 -p 6379 -a your_password
##### 查看容器ip
> docker inspect redis_s | grep IPAddress

### 3.RabbitMQ
> docker run --name rabbitmq -d -p 15672:15672 -p 5672:5672 rabbitmq:management



### 4.Kafka
1、kafka需要zookeeper管理，所以需要先安装zookeeper。 
下载zookeeper镜像
>  docker pull wurstmeister/zookeeper

2、启动镜像生成容器
>  docker run -d --restart=always --log-driver json-file --log-opt max-size=100m --log-opt max-file=2  --name zookeeper -p 2181:2181 -v /etc/localtime:/etc/localtime wurstmeister/zookeeper

3、下载kafka镜像
>  docker pull wurstmeister/kafka

4、启动kafka镜像生成容器
>  docker run -d --restart=always --log-driver json-file --log-opt max-size=100m --log-opt max-file=2 --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=172.17.0.12:2181/kafka -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://175.24.50.204:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -v /etc/localtime:/etc/localtime wurstmeister/kafka

> 参数说明：<br/>
> -e KAFKA_BROKER_ID=0  在kafka集群中，每个kafka都有一个BROKER_ID来区分自己<br/>
> -e KAFKA_ZOOKEEPER_CONNECT=172.17.0.12:2181/kafka 配置zookeeper管理kafka的路径172.17.0.12:2181/kafka<br/>
> -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://172.17.0.12:9092  把kafka的地址端口注册给zookeeper，如果是远程访问要改成外网IP,类如Java程序访问出现无法连接。<br/>
> -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 配置kafka的监听端口<br/>
> -v /etc/localtime:/etc/localtime 容器时间同步虚拟机的时间<br/>

5、验证kafka是否可以使用
> 5.1、进入容器
docker exec -it kafka bash<br/>
> 5.2、进入 /opt/kafka_2.12-2.3.0/bin/ 目录下
 cd /opt/kafka_2.12-2.3.0/bin/<br/>
> 5.3、运行kafka生产者发送消息
./kafka-console-producer.sh --broker-list localhost:9092 --topic sun<br/>

发送消息
> {"datas":[{"channel":"","metric":"temperature","producer":"ijinus","sn":"IJA0101-00002245","time":"1543207156000","value":"80"}],"ver":"1.0"}
 
5.4、运行kafka消费者接收消息
>  ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic sun --from-beginning

### 5.es
拉取镜像
> docker pull elasticsearch:6.8.3
启动
> docker run --name=test_es -d -p 9200:9200 -p 9300:9300 docker.io/elasticsearch:6.8.3
单节点启动

> docker run --name=elasticsearch6.8.3 -e "discovery.type=single-node" -d -p 9200:9200 -p 9300:9300 docker.io/elasticsearch:6.8.3
> docker run --name=elasticsearch6.8.3 -e "discovery.type=single-node" -d -p 9200:9200 -p 9300:9300  -v /program/elasticsearch/data:/usr/share/elasticsearch/data  -v /program/elasticsearch/logs:/usr/share/elasticsearch/logs  -v /program/elasticsearch/config:/usr/share/elasticsearch/config docker.io/elasticsearch:6.8.3


> docker run --name=elasticsearch6.8.3 -e "discovery.type=single-node" -d -p 9200:9200 -p 9300:9300 docker.io/elasticsearch:6.8.3

es 启动常见问题
> https://www.cnblogs.com/ming-blogs/p/11184932.html
启动kibana
> docker run --name kibana6.8.3 -d -p 5601:5601 -e ELASTICSEARCH_URL=http://172.17.0.12:9200 kibana:6.8.3
es简单查询语句
https://www.jianshu.com/p/c377477df7fc

查询文件

find / -name jvm.options

docker删除推出的容器
> docker rm $(docker ps -q -f status=exited)

centos 添加swap空间
> https://www.jianshu.com/p/48392f3eef76

解决办法：
在/etc/sysctl.conf文件最后添加一行：
vm.max_map_count=262144

立即生效, 执行：
/sbin/sysctl -p

> docker run -d -p 9200:9200 -p 9300:9300 --name es1 -h es1\
 -e cluster.name=yun -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -e xpack.security.enabled=false\
  docker.elastic.co/elasticsearch/elasticsearch:6.8.3

> docker run -d -p 9201:9200 -p 9301:9300 --link es1\
  --name es2 -e cluster.name=yun -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -e xpack.security.enabled=false\
  -e discovery.zen.ping.unicast.hosts=es1 docker.elastic.co/elasticsearch/elasticsearch:6.8.3


> docker run -d --name kafka-manager --link zookeeper:zookeeper --link kafka:kafka -p 9001:9000 --restart=always --env ZK_HOSTS=zookeeper:2181 sheepkiller/kafka-manager
