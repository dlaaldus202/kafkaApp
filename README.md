## kafka설치
```bash
//설치
brew install kafka

//설치경로(.sh 파일)
/opt/homebrew/Cellar/kafka/3.6.1/libexec/bin

//설치경로(kafka server.properties)
/opt/homebrew/Cellar/kafka/3.6.1/.bottle/etc/kafka/kraft

```
<br/><br/>
<br/><br/>


## kafka실행
```bash
//초기에 브로커에 할당될 UUID를 지정
//임의의 UUID할당 
/opt/homebrew/Cellar/kafka/3.6.1/libexec/bin/kafka-storage.sh random-uuid

//초기에 브로커에 할당될 UUID를 지정
/opt/homebrew/Cellar/kafka/3.6.1/libexec/bin/kafka-storage.sh format -t yGlZ6J9WSa2IT0kT0vC3qA --config /opt/homebrew/Cellar/kafka/3.6.1/.bottle/etc/kafka/kraft/server.properties

//kafka 실행
/opt/homebrew/Cellar/kafka/3.6.1/libexec/bin/kafka-server-start.sh /opt/homebrew/Cellar/kafka/3.6.1/.bottle/etc/kafka/kraft/server.properties
```
<br/><br/>
<br/><br/>


## 파티션 할당 및 콘솔 실행
```bash
//demo_java : 3 개의 파티션 할당
kafka-topics.sh --bootstrap-server localhost:9092 --topic demo_java --create --partitions 3 --replication-factor 1

//consumer 동작 확인을 위한 .sh
//해당 경로로 이동
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic demo_java
```
<br/><br/>
<br/><br/>


## 프로젝트 생성 
```java
1. command + shift + > 
2. Spring Intializr : Create a Maven Project.. 클릭
3. war
4. pom.xml dependency 확인 
```
<br/><br/>
<br/><br/>


## Producer 소스코드 작성
```java
		//어디서 client.id = producer-3 어디서 설정하는건지 by 미연
		log.info("welcome KafkaPorducerApplication!!");

		String bootstrapServers = "127.0.0.1:9092";

    // create Producer properties 설정
    Properties properties = new Properties();
    properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
    properties.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
    properties.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

    // create the producer
    KafkaProducer<String, String> producer = new KafkaProducer<>(properties);

    //key는 선택
		String key = "I'm key!!";
    ProducerRecord<String, String> producerRecord = new ProducerRecord<>("demo_java", key, "hello world");

    // send data - asynchronous
    producer.send(producerRecord);
    // flush data - synchronous
    producer.flush();
    // flush and close producer
    producer.close();

```
<br/><br/>
<br/><br/>


## Consumer 소스코드 작성
```java
  		SpringApplication.run(KafkaConsumerApplication.class, args);
        log.info("I am a Kafka Consumer");

        String bootstrapServers = "127.0.0.1:9092";
        String groupId = "my-fifth-application";
        String topic = "demo_java";

        // create consumer configs
		    // AUTO_OFFSET_RESET_CONFIG Option :  earliest, latest, none
        Properties properties = new Properties();
        properties.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        properties.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.setProperty(ConsumerConfig.GROUP_ID_CONFIG, groupId);
        properties.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

        // create consumer
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);

        // get a reference to the current thread
        final Thread mainThread = Thread.currentThread();
      
       try {
      
                  // subscribe consumer to our topic(s)
                  consumer.subscribe(Arrays.asList(topic));
      
                  // poll for new data
                  while (true) {
                      ConsumerRecords<String, String> records =
                              consumer.poll(Duration.ofMillis(100));
      
                      for (ConsumerRecord<String, String> record : records) {
                          log.info("Key: " + record.key() + ", Value: " + record.value());
                          log.info("Partition: " + record.partition() + ", Offset:" + record.offset());
                      }
                  }
      
              } catch (WakeupException e) {
                  log.info("Wake up exception!");
                  // we ignore this as this is an expected exception when closing a consumer
              } catch (Exception e) {
                  log.error("Unexpected exception", e);
              } finally {
                  consumer.close(); // this will also commit the offsets if need be.
                  log.info("The consumer is now gracefully closed.");
              }
      
      	}


```
<br/><br/>
<br/><br/>



## 레퍼런스
[KRaft mode] : https://www.conduktor.io/kafka/how-to-install-apache-kafka-on-linux-without-zookeeper-kraft-mode/
KRaft mode : 21년전은 zookeeper 와 Kafka 를 별도 설치를 헤야했음, 현재는 KRaft (zookeeper, kafka 포함) 
