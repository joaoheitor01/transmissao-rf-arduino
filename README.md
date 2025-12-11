# Implementação da Camada de Enlace em Sistema de Comunicação RF 433MHz

Este repositório contém o código fonte e a documentação de um sistema de transmissão de dados sem fio utilizando módulos RF 433MHz. O projeto implementa técnicas da **Camada de Enlace** (modelo OSI), incluindo detecção de erros via Checksum e controle de fluxo ARQ (Stop-and-Wait).

## 👥 Autores
Projeto desenvolvido pelos discentes de Engenharia da Computação do IFMT:
* João Heitor Kozow Bittencourt Bertoloto
* Henzo Henrique Ferreira Moraes
* Enzo Bernardo de Campos
* Mauricio Matias Marques

## 🛠️ Hardware Utilizado
* 2x Microcontroladores Arduino Uno R3 
* 1x Par de Módulos RF (Transmissor e Receptor) 433 MHz
* 5x Botões (Push Buttons) para controle do emissor
* 3x LEDs para feedback visual no receptor
* Resistores e Jumpers

## ⚙️ Funcionamento do Protocolo

O sistema foi projetado para operar em um canal ruidoso. Para garantir a integridade, foram implementadas as seguintes lógicas:

1.  **Estrutura do Pacote:** O emissor envia uma `String` contendo a sequência de dados seguida por um dígito verificador (Checksum).
    *Cálculo:* Soma-se os números da sequência e utiliza-se o resto da divisão por 10 (módulo) como verificador.
2.  **Verificação (Receiver):** O receptor separa o último dígito, recalcula a soma dos dados recebidos e compara com o checksum enviado.
3.  **Controle de Fluxo (ARQ):**
    **ACK (111):** Se os dados estiverem íntegros, o receptor envia o código `111` e executa a sequência nos LEDs.
    **NACK (999):** Se houver erro de integridade, o receptor envia `999`, solicitando retransmissão automática.

## 🔌 Esquema de Ligação

### Transmissor (Emissor)
| Componente | Pino Arduino | Função |
| :--- | :--- | :--- |
| Módulo RF  | D12 | Transmissão de Dados  |
| Botão Reset | D5 | Limpa a sequência |
| Botão Send | D6 | Envia o pacote |
| Botão P1 | D2 | Adiciona "1" à sequência  |
| Botão P2 | D3 | Adiciona "2" à sequência |
| Botão P3 | D4 | Adiciona "3" à sequência |
| LED Status | D7 | Feedback de envio/erro |

### Receptor
| Componente | Pino Arduino | Função |
| :--- | :--- | :--- |
| Módulo RF (RX) | D11 | Recebimento de Dados |
| Módulo RF (TX) | D12 | Envio de ACK/NACK |
| LED Verde | D2 | Representa o dado '1' |
| LED Amarelo | D3 | Representa o dado '2' |
| LED Vermelho | D4 | Representa o dado '3' |

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
