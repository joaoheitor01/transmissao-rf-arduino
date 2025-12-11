                                                                                                                                                                                                                                                                                                                                               #include <RH_ASK.h>
#include <SPI.h>

// Configuração do RadioHead:
// Velocidade: 2000 bps
// Pino RX (Receptor): 11
// Pino TX (Transmissor): 12
RH_ASK driver;

// --- PINOS DOS LEDS ---
#define LED_VERDE    2  // Representa o '1'
#define LED_AMARELO  3  // Representa o '2'
#define LED_VERMELHO 4  // Representa o '3'

void setup() {
  Serial.begin(9600);

  // Configura os LEDs
  pinMode(LED_VERDE, OUTPUT);
  pinMode(LED_AMARELO, OUTPUT);
  pinMode(LED_VERMELHO, OUTPUT);

  // Inicializa o sistema de rádio
  if (!driver.init()) {
    Serial.println("Falha ao iniciar o receptor RadioHead!");
  } else {
    Serial.println("Receptor pronto. Aguardando sequencias...");
  }
}

// Função para piscar um LED específico
void piscar(int pino) {
  digitalWrite(pino, HIGH);
  delay(300);
  digitalWrite(pino, LOW);
  delay(200);
}

// Executa a sequência de luzes recebida
void executarSequencia(String seq) {
  Serial.print("Executando: ");
  Serial.println(seq);
  
  for (unsigned int i = 0; i < seq.length(); i++) {
    char c = seq.charAt(i);
    if (c == '1') piscar(LED_VERDE);
    else if (c == '2') piscar(LED_AMARELO);
    else if (c == '3') piscar(LED_VERMELHO);
  }
}

void loop() {
  uint8_t buf[50]; 
  uint8_t buflen = sizeof(buf);

  // Verifica se chegou mensagem
  if (driver.recv(buf, &buflen)) {
    buf[buflen] = 0; // Finaliza a string
    String mensagemRecebida = String((char*)buf);
    
    Serial.print("Pacote recebido: ");
    Serial.println(mensagemRecebida);

    // --- VALIDAÇÃO DE INTEGRIDADE (CHECKSUM) ---
    // O pacote é: [DADOS][ÚLTIMO DIGITO É O CHECKSUM]
    // Ex: "1236" -> Dados="123", ChecksumRecebido='6'
    
    if (mensagemRecebida.length() < 2) return; // Ignora ruído muito curto

    // Separa os dados do digito verificador
    String dados = mensagemRecebida.substring(0, mensagemRecebida.length() - 1);
    char checksumRecebidoChar = mensagemRecebida.charAt(mensagemRecebida.length() - 1);
    int checksumRecebido = checksumRecebidoChar - '0';

    // Recalcula o checksum localmente para ver se bate
    int checksumCalculado = 0;
    for (unsigned int i = 0; i < dados.length(); i++) {
      checksumCalculado += (dados.charAt(i) - '0');
    }
    
    // Pegamos apenas o último dígito da soma (caso a soma seja > 9)
    // Isso é necessário para bater com a lógica matemática do transmissor
    int digitoVerificadorCalculado = checksumCalculado % 10;

    // --- VERIFICAÇÃO ---
    if (digitoVerificadorCalculado == checksumRecebido) {
      // 1. DADOS CORRETOS!
      Serial.println("Integridade OK. Enviando ACK.");
      
      // Envia confirmação (ACK = 111)
      const char *ack = "111";
      driver.send((uint8_t *)ack, strlen(ack));
      driver.waitPacketSent();

      // Executa a ação (piscar os LEDs)
      executarSequencia(dados);

    } else {
      // 2. DADOS CORROMPIDOS!
      Serial.print("Erro de integridade! Calc: ");
      Serial.print(digitoVerificadorCalculado);
      Serial.print(" Rec: ");
      Serial.println(checksumRecebido);
      
      Serial.println("Enviando NACK.");
      
      // Envia erro (NACK = 999) para o transmissor tentar de novo
      const char *nack = "999";
      driver.send((uint8_t *)nack, strlen(nack));
      driver.waitPacketSent();
    }
  }
}