#include <SoftwareSerial.h>
const int rxPin = 2;
const int txPin = 3;
SoftwareSerial mySerial(2, 3);

byte messege[256];
uint8_t index_request = 7;
float parameters[64];

// register list : http://www.toyotech.com.tw/upload/pdf/Sch-PM5350rs485.pdf
#define VOLTAGE_C_ADDRESS 0x0BD40002
#define CURRENT_C_ADDRESS 0x0BB80002
#define FREQUENCY_ADDRESS 0x0C260002
#define ACTIVE_POWER_ADDRESS 0x0BEE0002
#define REACTIVE_POWER_ADDRESS 0x0BF60002
#define POWER_FACTOR_ADDRESS 0x0C060002
#define ENERGY_ADDRESS 0x6C7A0002

struct modbus_transmit
{
  uint8_t address = 0x01;
  uint8_t function = 0x03;

  uint32_t startByte_H = 0xFF000000;
  uint32_t startByte_L = 0x00FF0000;

  uint16_t endByte_H = 0x0000FF00;
  uint16_t endByte_L = 0x000000FF;

  uint8_t crc_L = 0xF0;
  uint8_t crc_H = 0x0F;

} MODBUS_REQ;


void setup()
{

  Serial.begin(57600);
  mySerial.begin(9600);
}


void loop()
{
  uint32_t param[index_request] = { VOLTAGE_C_ADDRESS, CURRENT_C_ADDRESS,
                                    FREQUENCY_ADDRESS, ACTIVE_POWER_ADDRESS,
                                    REACTIVE_POWER_ADDRESS, POWER_FACTOR_ADDRESS,
                                    ENERGY_ADDRESS
                                  };

  String command[index_request] = {"voltage", "current", "frequency", "active power",
                                   "reactive power", "power_factor", "energy"
                                  };

  //multiple request-respond
  for (int i = 0; i < 3; i++)
  {

    READING_PARAM( param[i] );
    delay(15);

    parameters[i] = get_param();
    Serial.print(command[i]); Serial.print(" :  ");
    Serial.println( parameters[i] );

    Serial.println();
    delay(250);
  }



  // Jeda looping 8 second
  Serial.println();
  delay(10000);
}

void READING_PARAM(uint32_t PARAM)
{
  // Declare variable Transmit ..
  messege[0] = MODBUS_REQ.address;
  messege[1] = MODBUS_REQ.function;
  messege[2] = (MODBUS_REQ.startByte_H &  PARAM)  >>  24;
  messege[3] = (MODBUS_REQ.startByte_L &  PARAM )  >>  16;

  //register request -1
  messege[3] = messege[3] - 1;

  messege[4] = (MODBUS_REQ.endByte_H  &  PARAM)  >>  8;
  messege[5] = MODBUS_REQ.endByte_L &   PARAM;

  // calculate crc and porting to 8 bit
  uint16_t crc = RTU_Vcrc(messege, 6);
  uint8_t crc_L = crc >> 8;
  uint8_t crc_H = crc;

  messege[6] = crc_L;
  messege[7] = crc_H;

  // Transmit variable messege.
  mySerial.write(messege, 8);
  Serial.print("transmit : .. ");
  //print request
  for (int i = 0; i < 8; i++)
  {
    Serial.print(messege[i], HEX);
    Serial.print(' ');
  }
  Serial.println();
}


uint16_t RTU_Vcrc(uint8_t RTU_Buff[], uint16_t RTU_Leng)
{
  uint16_t temp = 0xFFFF, temp2, flag;
  for (uint16_t i = 0; i < RTU_Leng; i++)
  {
    temp = temp ^ RTU_Buff[i];
    for (uint8_t e = 1; e <= 8; e++)
    {
      flag = temp & 0x0001;
      temp >>= 1;
      if (flag) temp ^= 0xA001;
    }
  }
  // Reverse MSB LSB
  temp2 = temp >> 8;
  temp = (temp << 8) | temp2;
  temp &= 0xFFFF;
  return temp;
}

float get_param()
{

  float variable_param;

  // Check uart comunication
  if (mySerial.available() > 0)
  {

    uint8_t u = 0;
    uint32_t recv [128];

    // use delay for catch all data recieved (for stable)
    delay(10);

    // catch data received from uart
    while (mySerial.available() > 0)
    {

      recv[u] = mySerial.read();
      u++;
    }


    // collect value target
    Serial.print("data masuk : ");

    for ( int i  = 0;  i  <  9;  i++)
    {

      Serial.print(recv[i], HEX);
      Serial.print(' ');
    }
    Serial.println();

    recv[3] = recv[3] << 24;
    recv[4] = recv[4] << 16;
    recv[5] = recv[5] << 8;

    uint32_t data[1];
    data[0] = recv[3]  | recv[4]  | recv[5]  | recv[6];


    //PARSING KEDALAM 3 VARIABLE : S E DAN F
    uint8_t S = ( data[0] >>  31  );
    uint32_t E = (data[0]  >>  23 );
    E = 0b011111111 & E;
    uint32_t F = 0x7FFFFF & data[0] ;

    Serial.print("data[0]: ");
    Serial.print(data[0], HEX);
    Serial.print(' ');

    Serial.print("S: ");
    Serial.print(S, HEX);
    Serial.print(' ');

    Serial.print("E: ");
    Serial.print(E, HEX);
    Serial.print(' ');

    Serial.print("F: ");
    Serial.println(((0x800000 | F)/pow(2,23)), DEC);
    
    
    // convert float32 to decimal
    // https://www.se.com/ww/en/faqs/FA204140/
    variable_param = pow(-1, S) * (pow(2, E - 127)) * (( 0x800000  |  F ) / pow(2,23));
    
    Serial.print("variable_param : ");
    Serial.println(variable_param);
  }

  return variable_param;
}
