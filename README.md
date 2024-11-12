# Atendimento do Professor

## Terças e quintas. Professor de horário parcial. Nesses 2 dias, podem contar comigo!

# Semana 05 - Comunicação à distância (Atividade Ponderada em sala)

* Comunicação entre o microcontrolador e os blocos externos. Comunicação WiFi com protocolo MQTT no ESP32. 

* Apresentação do Ubidots e suas vantagens

* Modelo TCP/IP: rede, Internet, transporte, aplicação. Controle de um LED por meio da Internet.

## Arquitetura Ubidots

![Arquitetura Ubidots](https://github.com/agodoi/m04-semana05/blob/main/imgs/arquiteturaUbidots.jpg)

## Funcionamento do Ubidots

![Fluxograma MQTT](https://github.com/agodoi/m04-semana05/blob/main/imgs/fluxogramaMqtt.png)

## Exemplos


### Clássico Código Envia Dados ao Broker [Ubidots]

Se não quiser usar o Ubidots, basta substituir algumas linhas pelo HiveMQ.

```
#include "UbidotsEsp32Mqtt.h"

/****************************************
 * Define Constants
 ****************************************/
const char *UBIDOTS_TOKEN = ""; //pegue o token no menu vertical esquerdo de quando vc criou o "black device"
const char *WIFI_SSID = "";      // Put here your Wi-Fi SSID
const char *WIFI_PASS = "";      // Put here your Wi-Fi password
const char *DEVICE_LABEL = "testeGodoi";   // pegue o nome do device que você criou do "blank device"
const char *VARIABLE_LABEL1 = "potenciometro"; // Put here your Variable label to which data  will be published
const char *VARIABLE_LABEL2 = "botao"; // Put here your Variable label to which data  will be published

const int PUBLISH_FREQUENCY = 6000; // Update rate in milliseconds

unsigned long timer;
uint8_t pinPotenciometro = 34;
uint8_t pinBotao = 33;
uint8_t pinLED = 2;
bool statusLED = false;

Ubidots ubidots(UBIDOTS_TOKEN);

/****************************************
 * Auxiliar Functions
 ****************************************/

void callback(char *topic, byte *payload, unsigned int length)
{
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++)
  {
    Serial.print((char)payload[i]);
  }
  Serial.println();
}

/****************************************
 * Main Functions
 ****************************************/

void setup()
{
  // put your setup code here, to run once:
  Serial.begin(115200);
  ubidots.setDebug(true);  // uncomment this to make debug messages available
  ubidots.connectToWifi(WIFI_SSID, WIFI_PASS);
  ubidots.setCallback(callback);
  ubidots.setup();
  ubidots.reconnect();
  pinMode(pinBotao, INPUT_PULLUP);
  pinMode(pinPotenciometro, INPUT);
  pinMode(pinLED, OUTPUT);

  timer = millis();
}

void loop()
{
  // put your main code here, to run repeatedly:
  if (!ubidots.connected())
  {
    ubidots.reconnect();
  }
  if (millis() - timer > PUBLISH_FREQUENCY) // triggers the routine every X seconds
  {
    float value1 = analogRead(pinPotenciometro);
    bool value2 = digitalRead(pinBotao);
    Serial.println(value1);
    Serial.println(value2);
    
    ubidots.add(VARIABLE_LABEL1, value1); // Insert your variable Labels and the value to be sent
    ubidots.publish(DEVICE_LABEL);
    ubidots.add(VARIABLE_LABEL2, value2); // Insert your variable Labels and the value to be sent
    ubidots.publish(DEVICE_LABEL);
    timer = millis();
    //statusLED = !statusLED;
    //digitalWrite(pinLED, statusLED);
    
  }
  ubidots.loop();
}
```

### Clássio Código Recebe Dados do Broker [Ubidots]

## Acendendo um LED do Ubidots

```
#include <WiFi.h>
#include "UbidotsEsp32Mqtt.h"

const uint8_t pinLED = 22;
const char *UBIDOTS_TOKEN = ""; //pegue o token no menu vertical esquerdo do seu ubidots quando criou um dispositivo novo
const char *WIFI_SSID = "";      // lembrando que Apple não é compatível com o ESP32 que trabalha apenas no 2.4GHz
const char *WIFI_PASS = "";      // Put here your Wi-Fi password
const char *DEVICE_LABEL = "testegodoi";   // coloque o nome do device que você criou no "blank device"
const char *VARIABLE_LABEL1 = "botao"; // Replace with your variable label to subscribe to

Ubidots ubidots(UBIDOTS_TOKEN);

void callback(char *topic, byte *payload, unsigned int length)
{
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++)
  {
    Serial.print((char)payload[i]);
  }
  
  String message;
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  }
  int state = message.toInt();
  if (state == 1) {
    Serial.println("\nLED ligado");
  } else {
    Serial.println("\nLED desligado");
  }
  Serial.println("");
 
}


void setup()
{
  // put your setup code here, to run once:
  Serial.begin(115200);
  ubidots.setDebug(true);  // uncomment this to make debug messages available
  ubidots.connectToWifi(WIFI_SSID, WIFI_PASS);
  ubidots.setCallback(callback);
  ubidots.setup();
  ubidots.reconnect();
  ubidots.subscribeLastValue(DEVICE_LABEL, VARIABLE_LABEL1); // Insert the dataSource and Variable's Labels
}

void loop()
{
  // put your main code here, to run repeatedly:
  if (!ubidots.connected())
  {
    ubidots.reconnect();
    ubidots.subscribeLastValue(DEVICE_LABEL, VARIABLE_LABEL1); // Insert the dataSource and Variable's Labels
  }
  ubidots.loop();
}
```

