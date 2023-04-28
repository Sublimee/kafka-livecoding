# Логи 

``` 
ls -la ./logs
total 56
drwxr-xr-x 2 root root  4096 Mar 31 15:57 .
drwxr-xr-x 1 root root  4096 Mar 31 15:57 ..
-rw-r--r-- 1 root root   414 Mar 31 15:57 controller.log
-rw-r--r-- 1 root root     0 Mar 31 15:57 kafka-authorizer.log
-rw-r--r-- 1 root root     0 Mar 31 15:57 kafka-request.log
-rw-r--r-- 1 root root   172 Mar 31 15:57 log-cleaner.log
-rw-r--r-- 1 root root 33612 Mar 31 15:57 server.log
-rw-r--r-- 1 root root     0 Mar 31 15:57 state-change.log
``` 

# Запись и чтение сообщений

В новой консоли создаем топик, указывая адрес брокера и порт, который слушает Kafka по умолчанию:

``` 
./bin/kafka-topics.sh --create --topic test --bootstrap-server localhost:9092
``` 

В случае успешного создания топика вы увидите следующее:

``` 
Created topic test.
``` 

Как это отражается в логах брокера? В логе мы увидим сообщение:

``` 
INFO Created log for partition test-0 in /tmp/kafka-logs/test-0 with properties {} (kafka.log.LogManager)
``` 

Была создана новая директория для хранения данных топика с одной партицией (значение по умолчанию, если явно не указано другое). Посмотрим на его конфигурацию:

``` 
./bin/kafka-topics.sh --describe --topic test --bootstrap-server localhost:9092
``` 

Вы увидите примерно следующее:

``` 
Topic: test	TopicId: BxJo_cf9QZqPI-CW-scVBQ PartitionCount: 1 	ReplicationFactor: 1	Configs: 
	Topic: test	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
``` 
Запишем первые сообщения:

``` 
./bin/kafka-console-producer.sh --topic test --bootstrap-server localhost:9092
>Hello world!
>Hello Alfa Mobile!
``` 

и попробуем их прочитать в соседней консоли:

``` 
./bin/kafka-console-consumer.sh --topic test --bootstrap-server localhost:9092
``` 

Какие сообщения вы получили?

<details>
  <summary>Спойлер</summary>
  
  Для того чтобы прочитать данные с начала, мы должны переопределить настройку  ```auto.offset.reset ```:
  
  ``` 
  ./bin/kafka-console-consumer.sh --topic test --bootstrap-server localhost:9092 --consumer-property auto.offset.reset=earliest
  ``` 
  
  или использовать более короткий вариант:
  
  ``` 
  ./bin/kafka-console-consumer.sh --topic test --bootstrap-server localhost:9092 --from-beginning
  ``` 
  
  В консоли мы должны увидеть ранее записанные сообщения:
  
  ``` 
  Hello world!
  Hello Alfa Mobile!
  ``` 
  
</details>

Запишем ещё одно сообщение через консоль продюсера и увидим, что оно появляется в консьюмере. Обратим внимание на лог брокера. Каждый запуск консольного консьюмера создает новую группу:
``` 
INFO [GroupCoordinator 0]: Dynamic member with unknown member id joins group console-consumer-72251 in Empty state. Created a new member id console-consumer-d95c604b-81ba-42b1-8a71-c0919820db5f and request the member to rejoin with this id. (kafka.coordinator.group.GroupCoordinator)
INFO [GroupCoordinator 0]: Preparing to rebalance group console-consumer-72251 in state PreparingRebalance with old generation 0 (__consumer_offsets-48) (reason: Adding new member console-consumer-d95c604b-81ba-42b1-8a71-c0919820db5f with group instance id None; client reason: rebalance failed due to MemberIdRequiredException) (kafka.coordinator.group.GroupCoordinator)
INFO [GroupCoordinator 0]: Stabilized group console-consumer-72251 generation 1 (__consumer_offsets-48) with 1 members (kafka.coordinator.group.GroupCoordinator)
INFO [GroupCoordinator 0]: Assignment received from leader console-consumer-d95c604b-81ba-42b1-8a71-c0919820db5f for group console-consumer-72251 for generation 1. The group has 1 members, 0 of which are static. (kafka.coordinator.group.GroupCoordinator)
``` 

Остановим консьюмер. В логе брокера по истечении таймаута увидим сообщение о выходе консьюмер из группы:

``` 
INFO [GroupCoordinator 0]: Member MemberMetadata(memberId=console-consumer-d95c604b-81ba-42b1-8a71-c0919820db5f, groupInstanceId=None, clientId=console-consumer, clientHost=/127.0.0.1, sessionTimeoutMs=45000, rebalanceTimeoutMs=300000, supportedProtocols=List(range, cooperative-sticky)) has left group console-consumer-72251 through explicit `LeaveGroup`; client reason: the consumer is being closed (kafka.coordinator.group.GroupCoordinator)
``` 

Давайте теперь явно зададим группу для нового консьюмера:

``` 
./bin/kafka-console-consumer.sh --topic test --group alfa --bootstrap-server localhost:9092 --consumer-property auto.offset.reset=earliest
``` 

В консоли видим ранее записанные в топик сообщения сообщения. В логах мы видим, что консьюмер подключился именно с той группой, которую мы задали:

