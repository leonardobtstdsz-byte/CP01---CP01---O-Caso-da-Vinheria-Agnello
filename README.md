# CP01---O-Caso-da-Vinheria-Agnello
Projeto de Arduino da "Vinharia Agnello"

## Contexto do projeto
Nosso grupo foi contratado pela Vinheria Agnello para desenvolver um sistema de monitoramento a ser instalado no ambiente em que os vinhos são armazenados. O dono a Vinheria informou que a qualidade do vinho é influenciada diretamente pelas condições de temperatura, umidade e luminosidade do ambiente.

## Descrição do projeto
O projeto tem como proposta elaborar um sistema que faça a captura das informações de luminosidade do ambiente para a Vinharia. Para isso isso, estaremos utilizando um Arduino de modelo UNO, contendo um sistema de alarme formado por 3 Leds; um LED verde para indicar que está OK, um LED amarelo para indica que está em níveis de alerta e um LED Vermelho para indicar que tem algum problema. Para complementar, o projeto contará com um display de LCD com o logo da empresa com mensagem de boas vindas.

## Dependências e Reprodução do projeto
Para reproduzir o projeto em mãos, será necessário um Arduino modelo UNO, um BreadBoard, 3 Leds de cores verde, vermelho e amarelo, 6 resistores de resistencias:" ", um buzzer para o alarme, um sensor para captura das informações de luminosidade do ambiente, e por fim, um display de LCD para simular a logo da empresa.

## Membros do Projeto
- Nicolas Forcione de Oliveira e Souza 
- Enrico Dellatorre da Fonseca
- Alexandre Constantino Furtado Junior
- Leonardo Batista de Souza
- Matheus Freitas dos Santos

## Código utilizado
#include <LiquidCrystal.h>

/*// Definição dos pinos do LCD (de acordo com o seu circuito)
const int rs = 12;   // Pino RS do LCD
const int en = 11;   // Pino EN do LCD
const int d4 = 5;    // Pino D4 do LCD
const int d5 = 4;    // Pino D5 do LCD
const int d6 = 3;    // Pino D6 do LCD
const int d7 = 2;    // Pino D7 do LCD*/
//LiquidCrystal lcd(rs, en, d4, d5, d6, d7);
LiquidCrystal lcd(12, 11, 10, 5, 4, 3, 2);
// Pinos dos componentes
const int LDR_PIN = A0;    // Pino do LDR (sensor de luminosidade)
const int LED_VERDE = 7;   // Pino do LED verde
const int LED_AMARELO = 8; // Pino do LED amarelo
const int LED_VERMELHO = 9; // Pino do LED vermelho
const int BUZZER = 6;      // Pino do buzzer

// Caracteres personalizados
byte garrafa1[8]    = {B00000, B01111, B01000, B01000, B01000, B01111, B00000, B00000}; 
byte garrafa2[8]    = {B00000, B11111, B11100, B11100, B11100, B11111, B00000, B00000}; 
byte garrafa3[8]    = {B00000, B11110, B00011, B00001, B00011, B11110, B00000, B00000}; 
byte garrafa4[8]    = {B00000, B00000, B00000, B11000, B00000, B00000, B00000, B00000}; 
byte gota[8]        = {B00000, B00000, B00000, B11000, B00100, B00100, B01110, B00100}; 
byte tacaVazia[8]   = {B10001, B10001, B01010, B00100, B00100, B00100, B11111, B00000};  
byte tacaMetade[8]  = {B10001, B11111, B01110, B00100, B00100, B00100, B11111, B00000}; 
byte tacaCheia[8]   = {B11111, B11111, B01110, B00100, B00100, B00100, B11111, B00000}; 
byte termometroBaixo[8] = {B01110, B01010, B01010, B01010, B01010, B01010, B11111, B01110}; 
byte termometroMedio[8] = {B01110, B01010, B01010, B01010, B01010, B11111, B11111, B01110}; 
byte termometroAlto[8]  = {B01110, B01010, B01010, B11111, B11111, B11111, B11111, B01110};  

// Variáveis
int leituraLDR = 0;
int luminosidade = 0;

// Limites de luminosidade (em porcentagem)
const int LIMITE_OK = 60;      // até 60% = OK (verde)
const int LIMITE_ALERTA = 85;  // entre 61% e 85% = alerta (amarelo + buzzer)
// acima de 85% = problema (vermelho)

