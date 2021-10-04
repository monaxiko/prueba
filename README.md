# **videoMQTT**
[![N|Solid](https://5g-ppp.eu/wp-content/uploads/2021/01/Logo-5G-META.png)](https://5g-ppp.eu/5gmeta/)
## _MQTT signalling and video delivery_


The aim of this repository is to create an complete environment where five entities take part in (MQTT Broker, WebRTC Server, IDManager, VideoTX and VideoRX ).

The repository structure has a structure defined by:
- [id_manager](https://google.es) folder contains the application which listen MQTT messages with token _/createID_ at port 1883, connect to _mysql_ database and get the next available id to connect with WebRTC Server.
- [mqtt_broker]() folder hosts the resources to deploy an MQTT Broker based on eclipse-mosquitto using Docker.
- [mysql_data]() includes a complete file system of a mysql database, but the most important is the folder [IDDelivery] which contains the information and the structure of the table to import into a central database.
- Finally, [simulated_devices]() include one script with the task for the video transmitter (VideoTX). 

```mermaid
sequenceDiagram
Demon->MQTT Broker: SUBSCRIBE [topic:createID]
MQTT Broker->Demon: SUBACK [topic:createID]

WebRTC Server -> WebRTC Server: deviceIP: A.A.A.A
WebRTC Server->MQTT Broker: SUBSCRIBE [topic: createID & topic: createIDresponse & topic: newdataflow & topic:sendMSG | deviceIP: A.A.A.A]
MQTT Broker->WebRTC Server: SUBACK [topic: createID & topic: createIDresponse & topic: newdataflow & topic:sendMSG]

TX->TX:deviceIP:B.B.B.B
TX->MQTT Broker: SUBSCRIBE [topic: createID & topic: createIDresponse & topic:newdataflow & topic:sendMSG | deviceIP: B.B.B.B]
MQTT Broker->TX: SUBACK [topic: createID & topic: createIDresponse & topic: newdataflow & topic:sendMSG]

TX->MQTT Broker: PUBLISH [topic: createID | deviceIP: B.B.B.B]
MQTT Broker->Demon: PUBLISH [topic: createID | deviceIP: B.B.B.B]
Demon -> Demon: DB request --> ID \n deviceIP: B.B.B.B \n ID: YYYY
Demon->MQTT Broker: PUBLISH [topic:createIDresponse  | deviceIP: B.B.B.B | ID: YYYY]
MQTT Broker -->WebRTC Server: PUBLISH [topic:createIDresponse  | deviceIP: B.B.B.B | ID: YYYY]
MQTT Broker ->TX: PUBLISH [topic:createIDresponse  | deviceIP: B.B.B.B | ID: YYYY]
note right of TX: ID: YYYY

TX->MQTT Broker: PUBLISH [topic:newdataflow | datatype: video | ID: YYYY]
MQTT Broker->WebRTC Server: PUBLISH [topic:newdataflow | datatype: video | ID: YYYY]

WebRTC Server -> WebRTC Server: newRequest() \n TX:YYYY

WebRTC Server ->MQTT Broker: PUBLISH [topic: createID | deviceIP: A.A.A.A | TX: YYYY]
MQTT Broker -> Demon: PUBLISH [topic: createID | deviceIP: A.A.A.A | TX: YYYY]
Demon -> Demon: DB request --> ID \n deviceIP: A.A.A.A \n ID: XXXX \n TX: YYYY
Demon->MQTT Broker: PUBLISH [topic:createIDresponse  | deviceIP: A.A.A.A | ID: XXXX | TX: YYYY]
MQTT Broker -->TX: PUBLISH [topic:createIDresponse  | deviceIP: A.A.A.A | ID: XXXX | TX: YYYY]
MQTT Broker ->WebRTC Server: PUBLISH [topic:createIDresponse  | deviceIP: A.A.A.A | ID: XXXX | TX: YYYY]
note right of WebRTC Server : ID: XXXX

WebRTC Server -> RX: LAUNCH RX \n ID: XXXX

WebRTC Server ->MQTT Broker: PUBLISH [topic: sendMSG | TX: YYYY | remoteID: XXXX]
MQTT Broker ->TX: PUBLISH [topic: sendMSG | TX: YYYY | remoteID: XXXX]

TX->RX: VIDEO [srcID: YYYY | remoteID: XXXX]

```

## Authors
* José Manuel Martínez (jmmartinez@vicomtech.org)



