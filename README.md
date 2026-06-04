# Projeto IoT para Sistemas Embarcados e Sensores de Açudes

![ESP32](https://img.shields.io/badge/ESP32-E7352C?style=flat&logo=espressif&logoColor=white) ![Raspberry PI](https://img.shields.io/badge/Raspberry%20Pi-A22846?style=flat&logo=raspberrypi&logoColor=white) ![MQTT](https://img.shields.io/badge/MQTT-660066?style=flat&logo=mqtt&logoColor=white) ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white) ![Postgres](https://img.shields.io/badge/Postgres-%23316192.svg?logo=postgresql&logoColor=white) ![TypeScript]() ![React](https://img.shields.io/badge/React-61DAFB?style=flat&logo=react&logoColor=black) ![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat&logo=amazonaws&logoColor=white)

## Descrição do Projeto

_Projeto dedicado a estudo de embarcados IoT, transformar ruídos de sensores analógicos em dados digitais_



## Stack

- ESP32 (simulado)
- Raspberry PI (simulado)
- MQTT
- Docker
- PostgreSQL
- TypeScript
- React
- AWS

## Diagrama

 ```mermaid
  graph LR
      ESP32[ESP32 + Sensores] -->|MQTT| Broker[Mosquitto Broker]
      Broker -->|Subscribe| Gateway[Raspberry Pi / Node-RED]
      Gateway -->|INSERT| DB[(PostgreSQL)]
      DB -->|Query| API[Backend TypeScript]
      API -->|HTTP| Front[React Dashboard]
```

## Como Rodar Localmente

1. Clone o repositório
2. Copie o .env.example para .env e preencha as variáveis
3. docker compose up -d
4. Acesse o pgAdmin em localhost:PORTA

## ROADMAP

[Ver ROADMAP completo](docs/ROADMAP.md)

## Licença MIT