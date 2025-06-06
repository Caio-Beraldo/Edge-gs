# Edge-gs
Integrantes:
Caio Costa Beraldo – RM560775

Victor Kenzo Mikado – RM560057


Problema:

-Enchentes urbanas causadas por:

-Drenagem ineficiente em bueiros

-Falta de monitoramento em tempo real

-Alertas tardios à população

Impactos:

-Risco à vida humana (afogamentos, doenças)

-Danos materiais (veículos, comércios, residências)

-Interrupção do tráfego e serviços públicos

Componentes Principais:

-Sensor ultrassônico (HC-SR04): Mede nível de água em cm

-Servomotor: Simula bomba d'água (0°=ligado, 90°=desligado)

-LEDs indicadores: Vermelho (alerta) e Verde (normal)

-Conexão MQTT: Comunicação com Node-RED para dashboard

Lógica de Funcionamento:

Integrações Avançadas:

-API OpenWeather: Previsão de chuva em tempo real

-Node-RED: Dashboard para visualização remota

-Broker MQTT: Mosquitto para comunicação IoT

Instruções para Execução e Simulação

Materiais Necessários:

-ESP32

-Sensor HC-SR04

-ServoMotor

-LEDs (vermelho e verde)

-Resistor 220Ω para LEDs

-Fonte de alimentação 5V

Passo a Passo:


-Instale as bibliotecas no Arduino IDE:

#include <WiFi.h>
#include <ArduinoJson.h>
#include <ESP32Servo.h>
#include <PubSubClient.h>
#include <HTTPClient.h>

Simulação no Wokwi:

-Cole o código fornecido

-Adicione componentes virtuais:

Teste do Sistema:

Simule chuva forte modificando o link do openweather

Aproxime objeto do sensor ultrassônico (<20cm)

Verifique:

-Servo move para 0°

-LED vermelho acende

-Mensagens MQTT são enviadas

Código-fonte:

#include <WiFi.h>
#include<ArduinoJson.h>
#include <ESP32Servo.h>
#include <PubSubClient.h>
#include <HTTPClient.h>
#define trigger 5
#define echo 18
#define ledR 21
#define ledG 22

Servo bombaServo;
#define servopin 19

int dist = 0;

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqttServer = "test.mosquitto.org";
int mqttPort = 1883;

WiFiClient espClient;
PubSubClient client(espClient);

// Para usar a API do OpenWeather
float getPrevisaoChuva() {
    if (WiFi.status() != WL_CONNECTED) {
      Serial.println("WiFi desconectado!");
      return 0.0;
    }

    HTTPClient http;
    String url = "https://api.openweathermap.org/data/2.5/weather?q=oslo&appid=79eb04ab3706010f477e6e6a8da29dc8";
    http.begin(url);

    float chuva = 0.0;
    int httpCode = http.GET();
    
    if (httpCode > 0) {
      String payload = http.getString();
      DynamicJsonDocument doc(1024);
      deserializeJson(doc, payload);

      if (doc.containsKey("rain") && doc["rain"].containsKey("1h")) {
        chuva = doc["rain"]["1h"];
      }
      Serial.println("Dados da API: " + payload);
    } else {
      Serial.println("Erro na requisição HTTP: " + String(httpCode));
    }
    http.end();
    return chuva;
}
// Definir a mensagem do status da chuva
String statusChuva(float chuvaPrevista){
  String status;
  if (chuvaPrevista > 10.0) {
  status = "CUIDADO! Chuva forte";
  } 
  else if (chuvaPrevista > 0) {
  status = "Chovendo";
  }
  else {
  status = "Sem chuva";
  }
  return String(chuvaPrevista)+"mm" + "|" + status;
}
// Conectar o Esp32 com o NodeRed
void setup_wifi() {
 delay(10);
 Serial.println();
 Serial.print("Connecting to ");
 Serial.println(ssid);


 WiFi.mode(WIFI_STA);
 WiFi.begin(ssid, password);


 while (WiFi.status() != WL_CONNECTED) {
   delay(500);
   Serial.print(".");
 }


 randomSeed(micros());


 Serial.println("");
 Serial.println("WiFi connected");
 Serial.println("IP address: ");
 Serial.println(WiFi.localIP());
}
void callback(char* topic, byte* payload, unsigned int length) {
 Serial.print("Message arrived [");
 Serial.print(topic);
 Serial.print("] ");
 for (int i = 0; i < length; i++) {
   Serial.print((char)payload[i]);
 }}
void reconnect() {
 while (!client.connected()) {
   Serial.print("Attempting MQTT connection...");
   String clientId = "ESP32Client-";
   clientId += String(random(0xffff), HEX);
   if (client.connect(clientId.c_str())) {
     Serial.println("Connected");
     client.publish("/sensor/Publish", "Welcome");
     client.subscribe("/sensor/Subscribe");
   } else {
     Serial.print("failed, rc=");
     Serial.print(client.state());
     Serial.println(" try again in 5 seconds");
     delay(5000);
   }}
}
void setup() {
 setup_wifi();
 client.setServer(mqttServer, 1883);
 client.setCallback(callback);
// Conectar o Ultrassonico e os Leds
pinMode(trigger, OUTPUT);
pinMode(echo, INPUT);
pinMode(ledG, OUTPUT);
pinMode(ledR, OUTPUT);

// Comandos para poder girar o Servo
ESP32PWM::allocateTimer(0);
bombaServo.setPeriodHertz(50);
bombaServo.attach(servopin, 500, 2400);
bombaServo.write(0);
}




void loop() {
  // Reconectar ao Esp32 , caso caia a rede.
  if (!client.connected()) {
   reconnect();
 }
 client.loop();
  
  float chuvaPrevista = getPrevisaoChuva();
  // Definir a distancia em cm.
  digitalWrite(trigger, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigger, LOW);
  dist = pulseIn(echo, HIGH);
  dist = dist / 58;
// Condição para acionamento da bomba automaticamente.
if (dist < 20 || chuvaPrevista > 10.0) {
    
    bombaServo.write(0);
    digitalWrite(ledR, HIGH);
    digitalWrite(ledG, LOW);
    
  } else {
    
    bombaServo.write(90);
    digitalWrite(ledR, LOW);
    digitalWrite(ledG, HIGH);
    
  }
  // Topicos para o NodeRed
  String msg = statusChuva(chuvaPrevista);
  client.publish("sensor/nivel", String(dist).c_str(),false);
  client.publish("sensor/chuva", String(chuvaPrevista).c_str(),false);
  client.publish("sensor/msg", msg.c_str(),false);
  delay(3000);
  

}

