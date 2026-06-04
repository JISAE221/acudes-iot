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

### O que ficou confuso / precisa revisão
- [ ] Diferença entre `image:` e `build:` no docker-compose (dockerfile não está sendo usado)
- [ ] Por que o volume nomeado precisa ser declarado no final do arquivo

### Conceitos validados pelo tutor
- Nenhum ainda (bugs encontrados antes da validação)

### Próximo passo
- Corrigir os bugs do docker-compose.yaml
- Subir o container e conectar via psql
- Checkpoint da Fase 0

---

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
