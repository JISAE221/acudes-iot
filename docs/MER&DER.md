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
| `id_usuario` | `UUID`          | `NOT NULL REFERENCES usuario(id_usuario) ON DELETE RESTRICT` | FK para o usuário proprietário do açude    |

**Observações:**
- Coordenadas em `NUMERIC(9,6)` permitem precisão de ~10 cm — suficiente para geolocalização de açudes.
- `status` é `BOOLEAN` por eficiência no banco; a UI faz a tradução (`TRUE` → "Ativo").
- `ON DELETE RESTRICT` impede que um usuário proprietário seja deletado enquanto possuir açudes cadastrados (proteção contra perda de referência).


### Entidade: `dispositivo`

Representa o dispositivo físico instalado em cada açude — pode ser o **monitor principal** ou um dos **módulos de expansão** (partidas, sonda d'água, alimentador). Cada açude pode ter vários dispositivos.

**Tipo customizado necessário** (PostgreSQL exige declarar o ENUM antes da tabela):

```sql
CREATE TYPE tipo_dispositivo_enum AS ENUM ('MONITOR','PARTIDA','SONDA_AGUA','ALIMENTADOR');
```

| Coluna              | Tipo                    | Constraints                                                       | Descrição                                                 |
| ------------------- | ----------------------- | ----------------------------------------------------------------- | --------------------------------------------------------- |
| `id_dispositivo`    | `UUID`                  | `PRIMARY KEY DEFAULT gen_random_uuid()`                           | Identificador único do dispositivo                        |
| `id_acude`          | `UUID`                  | `NOT NULL REFERENCES acude(id_acude) ON DELETE RESTRICT`          | FK para o açude onde está instalado                       |
| `n_serie`           | `VARCHAR(20)`           | `NOT NULL UNIQUE`                                                 | Número de série de fábrica (único)                        |
| `endereco_mac`      | `MACADDR8`              | `NOT NULL`                                                        | Endereço MAC (suporta EUI-64)                             |
| `ip`                | `INET`                  | `NOT NULL`                                                        | Endereço IP do dispositivo na rede                        |
| `tipo_dispositivo`  | `tipo_dispositivo_enum` | `NOT NULL`                                                        | Tipo do dispositivo (MONITOR, PARTIDA, SONDA_AGUA, ALIMENTADOR) |
| `data_fabricacao`   | `DATE`                  | `NOT NULL`                                                        | Data de fabricação do hardware                            |
| `nome_dispositivo`  | `VARCHAR(200)`          | `NOT NULL`                                                        | Nome amigável do dispositivo                              |
| `instalado_em`      | `TIMESTAMPTZ`           | `NOT NULL DEFAULT NOW()`                                          | Timestamp da instalação em campo                          |
| `ultimo_contato`    | `TIMESTAMPTZ`           |                                                                   | Timestamp do último heartbeat (`NULL` = nunca se comunicou) |
| `versao_firmware`   | `VARCHAR(20)`           |                                                                   | Versão do firmware atual (suporte/debug remoto)           |
| `status`            | `BOOLEAN`               | `NOT NULL DEFAULT TRUE`                                           | `TRUE` = ativo, `FALSE` = inativo                         |

**Observações:**
- `ON DELETE RESTRICT` impede que um açude seja deletado enquanto possuir dispositivos vinculados — proteção contra perda acidental de histórico em sistema industrial. Para "remover" um dispositivo, usar `status = FALSE` (soft delete).
- `MACADDR8` (em vez de `MACADDR`) suporta endereços EUI-64, padrão moderno usado em IoT/IPv6.
- `INET` é tipo nativo do PostgreSQL — valida formato de IP automaticamente e suporta IPv4 e IPv6.
- `ultimo_contato` admite `NULL` propositalmente: significa "dispositivo cadastrado mas ainda não enviou heartbeat". Forçar `DEFAULT NOW()` mentiria nas queries de monitoramento (ex.: "dispositivos sumidos há mais de 1 dia").
- `tipo_dispositivo_enum` é declarado com sufixo `_enum` para diferenciar o **tipo** da **coluna** de mesmo nome semântico.

### Entidade: `usuario`

Representa uma pessoa com acesso ao sistema. Pode ser **proprietário** de açudes (referenciada por `acude.proprietario_id`) e/ou possuir vínculo com vários açudes via tabela intermediária `usuario_acude` (N:N — onde mora o perfil de acesso).

| Coluna           | Tipo            | Constraints                                                            | Descrição                                                            |
| ---------------- | --------------- | ---------------------------------------------------------------------- | -------------------------------------------------------------------- |
| `id_usuario`     | `UUID`          | `PRIMARY KEY DEFAULT gen_random_uuid()`                                | Identificador único do usuário                                       |
| `nome`           | `VARCHAR(200)`  | `NOT NULL`                                                             | Nome completo do usuário                                             |
| `email`          | `VARCHAR(200)`  | `NOT NULL UNIQUE CHECK (email ~ '^[^@]+@[^@]+\.[^@]+$')`               | Email de login (validação completa de formato fica na aplicação)     |
| `telefone`       | `VARCHAR(20)`   | `NOT NULL UNIQUE CHECK (telefone ~ '^\+[1-9]\d{1,14}$')`               | Telefone em formato internacional E.164 (ex: `+5585999990000`)       |
| `senha`          | `TEXT`          | `NOT NULL`                                                             | Hash da senha (bcrypt/argon2 — hashing feito na aplicação)           |
| `status`         | `BOOLEAN`       | `NOT NULL DEFAULT TRUE`                                                | `TRUE` = ativo, `FALSE` = inativo (soft delete)                      |
| `criado_em`      | `TIMESTAMPTZ`   | `NOT NULL DEFAULT NOW()`                                               | Timestamp de criação do registro                                     |
| `atualizado_em`  | `TIMESTAMPTZ`   | `NOT NULL DEFAULT NOW()`                                               | Timestamp da última atualização                                      |
| `ultimo_login`   | `TIMESTAMPTZ`   |                                                                        | Timestamp do último login (`NULL` = nunca logou)                     |

**Observações:**
- **LGPD — CPF intencionalmente omitido:** o sistema não exige CPF para sua função (autenticação por email + senha), portanto não é coletado. Princípio da minimização de dados pessoais.
- **Senha nunca é armazenada em texto puro.** O hash é gerado na **aplicação** (bcrypt/argon2), nunca no banco. Motivos: (1) separação de responsabilidades; (2) flexibilidade para trocar algoritmo de hash; (3) senha em texto nunca trafega até o banco (logs, traces, dumps).
- `TEXT` (em vez de `VARCHAR(60)`) acomoda hashes de qualquer algoritmo moderno — bcrypt gera ~60 chars, argon2id pode passar de 100.
- **Email:** regex no `CHECK` é uma validação básica (existe `@` e `.`). Validação completa (RFC 5322, MX record, etc.) é responsabilidade da aplicação — regex de email "100% correta" é monstruosa e raramente vale a pena no banco.
- **Telefone em E.164:** padrão internacional (`+` + país + número, até 15 dígitos). Funciona para qualquer país, suporta integração com gateways de SMS/WhatsApp.
- `ultimo_login` admite `NULL` propositalmente — significa "usuário cadastrado mas nunca logou". Mesma lógica do `dispositivo.ultimo_contato`.
- Campo de **perfil de acesso** (dono/operador/visualizador) **não** mora aqui — mora na tabela `usuario_acude`, pois o mesmo usuário pode ter perfis diferentes em açudes diferentes.


---



```mermaid
```