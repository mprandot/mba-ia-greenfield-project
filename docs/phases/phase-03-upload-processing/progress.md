# phase-03-upload-processing — Progress

**Status:** in_progress
**SIs:** 4/7 completed

### SI-03.1 — Infraestrutura: Docker Compose, config namespaces e variáveis de ambiente
- **Status:** completed
- **Tests:** no tests (infra) — 4 ACs verified manually (docker compose up com 5 serviços, MinIO console em :9001, Redis PONG, Joi rejeita MINIO_ENDPOINT ausente)
- **Observations:**
  - Registrei `storageConfig` e `queueConfig` no `load` array do `ConfigModule.forRoot` em `app.module.ts` (não listado explicitamente nas Technical actions, mas necessário para injeção via `ConfigType` nos SIs seguintes).
  - Atualizei `env.validation.integration-spec.ts` (`requiredEnv`) para incluir os novos campos obrigatórios (MINIO_ENDPOINT, MINIO_ACCESS_KEY, MINIO_SECRET_KEY, MINIO_BUCKET_NAME, REDIS_HOST) — sem isso os testes existentes de `validate({})` quebrariam.

### SI-03.2 — Entidade Video e Migração de banco
- **Status:** completed
- **Tests:** 7 passing (6 entity integration + 1 module compilation)
- **Observations:**
  - O banco de dev estava completamente vazio (migrações CreateUsersAndChannels e CreateAuthTokens nunca haviam sido aplicadas neste ambiente) — a primeira tentativa de `migration:generate` gerou um diff recriando todas as tabelas existentes. Rodei `migration:run` para aplicar as duas migrações pendentes primeiro, depois regenerei `CreateVideos` limpo (apenas a tabela `videos`).
  - Atualizei `src/test/create-test-data-source.ts` (`cleanAllTables`) para incluir `DELETE FROM "videos"` antes de `channels` — sem isso, qualquer suíte de integração existente que usa esse helper quebraria por violação de FK assim que houver linhas em `videos` referenciando `channels`.

### SI-03.3 — Módulo de Storage (MinIO/S3)
- **Status:** completed
- **Tests:** 7 passing (1 module compilation + 6 integration against real MinIO)
- **Observations:**
  - context7 MCP não estava disponível neste ambiente (server não conectado); usei WebFetch/WebSearch para validar a API do AWS SDK v3 (`@aws-sdk/client-s3` + `@aws-sdk/s3-request-presigner`) como alternativa, conforme exigido pela regra de "Library Documentation Lookup" do CLAUDE.md.
  - `initializeBucket()` distingue erro 404 (bucket ausente, cria) de qualquer outro erro (relança), em vez de um `catch` genérico — evita mascarar falhas de rede/credenciais como "bucket ausente".
  - Os testes de integração criam alguns uploads multipart que nunca são completados (`test-key-create`, `test-key-parts` — usados só para validar forma do retorno, não fluxo completo), o que deixa multipart uploads incompletos no MinIO entre execuções de teste. Não há endpoint de abort/cleanup implementado nesta SI; se isso virar um problema de acúmulo, considerar um `AbortMultipartUploadCommand` de limpeza ou lifecycle policy no bucket — fora do escopo desta SI.

### SI-03.4 — Módulo de Fila (BullMQ + Redis)
- **Status:** completed
- **Tests:** 3 passing (1 module compilation + 2 integration against real Redis/BullMQ)
- **Observations:**
  - context7 MCP não disponível novamente; usei WebSearch para confirmar a API do `@nestjs/bullmq` (`BullModule.forRootAsync`/`registerQueue`, `@InjectQueue`, opts `attempts`/`backoff`) como alternativa.
  - Teste de integração usa `queue.obliterate({ force: true })` em `beforeEach` para isolar jobs entre testes (fila real compartilhada no Redis do compose), e `queue.close()` em `afterAll` para não deixar conexão pendurada.

### SI-03.5 — Videos API: Criar rascunho e concluir upload
- **Status:** pending
- **Tests:** no tests
- **Observations:** none

### SI-03.6 — Worker: Processamento de Vídeos com FFmpeg
- **Status:** pending
- **Tests:** no tests
- **Observations:** none

### SI-03.7 — Videos API: Endpoints de Recuperação (streaming e download)
- **Status:** pending
- **Tests:** no tests
- **Observations:** none
