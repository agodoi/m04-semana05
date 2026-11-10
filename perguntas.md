# Aula invertida  Broker MQTT e Node-RED

## Como vamos trabalhar

Vocês já estudaram a teoria do broker. Nesta aula, cada grupo vai discutir as perguntas de cada etapa, registrar uma resposta breve e escolher um integrante para compartilhar o raciocínio com a turma.

O material da Semana 05 ficará disponível para consulta. Primeiro, conversem e formulem uma hipótese; depois, expliquem a resposta e indiquem quais pontos ainda geram dúvidas. O professor retomará cada tema usando o material da aula.

**Combinado:** não compartilhem senhas nem dados de acesso do grupo nas respostas, capturas de tela ou apresentações. Usem apenas a instância e os dispositivos autorizados para sua equipe.

---

<a id="momento-1"></a>

## Momento 1  Conhecendo a arquitetura

**Quando usar:** antes da seção “Arquitetura da infraestrutura local”.

1. Qual é a função de cada elemento na solução: ESP32, roteador local, Raspberry Pi, Mosquitto, Node-RED e notebook?
2. Por que o ESP32 se conecta à rede IoT da sala enquanto o notebook deve permanecer, preferencialmente, na rede do Inteli? Como o Raspberry Pi participa das duas redes?
3. Se o notebook estiver na rede do Inteli, qual endereço e qual porta o grupo deve usar para abrir seu editor? Em que situação o endereço `192.168.10.2` entra na comunicação?

**Desafio do grupo:** desenhem o percurso de uma leitura do sensor até aparecer no notebook. Identifiquem onde atua cada rede.

**Retomada pelo professor:** arquitetura da infraestrutura local e acesso à infraestrutura da T29.


