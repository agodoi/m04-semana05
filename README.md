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


### (1) Clássico Código Envia Dados ao Broker [Ubidots]

#### Esse código está atualizado e funcionando. Alterem as variáveis com const char para as suas credenciais.

```
#include "UbidotsEsp32Mqtt.h"

/****************************************
 * Define Constants
 ****************************************/

const char *WIFI_SSID = ""; // Put here your Wi-Fi SSID
const char *WIFI_PASS = ""; // Put here your Wi-Fi password

const char *UBIDOTS_TOKEN = "PEGAR O TOKEN QUE CHEGOU NO SLACK PARA O SEU GRUPO"; //PROCURE PELO SEU TOKEN COM O SEU PROFESSOR DE PROG.
const char *DEVICE_LABEL = "esp32_t12_godoi"; // Coloque o nome do dispositivo que você deseja apelidar
const char *VARIABLE_LABEL1 = "potenciometro"; // Coloque o nome da variável que deseja publicar
const char *VARIABLE_LABEL2 = "botao"; // Outra variável que será publicada
const char *CLIENT_ID = "godoi"; //PULO DO GATO! Coloque um nome qualquer para te identificar no Ubidots de 5 a 20 caracteres e que seja diferente dos demais integrantes
//esse cliente_id é apenas para não sobrepor os tópicos dentro da conta do Ubidots. Ele é importante aqui no código, mas não vai aparecer na sua dashboard do Ubidots.



Ubidots ubidots(UBIDOTS_TOKEN, CLIENT_ID); //essa linha é novidade

const int PUBLISH_FREQUENCY = 3000; // Update rate in milliseconds
unsigned long timer;
uint8_t pinPotenciometro = 34;
uint8_t pinBotao = 33;
uint8_t pinLED = 2;
bool statusLED = false;

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
    ubidots.add(VARIABLE_LABEL2, value2); // Insert your variable Labels and the value to be sent
    ubidots.publish(DEVICE_LABEL); //pode usar um único publish para várias variáveis
    timer = millis();
  }
  ubidots.loop();
}
```

### (2) Clássio Código Recebe Dados do Broker [Ubidots]

## Acendendo um LED do Ubidots

```
#include <WiFi.h>
#include "UbidotsEsp32Mqtt.h"

const uint8_t pinLED = 22;
const char *UBIDOTS_TOKEN = "PEGAR O TOKEN QUE CHEGOU NO SLACK PARA O SEU GRUPO"; //PROCURE PELO SEU TOKEN COM O SEU PROFESSOR DE PROG.
const char *WIFI_SSID = "";      // lembrando que Apple não é compatível com o ESP32 que trabalha apenas no 2.4GHz
const char *WIFI_PASS = "";      // Put here your Wi-Fi password
const char *DEVICE_LABEL = "testegodoi";   // coloque o nome do device que você criou no "blank device"
const char *VARIABLE_LABEL3 = "botao_no_ubidots"; // Replace with your variable label to subscribe to
const char *CLIENT_ID = "godoi"; //PULO DO GATO! Coloque um nome qualquer que seja diferente dos demais integrantes

Ubidots ubidots(UBIDOTS_TOKEN, CLIENT_ID); //essa linha é novidade

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
  ubidots.subscribeLastValue(DEVICE_LABEL, VARIABLE_LABEL3); // Insert the dataSource and Variable's Labels
}

void loop()
{
  // put your main code here, to run repeatedly:
  if (!ubidots.connected())
  {
    ubidots.reconnect();
    ubidots.subscribeLastValue(DEVICE_LABEL, VARIABLE_LABEL3); // Insert the dataSource and Variable's Labels
  }
  ubidots.loop();
}
```

### (3) Juntando Envia e Recebe Dados [Ubidots]

