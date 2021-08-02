/**
   BasicHTTPClient.ino
    Created on: 24.05.2015
*/

#include <Arduino.h>

#include <ESP8266WiFi.h>

#include <ESP8266HTTPClient.h>


#ifndef STASSID
#define STASSID "HMI_Enterence"
#define STAPSK  "trudnehasloduzymiliterami"
#endif



//char* sensor_id_number = "3"; //Id utworzonego w bazie danych sensora
//int numer_licznika = atoi(sensor_id_number);//sensor_id_number.toInt();
const String sensor_id_number = "5";
const int    numer_licznika   =  5 ;


int koncowka_ip = 4 + numer_licznika; //lenistwo do 100 są rezerwowane na liczniki. Czyli możemy zainstalować 60 urządzeń.

long interval = 10000; //Co ile ma wysyłać stan //10 sekund.
unsigned long previousMillis = 0;
unsigned long currentMillis = 0;

const String server_adress = "http://10.186.10.2/test/"; //Aktualnie jest serwer
const String php_key_target = "key.php";
const String field_for_key = "?key=";
const String php_target = "insert.php"; //nazwa skryptu uzupelniajacego. Trzeba pamiętać o "?"
const String field_one = "?nazwa=";
const String field_two = "&dane=";


int i = 0;

int flip_flop = 0;
int flip_flop_komunikacji = 0;
int flip_flop_neta = 0;
int led = D8;
int led_komunikacji = D4;
int led_neta = D0;
int czujnik = D7;
int licznik_impulsow = 0;

void ICACHE_RAM_ATTR dodaj_l_jeden() { //Ta funkcja jest realizowana przez drugi wątek esp.
  {
    //dodałem debouncing bo zliczało nieraz 3 krotnie więcej niż powinno.
    static unsigned long last_interrupt_time = 0;
    unsigned long interrupt_time = millis();
    // If interrupts come faster than 1000ms, assume it's a bounce and ignore
    if (interrupt_time - last_interrupt_time > 1000)
    {
      licznik_impulsow++;

    }
    last_interrupt_time = interrupt_time;
  }

}



void WIFI_Connect()
{
  WiFi.disconnect();
  WiFi.mode(WIFI_STA);
  WiFi.begin(STASSID, STAPSK);
  for (int i = 0; i < 25; i++)
  {
    if ( WiFi.status() != WL_CONNECTED ) {
      delay ( 250 );
      Serial.print ( "." );
      delay ( 250 );
      if (flip_flop_neta == 0) {
        digitalWrite(led_neta, HIGH);
        flip_flop_neta = 1;
      } else {
        digitalWrite(led_neta, LOW);
        flip_flop_neta = 0;
      }
    }
  }
}




void setup() {

  pinMode(led_komunikacji, OUTPUT);
  pinMode(led_neta, OUTPUT);
  pinMode(led, OUTPUT);
  pinMode(czujnik, INPUT); //nie moze byc pullup
  Serial.begin(115200);
  // Serial.setDebugOutput(true);
  Serial.println();
  for (uint8_t t = 4; t > 0; t--) {
    Serial.printf("[SETUP] WAIT %d...\n", t);
    Serial.flush();
    delay(1000);
  }
  Serial.print("yo im ");
  Serial.println(numer_licznika);
  Serial.println("https://github.com/MarcinanBarbarzynca/ESP8266-PNP-Sensor-counter");
  WiFi.mode(WIFI_STA);

  //Statyczne IP Tu musisz zmienić na swoje ustawienia sieci
  IPAddress ip(10, 186, 10, koncowka_ip); //34-->1 35-->2 36-->3 37--> 4
  IPAddress gateway(10, 186, 10, 3);
  IPAddress subnet(255, 255, 255, 0);
  WiFi.config(ip, gateway, subnet);


  WiFi.begin(STASSID, STAPSK);


  attachInterrupt(digitalPinToInterrupt(czujnik), dodaj_l_jeden, FALLING); //Ten interrupt musi znaleźć się na końcu całego setupa


}

void loop() {
  if (digitalRead(D7) == HIGH) {
    digitalWrite(led, HIGH);
  } else {
    digitalWrite(led, LOW);
  }

  if ((WiFi.status() == WL_CONNECTED)) {
    currentMillis = millis();
    if (currentMillis - previousMillis >= interval) {
      previousMillis = currentMillis;
      Serial.print("[HTTP] begin... \n");
      WiFiClient client;
      HTTPClient http;


      int key = random(1000000, 9999999);
      String url_for_key = server_adress + php_key_target + field_for_key + key;
      Serial.println(url_for_key);
      Serial.println();

      String complete_adress = server_adress + php_target + field_one + sensor_id_number + field_two + licznik_impulsow;
      Serial.println(complete_adress);
      Serial.println();


      if (http.begin(client, url_for_key)) {
        Serial.print("[HTTP] GET...\n");
        // start connection and send HTTP header
        int httpCode = http.GET();

        // httpCode will be negative on error
        if (httpCode > 0) {
          // HTTP header has been send and Server response header has been handled
          Serial.printf("[HTTP] GET... code: %d\n", httpCode);

          // file found at server
          if (httpCode == HTTP_CODE_OK || httpCode == HTTP_CODE_MOVED_PERMANENTLY) {
            String payload = http.getString();
            Serial.print("Otrzymany klucz: ");
            Serial.println(payload);
            http.end(); //Zamykanie połączenia a potem ponowne otwieranie w celu przesłania danych.
            if (payload.toInt() == key) {
              if (http.begin(client, complete_adress)) {
                Serial.print("[HTTP] GET...\n");
                // start connection and send HTTP header
                int httpCode = http.GET();

                // httpCode will be negative on error
                if (httpCode > 0) {
                  // HTTP header has been send and Server response header has been handled
                  Serial.printf("[HTTP] GET... code: %d\n", httpCode);

                  // file found at server
                  if (httpCode == HTTP_CODE_OK || httpCode == HTTP_CODE_MOVED_PERMANENTLY) {
                    String payload = http.getString();
                    licznik_impulsow = 0;
                    if (flip_flop_komunikacji == 0) {
                      digitalWrite(led_komunikacji, HIGH);
                      flip_flop_komunikacji = 1;
                    } else {
                      digitalWrite(led_komunikacji, LOW);
                      flip_flop_komunikacji = 0;
                    }
                    Serial.println(payload);
                  }
                } else {
                  Serial.printf("[HTTP] GET... failed, error: %s\n", http.errorToString(httpCode).c_str());
                }


              } else {
                Serial.printf("[HTTP} Unable to connect\n");
              }
            }


          }
        } else {
          Serial.printf("[HTTP] GET... failed, error: %s\n", http.errorToString(httpCode).c_str());
        }

        http.end();
      } else {
        Serial.printf("[HTTP} Unable to connect\n");

      }

    }
  } else {
    //Tu kod rekonekta i informacji o braku połączenia
    WIFI_Connect();


  }

  //delay(10000);
}
