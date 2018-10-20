[Go back to the beginning](index.md)
## Digital Input - sending button status.
Let's connect a button to our board - it should short the D7 with GND (G on the PCB);

And then add suitable code to use it. First let's configure the input:

```
void setup() {
  pinMode(13, INPUT_PULLUP);
  Serial.begin(115200);

  setup_wifi();
  client.setServer(MQTT_SERVER, 1883);
  client.setCallback(callback);
}
```
Then let's add a piece of code that will send an event when the button is pressed:
```
if(digitalRead(13) == LOW){
  client.publish("yourName/Input","1");
  delay(300);
}
```
Use the HiveMQ tool to check whether you actually see the events triggered by the button.
## Receving commands via MQTT
Now we will try something different - let's try to control the status of a digital output via MQTT. It could be used eg. to control a status of a lamp, AC or something similar.

First of all - let's set the `LED_BUILTIN` GPIO as digital output:
```
void setup() {
  pinMode(13, INPUT_PULLUP);
  pinMode(LED_BUILTIN, OUTPUT);
  Serial.begin(115200);
  setup_wifi();
  client.setServer(MQTT_SERVER, 1883);
  client.setCallback(callback);
}
```

Then - let's add a subscription to the reconnect function:
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
      client.subscribe("yourName");
      //Added subscription:

      client.subscribe("yourName/Control");


    } else {
      delay(5000);
    }
  }
}
```

And at last let's prepare the logic that will actually control the output:
```
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  if(strcmp(topic, "yourName/Control") == 0){
    if ((char)payload[0] == '1') {
      digitalWrite(LED_BUILTIN, HIGH);
    } else {
      digitalWrite(LED_BUILTIN, LOW);
    }
  }
}
```
Upload the code and try to send `1` or `0` to `yourName/Control` topic and see whether the LED lights up.
You might notice that something is not exactly right - how will you fix it?


*Please save the sketch for further use - we will build on top of it in the next excersises.*

[OneWire temperature and humidity sensor](dht11.md)


In case something went wrong - you should end up with something like this:
```
#include <ESP8266WiFi.h>
#include <PubSubClient.h>

#define WIFI_SSID "WorkshopNet"
#define WIFI_PASSWORD "iotworkshop"

#define MQTT_SERVER "10.64.0.55"
#define SENSOR_NAME "yourName"

WiFiClient espClient;
PubSubClient client(espClient);

void setup() {
  pinMode(13, INPUT_PULLUP);
  pinMode(LED_BUILTIN, OUTPUT);
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
  if(digitalRead(13) == LOW){
    client.publish("yourName/Input","1");
    delay(300);
  }
}
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  if(strcmp(topic, "yourName/Control") == 0){
    if ((char)payload[0] == '1') {
      digitalWrite(LED_BUILTIN, LOW);
    } else {
      digitalWrite(LED_BUILTIN, HIGH);
    }
  }
}
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
      client.subscribe("sensorNode");
      client.subscribe("yourName/Control");

    } else {
      delay(5000);
    }
  }
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
