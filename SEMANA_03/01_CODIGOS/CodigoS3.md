#include <Arduino.h>
//portas
const int buttonPins[] = {16, 17, 18, 19};
const int ledPins[] = {12, 27, 25, 32}; 
const int startButtonPin = 35; 
//quantidades
const int numberOfLedsAndButtons = 4;
const int maxSequence = 100;
int gameSequence[maxSequence];
int currentSequenceLength = 0;
int inputIndex = 0;
bool gameStarted = false;
// Configura comunicação serial e pinos dos botões e LEDs
void setup() {
  Serial.begin(9600);

  for (int i = 0; i < numberOfLedsAndButtons; i++) {
    pinMode(ledPins[i], OUTPUT);
    pinMode(buttonPins[i], INPUT_PULLDOWN); 
  }

  pinMode(startButtonPin, INPUT_PULLDOWN); 

  Serial.println("Welcome to Genius Game!");
  Serial.println("Press the start button to begin.");
}
// Loop principal que verifica se o jogo começou e controla o fluxo
void loop() {
  if (!gameStarted && digitalRead(startButtonPin) == HIGH) {
    delay(300);
    gameStarted = true;
    startNewGame();
  }

  if (gameStarted) {
    playGame();
  }
}
// Inicia um novo jogo, criando uma nova sequência aleatória
void startNewGame() {
  randomSeed(millis());

  for (int i = 0; i < maxSequence; i++) {
    gameSequence[i] = random(numberOfLedsAndButtons);
  }

  currentSequenceLength = 1;
  inputIndex = 0;
  gameStarted = true; 

  Serial.println("Starting new game!");
}
// Executa um nível do jogo, mostrando a sequência e lendo a entrada do jogador
void playGame() {
  for (int i = 0; i < currentSequenceLength; i++) {
    digitalWrite(ledPins[gameSequence[i]], HIGH);
    delay(500);
    digitalWrite(ledPins[gameSequence[i]], LOW);
    delay(250);
  }
// Reinicia o jogo apos o jogador perder
  for (int i = 0; i < currentSequenceLength; i++) {
    int buttonIndex = waitForButtonPress();
    if (buttonIndex != gameSequence[i]) {
      Serial.println("Wrong button! Starting over.");
      gameStarted = false; 
      delay(1000); 
      return; 
    }
  }

  Serial.println("Correct! Next sequence.");
  currentSequenceLength++;
  if (currentSequenceLength > maxSequence) {
    Serial.println("Congratulations! You've completed the game!");
    gameStarted = false; 
  }

  delay(1000); 
}

int waitForButtonPress() {
  while (true) {
    for (int i = 0; i < numberOfLedsAndButtons; i++) {
      if (digitalRead(buttonPins[i]) == HIGH) { 
        delay(200); 
        while (digitalRead(buttonPins[i]) == HIGH); 
        return i; 
      }
    }
  }
}
