#include <RH_ASK.h>
#include <SPI.h> 

// Cria o objeto driver. 
// Padrões: velocidade 2000bps, RX pino 11, TX pino 12
RH_ASK driver;

// ----- PINOS -----
#define BTN_RESET 5
#define BTN_SEND  6
#define BTN_P1    2
#define BTN_P2    3
#define BTN_P3    4
#define LED_STATUS 7

String sequencia = "";

// Função para piscar o LED
void piscaLED(int vezes) {
  for (int i = 0; i < vezes; i++) {
    digitalWrite(LED_STATUS, HIGH);
    delay(200);
    digitalWrite(LED_STATUS, LOW);
    delay(200);
  }
}

// Monta um inteiro [SEQUÊNCIA][CHECKSUM]
String montarPacote() {
  int checksum = 0;
  // Calcula a soma de todos os números da sequência
  for (char c : sequencia) {
    checksum += (c - '0');
  }
  
  // Pega apenas o último dígito da soma (ex: soma 15 vira 5)
  int digitoVerificador = checksum % 10;

  // Cola o dígito no final da sequência original
  // Ex: Sequencia "111111" + Digito "6" = "1111116"
  return sequencia + String(digitoVerificador); 
}

void enviarSequencia() {
  // Agora recebemos direto a String pronta
  String pacoteStr = montarPacote();
  
  Serial.print("Enviando pacote: ");
  Serial.println(pacoteStr);

  // Envia a mensagem
  driver.send((uint8_t *)pacoteStr.c_str(), pacoteStr.length());
  driver.waitPacketSent(); 
  
  // ... (O RESTO DA LÓGICA DE ACK/NACK CONTINUA IGUAL ABAIXO) ...
  // Apenas certifique-se de manter o restante da função enviarSequencia 
  // (parte do while e millis) igual ao código anterior.
  
  // ----- TRECHO DE ESPERA (Copie isso para completar a função) -----
  unsigned long inicio = millis();
  bool respostaRecebida = false;

  while (millis() - inicio < 2000) {
    uint8_t buf[10]; 
    uint8_t buflen = sizeof(buf);

    if (driver.recv(buf, &buflen)) {
      buf[buflen] = 0; 
      String resposta = String((char*)buf);
      
      // A resposta do ACK (111) ou NACK (999) é pequena, 
      // então podemos usar toInt() aqui sem medo.
      long valor = resposta.toInt(); 

      Serial.print("Resposta recebida: ");
      Serial.println(valor);

      if (valor == 111) {            
        Serial.println("ACK recebido.");
        piscaLED(2);
        respostaRecebida = true;
        return; 
      } 
      else if (valor == 999) {     
        Serial.println("NACK recebido. Retransmitindo...");
        piscaLED(3);
        driver.send((uint8_t *)pacoteStr.c_str(), pacoteStr.length());
        driver.waitPacketSent();
        delay(200);
        inicio = millis(); 
      }
    }
  }

 if (!respostaRecebida) {
    Serial.println("Timeout: Nenhuma resposta do receptor."); 
    piscaLED(3);
}
}

void setup() {
  Serial.begin(9600);
  
  // Configuração dos pinos (INPUT_PULLUP)
  pinMode(BTN_RESET, INPUT_PULLUP);
  pinMode(BTN_SEND,  INPUT_PULLUP);
  pinMode(BTN_P1,    INPUT_PULLUP);
  pinMode(BTN_P2,    INPUT_PULLUP);
  pinMode(BTN_P3,    INPUT_PULLUP);
  pinMode(LED_STATUS, OUTPUT);

  // Inicializa o RadioHead
  if (!driver.init()) {
    Serial.println("Falha na inicializacao do RadioHead!");
  } else {
    Serial.println("Emissor RadioHead iniciado.");
  }
}
void loop() {
  // Botão 1 - Só aceita se tiver menos de 3 digitos gravados
  if (!digitalRead(BTN_P1) && sequencia.length() < 3) {
    sequencia += "1";
    Serial.print("Sequencia: ");
    Serial.println(sequencia);
    delay(250);
  }
  
  // Botão 2
  if (!digitalRead(BTN_P2) && sequencia.length() < 3) {
    sequencia += "2";
    Serial.print("Sequencia: ");
    Serial.println(sequencia);
    delay(250);
  }
  
  // Botão 3
  if (!digitalRead(BTN_P3) && sequencia.length() < 3) {
    sequencia += "3";
    Serial.print("Sequencia: ");
    Serial.println(sequencia);
    delay(250);
  }

  // Reset da sequência
  if (!digitalRead(BTN_RESET)) {
    sequencia = "";
    Serial.println("Sequencia resetada.");
    delay(300);
  }

  // Enviar sequência
  if (!digitalRead(BTN_SEND)) {
    if (sequencia.length() > 0) {
      enviarSequencia();
    } else {
      Serial.println("Nenhuma sequencia definida.");
    }
    delay(300);
  }
}