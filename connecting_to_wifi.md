## Connecting to connecting_to_wifi
Now we’ll connect to the WiFi Network - using a simple function setup_wifi() that we’ll write somewhere in our code and call it in setup();
```
#include <ESP8266WiFi.h>

#define WIFI_SSID "WorkshopNet"
#define WIFI_PASSWORD "iotworkshop"

void setup() {              
  Serial.begin(115200);   
  setup_wifi();
}

void loop() {

}

void setup_wifi() {

  delay(10);

  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(WIFI_SSID);

  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

```
It just uses the WiFi library from the core package, calls the .begin() method with SSID and PASSWORD as statically defined on top and then waits until we successfully connect to the network.

Write the function, upload the sketch and check the output in the serial manager. If the IP address appears there, you’re good to go further.
![ip addr](images/ipaddr.png)


## MQTT connection from ESP8266
Now we’ll connect to the MQTT broker from ESP8266. To do that we’ll need to a PubSubClient library to our sketch and then write some functions.
To install the library you need to find it via the Library Manager. It’s available via: Sketch -> Add Library -> Manage Libraries menu. Just search for “pubsubclient” in the search box and install it.
![ip addr](images/pubsubclient.png)
### Headers and defines
Now let’s write functions that we’ll need. First of all let’s add needed headers and defines:
```
#include <ESP8266WiFi.h>
#include <PubSubClient.h>

#define WIFI_SSID "WorkshopNet"
#define WIFI_PASSWORD "iotworkshop"

#define MQTT_SERVER "10.64.0.55"
#define SENSOR_NAME "yourName"

WiFiClient espClient;
PubSubClient client(espClient);
```
Replace `"yourName"` with your name and first letter of your surname - so everyone uses one of his own.

### reconnect() - maintaining connectivity with the broker
Then, write a  reconnect()  functionthat’s going to maintain connectivity with the broker:
```
void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str())) {
      Serial.println("connected");
      client.publish(SENSOR_NAME, "hello world");
      client.subscribe(SENSOR_NAME);
    } else {
      delay(5000);
    }
  }
}
```
### callback() - triggered when some message arrives at the device
Next, write a simple callback function that is going to be triggered every time that some message arrives in our device.
It just prints out the topic and payload of the message that arrived.

```
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();
}
```


### setup() and loop()
At last let’s update our setup and loop functions:
```
void setup() {
  Serial.begin(115200);
  setup_wifi();
  client.setServer(MQTT_SERVER, 1883);
  client.setCallback(callback);
}
void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
}
```

In the setup we perform client initialization and setup -> we set the MQTT server and port (default 1883) and attach a callback function.
In the loop if the client is not connected we call the reconnect() function to get the connection back.
Then, as often as possible, we call client.loop() so the incoming messages get parsed correctly.

Upload the sketch and check whether it is parsed correctly. Ask the instructor to display incoming messages at the broker so you’re sure that your sketch worked correctly.
![ip addr](images/mqttConnected.png)

## Sending and receving messages with MQTT from web
We can use a web tool to monitor data that we pass to broker via additional websockets listener in the broker.
Go to the tool here:
```
http://www.hivemq.com/demos/websocket-client/
```
And configure it as follows: the client id should remain a random value
![ip addr](images/hivemq.png)
