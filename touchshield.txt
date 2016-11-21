/*
 * 터치쉴드 mpr121은 아두이노와 I2C(Inter Integrated Circle) 통신방식을 사용
 * I2C란 복수개의 슬레이브 장치가 복수개의 마스터 장치와 통신하기 위한 프로토콜
 * 여기서는 1개의 마스터(아두이노)가 다수의 슬레이브(mpr121의 터치센서)를 제어하는 방식
 * 데이터 전송 및 제어를 하기 위해 아두이노 기본 라이브러리인 Wire.h 헤더파일을 사용
 * 아두이노에서 I2C 통신을 쉽게 사용하기 위해 'Wire'라는 객체를 제공하는데 I2C 통신을 위한 핀으로 SDA, SCL핀을 하나씩 제공
 */

#include "mpr121.h"
#include <Wire.h>

int irqpin = 2;  // 터치쉴드에 연결되는 아두이노 2번 단자
boolean touchStates[12]; // 터치쉴드에 있는 총 12개의 터치센서

void setup(){
  pinMode(irqpin, INPUT);     // 아두이노 2번 단자에 연결되어있는 터치쉴드를 입력으로 설정
  digitalWrite(irqpin, HIGH); // 2번 단자를 항상 HIGH로 설정 - pull up 저항에서 동작되도록
  
  Serial.begin(9600); // 시리얼 통신의 속도를 9600bs(초당 받는 비트의 수)로 설정  
  Wire.begin();       // I2C 통신을 초기화하고 활성화해줌

  mpr121_setup();     // 터치센서마다 어드레스 초기화
}

void loop(){
  readTouchInputs();  // 터치센서의 입력에 따라 메시지를 출력
}


void readTouchInputs(){

  if(!checkInterrupt()){ // irqpin의 입력을 읽는다. pull up 저항이라고 가정하고 짠 코드이므로 값을 반전시킴
    
    
    Wire.requestFrom(0x5A,2); // mpr121의 어드레스에 2만큼의 데이터를 요청
    
    byte LSB = Wire.read();   // 데이터를 읽어옴
    byte MSB = Wire.read();
    
    uint16_t touched = ((MSB << 8) | LSB); //16bits that make up the touch states

    
    for (int i=0; i < 12; i++){  // Check what electrodes were pressed
      if(touched & (1<<i)){
      
        if(touchStates[i] == 0){   // 입력이 들어왔을 때
          // 핀 번호의 입력 안내문구 출력
          Serial.print("pin ");
          Serial.print(i);
          Serial.println(" was just touched");
        
        }else if(touchStates[i] == 1){
          //pin i is still being touched
        }  
      
        touchStates[i] = 1;      
      }
      else{
        if(touchStates[i] == 1){  // 입력이 들어오지 않을 때
          // 입력이 중단되었다는 문구 출력
          Serial.print("pin ");
          Serial.print(i);
          Serial.println(" is no longer being touched");
          
          //pin i is no longer being touched
       }
        
        touchStates[i] = 0;
      }
    
    }
    
  }
}




void mpr121_setup(void){

  set_register(0x5A, ELE_CFG, 0x00); 
  
  // Section A - Controls filtering when data is > baseline.
  set_register(0x5A, MHD_R, 0x01);
  set_register(0x5A, NHD_R, 0x01);
  set_register(0x5A, NCL_R, 0x00);
  set_register(0x5A, FDL_R, 0x00);

  // Section B - Controls filtering when data is < baseline.
  set_register(0x5A, MHD_F, 0x01);
  set_register(0x5A, NHD_F, 0x01);
  set_register(0x5A, NCL_F, 0xFF);
  set_register(0x5A, FDL_F, 0x02);
  
  // Section C - Sets touch and release thresholds for each electrode
  set_register(0x5A, ELE0_T, TOU_THRESH);
  set_register(0x5A, ELE0_R, REL_THRESH);
 
  set_register(0x5A, ELE1_T, TOU_THRESH);
  set_register(0x5A, ELE1_R, REL_THRESH);
  
  set_register(0x5A, ELE2_T, TOU_THRESH);
  set_register(0x5A, ELE2_R, REL_THRESH);
  
  set_register(0x5A, ELE3_T, TOU_THRESH);
  set_register(0x5A, ELE3_R, REL_THRESH);
  
  set_register(0x5A, ELE4_T, TOU_THRESH);
  set_register(0x5A, ELE4_R, REL_THRESH);
  
  set_register(0x5A, ELE5_T, TOU_THRESH);
  set_register(0x5A, ELE5_R, REL_THRESH);
  
  set_register(0x5A, ELE6_T, TOU_THRESH);
  set_register(0x5A, ELE6_R, REL_THRESH);
  
  set_register(0x5A, ELE7_T, TOU_THRESH);
  set_register(0x5A, ELE7_R, REL_THRESH);
  
  set_register(0x5A, ELE8_T, TOU_THRESH);
  set_register(0x5A, ELE8_R, REL_THRESH);
  
  set_register(0x5A, ELE9_T, TOU_THRESH);
  set_register(0x5A, ELE9_R, REL_THRESH);
  
  set_register(0x5A, ELE10_T, TOU_THRESH);
  set_register(0x5A, ELE10_R, REL_THRESH);
  
  set_register(0x5A, ELE11_T, TOU_THRESH);
  set_register(0x5A, ELE11_R, REL_THRESH);
  
  // Section D
  // Set the Filter Configuration
  // Set ESI2
  set_register(0x5A, FIL_CFG, 0x04);
  
  // Section E
  // Electrode Configuration
  // Set ELE_CFG to 0x00 to return to standby mode
  set_register(0x5A, ELE_CFG, 0x0C);  // Enables all 12 Electrodes
  
  
  // Section F
  // Enable Auto Config and auto Reconfig
  /*set_register(0x5A, ATO_CFG0, 0x0B);
  set_register(0x5A, ATO_CFGU, 0xC9);  // USL = (Vdd-0.7)/vdd*256 = 0xC9 @3.3V   set_register(0x5A, ATO_CFGL, 0x82);  // LSL = 0.65*USL = 0x82 @3.3V
  set_register(0x5A, ATO_CFGT, 0xB5);*/  // Target = 0.9*USL = 0xB5 @3.3V
  
  set_register(0x5A, ELE_CFG, 0x0C);
  
}


boolean checkInterrupt(void){
  return digitalRead(irqpin);
}


void set_register(int address, unsigned char r, unsigned char v){
    Wire.beginTransmission(address); //지정된 주소의 슬레이브에 송신이 시작됨을 알림
    Wire.write(r);          // 슬레이브에 데이터를 보냄
    Wire.write(v);
    Wire.endTransmission(); // 지정된 주소의 슬레이브에 송신이 끝남.
}