```
#include "UbidotsEsp32Mqtt.h"

/****************************************
 * Define Constants
 ****************************************/
const char *UBIDOTS_TOKEN = " ";
//const char *WIFI_SSID = " "; // Put here your Wi-Fi SSID
//const char *WIFI_PASS = " "; // Put here your Wi-Fi password

const char *DEVICE_LABEL = "esp32_t12_godoi"; // Put here your Device label to which data  will be published
const char *VARIABLE_LABEL1 = "potenciometro_no_proto"; // Put here your Variable label to which data  will be published
const char *VARIABLE_LABEL2 = "botao_no_proto"; // Put here your Variable label to which data  will be published
const char *VARIABLE_LABEL3 = "botao_no_ubidots"; // Put here your Variable label to which data  will be published
const char *CLIENT_ID = "godoi"; //PULO DO GATO! Coloque um nome qualquer que seja diferente dos demais integrantes

const int PUBLISH_FREQUENCY = 5000; // Update rate in milliseconds
const int SUBSCRIBER_FREQUENCY = 2000; // Update rate in milliseconds

unsigned long timer1, timer2;
uint8_t pinPotenciometro = 34;
uint8_t pinBotao = 33;
uint8_t pinLED = 2;
bool statusLED = false;

Ubidots ubidots(UBIDOTS_TOKEN, CLIENT_ID);

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

  timer1 = millis();
  timer2 = millis();
}

void loop()
{
  // put your main code here, to run repeatedly:
  if (!ubidots.connected())
  {
    ubidots.reconnect();
  }
  if (millis() - timer1 > PUBLISH_FREQUENCY) // triggers the routine every X seconds
  {
    float value1 = analogRead(pinPotenciometro);
    bool value2 = digitalRead(pinBotao);
    Serial.print("Valor do pot: ");
    Serial.println(value1);
    ubidots.add(VARIABLE_LABEL1, value1); // Insert your variable Labels and the value to be sent
    ubidots.add(VARIABLE_LABEL2, value2); // Insert your variable Labels and the value to be sent
    ubidots.publish(DEVICE_LABEL); //pode usar um único publish para várias variáveis
    timer1 = millis();
  }


  if (millis() - timer2 > SUBSCRIBER_FREQUENCY) // triggers the routine every X seconds
  {
    ubidots.subscribeLastValue(DEVICE_LABEL, VARIABLE_LABEL3); // Insert the dataSource and Variable's Labels
    timer2 = millis();
  }
  ubidots.loop();
}
```

# Teoria Completa

# Comunicação à Distância

Nesse encontro, veremos como a comunicação entre o microcontrolador e os blocos externos acontece, como por exemplo, como o ESP32 utiliza a comunicação WiFi com o protocolo MQTT passando pelas camadas de Rede, Internet, Transporte e Aplicação. A parte prática será o controle de um LED por meio da Internet e Ubidots.

## Arquitetura do projeto usando Ubidots

Confira na figura a seguir, como será o seu projeto usando a plataforma Ubidots.

O Ubidots é uma plataforma IoT que oferece uma variedade de recursos para facilitar o desenvolvimento e gerenciamento de projetos IoT. Vatagens:

1. **Interface Amigável:** O Ubidots possui uma interface de usuário intuitiva e amigável, o que facilita a configuração, monitoramento e análise de dados do seu dispositivo IoT. Não precisa desenvolver seu front-end nesse projeto.

2. **Suporte a Diversos Dispositivos e Protocolos:** A plataforma suporta uma variedade de dispositivos e alguns protocolos IoT (MQTT e HTTP), permitindo a integração fácil de diferentes tipos de sensores e hardware. Em especial, vamos usar o protocolo MQTT e a quantidade de sensor que o ESP32 suportar.

3. **Visualização de Dados em Tempo Real:** Fornece ferramentas para visualização de dados em tempo real, incluindo gráficos e dashboards personalizáveis, permitindo que os usuários monitorem facilmente o status de seus dispositivos IoT.

4. **Regras e Ações Personalizadas:** Permite a configuração de regras e ações personalizadas com base nos dados coletados, facilitando a automação e a resposta a eventos específicos.

5. **Segurança:** Oferece recursos de segurança robustos para proteger os dados transmitidos e armazenados, garantindo a integridade e confidencialidade das informações.

6. **Escalabilidade:** A plataforma é projetada para ser escalável, permitindo que os usuários aumentem o número de dispositivos conectados e gerenciem grandes frotas de dispositivos IoT.

7. **Facilidade de Conectividade:** Simplifica a configuração de conexões entre dispositivos e a nuvem, proporcionando uma experiência mais suave para os desenvolvedores.

8. **Suporte Técnico:** Oferece suporte técnico para ajudar os desenvolvedores a superar desafios e resolver problemas durante a implementação de projetos IoT.

<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/m04-semana06/blob/main/imgs/arquitetura_geral_mqtt.jpg">
   <img alt="Família ESP32" src="[YOUR-DEFAULT-IMAGE](https://github.com/agodoi/m04-semana06/blob/main/imgs/arquitetura_geral_mqtt.jpg)">