``` 
INFO [GroupCoordinator 0]: Dynamic member with unknown member id joins group alfa in Empty state. Created a new member id console-consumer-7071eb26-cc6a-4618-aa1e-a3aa9e14eda2 and request the member to rejoin with this id. (kafka.coordinator.group.GroupCoordinator)
INFO [GroupCoordinator 0]: Preparing to rebalance group alfa in state PreparingRebalance with old generation 0 (__consumer_offsets-24) (reason: Adding new member console-consumer-7071eb26-cc6a-4618-aa1e-a3aa9e14eda2 with group instance id None; client reason: rebalance failed due to MemberIdRequiredException) (kafka.coordinator.group.GroupCoordinator)
INFO [GroupCoordinator 0]: Stabilized group alfa generation 1 (__consumer_offsets-24) with 1 members (kafka.coordinator.group.GroupCoordinator)
INFO [GroupCoordinator 0]: Assignment received from leader console-consumer-7071eb26-cc6a-4618-aa1e-a3aa9e14eda2 for group alfa for generation 1. The group has 1 members, 0 of which are static. (kafka.coordinator.group.GroupCoordinator)
``` 

Перезапустим консьюмер:

``` 
./bin/kafka-console-consumer.sh --topic test --group alfa --bootstrap-server localhost:9092 --consumer-property auto.offset.reset=earliest
``` 

Какие сообщения вы получили?

<details>
  <summary>Спойлер</summary>
  
  В консоли мы снова ничего не видим. Почему? Консьюмеры группы в Kafka могут коммитить свои оффсеты (смещения) для топик-партиции, чтобы при перезапуске продолжать чтение с последней запомненной позиции. Именно это поведение мы и видим. Давайте это проверим с помощью команды: 
  
  ``` 
  ./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group alfa --describe
  GROUP	  TOPIC   PARTITION	  CURRENT-OFFSET	LOG-END-OFFSET	LAG	CONSUMER-ID	                                          HOST        CLIENT-ID
  alfa    test   	0          	2		            2			          0   console-consumer-4c5a6d95-e88f-4e34-8af0-bc886d2d434f /127.0.0.1  console-consumer
  ``` 
  
  Группа alfa в топике test и партиции с идентификатором 0 сохранила свой оффсет в позиции 2, что и является концом топика (LOG-END-OFFSET=2), поэтому LAG=0.
  Если выполнить команду без подключенных консьюмеров, то увидим следующее:
  
  ``` 
  Consumer group 'alfa' has no active members.
  GROUP	  TOPIC	  PARTITION	  CURRENT-OFFSET	LOG-END-OFFSET	LAG	  CONSUMER-ID	HOST	CLIENT-ID
  alfa    test   	0          	2		            2			          0     -		        -     -
  ``` 
  
</details>

А теперь сбросим позицию “указателя” на начало:

``` 
./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group alfa --topic test --reset-offsets --to-earliest --execute
``` 

В консоли видим новое состояние метаданных топик-партиций:

``` 
GROUP   TOPIC   PARTITION  NEW-OFFSET     
alfa    test    0          0
``` 

Теперь запустим консьюмер с отключенным автоматическим коммитом оффсетов при чтении:

``` 
./bin/kafka-console-consumer.sh --topic test --bootstrap-server localhost:9092 --group alfa --consumer-property auto.offset.reset=earliest --consumer-property enable.auto.commit=false
```

Из раза в раз при таком запуске консьюмера видим, что все сообщения повторно считываются, так как оффсеты не коммитятся автоматически. В этом случае информация о группе будет выглядеть следующим образом:

``` 
./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group alfa --describe
GROUP	  TOPIC	  PARTITION	  CURRENT-OFFSET	LOG-END-OFFSET	LAG	CONSUMER-ID	                                          HOST	          CLIENT-ID
alfa    test   	0          	0		            2		            0   console-consumer-d9027f5a-d0a9-4031-93f8-4ed268a2e9a6 /127.0.0.1      console-consumer
``` 

# Удаление топика

Наконец, удалим топик, чтобы завершить эту серию экспериментов:

```
./bin/kafka-topics.sh --delete --topic test --bootstrap-server localhost:9092
```

# Информация о кластере Kafka в ZooKeeper

Давайте заглянем в ZooKeeper, и посмотрим, какую информацию он хранит. Откроем консоль ZooKeeper:

```
./bin/zookeeper-shell.sh localhost:2181
```

Информация в ZooKeeper хранится в иерархической структуре в виде папок и файлов. Посмотрим, что хранится в рутовом ключе:

```
ls /
[admin, brokers, cluster, config, consumers, controller, controller_epoch, feature, isr_change_notification, latest_producer_id_block, log_dir_event_notification, zookeeper]
```

Теперь мы можем получить информацию по соответствующему ключу:

```
get /controller
{"version":1,"brokerid":0,"timestamp":"1677731044279"}
get /brokers/topics/test/partitions/0/state
{"controller_epoch":1,"leader":0,"version":1,"leader_epoch":0,"isr":[0]}
stat /brokers/ids/0
cZxid = 0x9d
ctime = Thu Mar 02 07:24:04 MSK 2023
mZxid = 0x9d
mtime = Thu Mar 02 07:24:04 MSK 2023
pZxid = 0x9d
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x1000003e94a0001
dataLength = 226
numChildren = 0
```

# Резюме по уроку

Если вы хотите затащить Kafka в прод, то уделите внимание в интеграционном контуре на следующее:
* Kafka пишет логи, их может быть много, их ротацию надо настраивать
* при тестировании смотрите, в какие партиции попадают сообщения (поведение может отличаться в зависимости от версии Kafka и настроек продюсеров)
* убедитесь, что при первом запуске и перезапуске консьюмеры начинают вычитывать сообщения с нужной позиции (т.е. коммитят оффсеты и не пропускают сообщения, записанные в их отсутствие)

