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

### MER

### Entidade: `acude`

Representa um açude monitorado pelo sistema. É a entidade raiz — todos os dispositivos, medições e eventos pertencem a um açude.

| Coluna            | Tipo            | Constraints                                                  | Descrição                                  |
| ----------------- | --------------- | ------------------------------------------------------------ | ------------------------------------------ |
| `id_acude`        | `UUID`          | `PRIMARY KEY DEFAULT gen_random_uuid()`                      | Identificador único do açude               |
| `nome_acude`      | `VARCHAR(200)`  | `NOT NULL`                                                   | Nome do açude (ex: "Açude Castanhão")      |
| `endereco`        | `VARCHAR(255)`  | `NOT NULL`                                                   | Endereço/localização descritiva            |
| `municipio`       | `VARCHAR(100)`  | `NOT NULL`                                                   | Município onde o açude está localizado     |
| `estado`          | `CHAR(2)`       | `NOT NULL`                                                   | Sigla do estado (ex: "CE", "PB")           |
| `latitude`        | `NUMERIC(9,6)`  | `NOT NULL`                                                   | Coordenada geográfica (latitude)           |
| `longitude`       | `NUMERIC(9,6)`  | `NOT NULL`                                                   | Coordenada geográfica (longitude)          |
| `status`          | `BOOLEAN`       | `NOT NULL DEFAULT TRUE`                                      | `TRUE` = ativo, `FALSE` = inativo          |
| `criado_em`       | `TIMESTAMPTZ`   | `NOT NULL DEFAULT NOW()`                                     | Timestamp de criação do registro           |
| `atualizado_em`   | `TIMESTAMPTZ`   | `NOT NULL DEFAULT NOW()`                                     | Timestamp da última atualização            |
| `proprietario_id` | `UUID`          | `NOT NULL REFERENCES usuario(id_usuario) ON DELETE RESTRICT` | FK para o usuário proprietário do açude    |

**Observações:**
- Coordenadas em `NUMERIC(9,6)` permitem precisão de ~10 cm — suficiente para geolocalização de açudes.
- `status` é `BOOLEAN` por eficiência no banco; a UI faz a tradução (`TRUE` → "Ativo").
- `ON DELETE RESTRICT` impede que um usuário proprietário seja deletado enquanto possuir açudes cadastrados (proteção contra perda de referência).
  
```mermaid
```