# PRACTICA - NODE RED

## Introducción
En este ejercicio aprenderemos como usar un servidor en linea conectado entre el NODE RED y WOKWI

## Materiales
- Simulador NODE RED
- Simulador WOKWI
- DHT22
- Ultrasonico
  
## Procedimiento
### INSTALACIÓN NODE - RED

1. Entrar a la pagina https://nodejs.org/en y descargar el archivo

![](https://github.com/ximena01ta/practica-node-red/blob/main/Captura%20de%20pantalla%202025-12-07%20222309.png)

3. Abrir el archivo e instalar el programa node.js
4. Abrir terminal en modo administrador y escribir lo siguente:
``npm install -g --unsafe-perm node-red``
5. Despues comprobamos que funcione node-red con el siguente codigo: (con este mismo codigo podemos arrancar el programa siempre que lo necesitemos)
``node-red``
6. Para arrancar el Programa Node-red se usa el codigo :
``node-red``
7. Para abrir la aplicación nos vamos al buscador, colocamos ``Node-Red`` nos habilitara el servidor
8. En el navegador anotamos la siguiente direccion: 127.0.0.1:1880

### EJERCICIO

1. Diseñar nuestro ejercicio en NODE RED:

![](https://github.com/ximena01ta/practica-node-red/blob/main/Captura%20de%20pantalla%202025-12-07%20213417.png) 

2. En WOKWI armar de la siguiente manera

![](https://github.com/ximena01ta/practica-node-red/blob/main/Captura%20de%20pantalla%202025-12-07%20233151.png)

3. Escribir el soguiente codigo: 
````
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
const int DHT_PIN = 15;
const int Trigger = 4;   //Pin digital 2 para el Trigger del sensor
const int Echo = 2; 
DHTesp dhtSensor;
long duration;
int distance;
int safetyDistance;
// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "3.121.19.141";
String username_mqtt="educatronicosiot";
String password_mqtt="12345678";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

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

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
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
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
  Serial.begin(9600);//iniciailzamos la comunicación
  pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0
}

void loop() {

// Reads the echoPin, returns the sound wave travel time in microseconds


// Calculating the distance
distance= duration*0.034/2;

safetyDistance = distance;
long t; //timepo que demora en llegar el eco
  long d; //distancia en centimetros

  digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(Trigger, LOW);
  
  t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm
  
  Serial.print("Distancia: ");
  Serial.print(d);      //Enviamos serialmente el valor de la distancia
  Serial.print("cm");
  Serial.println();
  delay(2000);     

delay(1000);
TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "XIMENA";
    //doc["Anho"] = 2025;
    //doc["Empresa"] = "Educatronicos";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
   doc["DISTANCIA"] = String(d);

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("XIMENA", output.c_str());
  }
}
````
## Resultados

![](https://github.com/ximena01ta/practica-node-red/blob/main/Captura%20de%20pantalla%202025-12-07%20233243.png)
![](https://github.com/ximena01ta/practica-node-red/blob/main/Captura%20de%20pantalla%202025-12-07%20213417.png)
