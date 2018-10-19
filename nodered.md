## Basics of Node-RED

## Getting data from MQTT and displaying it in the debug pane

Start by dragging an MQTT input block into your flow:
![ip addr](images/mqttIn.png)

Double click it to open configuration pane:
![ip addr](images/confPanel.png)
Insert the topic name `yourName/Temperature` and click the edit button next to the server settings and configure the broker:
![ip addr](images/brokerConf.png)

Next - drag a debug block into your flow and configure it to display a complete msg object:
![ip addr](images/msgObj.png)

Connect these two and click deploy in right upper corner. After clicking into the bug icon right below deploy you should be able to see flowing mqtt messages there.
![ip addr](images/messagesDebug.png)



## Sending data to your device

## Simple data filtering

## Creating triggers in Node-RED
