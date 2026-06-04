# Roadmap de Aprendizado — Projeto Açudes IoT

> Metodologia: Ultralearning | Projeto: Sistema de monitoramento de açudes
> Stack: ESP32 → Raspberry Pi → AWS → Docker → PostgreSQL → TypeScript → React

---

## Visão geral das fases

```
FASE 0 → FASE 1 → FASE 2 → FASE 3 → FASE 4 → FASE 5 → FASE 6
Docker    Redes    ESP32    Rasp Pi   Backend  Frontend  AWS
Postgres  MQTT     Eletr.   Gateway   TypeSc.  React     Deploy
  ✅        →        →        →         →        →         →
```

---

## FASE 0 — Infraestrutura Local (atual)

**Objetivo:** Ter PostgreSQL rodando via Docker Compose com boas práticas.

### O que estudar
- Docker: imagens, containers, volumes, redes
- Docker Compose: multi-serviço, variáveis de ambiente, healthcheck
- PostgreSQL: DDL, DML, tipos de dados, índices básicos

### Conceitos matemáticos
- Nenhum nesta fase (foco em infra)

### Recursos
- Documentação oficial Docker: https://docs.docker.com
- Imagem oficial Postgres: https://hub.docker.com/_/postgres
- **Livro:** "The Docker Book" — James Turnbull (prático e direto)
- PostgreSQL oficial: https://www.postgresql.org/docs/current/

### Checkpoint (você prova que aprendeu quando...)
- [ ] Consegue explicar a diferença entre imagem e container
- [ ] Sabe por que volumes existem
- [ ] Consegue criar um banco, tabela e inserir dados via psql
- [ ] Sabe o que o healthcheck faz e por que importa

---

## FASE 1 — Redes e Protocolos IoT

**Objetivo:** Entender como dados viajam do sensor até o banco.

### O que estudar
- Modelo OSI (camadas de rede) — visão geral
- TCP/IP básico
- **MQTT:** publish/subscribe, broker, tópicos, QoS
- JSON como formato de payload

### Conceitos matemáticos
- Nenhum crítico — entender latência e throughput conceptualmente

### Recursos
- **MQTT Essentials (série completa):** https://www.hivemq.com/mqtt-essentials/
- **Livro:** "Computer Networking: A Top-Down Approach" — Kurose & Ross (cap. 1 e 2 são suficientes por agora)
- Mosquitto docs: https://mosquitto.org/documentation/

### Checkpoint
- [ ] Explica publish/subscribe com analogia própria
- [ ] Sabe a diferença entre QoS 0, 1 e 2
- [ ] Consegue publicar e receber uma mensagem MQTT no terminal

---

## FASE 2 — Eletrônica e ESP32

**Objetivo:** Entender o hardware e programar o ESP32 para enviar dados reais.

### O que estudar
- Eletrônica básica: tensão, corrente, resistência (Lei de Ohm)
- GPIO, ADC, I2C, SPI, UART
- Programação do ESP32 (C++/Arduino)
- Sensores relevantes para açudes: ultrassônico (nível), DHT22 (temperatura), turbidez, pH
- MQTT no ESP32 (biblioteca PubSubClient)
- Simulação no Wokwi

### Conceitos matemáticos
- **Lei de Ohm:** V = R × I
- **Divisor de tensão** (para leitura de sensores)
- **ADC:** conversão analógico-digital, resolução em bits
- Básico de filtros (média móvel para ruído de sensor)

### Recursos
- **Livro (essencial):** "Practical Electronics for Inventors" — Scherz & Monk
- **Livro ESP32:** "Programming ESP32 with Arduino IDE" — Dogan Ibrahim
- **Referência eletrônica top:** "The Art of Electronics" — Horowitz & Hill (mais avançado, consulta)
- Wokwi: https://wokwi.com
- ESP32 Arduino Core: https://docs.espressif.com/projects/arduino-esp32
- PubSubClient (MQTT): https://pubsubclient.knolleary.net

### Checkpoint
- [ ] Explica a diferença entre sinal analógico e digital
- [ ] Monta um circuito com LED e resistor no Wokwi sem errar
- [ ] ESP32 no Wokwi publicando temperatura via MQTT num broker real

---

## FASE 3 — Raspberry Pi como Gateway

**Objetivo:** Raspberry Pi recebe dados do ESP32 via MQTT, processa e envia ao banco.

### O que estudar
- Linux básico (terminal, arquivos, processos, systemd)
- Python para IoT (paho-mqtt, psycopg2)
- Node-RED como alternativa visual ao gateway
- Conceito de gateway: validação, transformação e roteamento de dados

