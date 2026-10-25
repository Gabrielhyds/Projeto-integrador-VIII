# 🌡️ Monitoramento de Temperatura com ESP32

Sistema de monitoramento e controle automático de temperatura utilizando **ESP32, sensor DS18B20, display LCD I2C e módulo relé** para acionamento de um cooler.

O projeto permite visualizar a temperatura em tempo real por meio de um display LCD e de uma interface web acessível por dispositivos conectados à mesma rede Wi-Fi.

O cooler é acionado automaticamente conforme a temperatura detectada pelo sensor, utilizando um sistema de controle com histerese para evitar acionamentos frequentes.

## 📋 Funcionalidades

- 🌡️ Monitoramento de temperatura com o sensor DS18B20.
- 🖥️ Exibição da temperatura e do estado do cooler no display LCD.
- 🌐 Interface web para monitoramento remoto.
- 🔄 Conversão de temperatura entre Celsius e Fahrenheit.
- ⚙️ Controle automático do cooler com histerese.
- 📶 Conectividade Wi-Fi.
- 🔌 Servidor HTTP para disponibilização dos dados.
- 🛡️ Identificação de falhas de leitura do sensor.

## 🧰 Componentes utilizados

| Componente | Descrição |
|---|---|
| ESP32 | Microcontrolador responsável pelo processamento e pela conexão Wi-Fi |
| DS18B20 | Sensor digital de temperatura |
| LCD 16x2 I2C | Exibição local das informações |
| Módulo relé | Controle elétrico do cooler |
| Cooler | Equipamento de refrigeração |
| Resistor de 4,7 kΩ | Pull-up para o sensor DS18B20 |
| Jumpers e protoboard | Conexões elétricas |

## 🔌 Esquema de ligações

### Sensor de temperatura DS18B20

| Pino do sensor | Conexão |
|---|---|
| VCC | 3,3 V do ESP32 |
| GND | GND do ESP32 |
| DATA (DQ) | GPIO 23 |

**Importante:** utilize um resistor de pull-up de 4,7 kΩ entre DATA e 3,3 V.

### Módulo relé

| Pino do módulo | Conexão |
|---|---|
| IN | GPIO 26 do ESP32 |
| VCC | Alimentação compatível com o módulo |
| GND | GND comum, conforme os requisitos do módulo |

O código considera um módulo relé acionado em nível lógico baixo (*active LOW*).

- **LOW:** aciona o relé.
- **INPUT:** libera o pino para que o circuito do módulo mantenha o relé desligado.

O comportamento depende do circuito do módulo utilizado. Verifique se ele permanece desligado quando o pino está em alta impedância.

### Display LCD 16x2 I2C

| Pino do display | Conexão |
|---|---|
| SDA | GPIO 21 |
| SCL | GPIO 22 |
| VCC | Alimentação compatível com o módulo |
| GND | GND do ESP32 |

O endereço I2C configurado no código é `0x27`. Alguns módulos utilizam `0x3F`.

**Atenção:** verifique os níveis lógicos do adaptador I2C. Se houver pull-ups para 5 V, utilize uma solução de compatibilidade de níveis para proteger o ESP32.

### Cooler

O cooler deve ser conectado ao circuito de potência controlado pelo relé, respeitando a tensão e a corrente nominais dos componentes.

**Nunca conecte um cooler de potência diretamente a um GPIO do ESP32.**

## ⚙️ Funcionamento do sistema

O funcionamento está dividido em cinco etapas:

1. **Inicialização:** o ESP32 configura o relé, inicializa o LCD e prepara o sensor.
2. **Leitura da temperatura:** o DS18B20 realiza medições a cada dois segundos.
3. **Controle automático:** a temperatura é comparada com os limites configurados.
4. **Atualização das informações:** o LCD apresenta a temperatura e o estado do cooler.
5. **Monitoramento remoto:** a interface web consulta os dados disponibilizados pelo servidor HTTP.

### Controle de temperatura com histerese

O projeto utiliza dois limites de temperatura para evitar que o relé fique ligando e desligando constantemente próximo de um único valor de referência.

| Condição | Comportamento |
|---|---|
| Temperatura acima de 28 °C, com cooler desligado | Liga o cooler |
| Temperatura abaixo de 26 °C, com cooler ligado | Desliga o cooler |
| Temperatura entre os limites | Mantém o estado anterior |

Os limites são definidos no código:

```cpp
constexpr float TEMP_LIGAR = 28.0f;
constexpr float TEMP_DESLIGAR = 26.0f;
```

O intervalo entre esses valores é chamado de **histerese**.

> O código utiliza comparações estritas (`>` e `<`). Portanto, atingir exatamente 28 °C não liga o cooler, e atingir exatamente 26 °C não o desliga.

## 🌐 Interface web e API HTTP

Após conectar-se à rede Wi-Fi, o ESP32 inicia um servidor HTTP na porta 80.

A interface web pode ser acessada pelo endereço IP exibido no Monitor Serial da Arduino IDE.

Exemplo:

```text
http://192.168.1.100
```

O endereço é ilustrativo. Utilize o IP informado pelo ESP32.

### Rotas disponíveis

