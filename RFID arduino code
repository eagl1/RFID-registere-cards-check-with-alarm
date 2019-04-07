/****************************** LIBRARIES ******************************/
#include <SPI.h>
#include <LiquidCrystal_I2C.h>
#include <RFID.h>

/****************************** DEFINITIONS ******************************/
// pins definitions
#define SS_PIN          10
#define RST_PIN         9
#define IR_1            A0
#define BUZZER          7
#define SERVO1          3
#define ACCEPT_LED      4 
#define DENIED_LED      5

// setup definitions
#define CARDS           1
#define RFID_ARR_SIZE   CARDS*5

// lcd status defines
#define START_MSG       0
#define CHECK_MSG       1
#define REGISTER_MSG    2
#define ACCEPTED_MSG    3
#define DENIED_MSG      4
#define ALREADY_MSG     5
#define FINISH_MSG      6

/****************************** VARIABLES ******************************/
// lcd variables:
byte sys_state;
uint32_t sys_st;
uint32_t sys_cr;
uint16_t sys_pr = 500;

  
// lcd variables:
byte lcd_state;
uint32_t lcd_st;
uint32_t lcd_cr;
uint16_t lcd_pr;

// rfid variables:
byte crd_buf[RFID_ARR_SIZE],card_no;
byte register_lock,check_lock,print_lock;

/****************************** FUNCTIONES ******************************/
// system functions
void perphiral_reset(void);

// rfid functions
void register_accepted_card(void);
void record_new_cards(void);
void check_card(void);
void print_registered_cards(void);

// lcd functions
void lcd_start(void);
void lcd_msg_mgt(void);

// objects
RFID rfid(SS_PIN, RST_PIN); 
LiquidCrystal_I2C lcd(0x27,16,2);

//////////////////////////////////////////////////////////////////////
void setup(){ 
  Serial.begin(9600);
  SPI.begin(); 
  rfid.init();
  lcd_start();
  pinMode(BUZZER,OUTPUT);
  pinMode(ACCEPT_LED,OUTPUT);
  pinMode(DENIED_LED,OUTPUT);
  pinMode(SERVO1,OUTPUT);
}

void loop(){
    register_accepted_card(); // apply only once
    check_card();
    lcd_msg_mgt();
    perphiral_reset();
    //servo_ctl_cmd();
}

void perphiral_reset(void){
  if(sys_state){  
    sys_cr = millis();
    if(sys_cr - sys_st >= sys_pr){  
      digitalWrite(ACCEPT_LED,LOW);
      digitalWrite(DENIED_LED,LOW);
      digitalWrite(BUZZER,LOW);
      sys_st = millis();
    }
  }
}

void lcd_start(void){
  lcd.init();
  lcd.backlight();
  lcd_state = START_MSG;
  lcd_st = millis();
}

void lcd_msg_mgt(void){
  lcd_cr = millis();
  if(lcd_cr - lcd_st >= lcd_pr){
    switch(lcd_state){
      
      case START_MSG:
        lcd.clear();
        lcd.print("RFID SYSTEM");
        lcd.setCursor(0, 1);
        lcd.print("Hi ..");
        lcd_state = REGISTER_MSG;
        lcd_pr = 1000;
      break;

      case REGISTER_MSG:
        lcd.clear();
        lcd.print("REGISTER your");      
        lcd.setCursor(0, 1);
        lcd.print("RFID CARD no: ");
        lcd.print(card_no+1);
        register_lock = 0;
        lcd_pr = 300;
      break;
      
      case CHECK_MSG:
        lcd.clear();
        lcd.print("ENTER your");      
        lcd.setCursor(0, 1);
        lcd.print("RFID CARD:");
        check_lock = 1;
        sys_state = 0;
        lcd_pr = 500;  
      break;
             
      case ACCEPTED_MSG:
        lcd.clear();
        lcd.print("CARD ACCEPTED");
        lcd_state = CHECK_MSG;
        lcd_pr = 1000;
      break;  
      
      case DENIED_MSG:
        lcd.clear();
        lcd.print("CARD DENIED");
        lcd_state = CHECK_MSG;
        lcd_pr = 1000;
      break;   

      case ALREADY_MSG:
        lcd.clear();
        lcd.print("CARD ALREADY");
        lcd.setCursor(0, 1);
        lcd.print("REGISTERED");
        lcd_state = REGISTER_MSG;
        register_lock = 0;
        lcd_pr = 1000;
      break;  
       
      case FINISH_MSG:
        lcd.clear();
        lcd.print("FINISHED");
        lcd.setCursor(0, 1);
        lcd.print("REGISTERATION");
        check_lock  = 1;
        lcd_state = CHECK_MSG;
        lcd_pr = 1500;
      break;        
    }
    lcd_st = millis();
  }
}

