##  AMQP&JMS, Some imp things you should know
1. Keep in mind that AMQP is a Messaging technologies that do not implement the JMS API.
2. JMS is API and AMQP is a protocol.So it doesn't make sense to say that what is default protocol of JMS , Ofcourse client applications use HTTP/S as the connection protocol when invoking a WebLogic Web Service.
3. JMS is only a API spec. It doesnt use any protocol. A JMS provider (like ActiveMQ) could be using any underlying protocol to realize the JMS API. For ex: Apache ActiveMQ can use any of the following protocols: AMQP, MQTT, OpenWire, REST(HTTP), RSS and Atom, Stomp, WSIF, WS Notification, XMPP


JMS, when it was defined did not define a protocol between the JMS client and a messaging server. The JMS client, which implement the JMS API can use whatever protocol to communicate with messaging server. The client just need to be compliant with JMS api. Thats all. Ususally JMS clients use a custom protocol that their messaging server understands.  
AMQP on other hand is a protocol between a messaging client and messaging server. A JMS client can use AMQP as the protocol to communicate with the messaging server. And there are clients like that available.

From:  
<http://stackoverflow.com/questions/15150133/jms-and-amqp-rabbitmq>