### Recursos
- **Livro:** "Linux Command Line" — William Shotts (grátis online: https://linuxcommand.org/tlcl.php)
- Paho MQTT Python: https://eclipse.dev/paho/index.php?page=clients/python/index.php
- Node-RED: https://nodered.org/docs/getting-started/

### Checkpoint
- [ ] Consegue navegar e operar no terminal Linux com confiança
- [ ] Script Python que recebe MQTT e insere no PostgreSQL funcionando

---

## FASE 4 — Backend TypeScript

**Objetivo:** API REST que expõe os dados dos sensores para o frontend.

### O que estudar
- TypeScript: tipos, interfaces, generics, async/await
- Node.js: event loop, módulos
- Express ou Fastify (API REST)
- Clean Architecture: separação de camadas (controller, service, repository)
- Testes: Jest + Supertest
- PostgreSQL via código: biblioteca pg ou Prisma

### Conceitos matemáticos
- Complexidade de algoritmos: O(n), O(log n) — para queries e estruturas de dados
- Básico de estatística para agregação de sensores: média, mediana, percentis

### Recursos
- **Livro:** "Programming TypeScript" — Boris Cherny (O'Reilly)
- **Livro:** "Effective TypeScript" — Dan Vanderkam
- **Livro:** "Clean Architecture" — Robert C. Martin
- **Livro:** "Clean Code" — Robert C. Martin
- **Livro:** "The Pragmatic Programmer" — Hunt & Thomas
- TypeScript docs: https://www.typescriptlang.org/docs/
- Fastify: https://fastify.dev/docs/latest/

### Checkpoint
- [ ] Explica a diferença entre `type` e `interface` no TypeScript
- [ ] API com endpoint `/sensors/readings` retornando dados do banco
- [ ] Pelo menos 1 teste de integração passando

---

## FASE 5 — Frontend React + Vite

**Objetivo:** Dashboard visual mostrando dados dos sensores em tempo real.

### O que estudar
- React: componentes, props, estado (useState, useEffect)
- TypeScript com React
- Fetch / Axios para consumir a API
- Gráficos: Recharts ou Chart.js
- Tempo real: WebSocket ou Server-Sent Events (SSE)
- Vite como bundler

### Recursos
- **Docs oficiais React (melhores do mercado):** https://react.dev
- Recharts: https://recharts.org
- **Livro:** "Full Stack React" (Acilio Martinez et al.)
- Vite: https://vitejs.dev/guide/

### Checkpoint
- [ ] Explica o ciclo de vida de um componente React
- [ ] Dashboard com gráfico de linha mostrando histórico de temperatura do açude
- [ ] Atualização automática a cada 30 segundos

---

## FASE 6 — AWS e Deploy

**Objetivo:** Toda a arquitetura rodando na nuvem.

### O que estudar
- AWS fundamentals: IAM, VPC, EC2, RDS, ECS
- Docker no AWS: ECS Fargate vs EC2
- Networking AWS: security groups, subnets, load balancer
- CI/CD básico: GitHub Actions

### Recursos
- **AWS Free Tier:** https://aws.amazon.com/free/
- **Curso:** AWS Certified Solutions Architect Associate (A Cloud Guru ou Udemy — Adrian Cantrill é o melhor)
- **Livro:** "AWS in Action" — Andreas Wittig (Manning)
- GitHub Actions: https://docs.github.com/en/actions

### Checkpoint
- [ ] Explica a diferença entre EC2, ECS e RDS
- [ ] Sistema completo rodando na AWS acessível via URL pública

---

## Trilha de leitura recomendada (ordem)

### Prioridade imediata (Fase 0-1)
1. Docker docs (online)
2. MQTT Essentials — HiveMQ (online, grátis)

### Assim que entrar na Fase 2
3. Practical Electronics for Inventors — Scherz & Monk
4. Programming ESP32 — Dogan Ibrahim

### Fase 4 em diante
5. Clean Code — Robert C. Martin
6. Programming TypeScript — Boris Cherny
7. Clean Architecture — Robert C. Martin
8. The Pragmatic Programmer — Hunt & Thomas

### Paralelo (leitura leve, qualquer fase)
9. Computer Networking: A Top-Down Approach — Kurose & Ross (cap. 1-2)
10. Linux Command Line — William Shotts (grátis online)

---

## Progresso atual

| Fase | Status | Iniciado | Concluído |
|------|--------|----------|-----------|
| Fase 0 — Docker + PostgreSQL | 🔄 Em andamento | 2026-06-04 | — |
| Fase 1 — Redes e MQTT | ⏳ Próximo | — | — |
| Fase 2 — ESP32 + Eletrônica | ⏳ Aguardando | — | — |
| Fase 3 — Raspberry Pi Gateway | ⏳ Aguardando | — | — |
| Fase 4 — Backend TypeScript | ⏳ Aguardando | — | — |
| Fase 5 — Frontend React | ⏳ Aguardando | — | — |
| Fase 6 — AWS Deploy | ⏳ Aguardando | — | — |