void register_accepted_card(void){
  if (!register_lock) {
    if (rfid.isCard()) {
      if(rfid.readCardSerial()){
        if(card_no>0){
          for(byte i = 0; i<card_no; i++){
            if(!memcmp((crd_buf+(i*5)),rfid.serNum,5)){
              
              Serial.println("already card is");
              for(byte p = i*5; p<(i*5)+5; p++){
                Serial.print(crd_buf[p],HEX);Serial.print(", ");}
                Serial.println();
                
              lcd_state = ALREADY_MSG;
              register_lock = 1;
            }
          }
        }
        
        if(!register_lock){
          
          for(byte i=0;i<5;i++){
            crd_buf[(card_no*5)+i] = rfid.serNum[i];}

          Serial.print("loop#"); Serial.println(card_no);
          Serial.print("start index ");Serial.println(card_no*5);
          Serial.print("end index ");Serial.println((card_no*5)+5);
          Serial.println();

          
          Serial.print("card#"); Serial.println(card_no+1);
          for(byte i=card_no*5;i<(card_no*5)+5;i++){
            Serial.print(crd_buf[i],HEX);Serial.print(", ");}
          Serial.println();Serial.println("//////////////////////////");
          
          card_no++;
          if(card_no >= CARDS){lcd_state = FINISH_MSG;}
          else{lcd_state = REGISTER_MSG;}
          register_lock = 1;          
        }
      }
    }
    rfid.halt();
  }
}

void print_registered_cards(void){
  if (print_lock){
    Serial.println("total no. of cards is:");
    Serial.println(card_no);
    Serial.println();
    
    for(byte s=0;s<card_no;s++){
      Serial.print("card#"); Serial.println(s+1);
      for(byte i=0;i<5;i++){
        Serial.print(crd_buf[(card_no*5)+i],HEX);Serial.print(", ");} 
        Serial.println();     
    }
    print_lock = 0;
  }
}


void check_card(void){
  bool accessGranted = false;  // A flag to store the state - start off denied.  
  if (check_lock){
    if (rfid.isCard()){
      if (rfid.readCardSerial()){
        for(byte i=0;i<card_no;i++){
          if(!memcmp(crd_buf+(i*5),rfid.serNum,5)){
            accessGranted = true;
          }
        }

        if(accessGranted){
          Serial.println("card accepted");
          digitalWrite(ACCEPT_LED,HIGH);
          digitalWrite(DENIED_LED,LOW);
          digitalWrite(BUZZER,LOW);
          check_lock = 0;
          sys_state = 1;
          lcd_state = ACCEPTED_MSG;
          sys_st = millis();
        }
        else{
          Serial.println("card denied");
          digitalWrite(ACCEPT_LED,LOW);
          digitalWrite(DENIED_LED,HIGH);
          digitalWrite(BUZZER,HIGH);
          check_lock = 0;
          sys_state = 1;
          lcd_state = DENIED_MSG;
          sys_st = millis();
        }
      }
      rfid.halt();
    }
  }
}
