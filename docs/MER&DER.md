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

### Entidade: `medicao_eletrica`

Armazena leituras elétricas trifásicas enviadas pelo dispositivo monitor. Cada linha representa uma amostra completa do sistema em um instante. Esta entidade é uma **hypertable do TimescaleDB**, particionada por `medido_em`.

**Tipo customizado necessário** (PostgreSQL exige declarar o ENUM antes da tabela):

```sql
CREATE TYPE tipo_qualidade_enum AS ENUM ('OK','SUSPECT','BAD','INTERPOLATED');
```

| Coluna                | Tipo                  | Constraints                                                          | Descrição                                                             |
| --------------------- | --------------------- | -------------------------------------------------------------------- | --------------------------------------------------------------------- |
| `id_medicao_eletrica` | `BIGSERIAL`           | parte da PK composta                                                 | Identificador sequencial da medição                                   |
| `id_dispositivo`      | `UUID`                | `NOT NULL REFERENCES dispositivo(id_dispositivo) ON DELETE RESTRICT` | FK para o dispositivo que enviou a leitura                            |
| `medido_em`           | `TIMESTAMPTZ`         | `NOT NULL` (parte da PK composta)                                    | Timestamp da leitura no relógio do sensor                             |
| `recebido_em`         | `TIMESTAMPTZ`         | `NOT NULL DEFAULT NOW()`                                             | Timestamp do recebimento no servidor                                  |
| `tensao_a`            | `REAL`                |                                                                      | Tensão da fase A em volts (`NULL` = sensor falhou)                    |
| `tensao_b`            | `REAL`                |                                                                      | Tensão da fase B em volts (`NULL` = sensor falhou)                    |
| `tensao_c`            | `REAL`                |                                                                      | Tensão da fase C em volts (`NULL` = sensor falhou)                    |
| `corrente_a`          | `REAL`                |                                                                      | Corrente da fase A em ampères (`NULL` = sensor falhou)                |
| `corrente_b`          | `REAL`                |                                                                      | Corrente da fase B em ampères (`NULL` = sensor falhou)                |
| `corrente_c`          | `REAL`                |                                                                      | Corrente da fase C em ampères (`NULL` = sensor falhou)                |
| `angulo_a`            | `REAL`                |                                                                      | Ângulo de fase A em graus (`NULL` = sensor falhou)                    |
| `angulo_b`            | `REAL`                |                                                                      | Ângulo de fase B em graus (`NULL` = sensor falhou)                    |
| `angulo_c`            | `REAL`                |                                                                      | Ângulo de fase C em graus (`NULL` = sensor falhou)                    |
| `frequencia`          | `REAL`                |                                                                      | Frequência do sistema em Hz — única para as 3 fases acopladas         |
| `assimetria`          | `REAL`                |                                                                      | Desequilíbrio entre fases em % — valor único por definição            |
| `fator_potencia_a`    | `REAL`                |                                                                      | Fator de potência da fase A (entre -1 e 1)                            |
| `fator_potencia_b`    | `REAL`                |                                                                      | Fator de potência da fase B (entre -1 e 1)                            |
| `fator_potencia_c`    | `REAL`                |                                                                      | Fator de potência da fase C (entre -1 e 1)                            |
| `qualidade`           | `tipo_qualidade_enum` | `NOT NULL DEFAULT 'OK'`                                              | Status da amostra (`OK`, `SUSPECT`, `BAD`, `INTERPOLATED`)            |

**Constraints de tabela e cláusula `WITH` (hypertable)** — declaradas após as colunas no `CREATE TABLE`:

```sql
PRIMARY KEY (id_medicao_eletrica, medido_em),
CONSTRAINT chk_tensao     CHECK (tensao_a    >= 0 AND tensao_b    >= 0 AND tensao_c    >= 0),
CONSTRAINT chk_corrente   CHECK (corrente_a  >= 0 AND corrente_b  >= 0 AND corrente_c  >= 0),
CONSTRAINT chk_angulo     CHECK (angulo_a BETWEEN -180 AND 180
                              AND angulo_b BETWEEN -180 AND 180
                              AND angulo_c BETWEEN -180 AND 180),
CONSTRAINT chk_fp         CHECK (fator_potencia_a BETWEEN -1 AND 1
                              AND fator_potencia_b BETWEEN -1 AND 1
                              AND fator_potencia_c BETWEEN -1 AND 1),
CONSTRAINT chk_freq       CHECK (frequencia >= 0),
CONSTRAINT chk_assimetria CHECK (assimetria BETWEEN 0 AND 100)
)
WITH (
  timescaledb.hypertable,
  timescaledb.partition_column = 'medido_em'
);
```

**Observações:**

- **Hypertable do TimescaleDB:** a tabela é particionada internamente em **chunks** por intervalo de tempo (default 7 dias). Por isso `medido_em` precisa estar na PK — o TimescaleDB usa essa coluna pra rotear cada insert/query para o chunk correto. Queries por intervalo de tempo se beneficiam de **chunk exclusion** automático.
- **PK em `BIGSERIAL` (não `UUID`):** time-series tem inserção pesada e índice grande. Sequencial de 8 bytes performa melhor que UUID de 16 bytes. Escolha consciente diferente das entidades de domínio (`acude`, `dispositivo`, `usuario`).
- **PK composta `(id_medicao_eletrica, medido_em)`:** exigência da hypertable. Em sistema não-time-series, `id_medicao_eletrica` sozinho bastaria.
- **Dois timestamps por amostra:** `medido_em` é o relógio do sensor (momento real da medição); `recebido_em` é o relógio do servidor. Cobre o caso do dispositivo perder conexão, acumular leituras em buffer local e enviar tudo de uma vez ao reconectar — sem o segundo timestamp seria impossível distinguir "leitura recente" de "buffer offline".
- **Tabela larga (1 linha = 1 amostra completa):** rejeitada a "tabela longa" (1 linha por grandeza) — multiplicaria as linhas em ~12x, perderia tipagem por grandeza e dificultaria CHECKs específicos.
- **`NULL` semântico nas grandezas (exceção à regra "NOT NULL por padrão"):** aqui `NULL` carrega significado próprio — "o sensor desta grandeza falhou em ler nesta amostra". Forçar `NOT NULL` obrigaria valor sentinela (`-9999`), poluiria agregações e exigiria filtragem manual em toda query. Com `NULL`, `AVG()` ignora automaticamente e falhas se contam com `COUNT(*) FILTER (WHERE coluna IS NULL)`.
- **Frequência e assimetria sem `_a/_b/_c`:** em sistema trifásico acoplado, frequência é propriedade do sistema (as três fases giram juntas em mesma frequência, defasadas em 120°). Assimetria é por definição uma medida do desequilíbrio **entre** as fases — não tem sentido elétrico ter "assimetria da fase A".
- **`REAL` (float 32 bits) em todas as grandezas:** sensor de campo já não tem precisão infinita, e `REAL` economiza metade do espaço vs `DOUBLE PRECISION`. Para 1M+ amostras/dia isso pesa em armazenamento e cache.
- **Filosofia dos CHECKs — "sanity check", não calibração:** as faixas servem pra rejeitar valor absurdo (sensor com defeito, bug na aplicação), não pra dizer "leitura ótima". Por isso `tensao_a >= 0` em vez de `BETWEEN 200 AND 240` — queda de tensão é o **evento de falha que se quer registrar**, não lixo. Detecção de alarme operacional fica em outra camada (queries de monitoramento ou tabela `evento_alarme`).
- **Flag `qualidade` única por amostra (não por grandeza):** evita **multicolinearidade** em análises estatísticas/ML — quando o dispositivo inteiro falha, todas as grandezas falham juntas, e três flags separadas seriam features redundantes. Combinada com o `NULL` semântico por grandeza, dá granularidade sem ruído estatístico.
- **`ON DELETE RESTRICT` em `id_dispositivo`:** histórico de medição é sagrado em sistema industrial — proteção contra perda acidental por deleção do dispositivo que originou os dados.

---



```mermaid
```