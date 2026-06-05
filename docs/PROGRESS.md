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
