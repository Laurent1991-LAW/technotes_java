# Redis主从、哨兵、集群



## 哨兵配置

**结构**：一主二从三哨兵

**前提** : docker已安装且已有redis镜像



### 一、创建配置文件

在/opt文件中创建文件夹6380、6381，`mkdir 6380 6381`

```
# 分别在两个文件夹中创建redis.conf, 均配置为6379端口的从服务器

cat <<EOF >6380/redis.conf
port 6380
slaveof 192.168.64.150 6379
appendonly yes
EOF

cat <<EOF >6381/redis.conf
port 6381
slaveof 192.168.64.150 6379
appendonly yes
EOF
```



### 二、启动三个服务器

```
# 主服务器, 直接使用默认6379端口即可
docker run -d --name redis6379 --net=host --restart=always redis 

# 从服务器1, 文件挂载且直接读取redis.conf中的配置
docker run -d --name redis6380 \
-v /opt/redis/6380/redis.conf:/redis.conf \
--net=host \
--restart=always \
redis \
redis-server /redis.conf

# 同上
docker run -d --name redis6381 \
-v /opt/redis/6381/redis.conf:/redis.conf \
--net=host \
--restart=always \
redis \
redis-server /redis.conf
```



**备注 :** redis容器的启动也可不建redis.conf, 直接在启动语句中配上端口及主从信息也可

![Snipaste_2022-07-25_15-51-58](E:\doc_repo\005-redis\redis主从哨兵集群\images\Snipaste_2022-07-25_15-51-58.png)



### 三、查看主从关系

进入各容器, 查看主从关系

```
docker exec -it redis6379 redis-cli
> info replication

docker exec -it redis6380 redis-cli -p 6380
> info replication

docker exec -it redis6381 redis-cli -p 6381
> info replication
```





### 四、创建哨兵配置文件

在/opt文件中创建文件夹sentinel，内部创建哨兵配置文件 ：

- sentinel monitor mymaster 192.168.64.150 6379 2" 中的mymaster为**自定义的主服务器名称**，末尾的 **2 表示两个哨兵投票确认主服务器宕机**，哨兵才会认为主服务器宕机；
- down-after-milliseconds ：几毫秒后 主服务器无心跳 将被判定为主观下线；
- failover-timeout：当进行failover时，配置所有slaves指向新的master所需的最大时间 ；
- parallel-syncs：这个配置项 指定了 在发生**failover主备切换**时, 最多可以有多少个slave同时对新的master进行 同步，这个数字越小，完成failover所需的时间就越长，但是如果这个数字越大，就意味着越 多的slave因为复制replication而不可用。可以通过**将这个值设为 1 来保证每次只有一个slave 处于不能处理命令请求的状态**。
  

```
cat <<EOF > /sentinel/5000.conf
port 5000
sentinel monitor mymaster 192.168.64.150 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1
EOF

cat <<EOF > /sentinel/5001.conf
port 5001
sentinel monitor mymaster 192.168.64.150 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1
EOF

cat <<EOF > /sentinel/5002.conf
port 5002
sentinel monitor mymaster 192.168.64.150 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1
EOF
```



### 五、启动三个哨兵



```
# 启动哨兵1
docker run -d --name sentinel5000 \
-v /opt/sentinel/5000.conf:/sentinel.conf \
--net=host \
redis \
redis-sentinel /sentinel.conf

# 启动哨兵2
docker run -d --name sentinel5001 \
-v /opt/sentinel/5001.conf:/sentinel.conf \
--net=host \
redis \
redis-sentinel /sentinel.conf

# 启动哨兵3
docker run -d --name sentinel5002 \
-v /opt/sentinel/5002.conf:/sentinel.conf \
--net=host \
redis \
redis-sentinel /sentinel.conf
```



### 六、查看哨兵情况

```
# 进入容器并开启哨兵程序
docker exec -it sentinel5000 redis-cli -p 5000

# 输入指令，查看哨兵监控的主服务器、从服务器和所有哨兵情况
> sentinel master mymaster
> sentinel slaves mymaster
> sentinel sentinels mymaster
```



### 七、模拟主服务器宕机

```
# 停止主服务器
docker stop redis6379
# 在哨兵日志中查看服务器切换日志： +switch-master mymaster 192.168.64.150 6379 192.168.64.150 6381
docker logs sentinel5000

# 查看 6380 和 6381 服务器的角色变化
docker exec -it redis6380 redis-cli -p 6380
> info replication

docker exec -it redis6381 redis-cli -p 6381
> info replication

# 重新启动6379，不会把6379切换成主服务器，而是作为从服务器

```