</picture>

O Ubidots é uma plataforma IoT baseada em nuvem que permite a coleta, armazenamento, visualização e análise de dados provenientes de dispositivos IoT. O processo de como os dados chegam ao Ubidots, são validados por meio de token e processados pelas informações do dispositivo IoT geralmente segue os seguintes passos:

## Documentação Ubidots

[Referência](https://docs.ubidots.com/reference/welcome)

[Códigos](https://github.com/ubidots)

[Exemplos de Desenvolvimento](https://dev.ubidots.com/devices/industrial-iot?_gl=1*dmqooy*_ga*NDExMzY1MTYxLjE2OTQ1NDI2MTA.*_ga_VEME7QQ5EZ*MTcwMDY1NzMwNy4xNi4xLjE3MDA2NTc1MDguNjAuMC4w)

[Exemplo de Projetos](https://www.hackster.io/ubidots/projects)

## Diagrama de Blocos do Funcionamento

1. **Criação de uma Conta e Projeto:**
   - Primeiramente, você precisa criar uma conta no Ubidots e iniciar um projeto. Durante esse processo, você obtém um token de acesso exclusivo associado ao seu projeto. Para esse caso do Inteli, você terá que abrir a sua conta pessoal Ubidots e ainda, terá outra conta que será criada pela coordenação/instrutor de programação. Essa segunda conta será a oficial do seu projeto. Hoje vamos criar a sua conta pessoal, e depois, num segundo momento, chegará a segunda conta. Essa segunda conta estará no nome do representante do grupo, mas a senha será pública entre as pessoas do mesmo grupo.

2. **Registro do Dispositivo:**
   - Para cada dispositivo IoT que você deseja conectar ao Ubidots, você precisará registrá-lo no seu projeto com um label. Isso é feito no seu código-fonte do ESP32 e o Ubidots criará automaticamente caso ele nãop esteja criado. Veja o diagrama de blocos a seguir para melhor entender.

3. **Integração do Dispositivo com a API Ubidots:**
   - No firmware ou software do seu dispositivo IoT, você incorporará a lógica para enviar dados para o Ubidots usando a API fornecida pela plataforma. Isso geralmente é feito por meio de solicitações HTTP ou MQTT, dependendo do protocolo que você escolheu no seu projeto. Nesse projeto, vamos focar no MQTT.

4. **Envio de Dados para o Ubidots:**
   - Periodicamente, o dispositivo envia dados para o Ubidots que ele chama de **DOT**. Esses DOT inclui **valeu**, **timestamp** e **context**. Veja a figura a seguir:

<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/m04-semana06/blob/main/imgs/dc0ffa6-ubidots-data.png">
   <img alt="Família ESP32" src="[YOUR-DEFAULT-IMAGE](https://github.com/agodoi/m04-semana06/blob/main/imgs/dc0ffa6-ubidots-data.png)">
</picture>


#### Valores [value]
Um valor numérico. O Ubidots aceita número de ponto flutuante de até 16 casas decimais.

{"valor" : 34.87654974}

#### Carimbos de Tempo [timestamp]

Um carimbo de tempo é uma maneira de rastrear o tempo como um total contínuo de segundos. Esta contagem começou no Unix Epoch em 1º de janeiro de 1970, no UTC. Portanto, o carimbo de tempo Unix é meramente o número de segundos entre uma data específica e o Unix Epoch. Por favor, tenha em mente que ao enviar dados para o Ubidots, você deve definir o carimbo de tempo em milissegundos. Além disso, se você recuperar o carimbo de tempo de um ponto, este estará em milissegundos.

"carimbo de tempo" : 1537453824000

O carimbo de tempo acima corresponde a quinta-feira, 20 de setembro de 2018, às 14:30:24.

DICA PROFISSIONAL: Uma ferramenta útil para converter entre carimbos de tempo Unix e datas legíveis por humanos é o Epoch Converter.

#### Contexto [context]

Valores numéricos não são o único tipo de dado suportado. Você também pode armazenar tipos de dados de string ou char no que chamamos de contexto. O contexto é um objeto de chave-valor que permite armazenar não apenas valores numéricos, mas também valores de string. Um exemplo de uso do contexto poderia ser:

"contexto" : {"status" : "ligado", "clima" : "ensolarado"}

O contexto é comumente usado para armazenar as coordenadas de latitude e longitude do seu dispositivo para casos de uso de GPS/rastreamento. Todos os mapas do Ubidots usam as chaves lat e lng do contexto de um ponto para extrair as coordenadas do seu dispositivo. Dessa forma, você só precisa enviar um único ponto com os valores das coordenadas no contexto da variável para plotar um mapa, em vez de enviar separadamente tanto a latitude quanto a longitude em duas variáveis diferentes. Abaixo, você pode encontrar um contexto típico com valores de coordenadas:

"contexto" : {"lat":-6.2, "lng":75.4, "clima" : "ensolarado"}

Observe que você pode misturar tanto valores de string quanto numéricos no contexto. Se sua aplicação for para fins de geo-localização, certifique-se de que as coordenadas estejam definidas em graus decimais.

5. **Validação do Token:**
   - Cada solicitação de dados enviada pelo dispositivo inclui o token de acesso associado ao seu projeto Ubidots. Esse token é validado pela plataforma para garantir que a solicitação seja legítima e autorizada.

6. **Armazenamento dos Dados:**
   - Após a validação do token, os dados enviados pelo dispositivo são armazenados nos servidores do Ubidots. Eles podem ser organizados em variáveis específicas, que representam diferentes tipos de dados ou informações do dispositivo.

7. **Processamento e Visualização dos Dados:**
   - Os dados armazenados podem ser processados e visualizados por meio do painel de controle do Ubidots. Os desenvolvedores podem criar dashboards personalizados para monitorar em tempo real o desempenho de seus dispositivos IoT.

8. **Configuração de Regras e Ações (Opcional):**
   - O Ubidots oferece recursos avançados, como a configuração de regras e ações automáticas com base nos dados recebidos. Isso permite automatizar respostas a determinados eventos, como alertas ou acionamento de dispositivos.

<picture>
   <source media="(prefers-color-scheme: light)" srcset="https://github.com/agodoi/m04-semana06/blob/main/imgs/diagrama_blocos.png">
   <img alt="Família ESP32" src="[YOUR-DEFAULT-IMAGE](https://github.com/agodoi/m04-semana06/blob/main/imgs/diagrama_blocos.png)">
</picture>

## Parte prática

Etapas:

1) Abra sua conta particular e gratuita do [Ubidots Stem](https://ubidots.com/stem). Fique atento que o final da URL tem que ser STEM para ser uma conta de estudante;
2) Instale a biblioteca Ubidots no Arduino IDE. Puxe os dois arquivos zip [dessa pasta](https://drive.google.com/drive/folders/10FC273CVdW05TD02czb3Ic7egROspJ67?usp=share_link) e adicione na sua IDE;
3) Faça o [donwload](https://docs.google.com/document/d/16XECwRzPNouLuc8NXMfkkcSZ2op2C6hjdl63yghBveo/edit?usp=sharing) desse exemplo de Liga-Desliga LED e salve no seu computador;
4) Deixe o seu ESP32 próximo do seu protoboard (ou até afixe seu ESP32 no próprio protoboard) para conectar a um LED externo;
5) Coloque um resistor de aproximadamente 330 ohms no pino positivo do LED, e no pino negativo do LED, conecte ao GND do ESP32;
6) Ligue o outro terminal do R no pino D2 do ESP32;
7) Atualize esses campos do código-fonte da sua forma:
```
const char *UBIDOTS_TOKEN = "";  // peque o seu TOKEN da sua conta particular. Vá na sua foto do seu perfil do Ubidots, e clique em "Credenciais da API" e pegue o "Default Token" e cole aqui
const char *WIFI_SSID = "";      // use o WiFi do seu celular nesse primeiro momento, pois o proxy local pode bloquear seu acesso
const char *WIFI_PASS = "";      // use a senha senha do roteador do seu celular
const char *DEVICE_LABEL = "";   // crie um nome qualquer para o seu dispositivo
const char *VARIABLE_LABEL = ""; // crie um nome qualquer de variável que terá os dados da sua aplicação

```
8) Grave esse código no seu ESP32;
9) Vá no seu ambiente Ubidots, e clique em **Dispositivos** e novamente, selecione a opção **Dispositivos**;
10) Você já verá o label do device que você criou em ```const char *DEVICE_LABEL = ""```;
11) Clique na variável ```const char *VARIABLE_LABEL = ""``` para ter acesso aos dados chegando, que nesse caso, será **0** ou **1** que indica se o LED está ligado ou desligado;
12) Associe um widget à essa variável e controle o ligar-desligar do seu LED.

