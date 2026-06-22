#include <SPI.h>
#include <MFRC522.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SS_PIN 5
#define RST_PIN 27

MFRC522 rfid(SS_PIN, RST_PIN);

// OLED
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

// LED + BUZZER
#define LED 25
#define BUZZER 26

//  LIVRES
byte livre1[4] = {0x10, 0xBA, 0x07, 0x43};
byte livre2[4] = {0x30, 0x9E, 0x0A, 0x43};

// ADMIN
byte adminUID[4] = {0x70, 0x90, 0x0A, 0x43};

bool etat1 = false;
bool etat2 = false;
bool adminMode = false;

void setup() {
  Serial.begin(115200);

  SPI.begin(18, 19, 23, 5);
  rfid.PCD_Init();

  Wire.begin(21, 22);
  display.begin(SSD1306_SWITCHCAPVCC, 0x3C);

  pinMode(LED, OUTPUT);
  pinMode(BUZZER, OUTPUT);

  afficher("Pret");
}

void loop() {

  if (!rfid.PICC_IsNewCardPresent()) return;
  if (!rfid.PICC_ReadCardSerial()) return;

  Serial.print("UID: ");
  for (byte i = 0; i < rfid.uid.size; i++) {
    Serial.print(rfid.uid.uidByte[i], HEX);
    Serial.print(" ");
  }
  Serial.println();

  if (match(adminUID)) {
    toggleAdmin();
  }
  else if (match(livre1)) {
    if (!adminMode) etat1 = toggleLivre("HPC Tome 3", etat1);
    else afficher("ADMIN ACTIVE\nHPC Tome 3 bloque");
  }
  else if (match(livre2)) {
    if (!adminMode) etat2 = toggleLivre("HPC Tome 4", etat2);
    else afficher("ADMIN ACTIVE\nHPC Tome 4 bloque");
  }

  rfid.PICC_HaltA();
}

bool match(byte *uid) {
  for (byte i = 0; i < 4; i++) {
    if (rfid.uid.uidByte[i] != uid[i]) return false;
  }
  return true;
}

bool toggleLivre(String nom, bool etat) {

  etat = !etat;

  digitalWrite(LED, HIGH);
  tone(BUZZER, 1200, 200);

  if (etat) afficher(nom + "\nEmprunte");
  else afficher(nom + "\nRetour");

  delay(1500);
  digitalWrite(LED, LOW);

  return etat;
}

void toggleAdmin() {

  adminMode = !adminMode;

  digitalWrite(LED, HIGH);
  tone(BUZZER, 2000, 300);

  if (adminMode) {
    afficher("ADMIN MODE\nON");
  } else {
    afficher("ADMIN MODE\nOFF");
  }

  delay(1500);
  digitalWrite(LED, LOW);
}

void afficher(String txt) {
  display.clearDisplay();
  display.setTextSize(2);
  display.setTextColor(WHITE);
  display.setCursor(0, 20);
  display.println(txt);
  display.display();
}