unsigned long ultimoAlerta = 0;
const unsigned long intervaloAlerta = 3000; // 3 segundos

void setup() {
  lcd.begin(16, 2);

  // Criar caracteres personalizados
  lcd.createChar(0, garrafa1);
  lcd.createChar(1, garrafa2);
  lcd.createChar(2, garrafa3);
  lcd.createChar(3, garrafa4);
  lcd.createChar(4, gota);
  lcd.createChar(5, tacaVazia);
  lcd.createChar(6, tacaMetade);
  lcd.createChar(7, tacaCheia);

  // Configuração dos pinos
  pinMode(LED_VERDE, OUTPUT);
  pinMode(LED_AMARELO, OUTPUT);
  pinMode(LED_VERMELHO, OUTPUT);
  pinMode(BUZZER, OUTPUT);
  pinMode(LDR_PIN, INPUT);

  // --- Animação de inicialização ---
  lcd.clear(); 
  lcd.setCursor(5,0); lcd.write(byte(0));
  lcd.setCursor(6,0); lcd.write(byte(1));
  lcd.setCursor(7,0); lcd.write(byte(2));
  lcd.setCursor(8,0); lcd.write(byte(3));
  lcd.setCursor(8,1); lcd.write(byte(5)); // Taça vazia
  lcd.setCursor(10,0); lcd.write(byte(0)); // Termômetro baixo (simbólico)
  delay(1000); // Atraso aumentado para dar tempo de visualização

  lcd.clear();
  lcd.setCursor(5,0); lcd.write(byte(0));
  lcd.setCursor(6,0); lcd.write(byte(1));
  lcd.setCursor(7,0); lcd.write(byte(2));
  lcd.setCursor(8,0); lcd.write(byte(4)); // Gota
  lcd.setCursor(8,1); lcd.write(byte(6)); // Taça meio
  lcd.setCursor(10,0); lcd.write(byte(1)); // Termômetro médio
  delay(1000);

  lcd.clear();
  lcd.setCursor(5,0); lcd.write(byte(0));
  lcd.setCursor(6,0); lcd.write(byte(1));
  lcd.setCursor(7,0); lcd.write(byte(2));
  lcd.setCursor(8,0); lcd.write(byte(4));
  lcd.setCursor(8,1); lcd.write(byte(7)); // Taça cheia
  lcd.setCursor(10,0); lcd.write(byte(2)); // Termômetro alto
  delay(1000);

  // Mensagem de boas-vindas
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print(" BEM-VINDO(a)!  ");
  lcd.setCursor(0, 1);
  lcd.print("Vinheria Agnello");
  delay(3000); // Atraso para a visualização da mensagem de boas-vindas

  // Tela inicial
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Monitor Luminoso");
  delay(2000); // Atraso para exibir a tela inicial
  lcd.clear();
}

void loop() {
  // Leitura do LDR (0 a 1023)
  leituraLDR = analogRead(LDR_PIN);
  luminosidade = map(leituraLDR, 0, 1023, 0, 100);

  // Atualiza LCD
  lcd.setCursor(0, 0);
  lcd.print("Luz: ");
  lcd.print(luminosidade);
  lcd.print("%   ");

  lcd.setCursor(0, 1);
  if (luminosidade < 30) {
    lcd.write(byte(5)); // taça vazia
    lcd.print(" ALERTA BAIXA ");
    
    // LEDs e buzzer
    digitalWrite(LED_VERMELHO, HIGH);
    digitalWrite(LED_AMARELO, LOW);
    digitalWrite(LED_VERDE, LOW);
    tone(BUZZER, 1000); // buzzer toca contínuo
  } 
  else if (luminosidade < 70) {
    lcd.write(byte(6)); // taça meio
    lcd.print(" NIVEL OK    ");
    
    digitalWrite(LED_AMARELO, HIGH);
    digitalWrite(LED_VERMELHO, LOW);
    digitalWrite(LED_VERDE, LOW);
    noTone(BUZZER); // buzzer desligado
  } 
  else {
    lcd.write(byte(7)); // taça cheia
    lcd.print(" MUITA LUZ   ");
    
    digitalWrite(LED_VERDE, HIGH);
    digitalWrite(LED_AMARELO, LOW);
    digitalWrite(LED_VERMELHO, LOW);
    noTone(BUZZER); // buzzer desligado
  }

  delay(1000); // Atraso para facilitar a leitura
}