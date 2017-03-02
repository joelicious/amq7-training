# Networking Brokers

## Prerequisites
 
-   worksheet1
-   A loopback address configured  
    
    route add -net 224.0.0.0 netmask 240.0.0.0 dev lo

## Creating 2 Clustered Brokers

-   create broker 1

```code
(A_MQ_Install_Dir)/bin/artemis create --replicated --user admin --password password --role admin --allow-anonymous y --clustered --host 127.0.0.1 --cluster-user clusterUser --cluster-password clusterPassword --name broker1 --max-hops 1 broker1
```
-   create broker 2

```code
(A_MQ_Install_Dir)/bin/artemis create --replicated --user admin --password password --role admin --allow-anonymous y --clustered --host 127.0.0.1 --cluster-user clusterUser --cluster-password clusterPassword --name broker2 --max-hops 1 --port-offset 100 broker2
```

-   start both brokers
```code
(broker1_home)/bin/artemis run
(broker2_home)/bin/artemis run
```

After some initial negotiation you should see each broker log that a bridge has been created.

## clustering a queue

Both brokers need to be configured with the same anycast queue definition

-   stop both brokers

-   Add an anycast queue configuration for both brokers
```xml
<address name="exampleQueue">
  <anycast>
    <queue name="exampleQueue"/>
  </anycast>
</address>
```
-  Ensure cluster-connection will cluster the queue, by updating the cluster-connection definition. For now remove the address so every address is clustered.
```xml 
<!-- <address>jms</address> -->

-   start both brokers
```code
(broker1_home)/bin/artemis run
(broker2_home)/bin/artemis run
```

-   from the worksheet2 directory run 2 consumers connected to each broker
```code
mvn verify -PqueueReceiver1
mvn verify -PqueueReceiver2                                      
```

-   from the worksheet2 directory run a producer to send 10 messages
```code
mvn verify -PqueueSender
```

NB you will see each consumer receives 5 messages in a round robin fashion

-   kill the consumers and run the producer again.  This example works best with persistence.

```code
mvn verify -PqueueSender
```
-   now run the consumers again
```code
mvn verify -PqueueReceiver1
mvn verify -PqueueReceiver2                                      
```

NB you will see that only 1 consumer received the messages as they were load balanced on demand.

-   now stop the brokers and update the load balancing to strict in the broker.xml file
```xml
<message-load-balancing>STRICT</message-load-balancing>
```

-   now re run the exercise and notice that both the consumers receive messages even when disconnected.

## clustering a topic

Both brokers need to be configured with the same multicast topic definition

-   Add an multicast topic configuration for both brokers
```xml
<address name="exampleTopic">
  <multicast>
    <queue name="exampleTopic"/>
  </multicast>
</address>
```

-   from the worksheet2 directory run 2 consumers connected to each broker
```code
mvn verify -PtopicReceiver1
mvn verify -PtopicReceiver2                                      
```

-   from the worksheet2 directory run a producer to send 10 messages
```code
mvn verify -PtopicSender
```

NB you will see each consumer receives all 10 messages



