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


### Clássico Código Envia/RecebeDados ao Broker [Ubidots ou outro qualquer]

Se não quiser usar o Ubidots, basta substituir algumas linhas pelo HiveMQ.

```
#include "UbidotsEsp32Mqtt.h" // Inclui a biblioteca necessária para comunicação com o Ubidots via MQTT usando o ESP32

const char *UBIDOTS_TOKEN = "";  // Define o token de autenticação do Ubidots. Você pega isso em API Credentials da sua conta de grupo do Inteli.
const char *WIFI_SSID = "";      // Define o SSID (nome) da rede Wi-Fi que será usada. Sugestão: use o WiFi do seu celular para evitar o proxy.
const char *WIFI_PASS = "";      // Define a senha da rede Wi-Fi
const char *DEVICE_LABEL = "";   // Define o rótulo do dispositivo no Ubidots ao qual os dados serão publicados
const char *VARIABLE_LABEL1 = ""; // Define o rótulo da variável 1 no Ubidots que receberá os dados
const char *VARIABLE_LABEL2 = ""; // Define o rótulo da variável 2 no Ubidots que receberá os dados
//const char *VARIABLE_LABELX = ""; // Define o rótulo da variável X no Ubidots que receberá os dados
//No plano assinado pelo Inteli, o Ubidots suporta até 20 variáveis a cada segundo

const int PUBLISH_FREQUENCY = 5000; // Define a frequência de publicação dos dados em milissegundos (neste caso, a cada 5 segundos)

unsigned long timer; // Variável para armazenar o tempo atual
uint8_t analogPin = 34; // Define o pino analógico que será usado para leitura de dados (GPIO34)

Ubidots ubidots(UBIDOTS_TOKEN); // Instancia o objeto Ubidots com o token de autenticação fornecido

// Função de callback para lidar com mensagens recebidas do Ubidots via MQTT
void callback(char *topic, byte *payload, unsigned int length)
{
  Serial.print("Mensagem recebida ["); // Exibe uma mensagem de início no monitor serial
  Serial.print(topic); // Exibe o tópico recebido
  Serial.print("] ");
  for (int i = 0; i < length; i++) // Itera sobre o payload recebido
  {
    Serial.print((char)payload[i]); // Converte cada byte para caractere e exibe no monitor serial
  }
  Serial.println(); // Quebra de linha no monitor serial
  //aqui você inseri seu código para fazer o que deseja com o dado recebido do Ubidots
}

void setup()
{
  Serial.begin(115200); // Inicializa a comunicação serial com uma taxa de 115200 bauds
  // ubidots.setDebug(true);  // Descomente esta linha para exibir mensagens de depuração (debug)
  ubidots.connectToWifi(WIFI_SSID, WIFI_PASS); // Conecta o ESP32 à rede Wi-Fi usando o SSID e a senha fornecidos
  ubidots.setCallback(callback); // Define a função de callback para lidar com mensagens recebidas
  ubidots.setup(); // Configura a conexão MQTT com o Ubidots
  ubidots.reconnect(); // Reconecta ao servidor MQTT do Ubidots, caso necessário

  timer = millis(); // Armazena o tempo atual em milissegundos
}

void loop()
{
  if (!ubidots.connected()) // Verifica se o ESP32 está conectado ao Ubidots
  {
    ubidots.reconnect(); // Se não estiver conectado, tenta reconectar
  }
  if (abs(millis() - timer) > PUBLISH_FREQUENCY) // Verifica se o intervalo de tempo definido para publicação foi alcançado
  {
    float value1 = analogRead(analogPin); // Lê o valor do pino analógico definido (GPIO34)
    ubidots.add(VARIABLE_LABEL1, value1); // Adiciona o valor lido à variável que será enviada ao Ubidots
    ubidots.publish(DEVICE_LABEL); // Publica os dados para o dispositivo especificado no Ubidots
    ubidots.add(VARIABLE_LABEL2, value2); // Adiciona o segundo que será enviada ao Ubidots
    ubidots.publish(DEVICE_LABEL); // repete a linha que publica os dados para o dispositivo especificado no Ubidots
    timer = millis(); // Atualiza o temporizador para o tempo atual
  }
  ubidots.loop(); // Mantém a conexão e comunicação com o servidor Ubidots
}
```

### Código Controle de LED

## Acendendo um LED do Ubidots

```
#include "UbidotsEsp32Mqtt.h"

const char *UBIDOTS_TOKEN = "BBFF-...";
const char *WIFI_SSID = "";      // Put here your Wi-Fi SSID
const char *WIFI_PASS = "";      // Put here your Wi-Fi password
const char *DEVICE_LABEL = "";   // Replace with the device label to subscribe to
const char *VARIABLE_LABEL = ""; // Replace with your variable label to subscribe to


const uint8_t LED = 2; // Pin used to write data based on 1's and 0's coming from Ubidots
const int PUBLISH_FREQUENCY = 1000; // Update rate in milliseconds
unsigned long timer;
Ubidots ubidots(UBIDOTS_TOKEN);

void callback(char *topic, byte *payload, unsigned int length)
{
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++)
  {
    Serial.print((char)payload[i]);
    if ((char)payload[0] == '1')
    {
      digitalWrite(LED, HIGH);
    }
    else
    {
      digitalWrite(LED, LOW);
    }
  }
  Serial.println();
}

void setup()
{
  Serial.begin(115200);
  pinMode(LED, OUTPUT);

  ubidots.setDebug(true);  // uncomment this to make debug messages available
  ubidots.connectToWifi(WIFI_SSID, WIFI_PASS);
  ubidots.setCallback(callback);
  ubidots.setup();
  ubidots.reconnect();
  ubidots.subscribeLastValue(DEVICE_LABEL, VARIABLE_LABEL); 
}


void loop()
{

  if(millis() - timer > PUBLISH_FREQUENCY){
    ubidots.add(VARIABLE_LABEL, 0); // Insira variável e dispositivo a serem publicados
    ubidots.publish(DEVICE_LABEL);
    timer = millis();
  }
  if (!ubidots.connected())
  {
    ubidots.reconnect();
    ubidots.subscribeLastValue(DEVICE_LABEL, VARIABLE_LABEL); 
  }
  ubidots.loop();
}
```