[← Voltar para a explicação na aula](readme.md#arquitetura)

---

<a id="momento-2"></a>

## Momento 2  Quem publica e quem recebe?

**Quando usar:** antes da seção “Funcionamento da comunicação MQTT”.

1. O que são **publisher**, **subscriber** e **broker**? Qual deles recebe a publicação e a encaminha aos interessados?
2. O ESP32 precisa conhecer o endereço do Node-RED para enviar uma leitura? Expliquem o caminho da mensagem.
3. Na prática de hoje, qual dispositivo publica as leituras e qual componente está inscrito para recebê-las?

**Retomada pelo professor:** funcionamento da comunicação MQTT e percurso da telemetria.


[← Voltar para a explicação na aula](readme.md#publicacao)

---

<a id="momento-3"></a>

## Momento 3  Dentro do Mosquitto

**Quando usar:** antes da seção “O que acontece dentro do broker Mosquitto?”.

Considerem que o ESP32 publica no tópico `inteli/iot/222/telemetria`:

```json
{"device":222,"botao":0}
```

1. Antes de publicar, que conexões e identificações precisam ocorrer? O que acontece quando o Node-RED se inscreve em um tópico?
2. Quais informações ajudam o Mosquitto a decidir para quem encaminhar a mensagem? Qual é a diferença entre **tópico** e **payload**?
3. O Mosquitto precisa interpretar o JSON e entender o valor `2048` para encaminhá-lo? Que componente pode interpretar os campos depois?
4. O que pode acontecer se dois ESP32 conectados utilizarem o mesmo identificador de cliente? Como o grupo pode evitar esse problema?

**Desafio do grupo:** organizem em ordem as ações conectar, inscrever, ler sensor, publicar, encaminhar e interpretar. Expliquem onde cada ação ocorre.

**Retomada pelo professor:** etapas da conexão, inscrição, publicação, verificação e encaminhamento no Mosquitto.


[← Voltar para a explicação na aula](readme.md#mosquitto)

---

<a id="momento-4"></a>

## Momento 4  Tópicos, filtros e QoS

**Quando usar:** durante o fluxo do Mosquitto, antes da etapa 8, sobre inscrições e tópicos.

Uma mensagem foi publicada no tópico `inteli/iot/222/telemetria`. Para cada filtro abaixo, discutam se a mensagem corresponderia e expliquem por quê:

- `inteli/iot/222/telemetria`
- `inteli/iot/+/telemetria`
- `inteli/iot/#`
- `inteli/iot/223/telemetria`

Agora respondam:

1. O que os curingas `+` e `#` representam? Onde eles são usados?
2. Que diferenças existem entre QoS 0, QoS 1 e QoS 2? Qual deles admite que uma mensagem chegue mais de uma vez?
3. Se o ESP32 publicar com QoS 0 e o Node-RED solicitar QoS 1, qual é o limite da entrega? Justifiquem.
4. Uma confirmação MQTT, quando aplicável, comprova que o valor já apareceu corretamente no dashboard? Que verificação adicional ajudaria a confirmar isso?

**Retomada pelo professor:** comparação de tópicos, curingas e níveis de QoS.


[← Voltar para a explicação na aula](readme.md#topicos)

---

<a id="momento-5"></a>

## Momento 5  O que acontece com os dados?

**Quando usar:** antes da seção “O broker armazena as mensagens?”.

1. Se ninguém estiver inscrito em um tópico, toda mensagem publicada ficará automaticamente guardada para sempre? De quais configurações isso pode depender?
2. Qual é a diferença entre uma mensagem retida e um histórico com todas as leituras de um sensor?
3. Se o grupo quiser consultar as temperaturas registradas ao longo de uma semana, o Mosquitto sozinho resolve essa necessidade? Que outro componente ou fluxo pode ser necessário?
4. O que o Node-RED precisa fazer para transformar o payload JSON recebido em um valor utilizável por um medidor ou gráfico?

**Retomada pelo professor:** retenção, sessões persistentes, processamento e armazenamento de histórico.


[← Voltar para a explicação na aula](readme.md#armazenamento)

---

<a id="momento-6"></a>

## Momento 6  Preparando a T29

**Quando usar:** antes de “Acesso à infraestrutura da T29”; ao retomar, avance até “Entendendo o arquivo `inteli_broker.h`”.

1. Como o grupo escolhe um ID para cada ESP32? Por que não deve usar um ID fora da sua faixa ou repetido em outra placa?
2. Qual é a diferença entre o endereço do **editor Node-RED**, o endereço do **dashboard** e o endereço MQTT usado pelo ESP32?
3. Que informações o professor enviará no canal privado do grupo? Como elas serão usadas sem expor credenciais para outras equipes?
4. Qual é a função do arquivo `inteli_broker.h`? O que o grupo deve alterar no `.ino` para identificar sua placa?
5. O que fazem `InteliBroker.begin()`, `InteliBroker.send()` e `InteliBroker.loop()`? Qual é a diferença entre adicionar um dado à próxima mensagem e publicá-lo?

**Retomada pelo professor:** acesso por grupo, faixas de IDs, cuidado com credenciais e relação entre os arquivos `.ino` e `.h`.


[← Voltar para a explicação na aula](readme.md#acesso)

---

<a id="momento-7"></a>

## Momento 7  Diagnóstico em equipe

**Quando usar:** depois da montagem e dos primeiros testes, antes da seção “Diagnóstico e resolução de problemas”.

Escolham hipóteses para cada situação. Expliquem qual verificação o grupo faria primeiro e quando chamaria o professor.

1. **O ESP32 reinicia repetidamente.** Quais causas podem estar relacionadas à alimentação, à conexão, ao arquivo da turma ou ao ID? O que o Monitor Serial pode ajudar a observar?
2. **O ESP32 conecta ao Wi-Fi, mas não ao MQTT.** Por que uma conexão Wi-Fi funcionando não garante uma conexão ao broker?
3. **O ESP32 parece publicar, mas o dashboard está vazio.** Como verificar o tópico e usar o node `debug` para descobrir em que etapa os dados deixaram de aparecer?
4. **O estado do botão não aparece no dashboard.** Como verificar a montagem, o payload, o node `debug` e o campo selecionado no fluxo?
5. **O editor do grupo não abre.** Que rede, endereço e porta devem ser conferidos no notebook? Que informações podem ser encaminhadas ao professor sem revelar senhas?

**Retomada pelo professor:** diagnóstico na ordem ESP32 → rede → broker → Node-RED → dashboard e responsabilidades de alunos e equipe técnica.


[← Voltar para a explicação na aula](readme.md#diagnostico)

---

<a id="fechamento"></a>

## Fechamento  Expliquem com suas palavras

**Quando usar:** ao final da aula, antes da seção “Conclusões sobre o MQTT”.

Cada grupo deve apresentar uma explicação curta contemplando:

1. O caminho de uma leitura do sensor até o dashboard.
2. A diferença entre o trabalho do Mosquitto e o do Node-RED.
3. Uma falha que o grupo saberia investigar e o primeiro teste que faria.
4. Uma dúvida que ainda gostaria de discutir com o professor.

[← Voltar para as conclusões da aula](readme.md#conclusoes)