## Comentário do código

```
const int PUBLISH_FREQUENCY = 1000;    //essa linha é o tempo de atualização do envio do MQTT
unsigned long timer;
```

Dentro do Void loop ()... adição do envio da variável e device para o Ubidots para facilitar a configuração da dashboard lá dentro do Ubidots
```
if(millis() - timer > PUBLISH_FREQUENCY){
    
    ubidots.add(VARIABLE_LABEL, 0);       //essa linha está associando o valor 0 na variável em questão 
    ubidots.publish(DEVICE_LABEL);        //essa linha publica os dados no protocolo MQTT
    timer = millis();                     //essa linha atualiza o valor de millis() que é um contato interno do ESP32
}
```

## Onde se  publica as variáveis e seus valores usando MQTT?

ubidots.add(PUBLISH_VARIABLE_LABEL, value); → manda a variável

ubidots.publish(PUBLISH_DEVICE_LABEL); → manda o device

Para vários sensores, como publicar? Simples, adicione um par de linhas para cada sensor:

ubidots.add(PUBLISH_VARIABLE_LABEL1, value1);

ubidots.publish(PUBLISH_DEVICE_LABEL1);

ubidots.add(PUBLISH_VARIABLE_LABEL2, value2);

ubidots.publish(PUBLISH_DEVICE_LABEL2);

