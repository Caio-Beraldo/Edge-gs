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
