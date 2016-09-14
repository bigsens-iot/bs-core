# Service Gateway
Implementation of message pattern for microservices communication. Based on pure TCP sockets.

<p align="center">
  <img src="/resources/images/message-pattern.png">
</p>

## Microservices principles

The entire platform is based on microservice mechanism. Services can be developed on any programming language, keep a local storage and/or connecting to the remote services e.g. IFTTT,  voice recognition services, etc. Host interaction between services works on pure TCP sockets and messaging pattern. All services are deployed to the host machine e.g. Raspberry Pi, BeagleBone and others.

Service address format is `remote_address : remote_port`. After connection the `Root Service` assigns the port to a new service and puts new record to the routing table. Below is the example of the routing table and a few possible services...

| # | Service address | Endpoint name    | Message registry      |
|---|-----------------|------------------|-----------------------|
| 1 | 0.0.0.0:1000    | [root](https://github.com/bigsens-iot/bigsens-service-gateway/tree/master/lib/services/root-service) | `[ msg1, msg2, ... ]` |
| 2 | 0.0.0.0:1001    | [zigbee.cc2530/31](https://github.com/bigsens-iot/zigbee-service) | `[ msg1, msg2, ... ]` |
| 3 | 0.0.0.0:1002    | belkin.wemo      | `[ msg1, msg2, ... ]` |
| 4 | 0.0.0.0:1003    | lg.smarttv       | `[ msg1, msg2, ... ]` |
| 5 | 0.0.0.0:1004    | [proxy.ws](https://github.com/bigsens-iot/bigsens-service-gateway/tree/master/lib/services/websocket-proxy) | `[ msg1, msg2, ... ]` |
| 6 | 0.0.0.0:1005    | goog.voicectrl   | `[ msg1, msg2, ... ]` |
| 7 | 0.0.0.0:1006    | usb.3gmod        | `[ msg1, msg2, ... ]` |
| 8 | 0.0.0.0:1007    | ui.base          | `[ msg1, msg2, ... ]` |
| . | ...             | ...              | ...                   |

Service identification based on the `Universally Unique IDentifier (UUID)` [RFC4122](https://tools.ietf.org/html/rfc4122) standard. Every service contains mandatory metadata like `UUID` and `Service Name`. Identification goes during connection between service and `Root Service`, it's called the service announcement with the message `SERVICE_ANNCE`. The `SERVICE_ANNCE` is a broadcast message and others services will be notified about announcement.

## Message model

All messages can be sent as broadcast, multicast or unicast. In the current implementation are two types of messages.

### Event message
Several services would like to use event-notification to coordinate their actions, and would like to use messaging to communicate those events. This type of message does not generate any response from receiver. When a sender service has an event to announce, it will create an event object, wrap it in a message, and send it on a channel. The receiver service will receive the `event message`, get the event, and process it. Messaging does not change the event notification, just makes sure that the notification gets to the receiver. The `event message` can be used for events like announcement or state changing.

### Command message
An service needs to invoke functionality provided by other service. This type of message is generating a response from receiver. There is no specific message type for commands. A `command message` is simply a regular message that happens to contain a command. The `command message` is a text message containing the command arguments in `json` format, a `command message` is a message with a command stored in it. There are two sub-types of command messages.
* `asyncast` - message with asynchronous response
* `syncast` - message with synchronous response
