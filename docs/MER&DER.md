# Modelo Relacional do Banco de Dados

## Entidades do Sensor

- acude
- dispositivo (monitor principal + módulos)
- medicao_eletrica (dados das 3 fases)
- medicao_agua (OD + temperatura)
- status_motor (partidas, contator, relé)
- status_alimentador
- evento_alarme (permanente — não apaga em 30 dias)
- usuario
- perfil_acesso

### Relacionamento Puro Documentado

1. acude **(1:N)** dispositivo **(1:N)** medicao_eletrica, medicao_agua
2. dispositivo **(1:N)** evento_alarme, status_motor, status_alimentador
3. usuario **(N:N)** acude -> exige tabela intermediaria (__usuario_acude__)
    Nivel de usuario_acude:
        - usuario_id (FK)
        - acude_id (FK)
        - perfil (dono, operador, visualizador...)
        - criado_em

```mermaid
```