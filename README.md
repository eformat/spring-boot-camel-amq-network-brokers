# AMQ 7 - artemis testing

```
oc -n openshift replace --force  -f \
https://raw.githubusercontent.com/jboss-container-images/jboss-amq-7-broker-openshift-image/73-7.3.0.GA/amq-broker-7-image-streams.yaml

oc -n openshift import-image amq-broker-7/amq-broker-73-openshift --from=registry.redhat.io/amq-broker-7/amq-broker-73-openshift --confirm

for template in amq-broker-73-basic.yaml \
amq-broker-73-ssl.yaml \
amq-broker-73-custom.yaml \
amq-broker-73-persistence.yaml \
amq-broker-73-persistence-ssl.yaml \
amq-broker-73-persistence-clustered.yaml \
amq-broker-73-persistence-clustered-ssl.yaml;
 do
 oc -n openshift replace --force -f \
https://raw.githubusercontent.com/jboss-container-images/jboss-amq-7-broker-openshift-image/73-7.3.0.GA/templates/${template}
 done

oc new-project amq-test

oc new-app amq-broker-73-basic -p AMQ_PASSWORD=password -p AMQ_USER=admin

oc port-forward $(oc get pods -lapplication=broker -o name) 61616

--
-- Creates a multicast address (or topic) called jms.topic.incomingOrders
--

oc set env dc/broker-amq AMQ_ADDRESSES=jms.topic.incomingOrders
oc set env dc/broker-amq AMQ_ADDRESSES-

-- Producers
            <to id="route-to-incoming-orders" uri="amq:topic:incomingOrders?exchangePattern=InOnly"/>

-- Consumer(s)
            <from id="route-from-incoming-orders" uri="amq:topic:incomingOrders::Consumer.{{camel.springboot.name}}.incomingOrders"/>

-- Can start mutiple Consumers with different names (and they each get a copy)
mvn spring-boot:run -DskipTests
mvn spring-boot:run -DskipTests -Dserver.port=9081 -Dmanagement.port=9082 -Dcamel.springboot.name=foobar

--
-- Creates an anycast queue called jms.queue.incomingOrders (can be comma separated)
--
oc set env dc/broker-amq AMQ_QUEUES=jms.queue.incomingOrders
oc set env dc/broker-amq AMQ_QUEUES-

-- Producer
            <to id="route-to-incoming-orders" uri="amq:queue:incomingOrders?exchangePattern=InOnly"/>

-- Consumer(s)
            <!-- from id="route-from-incoming-orders" uri="amq:queue:incomingOrders"/-->
            <from id="route-from-incoming-orders" uri="amq:queue:incomingOrders::Consumer.{{camel.springboot.name}}.incomingOrders"/>	    

-- Can start mutiple Consumers (and they load balance across consumers groups (by name))

-- camel.springboot.name=consumer
mvn spring-boot:run -DskipTests
mvn spring-boot:run -DskipTests -Dserver.port=9081 -Dmanagement.port=9082

-- camel.springboot.name=abc
mvn spring-boot:run -DskipTests -Dserver.port=9091 -Dmanagement.port=9092 -Dcamel.springboot.name=abc
mvn spring-boot:run -DskipTests -Dserver.port=9093 -Dmanagement.port=9094 -Dcamel.springboot.name=abc


-- Can start mutiple Consumers (and they each get a copy)
mvn spring-boot:run -DskipTests
mvn spring-boot:run -DskipTests -Dserver.port=9081 -Dmanagement.port=9082 -Dcamel.springboot.name=foobar


-- Producer (request reply)
            <to id="route-to-incoming-orders" uri="amq:queue:incomingOrders?exchangePattern=InOut"/>

```

