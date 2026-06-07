# Diário de Progresso

> Atualizado a cada sessão de estudo. Registro honesto do que foi aprendido, o que ficou confuso e o que precisa revisão.

---

## Sessão 001 — 2026-06-04

**Fase:** 0 — Docker + PostgreSQL

### O que foi feito
- Criado `docker-compose.yaml` com PostgreSQL
- Criado `.env` com variáveis de ambiente
- Revisão do tutor identificou 4 bugs:
  - Porta mal formatada (`"``${POSTGRES_PORT}"` → `"${POSTGRES_PORT}:5432"`)
  - Backticks do PowerShell contaminando o YAML
  - Typo: `posgresql` → `postgresql`
  - Faltam seções `volumes:` e `networks:` no final do arquivo
- README criado e revisado (badges, diagrama bidirecional)
- Repositório público criado no GitHub
- Git configurado do zero
- Conflito de porta diagnosticado e resolvido
- DBeaver conectado com sucesso


### O que ficou confuso / precisa revisão
- [x] Diferença entre `image:` e `build:` no docker-compose (dockerfile não está sendo usado)
- [x] Por que o volume nomeado precisa ser declarado no final do arquivo
  - Entendi a diferenca de image e build, iamge e uma virtualziacao e pacote pronto, ja o build vc personaliza.
  - O volume precisa ser declarado na raiz para o Docker saber que deve cria-lo como volume persistente.

### Conceitos validados pelo tutor
- Image vs Build
- Volume nomeado na raiz
- Diagrama bidirecional MQTT
- Diagnostico de porta com Powershell: usados os comandos Get-NetTCPConnection (funcao grosseira e "pegar" a conexao setada pelo LocalPort e verificar com protocolo TCP(_transmission control protocol_) a porta indicada).
Select-Object e selecionar a porta setada, depois vc usa o Owning, qm esta possuindo essa porta, mostrando a OwningProcess(OP), e com esse OP, vc da um Get-Process -Id + OP`s, e ai localizamos quem q esta possuindo a porta.
-ErrorAction SilentlyContinue (Usado para verificar se a porta esta vazia)

### Próximo passo
-Verificar no Dbeaver(SGBD) se o banco esta funcionando
---

## Sessão 002 — 2026-06-05

**Fase:** 0 — Docker + PostgreSQL

### O que foi feito
- Criando o Modelo Relacional
- Aprendi uma coisa importante sobre relacionamentos, cardinalidades e chaves: Quando a cardinalidade é 1:N, a Entidade(1) não precisa de chaves estrangeiras da N, mas Entidade(N) precisa das chaves estrangeiras da 1. N:N; precisa de uma tabela intermediária, que faz o papel do relacionamento. Por fim, 1:1 Em qualquer um dos lados (geralmente no "filho"), adicione a chave estrangeira do "pai".

## Sessão 003 — 2026-06-06

**Fase:** 0 — Docker + PostgreSQL

### O que foi feito
  - Tipos nativos do Postgres: INET, MACADDR8, UUID, ENUM (CREATE TYPE)
  - ON DELETE RESTRICT vs CASCADE — integridade
  industrial
  - Semântica de NULL — quando ausência tem significado
  - ENUM vs tabela auxiliar — trade-off
  - Não modelar por especulação (Pensei que seria uma boa ideia colocar JSONB, mas é especulação para o uso dos sensores, qualquer coisa eu mudo depois)
  - PK/FK — quem gera valor (PK) vs quem aponta (FK)
  - MAC -> Media Address Control (A "Pessoa" q mora na "Casa"IP)
  - Identificado que vamos usar Timescale, tabela medicao vira composta
  - Decisão futura em deixar multivalorado o telefone depois

#### Aprendizados
**O que são CHUNKS?**
Chunk = partição interna de uma hypertable, recortada por intervalo de tempo. Cada chunk é uma tabela física separada por baixo dos panos. Você escreve queries como se fosse uma tabela só -> o Timescale roteia automaticamente.

#### Entidades finalizadas no MER&DER.md
- `dispositivo` — tabela markdown completa + Observações (tipo ENUM customizado, MACADDR8, INET, NULL semântico em `ultimo_contato`)
- `usuario` — tabela markdown completa + Observações (LGPD sem CPF, senha hash na aplicação, email regex básico, telefone E.164)

#### Decisões técnicas pra `medicao_eletrica` (próxima entidade)
- **PK:** `BIGSERIAL` (~9.2 quintilhões — sequencial, mais barato que UUID pra time-series)
- **PK composta:** `(id_medicao_eletrica, medido_em)` — exigência da hypertable do TimescaleDB
- **Formato:** **tabela larga** (1 linha = 1 amostra completa). Decidido NÃO usar tabela longa: aumentaria volume em 12x e perderia tipagem por grandeza
- **Timestamps:** 2 — `medido_em` (relógio do sensor) + `recebido_em` (relógio do servidor). Útil quando dispositivo manda buffer offline
- **Tipo numérico:** decidir grandeza a grandeza entre `NUMERIC(p,s)` (precisão exata) vs `REAL`/`DOUBLE PRECISION` (mais barato, menos preciso)
- **FK `id_dispositivo`:** `ON DELETE RESTRICT` (histórico de medição é sagrado em sistema industrial)
- **Atributos:** puxar do `sensor.md` — tensão/corrente/FP por fase (A,B,C), frequência, ângulo, etc.

#### TimescaleDB — onde encontrar (não usar tutorial random)
- Site: https://www.timescale.com/
- Docs: https://docs.timescale.com/
- Repo: https://github.com/timescale/timescaledb
- Docker Hub: https://hub.docker.com/r/timescale/timescaledb
- Imagem pro projeto: `timescale/timescaledb:latest-pg16` (combina TimescaleDB + Postgres 16)
- Versão Community = gratuita e open source (não confundir com Timescale Cloud/Enterprise pagos)

### Próximo passo
- Esboçar v1 da entidade `medicao_eletrica` usando as decisões acima
- Puxar atributos exatos do `sensor.md`
- Continuar com `medicao_agua`, `status_motor`, `status_alimentador`, `evento_alarme`, `usuario_acude`
- Trocar imagem Docker pra `timescale/timescaledb:latest-pg16` (depois, na hora de criar as tabelas)
<!-- Template para próximas sessões:

## Sessão 00X — YYYY-MM-DD

**Fase:** X — Nome

### O que foi feito

### O que ficou confuso / precisa revisão
- [ ]

### Conceitos validados pelo tutor
- [ ]

### Próximo passo

-->