# Quick Start

## Немного теории

Для тех, кто не ещё не знаком с Kafka, проведем краткий экскурс. В этом нам поможет ChatGPT. Зададим ему пару вопросов:

![Диалог с ChatGPT](https://user-images.githubusercontent.com/13710048/234185518-159e03fc-2cea-41ab-9596-eb53dd9b7da2.png "Диалог с ChatGPT")

Примерно так это можно отобразить схематично:

![Знакомьтесь, это наш первый кластер](https://user-images.githubusercontent.com/13710048/234182171-df6df6fc-2d63-414c-a878-5e681f230cc0.png "Знакомьтесь, это наш первый кластер")

Из ответа ChatGPT могло быть не совсем понятно, что такое топики (темы) и партиции (разделы). Какую аналогию можно провести, чтобы понять эти структуры? 

![Топики и партиции](https://user-images.githubusercontent.com/13710048/235059049-cf0502e8-3bd8-4516-a7e7-5b515470d1b3.png "Топики и партиции")

Скажем несколько слов о группах потребителей:

![Группы потребителей](https://user-images.githubusercontent.com/13710048/235061300-23418a2f-5ddc-4333-93f3-3bc169ce7760.png "Группы потребителей")

Уточним исходную схему:

![Уточнённая схема](https://user-images.githubusercontent.com/13710048/235059283-53ae37d3-f20f-451b-898c-234491927867.png "Уточнённая схема")

Это и будет наш первый кластер. Для начала практики этих знаний вполне достаточно.

## Проверим окружение

Перейдите в любую удобную для загрузки архива папку и скачайте последнюю версию Kafka по ссылке:
```
wget https://archive.apache.org/dist/kafka/3.4.0/kafka_2.13-3.4.0.tgz
```

Распакуем архив и перейдем в каталог с приложением:
``` 
tar -xzf kafka_2.13-3.4.0.tgz
cd kafka_2.13-3.4.0/
```

Первым делом запускаем ZooKeeper:
``` 
./bin/zookeeper-server-start.sh config/zookeeper.properties
```
В соседней консоли запускаем брокер Kafka:
```
./bin/kafka-server-start.sh config/server.properties
```
Мы должны увидеть сообщения, которые скажут о том, что брокер 
1) подключился к ZooKeeper:
```INFO [ZooKeeperClient Kafka server] Connected. (kafka.zookeeper.ZooKeeperClient)```
2) стартовал и получил идентификатор:
```INFO [KafkaServer id=0] started (kafka.server.KafkaServer)```

Если вы увидели эти строки в логе, то вы готовы к первой части практики. Теперь можете прервать выполнение ZooKeeper и Kafka в соответствующих консолях, отправив сигнал SIGINT или завершить работу кластера штатно:
```
./bin/kafka-server-stop.sh config/server.properties
./bin/zookeeper-server-stop.sh config/zookeeper.properties
```

Далее мы будем поднимать кластер Kafka с помощью docker compose. Убедитесь, что [кластер из трех брокеров Kafka и одной ноды ZooKeeper](https://github.com/Sublimee/kafka-livecoding/blob/main/simple-cluster/docker-compose.yml) успешно стартует. Вы уже знаете как это сделать (посмотреть в логи каждого контейнера с брокером Kafka по аналогии с тем, как мы делали это выше).