… e assim quantos sensores existirem no seu projeto

## Referência da Prática

[Referência](https://help.ubidots.com/en/articles/748067-connect-an-esp32-devkitc-to-ubidots-over-mqtt?_gl=1*1weeu9s*_ga*NTM1ODIzMjQzLjE2NzMyNjQ2NjY.*_ga_VEME7QQ5EZ*MTY4NDc2MzgxOC40OC4xLjE2ODQ3NjYwMTQuNTMuMC4w)

## Conclusões sobre o MQTT

Cada grupo deve descrecer sobre o que aprendeu ou entendeu nessa prática.

[Clique aqui](https://jamboard.google.com/d/1DeO6qMXhEGPfSgDyyLa0RFzKe3BcfY5c9m2odPYT3j4/edit?usp=sharing) e deixe sua conclusão em 1 ou 2 post-it no Jamboard, que ele será lido hoje.

## Conceitos sobre o dBm

O dB é um parâmetro de medição de intensidade de um sinal qualquer. Lembrando que potência é dado em Watt. 

d → dez

B → Bell → que veio de Alexandre Graham Bell (inventor do telefone fixo)

m → é de milésimo de dB (é opcional)

## Equação do dB

Equação do dB = 10 . log (P_desejado / P_referência) 

Note que esse número dB está na escala logarítmica;

Portanto, dB é um valor referencial, isto é, P_desejado está comparável a um outro número (P_referência)

Quais valores usados para P_ref? → 1W ou 1mW

Exemplo:

Tenho um sinal de WiFi chegando no meu ESP32 de intensidade de -10dBW

Significa que o sinal se enfraqueceu 10dB em relação ao P_referência de 1W

Outro exemplo:

Tenho um sinal de WiFi chegando no meu ESP32 de intensidade de -13dBm

Significa que o sinal se enfraqueceu 13dB em relação ao P_referência de 1mW

O que você precisa saber para fazer essas contas? Que…


1dB		→ ganho de intensidade de  1,25 (é o mesmo que multiplicar por 1,25 na escala linear)

-1dB 		→perda de intensidade de 0,79 (é o mesmo que multiplicar por 0,79 na escala linear)

3dB		→ ganho de 1,995

-3dB		→ perda na escala de 0,5

10dB		→ ganho de 10x

-10dB	→ perda na escala de 0,1

### Exemplos práticos

Um sinal está com 10dBm na entrada da minha antena. Quanto isso vale na escala linear comparado com 1mW, ou seja, quanto vale Pdesejado?

X 		= 10 . log (Pd / 1mW)

10dBm 	= 10. log (Pd / 1mW)

10dBm = 10. log (Pd / 1mW)		→ você pode cancelar os milis envolvidos

10/10 =     log (Pd)

1     =     log (Pd)

10^1  =	         Pd

Pd = 10mW	→ lembre-se que estava em mW no enunciado, então vc corta o mili nas contas, mas retorne ele no resultado

#### Agora tente resolver esses valores

a) Converta 13dBm para escala linear

b) Converta 17dBm para escala linear

c) Converta -41dBm para escala linear

d) Converta -35dBm para escala linear


