#include <Arduino.h>

volatile uint8_t cnt = 0;

// EEPROM READ address 0 (lecture sequence)
uint8_t eepromRead0() {
  while (EECR & 0x02);   // wait if EEPROM busy :contentReference[oaicite:5]{index=5}
  EEAR = 0;              // reading address 0, “Select EEPROM memory location 0.”
  EECR |= 0x01;
  // “Turn ON bit 0 of the EECR register, but keep all other bits unchanged.”
  return EEDR;      
}

// EEPROM WRITE address 0 (lecture sequence)
void eepromWrite0(uint8_t v) { //writing to EEPROM address 0
  while (EECR & 0x02);
  // checks if bit 1 is equal to 1
  //0x02 = 00000010 only bit 1 is TRUE
  // if EECR = 00000010 AND 00000010, result = TRUE
  EEAR = 0;    // select address 0
  EEDR = v;   // put value into EEPROM data register
  EECR |= 0x04;          // master write enable, “I am preparing to write to EEPROM.”
  EECR |= 0x02;          // “Now actually start writing.”
}

void setup() {
  Serial.begin(9600);

  // continue from last stored value (EEPROM persists) :contentReference[oaicite:8]{index=8}
  cnt = eepromRead0();
  // load last saved value
  // if removed -> counter always starts at 0

  Serial.print("Start cnt = ");
  Serial.println(cnt);
}

void loop() {
  // Reload r24 from cnt each second, increment r24, store back.
  asm volatile(
    "lds r24, cnt \n\t"
    // Load from SRAM variable cnt into register r24.
    "inc r24      \n\t"
    // increment r24
    // updates flags and stores result back into r24
    "sts cnt, r24 \n\t"
    // store directly to SRAM
    // “Take value in register r24 and store it into memory variable cnt.”
  );

  delay(1000); // 1 second step

  if (Serial.available()) {           // checks if data is received
    char c = (char)Serial.read();
    // reads one byte from serial buffer
    // (char) converts number into a character
    // if you remove char, it still works
    // if user types S, ASCII code = 83
   

    if (c == 'S') {                // stores current count
      eepromWrite0(cnt);
      Serial.println("Saved to EEPROM (addr 0)");
    }

    if (c == 'R') {              // reset RAM value
      cnt = 0;
      eepromWrite0(0);          // reset EEPROM
      Serial.println("Reset to 0 (and saved)");
    }
  }

  Serial.print("cnt = ");
  Serial.println(cnt);
}
