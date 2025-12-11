# Implementação da Camada de Enlace em Sistema de Comunicação RF 433MHz

Este repositório contém o código fonte e a documentação de um sistema de transmissão de dados sem fio utilizando módulos RF 433MHz. [cite_start]O projeto implementa técnicas da **Camada de Enlace** (modelo OSI), incluindo detecção de erros via Checksum e controle de fluxo ARQ (Stop-and-Wait)[cite: 58, 60].

## 👥 Autores
[cite_start]Projeto desenvolvido pelos discentes de Engenharia da Computação do IFMT[cite: 26, 41]:
* [cite_start]João Heitor Kozow Bittencourt Bertoloto [cite: 29]
* [cite_start]Henzo Henrique Ferreira Moraes [cite: 30]
* [cite_start]Enzo Bernardo de Campos [cite: 31]
* [cite_start]Mauricio Matias Marques [cite: 32]

## 🛠️ Hardware Utilizado
* [cite_start]2x Microcontroladores Arduino Uno R3 [cite: 75]
* [cite_start]1x Par de Módulos RF (Transmissor e Receptor) 433 MHz [cite: 76]
* [cite_start]5x Botões (Push Buttons) para controle do emissor [cite: 77]
* [cite_start]3x LEDs para feedback visual no receptor [cite: 78]
* [cite_start]Resistores e Jumpers [cite: 78]

## ⚙️ Funcionamento do Protocolo

O sistema foi projetado para operar em um canal ruidoso. Para garantir a integridade, foram implementadas as seguintes lógicas:

1.  **Estrutura do Pacote:** O emissor envia uma `String` contendo a sequência de dados seguida por um dígito verificador (Checksum).
    * [cite_start]*Cálculo:* Soma-se os números da sequência e utiliza-se o resto da divisão por 10 (módulo) como verificador[cite: 6, 7].
2.  [cite_start]**Verificação (Receiver):** O receptor separa o último dígito, recalcula a soma dos dados recebidos e compara com o checksum enviado[cite: 168, 170].
3.  **Controle de Fluxo (ARQ):**
    * [cite_start]**ACK (111):** Se os dados estiverem íntegros, o receptor envia o código `111` e executa a sequência nos LEDs[cite: 174].
    * [cite_start]**NACK (999):** Se houver erro de integridade, o receptor envia `999`, solicitando retransmissão automática[cite: 177, 15].

## 🔌 Esquema de Ligação

### Transmissor (Emissor)
| Componente | Pino Arduino | Função |
| :--- | :--- | :--- |
| Módulo RF (Data) | D12 | [cite_start]Transmissão de Dados  |
| Botão Reset | D5 | [cite_start]Limpa a sequência [cite: 2] |
| Botão Send | D6 | [cite_start]Envia o pacote [cite: 2] |
| Botão P1 | D2 | [cite_start]Adiciona "1" à sequência [cite: 2] |
| Botão P2 | D3 | [cite_start]Adiciona "2" à sequência [cite: 2] |
| Botão P3 | D4 | [cite_start]Adiciona "3" à sequência [cite: 2] |
| LED Status | D7 | [cite_start]Feedback de envio/erro [cite: 2] |

### Receptor
| Componente | Pino Arduino | Função |
| :--- | :--- | :--- |
| Módulo RF (RX) | D11 | [cite_start]Recebimento de Dados [cite: 155] |
| Módulo RF (TX) | D12 | [cite_start]Envio de ACK/NACK [cite: 155] |
| LED Verde | D2 | [cite_start]Representa o dado '1' [cite: 156] |
| LED Amarelo | D3 | [cite_start]Representa o dado '2' [cite: 156] |
| LED Vermelho | D4 | [cite_start]Representa o dado '3' [cite: 156] |

## 📸 Imagens do Projeto

![Esquemático](docs/images/esquematico.jpg)
*Esquemático do sistema (Tinkercad)*

![Montagem Física](docs/images/montagem_real.jpg)
*Montagem física realizada em protoboard*

## 🚀 Como Executar
1. Instale a biblioteca **RadioHead** no seu Arduino IDE.
2. Carregue o código `Transmissor.ino` em um Arduino.
3. Carregue o código `Receptor.ino` no segundo Arduino.
4. Conecte as fontes de alimentação e inicie a comunicação pressionando os botões P1, P2 ou P3 seguidos de "Send".