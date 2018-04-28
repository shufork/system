# system modules

system modules for cloud service support

# 构建
```
mvnw clean package -P 'docker-build'
```

# 部署

RabbitMQ(使用docker)
```
docker run -d --name rabbitmq --publish 5671:5671 rabbitmq:management
```

```
#!/bin/sh
## global
eureka_img=hub.docker.com/r/jcalzz/repo1/eureka
config_img=hub.docker.com/r/jcalzz/repo1/config
admin_img=hub.docker.com/r/jcalzz/repo1/admin

config_url=https://gitee.com/wsa10054/config-repo
eureka_host1=$(hostname)
eureka_port1=8761
eureka_host2=$(hostname)
eureka_port2=8762
zookeeper_host=192.168.0.35
zookeeper_port=2181
rabbitmq_host=192.168.0.166
rabbitmq_port=5671
kafka_host=192.168.0.35
kafka_port=9092
mysql_host=192.168.0.51
mysql_port=3306
mysql_user=dev
mysql_pwd=zfmm
redis_host=192.168.0.51
redis_port=6379

# eureka 1
docker run --rm -itd -p $eureka_port1:$eureka_port1 --network=host  --name="eureka1"  \
${eureka_img} \
--spring.profiles.active=peer1 \
--server.port=$eureka_port1 \
--eureka.instance.hostname=$eureka_host1 \
--eureka.client.service-url.defaultZone="http://$eureka_host1:$eureka_port1/eureka/,http://$eureka_host2:$eureka_port2/eureka/"

./wait-for-it.sh $eureka_host1:$eureka_port1 --timeout=30 --strict -- echo "eureka 1 is up"

# eureka 2
docker run --rm -itd -p $eureka_port2:$eureka_port2 --network=host  --name="eureka2"  \
${eureka_img} \
--spring.profiles.active=peer2 \
--server.port=$eureka_port2 \
--eureka.instance.hostname=$eureka_host2 \
--eureka.client.service-url.defaultZone="http://$eureka_host2:$eureka_port2/eureka/,http://$eureka_host1:$eureka_port1/eureka/"

./wait-for-it.sh $eureka_host2:$eureka_port2 --timeout=30 --strict -- echo "eureka 2 is up"

# config (8888)
docker run --rm -itd -p 8888:8888 --network=host  --name="config"  \
${config_img} \
--server.port=8888 \
--spring.cloud.config.server.git.uri=$config_url \
--spring.cloud.config.label=xd-office \
--RABBITMQ_HOST=$rabbitmq_host \
--RABBITMQ_PORT=$rabbitmq_port \
--eureka.client.service-url.defaultZone="http://$eureka_host1:$eureka_port1/eureka/,http://$eureka_host2:$eureka_port2/eureka/"

./wait-for-it.sh 127.0.0.1:8888 --timeout=30 --strict -- echo "config-server is up"

# admin (19001)
docker run --rm -itd -p 19001:19001 --network=host  --name="admin"  \
${admin_img} \
--spring.profiles.active=test \
--server.port=19001 \
--eureka.client.service-url.defaultZone="http://$eureka_host2:$eureka_port2/eureka/,http://$eureka_host1:$eureka_port1/eureka/"

./wait-for-it.sh 127.0.0.1:19001 --timeout=30 --strict -- echo "admin-server is up"

```

# 开发环境特殊配置

eureka 服务器
```
# 关闭自我保护
--eureka.server.enable-self-preservation=false
# 清理间隔(单位毫秒,默认是60*1000)
--eureka.server.eviction-interval-timer-in-ms=20000
```
eureka 客户端
```
# 开启健康检查(需要spring-boot-starter-actuator依赖)
--eureka.client.healthcheck.enabled=true
# 续约更新时间间隔(默认30秒)	
--eureka.instance.lease-renewal-interval-in-seconds=10	
# 续约到期时间(默认90秒)
--eureka.instance.lease-expiration-duration-in-seconds=30
```