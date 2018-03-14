
# Kafka

### 1.Installation

##### 1. Télécharger le code
```bash
> tar-xzf kafka_2.11-1.0.1.tgz

> cd kafka_2.11-1.0.1
```
##### 2. Démarrer le serveur
Ouvrir à chaque fois un nouveau bash pour chaque application (garder Zookeper, Kafka et topic ouverts dans des fenêtres différentes).

*Zookeeper:*
```bash
> bin/zookeeper-server-start.sh config/zookeeper.properties
```
*Kafka server:*
```bash
> bin/kafka-server-start.sh config/server.properties
```

##### 3. Créer un topic
```bash
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
> bin/kafka-topics.sh --list --zookeeper localhost:2181
test
```
 ##### 4. Envoyer un message: créer un producer
Pour se connecter depuis AWS, les consumers et le producer doivent utiliser le même zookerper: nécessaire de changer le 
```localhost```  avec la même adresse ip public.
```bash
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic `test`
This is a message
This is another message
```
##### 5.  Lire un message: créer un consumer
```bash
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
This is a message
This is another message
```
##### 6. Setter un cluster multi-brocker
Arrêter les services Ambari pour ne pas avoir de problèmes d'interaction. 
Créer un config file pour chaque broker: pour chaque broker changer ```server-1``` en ```server-2```etc..:
```bash
> cp config/server.properties config/server-1.properties
```

* Modifier dans le config file le ```broker.id```, le ```listener``` et le ```log.dir``` . Dans le cas du cluster AWS, chaque nœud modifie le ```broker.id``` et le ```zookeeper.connect```:
```bash
> config/server-1.properties:
>    broker.id=1
>    listeners=PLAINTEXT://:9093
>    log.dir=/tmp/kafka-logs-1
```
* Ou modifier directement dans le fichier en passant par nano:
```bash
sudo nano server-1.properties
```


Dans une nouvelle fenêtre du bash: lancer le broker:
```bash
> bin/kafka-server-start.sh config/server-1.properties
```
Créer un topic avec un facteur de réplication égal au nombre de brokers créés:
```bash
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic
```
On peut ensuite voir la description du nouveau topic:
```bash
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic: my-replicated-topic PartitionCount:1 ReplicationFactor:3 Configs:
	Topic: my-replicated-topic Partition: 0 Leader: 1 Replicas: 1,2,0 Isr: 1,2,0
```

Créer un groupe avec plusieurs utilisateurs: dans le fichier ```consumer.properties```, changer les champs ```zookeeper.connect``` et  ```group.id```. Mettre l'IP du serveur zookeper et un nom identique pour le groupe. Créer un fichier identique (changer le nom) pour chaque élément du groupe (```consumer.properties```, ```consumer-1.properties``` etc..).

```bash
sudo bin/kafka-console-consumer.sh --bootstrap-server 34.249.95.103:9092 --topic cars-topic --from-beginning --consumer.config config/consumer.properties
```

#### Tuer un broker et le relancer

```bash
bin/kafka-topics.sh --describe --zookeeper 00.000.00.000:2181 --topic cars-topic
Topic:cars-topic        PartitionCount:5        ReplicationFactor:3     Configs:
        Topic: cars-topic       Partition: 0    Leader: 2       Replicas: 2,0,1 Isr: 2,0,1
        Topic: cars-topic       Partition: 1    Leader: 3       Replicas: 3,1,2 Isr: 3,1,2
        Topic: cars-topic       Partition: 2    Leader: 0       Replicas: 0,2,3 Isr: 0,2,3
        Topic: cars-topic       Partition: 3    Leader: 1       Replicas: 1,3,0 Isr: 1,3,0
        Topic: cars-topic       Partition: 4    Leader: 2       Replicas: 2,1,3 Isr: 2,1,3
```
On tue le serveur 2:
```bash
> sudo ps aux | grep server-1.properties
ubuntu     515  0.0  0.0  21744  3720 pts/2    T    12:12   0:00 nano server-1.properties
root      3717  0.0  0.0  55740  4020 pts/2    S+   13:51   0:00 sudo bin/kafka-server-start.sh config/server-1.properties
root      3718  1.0  5.7 4836488 942764 pts/2  Sl+  13:51   1:13 java -Xmx1G -Xms1G -server -XX:+UseG1GC 

> kill -9 515
> sudo kill -9 3717
> sudo kill -9 3718
 ```

```bash
> bin/kafka-topics.sh --describe --zookeeper 00.000.00.000:2181 --topic cars-topic
Topic:cars-topic        PartitionCount:5        ReplicationFactor:3     Configs:
        Topic: cars-topic       Partition: 0    Leader: 0       Replicas: 2,0,1 Isr: 0,1,2
        Topic: cars-topic       Partition: 1    Leader: 3       Replicas: 3,1,2 Isr: 3,1,2
        Topic: cars-topic       Partition: 2    Leader: 0       Replicas: 0,2,3 Isr: 0,3,2
        Topic: cars-topic       Partition: 3    Leader: 1       Replicas: 1,3,0 Isr: 1,3,0
        Topic: cars-topic       Partition: 4    Leader: 1       Replicas: 2,1,3 Isr: 1,3,2
```
La partition 2 a été tuée.  Le broker 1 reprend le leader du broker mort.
Si tous les brokers sauf 1 sont tués, le broker restant devient leader seulement sur les followers qu'il avait en écriture. Les partitions sans leader deviennent inaccessibles pour le producer/consumer mais les données ne sont pas perdues.   

#### Supprimer un topic
Pour supprimer un topic, il se retrouve "marked for deletion": lors de la création d'un nouveau topic, celui marqué sera écrasé à ce moment.
```bash
> sudo bin/kafka-topics.sh --delete --zookeeper 00.000.00.000:2181 --topic yolo
> sudo bin/kafka-topics.sh --list --zookeeper 00.000.00.000:2181
__consumer_offsets
karine
test
tp_connect
yolo - marked for deletion
 ```
 