| Método | Endpoint | Descrição |
|---|---|---|
| GET | `/` | Retorna a página HTML de monitoramento |
| GET | `/temperaturec` | Retorna a temperatura em Celsius |
| GET | `/temperaturef` | Retorna a temperatura em Fahrenheit |
| GET | `/status` | Retorna o estado atual do cooler |

### Exemplos de respostas

**Temperatura em Celsius — `/temperaturec`**

```text
28.50
```

**Temperatura em Fahrenheit — `/temperaturef`**

```text
83.30
```

**Estado do cooler — `/status`**

```text
LIGADO
```

Em caso de falha na leitura do sensor, os endpoints de temperatura retornam:

```text
--
```

A interface web utiliza JavaScript para consultar os endpoints e atualizar os valores automaticamente a cada dois segundos.

## 💻 Tecnologias e bibliotecas

O projeto utiliza C++ no ambiente Arduino para ESP32.

| Biblioteca | Finalidade |
|---|---|
| `WiFi.h` | Conexão à rede Wi-Fi |
| `WebServer.h` | Criação do servidor HTTP |
| `OneWire.h` | Comunicação com o sensor DS18B20 |
| `DallasTemperature.h` | Leitura dos dados do sensor |
| `Wire.h` | Comunicação I2C |
| `LiquidCrystal_I2C.h` | Controle do display LCD |
| `math.h` | Validação de valores numéricos |

As bibliotecas de terceiros devem estar instaladas no ambiente Arduino.

## 🚀 Instalação e configuração

### Pré-requisitos

- Placa ESP32 compatível com o projeto.
- Arduino IDE instalada.
- Suporte à placa ESP32 configurado.
- Bibliotecas necessárias instaladas.
- Componentes conectados conforme o esquema apresentado.

### 1. Clone o repositório

```bash
git clone https://github.com/SEU-USUARIO/SEU-REPOSITORIO.git
```

Substitua o endereço pelo link real do seu repositório.

### 2. Instale as bibliotecas

No gerenciador de bibliotecas da Arduino IDE, instale:

- OneWire
- DallasTemperature
- LiquidCrystal I2C

As bibliotecas `WiFi.h`, `WebServer.h` e `Wire.h` normalmente acompanham o pacote de suporte à placa ESP32.

### 3. Configure a rede Wi-Fi

No início do código, configure as credenciais da rede:

```cpp
const char* ssid = "NOME_DA_REDE";
const char* password = "SENHA_DA_REDE";
```

**Não publique suas credenciais reais em um repositório público.**

### 4. Configure e envie o código

1. Conecte o ESP32 ao computador.
2. Abra o projeto na Arduino IDE.
3. Selecione a placa ESP32 correspondente.
4. Selecione a porta serial correta.
5. Compile e envie o código para a placa.

### 5. Acesse a interface web

1. Abra o Monitor Serial com velocidade de `115200 baud`.
2. Aguarde a conexão com o Wi-Fi.
3. Identifique o endereço IP informado.
4. Acesse o endereço pelo navegador de um dispositivo conectado à mesma rede.

## 🔍 Solução de problemas

### O LCD não apresenta informações

- Verifique as conexões SDA e SCL.
- Confirme o endereço I2C (`0x27` ou `0x3F`).
- Confira a alimentação do display.
- Verifique a compatibilidade dos níveis lógicos.

### O sensor apresenta falha de leitura

- Confira as conexões do DS18B20.
- Verifique o resistor de pull-up de 4,7 kΩ.
- Confirme se o sensor está conectado ao GPIO 23.
- Consulte as mensagens do Monitor Serial.

### O cooler não liga ou não desliga

- Confira a conexão do relé ao GPIO 26.
- Verifique a alimentação do módulo.
- Confirme se o módulo utiliza acionamento em nível baixo.
- Verifique os limites de temperatura.
- Teste o circuito de potência separadamente, seguindo as precauções elétricas.

### A página web não abre

- Verifique se o ESP32 está conectado ao Wi-Fi.
- Confirme se o dispositivo está na mesma rede.
- Utilize o IP correto.
- Verifique se a rede permite a comunicação entre dispositivos.

## 🛡️ Segurança e limitações

O código identifica falhas de leitura do sensor e solicita o desligamento do cooler.

Entretanto, o desligamento por software não substitui mecanismos independentes de proteção térmica. Em aplicações críticas, o comportamento seguro deve ser definido de acordo com o equipamento controlado.

Outras limitações:

- O monitoramento web depende da conectividade Wi-Fi.
- O servidor HTTP não implementa autenticação.
- A inicialização aguarda indefinidamente a conexão Wi-Fi.
- O comportamento do relé desligado depende do circuito do módulo utilizado.

## 🔮 Melhorias futuras

- Gráficos com histórico de temperatura.
- Configuração dos limites pela interface web.
- Reconexão automática ao Wi-Fi.
- Armazenamento do histórico de medições.
- Autenticação para acesso ao painel.
- Alertas de temperatura.
- Integração com MQTT e plataformas IoT.

## 📄 Licença

A licença do projeto deve ser definida pelo autor e incluída no repositório por meio de um arquivo `LICENSE`.

## 👨‍💻 Autor

Projeto desenvolvido para monitoramento e controle automático de temperatura com ESP32, integrando sensores, acionamento de hardware e monitoramento por interface web.
