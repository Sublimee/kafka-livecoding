# Отказоустойчивость кластера

Воспроизведем основные сценарии того, что может пойти не так в кластере и как на это реагирует система. Все эксперименты проводим в кластере с одним ZooKeeper.

## Эксперимент 1. Останавливаем брокер Kafka штатно

Останавливаем брокер Kafka штатно:

```
docker-compose stop kafka-1
```

Проверяем, что отключение произошло контролируемо:

```
docker-compose logs kafka-1 | grep "Starting controlled shutdown"
```

Брокер Kafka по умолчанию выполняет контролируемое отключение, чтобы минимизировать перебои в обслуживании клиентов. Подробнее [здесь](https://www.confluent.io/blog/apache-kafka-supports-200k-partitions-per-cluster/#:~:text=The%20Kafka%20broker%20does%20a,it%27s%20about%20to%20shut%20down).

При старте брокера мы должны увидеть сообщение следующего формата:

```
"Skipping recovery of ${logsToLoad.length} logs from $logDirAbsolutePath since clean shutdown file was found"
```

## Эксперимент 2. Останавливаем брокер Kafka нештатно

Останавливаем брокер Kafka нештатно:

```
docker-compose kill kafka-1
```

При старте брокера мы должны увидеть сообщение следующего формата:

```
"Recovering ${logsToLoad.length} logs from $logDirAbsolutePath since no clean shutdown file was found")
```

Наличие *clean shutdown файла* (*.kafka_cleanshutdown*) указывает на то, что брокер был отключен штатно. Он используется для того, чтобы сократить время старта в случае корректного завершения работы.

Запускаем брокер и проверяем, что отключение прошло нештатно и брокеру пришлось восстанавливаться:

```
docker-compose logs kafka-1 | grep "no clean shutdown file was found"
```

## Эксперимент 3. Останавливаем ZooKeeper

Создаем топик:

```
./bin/kafka-topics.sh --create --topic test --partitions 1 --bootstrap-server localhost:19092
```

Проверяем, что топик создан:

```
./bin/kafka-topics.sh --describe --topic test --bootstrap-server localhost:19092
```

Останавливаем ZooKeeper:

```
docker-compose stop zookeeper
```

Записываем данные в топик, после чего прерываем выполнение:

```
./bin/kafka-console-producer.sh --topic test --bootstrap-server localhost:19092
```

Читаем данные из топика. Там будет *0 messages*, это мы увидим, прервав выполнение:

```
./bin/kafka-console-consumer.sh --topic test --bootstrap-server localhost:19092 --from-beginning
```

Пытаемся создать новый топик и получаем ошибку *ERROR org.apache.kafka.common.errors.TimeoutException*:

```
./bin/kafka-topics.sh --create --topic test-2 --partitions 1 --bootstrap-server localhost:19092
```

Результаты эксперимента не изменятся, если в кластер будут 2 ноды ZooKeeper или в кластере из 3 нод упадут 2 из них. Проведите этот эксперимент самостоятельно.

Какое влияние на кластер производит отсутствия кворума кластера ZooKeeper? Действия, связанные с администрированием кластера (например, создание темы, редактирование ACL или изменение конфигурации) будут недоступны сразу, но клиенты некоторое время смогут отправлять и получать сообщения. Если кластер ZooKeeper недоступен продолжительное время, то клиенты постепенно начнут испытывать проблемы с отправкой/получением сообщений, особенно если некоторые брокеры выйдут из строя, потому что лидеры разделов не будут распределены между другими брокерами. Сами брокеры, однако, не пострадают от недоступности ZooKeeper. Они будут периодически пытаться повторно подключиться к кластеру ZooKeeper. *Если вы позаботитесь об использовании того же IP-адреса для восстановленного экземпляра ZooKeeper, который был у него до сбоя, брокеры не нужно будет перезапускать.*

## Эксперимент 4. Попытка создания топика при невозможности обеспечить *replication-factor*

Создадим топик:

```
./bin/kafka-topics.sh --create --topic test --partitions 3 --bootstrap-server localhost:19092 --replication-factor 3
```

Проверяем, что топик создан:

```
./bin/kafka-topics.sh --describe --topic test --bootstrap-server localhost:19092
```

Остановим один брокер:

```
docker-compose stop kafka-1
```

Пробуем создать еще один топик, и получим ошибку, связанную со слишком высоким фактором репликации:

```
./bin/kafka-topics.sh --create --topic test-2 --partitions 3 --bootstrap-server localhost:29092 --replication-factor 3
```

Пробуем создать топик снова с фактором репликации 2:

```
./bin/kafka-topics.sh --create --topic test-2 --partitions 3 --bootstrap-server localhost:29092 --replication-factor 2
```

## Эксперимент 5. Попытка записать данные в топик при невозможности обеспечить ответ от *min.insync.replicas*

Выполните эксперимент самостоятельно.

## Эксперимент 6. В кластере Kafka, состоящем из 3 нод, создадим тему с фактором репликации равным 2. Что произойдет с партициями, если остановить один из брокеров-фолловеров? 

Выполните эксперимент самостоятельно. Найдите этому поведению документальное подтверждение.

## Эксперимент 7*. Cоздайте ситуацию, когда несколько брокеров будут считать себя контроллерами (*zombie controller*)

Выполните эксперимент самостоятельно.
