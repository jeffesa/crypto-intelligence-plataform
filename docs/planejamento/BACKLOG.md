# BACKLOG — Crypto Intelligence Platform

## Plataforma de Inteligência de Mercado, Análise Preditiva e Investimento Automatizado com IA

> **Tech Challenge / FIAP** — Plataforma robusta de monitoramento, inteligência e simulação de
> investimentos em criptomoedas, utilizando Inteligência Artificial (IA), análise de dados históricos,
> indicadores técnicos, monitoramento de notícias e análise de sentimento de redes sociais.

---

## 📋 ÉPICO 1: Fundação — Estrutura, Backend, Frontend, PostgreSQL e Autenticação

### TASK-001: Criar estrutura multi-module Maven com Spring Boot 3 (Java 21)

**Labels:** `priority: high`, `épico: setup`, `pontos: 8`, `type: infra`
**Milestone:** Sprint 1 - Fundação
**Status:** 🟡 TODO

**Descrição:** Configurar o projeto Maven multi-module com parent POM e módulos por domínio
seguindo arquitetura hexagonal (ports & adapters). Módulos: `identity`, `market-data`, `news-intelligence`,
`social-intelligence`, `ai-engine`, `historical-data`, `strategy-engine`, `trading-engine`,
`risk-management`, `backtesting`, `portfolio`, `notification`, `reporting`. Cada módulo compila
independente.

**Critérios de Aceitação:**
- [ ] Parent POM na raiz com `<modules>` definidos
- [ ] 13 módulos filhos com `pom.xml` próprio
- [ ] Spring Boot 3.x BOM gerenciado no parent
- [ ] Java 21 configurado em todos os módulos
- [ ] Dependências: Spring Web, Security, Data JPA, WebSocket, Batch, AI, Redis, Validation
- [ ] Extensões: Flyway, Testcontainers, JaCoCo, MapStruct, Lombok
- [ ] Cada módulo compila sem erros (`mvn clean compile`)
- [ ] `.gitignore` configurado (target/, .idea/, .DS_Store, .env, etc.)
- [ ] Profiles: dev (local), test (Testcontainers), prod (AWS)

**Dependências Técnicas:** -

---

### TASK-002: Configurar arquitetura hexagonal (ports & adapters) em cada módulo

**Labels:** `priority: high`, `épico: setup`, `pontos: 5`, `type: infra`
**Milestone:** Sprint 1 - Fundação
**Status:** 🟡 TODO

**Descrição:** Organizar pacotes seguindo arquitetura hexagonal: `domain` → `application` (use cases)
→ `infra` (adapters in/out). Ports (interfaces) no `domain/port` e `application/port`, adapters
na `infra/adapter/{in,out}`. Drivers (REST, WebSocket, CLI) em `infra/adapter/in`, Driven
(persistence, external APIs, messaging) em `infra/adapter/out`.

**Critérios de Aceitação:**
- [ ] Estrutura: `domain/{entity,port,valueobject}`, `application/{usecase,port,service}`, `infra/{adapter,config}`
- [ ] Subpacotes: `infra/adapter/in/{web,websocket}`, `infra/adapter/out/{persistence,external,messaging}`
- [ ] Dependências fluem de fora para dentro (infra → application → domain)
- [ ] Classes de domínio são POJOs puros (sem anotações de framework)
- [ ] `package-info.java` documentando a arquitetura em cada módulo
- [ ] Inversão de dependência via interfaces (ports) para repositórios, gateways externos, notificações

**Dependências Técnicas:** TASK-001 concluída

---

### TASK-003: Configurar PostgreSQL com TimescaleDB, Flyway e Redis

**Labels:** `priority: high`, `épico: setup`, `pontos: 5`, `type: infra`
**Milestone:** Sprint 1 - Fundação
**Status:** 🟡 TODO

**Descrição:** Criar `docker-compose.yml` com PostgreSQL 16 + TimescaleDB (hypertables para séries
temporais), Redis 7. Flyway migrations iniciais para tabelas core. Configurar `application.yml`
por profile (dev/test/prod).

**Critérios de Aceitação:**
- [ ] `docker-compose.yml` com postgres, timescaledb, redis
- [ ] Serviços: postgres (5432), timescaledb (5433), redis (6379)
- [ ] Extensão TimescaleDB habilitada
- [ ] Migrations Flyway V1__init_core.sql (tabelas: cryptocurrency, user, portfolio, strategy)
- [ ] `application-dev.yml`, `application-test.yml`, `application-prod.yml`
- [ ] `run.sh` para subir ambiente local
- [ ] Testcontainers configurado para testes de integração

**Dependências Técnicas:** TASK-001 concluída

---

### TASK-004: Configurar CI/CD GitHub Actions (build + deploy automático), SonarCloud e Quality Gates

**Labels:** `priority: high`, `épico: setup`, `pontos: 5`, `type: infra`
**Milestone:** Sprint 1 - Fundação
**Status:** 🟡 IN_PROGRESS

**Descrição:** Pipeline completo de CI/CD em dois jobs (padrão fase 4): job **build** (build, testes
unitários/integração, cobertura JaCoCo ≥ 80%, SonarCloud analysis com quality gate bloquante,
validação de branch naming, auto PR develop→main) e job **deploy** (deploy automático SOMENTE após
merge em `main` e após build+Sonar passarem, com verificação de secrets e OIDC). Deploy adaptado à
estratégia free-tier (EC2 t3.micro + Docker Compose / SAM para RDS), não ECS Fargate.

**Estado atual (implementação):** Workflow criado em `.github/workflows/` (ci-cd.yml, branch-naming.yml, auto-pr-to-main.yml) e `sonar-project.properties`; YAML corrigido; SONAR_TOKEN configurado; branch protection de `main` exigindo `Build & Test & SonarCloud`; PR develop→main auto-criado/auto-merge; CI passou no PR e foi mergeado. Job deploy gateado por `vars.DEPLOY_ENABLED == 'true'` (para quando infra/OIDC secrets estiverem prontos).

**Critérios de Aceitação:**
- [x] `.github/workflows/ci-cd.yml` job `build`: trigger push/PR develop/main, Maven verify, cache, Java 21, SonarCloud
- [x] SonarCloud com `-Dsonar.qualitygate.wait=true` (build falha se quality gate falhar) — PR para `main` exige SONAR_TOKEN e gate bloquante
- [x] `.github/workflows/ci-cd.yml` job `deploy`: `needs: build`, `if: push && ref == refs/heads/main && vars.DEPLOY_ENABLED == 'true'`, environment `prod`
- [ ] Deploy job: verificação prévia de secrets (falha cedo com mensagem clara) + autenticação AWS via OIDC (`aws-actions/configure-aws-credentials`) + deploy SAM/script para EC2 (free-tier), região `sa-east-1`
- [x] `.github/workflows/branch-naming.yml`: prefixos feature/, bugfix/, hotfix/, release/, chore/, docs/, test/, refactor/
- [x] `.github/workflows/auto-pr-to-main.yml`: PR automático develop→main
- [x] `sonar-project.properties` multi-module configurado
- [ ] Quality Gate: 0 bugs, 0 vulnerabilities, coverage ≥ 80%, duplicated lines < 3%
- [ ] Pipeline verde no primeiro push; deploy comprovado em merge para main

**Dependências Técnicas:** TASK-001 concluída

---

### TASK-005: Criar módulo Identity — usuários, roles e autenticação JWT

**Labels:** `priority: high`, `épico: identity`, `pontos: 8`, `type: feature`
**Milestone:** Sprint 2 - Identity
**Status:** 🟡 TODO

**Descrição:** Domínio: `User`, `Role`, `Permission`. Use cases: `RegisterUserUseCase`, `AuthenticateUseCase`,
`RefreshTokenUseCase`, `ChangePasswordUseCase`. Ports: `UserRepository`, `PasswordEncoder`,
`TokenProvider`. Adapters: JPA repository, BCrypt encoder, JWT provider (access + refresh tokens).

**Critérios de Aceitação:**
- [ ] Entidades: `User` (id, email, passwordHash, roles, status, createdAt), `Role` (name, permissions)
- [ ] Use cases com validações de domínio (email único, senha forte, account lockout)
- [ ] JWT: access token 15min, refresh token 7d, rotação de refresh token
- [ ] Spring Security filter chain com JWT authentication
- [ ] Endpoints REST: `POST /api/auth/register`, `POST /api/auth/login`, `POST /api/auth/refresh`, `POST /api/auth/logout`
- [ ] Testes unitários ≥ 85% cobertura
- [ ] Testes de integração com Testcontainers

**Dependências Técnicas:** TASK-002, TASK-003 concluídas

---

### TASK-006: Configurar Spring Security global, CORS, CSRF, Rate Limiting

**Labels:** `priority: high`, `épico: identity`, `pontos: 5`, `type: feature`
**Milestone:** Sprint 2 - Identity
**Status:** 🟡 TODO

**Descrição:** Configuração centralizada de segurança: CORS, CSRF (stateless), rate limiting por IP/usuário,
headers de segurança, auditoria de acesso.

**Critérios de Aceitação:**
- [ ] `SecurityConfig` com bean `SecurityFilterChain`
- [ ] CORS configurado para frontend Angular
- [ ] Rate limiting: 100 req/min por IP, 1000 req/min por usuário autenticado
- [ ] Security headers: HSTS, CSP, X-Frame-Options, X-Content-Type-Options
- [ ] Auditoria: log de login, logout, tentativas falhadas, mudanças de senha
- [ ] Exception handling: 401, 403, 429 com ProblemDetail RFC 7807

**Dependências Técnicas:** TASK-005 concluída

---

## 📋 ÉPICO 2: Market Data — Coleta, Processamento e Indicadores Técnicos

### TASK-007: Criar domínio Market Data — entidades, value objects e ports

**Labels:** `priority: high`, `épico: market-data`, `pontos: 8`, `type: feature`
**Milestone:** Sprint 3 - Market Data
**Status:** 🟡 TODO

**Descrição:** Entidades: `Cryptocurrency` (id, symbol, name, status, marketCapRank),
`MarketData` (cryptocurrency, price, volume, timestamp, interval, open/high/low/close),
`TechnicalIndicator` (cryptocurrency, rsi, macd, movingAverages, bollingerBands, volatility, calculatedAt).
Ports: `MarketDataRepository`, `CryptocurrencyRepository`, `TechnicalIndicatorRepository`,
`PriceFeedPort` (interface para provedores externos).

**Critérios de Aceitação:**
- [ ] Entidades JPA com mapeamento TimescaleDB (hypertables para MarketData, TechnicalIndicator)
- [ ] Value objects: `Price`, `Volume`, `Percentage`, `Timestamp`, `Interval` (1m, 5m, 15m, 1h, 4h, 1d)
- [ ] Enums: `Interval`, `DataSource` (BINANCE, COINGECKO, COINMARKETCAP)
- [ ] Ports como interfaces no `domain/port`
- [ ] Índices compostos para consultas temporais eficientes
- [ ] Particionamento por tempo (chunk_time_interval = 1 day)

**Dependências Técnicas:** TASK-002, TASK-003 concluídas

---

### TASK-008: Implementar adapters de coleta — Binance WebSocket + REST fallback

**Labels:** `priority: high`, `épico: market-data`, `pontos: 8`, `type: feature`
**Milestone:** Sprint 3 - Market Data
**Status:** 🟡 TODO

**Descrição:** Adapter `BinancePriceFeedAdapter` implementando `PriceFeedPort`. Conexão WebSocket
para streams de trade/kline em tempo real (BTC, ETH, SOL + configuráveis). REST fallback para
histórico e reconciliação. Circuit breaker, retry exponencial, dead letter queue para falhas.

**Critérios de Aceitação:**
- [ ] WebSocket client com reconexão automática e heartbeat
- [ ] Subscrição dinâmica de símbolos (configuração em runtime)
- [ ] Processamento de mensagens: trade (preço atual), kline (OHLCV por intervalo)
- [ ] Persistência assíncrona via `MarketDataRepository` (batch inserts)
- [ ] Métricas: latência, mensagens processadas, erros, reconexões
- [ ] Health check: última mensagem recebida < 30s
- [ ] Testes de integração com Binance testnet

**Dependências Técnicas:** TASK-007 concluída

---

### TASK-009: Implementar cálculo de indicadores técnicos

**Labels:** `priority: high`, `épico: market-data`, `pontos: 8`, `type: feature`
**Milestone:** Sprint 3 - Market Data
**Status:** 🟡 TODO

**Descrição:** Use case `CalculateIndicatorsUseCase` disparado por agendamento (EventBridge/Scheduler)
ou evento de novo candle. Indicadores: RSI (14), MACD (12,26,9), Médias Móveis (SMA/EMA 9,20,50,200),
Bollinger Bands (20,2), ATR (14), Volatilidade (desvio padrão 20 períodos). Armazenamento em
`TechnicalIndicator` hypertable.

**Critérios de Aceitação:**
- [ ] Cálculos puros em `domain/service/TechnicalAnalysisService` (sem framework)
- [ ] Janelas deslizantes eficientes (não recalcular tudo)
- [ ] Suporte a múltiplos intervalos simultâneos
- [ ] Idempotência: mesmo candle não recalculado
- [ ] Métricas de performance (tempo de cálculo por símbolo/intervalo)
- [ ] Testes unitários com dados conhecidos (valores de referência)

**Dependências Técnicas:** TASK-007, TASK-008 concluídas

---

### TASK-010: API REST e WebSocket para dados de mercado

**Labels:** `priority: high`, `épico: market-data`, `pontos: 5`, `type: feature`
**Milestone:** Sprint 3 - Market Data
**Status:** 🟡 TODO

**Descrição:** Controllers (adapters in): `MarketDataController` (REST) e `MarketDataWebSocketHandler`
(WebSocket). Endpoints: `GET /api/market-data/{symbol}/latest`, `GET /api/market-data/{symbol}/history`,
`GET /api/market-data/{symbol}/indicators`, `WS /api/market-data/stream`.

**Critérios de Aceitação:**
- [ ] REST: paginação, filtros por intervalo e período, compressão gzip
- [ ] WebSocket: subscrição por símbolos, broadcast de ticks/indicadores em tempo real
- [ ] Autenticação JWT obrigatória
- [ ] Rate limiting específico para market data
- [ ] Documentação OpenAPI/Swagger
- [ ] Testes de contrato e carga

**Dependências Técnicas:** TASK-008, TASK-009 concluídas

---

## 📋 ÉPICO 3: News Intelligence — Coleta, Análise e Classificação de Notícias

### TASK-011: Criar domínio News — entidades, ports e fontes configuráveis

**Labels:** `priority: high`, `épico: news-intelligence`, `pontos: 5`, `type: feature`
**Milestone:** Sprint 4 - News Intelligence
**Status:** 🟡 TODO

**Descrição:** Entidades: `NewsArticle` (title, summary, url, source, publishedAt, collectedAt,
relatedCryptos, category, sentiment, relevance, impact, reliability), `NewsSource` (name, url,
type, apiKey, config, enabled, reliability). Ports: `NewsRepository`, `NewsSourceRepository`,
`NewsCollectorPort` (interface para coletores).

**Critérios de Aceitação:**
- [ ] Entidades com índices para busca por crypto, data, sentimento
- [ ] Enum `NewsCategory`: REGULATORY, TECHNICAL, PARTNERSHIP, MARKET_MOVE, MACRO, EXCHANGE, PROJECT_UPDATE
- [ ] Value objects: `SentimentScore` (-100 a +100), `RelevanceScore` (0-100), `ImpactScore` (0-100)
- [ ] Configuração de fontes via YAML/DB (RSS, REST, GraphQL)
- [ ] Deduplicação por URL/hash de conteúdo

**Dependências Técnicas:** TASK-002, TASK-003 concluídas

---

### TASK-012: Implementar coletores de notícias — RSS, NewsAPI, CryptoPanic, CoinDesk

**Labels:** `priority: high`, `épico: news-intelligence`, `pontos: 8`, `type: feature`
**Milestone:** Sprint 4 - News Intelligence
**Status:** 🟡 TODO

**Descrição:** Adapters `RssNewsCollector`, `NewsApiCollector`, `CryptoPanicCollector`, `CoinDeskCollector`
implementando `NewsCollectorPort`. Agendamento via Spring Batch / Scheduler. Respeito a rate limits,
tratamento de erros, armazenamento bruto + processado.

**Critérios de Aceitação:**
- [ ] Coleta a cada 5-15 min configurável por fonte
- [ ] Extração de metadados: título, resumo, URL, data, autor, tags
- [ ] Filtro de criptomoedas mencionadas (NER simples + lookup de símbolos)
- [ ] Classificação de categoria via regras + ML (TASK-017)
- [ ] Armazenamento: raw HTML/JSON em S3/Blob, processado no PostgreSQL
- [ ] Métricas: artigos coletados, duplicados, erros por fonte

**Dependências Técnicas:** TASK-011 concluída

---

### TASK-013: Pipeline de análise de sentimento de notícias

**Labels:** `priority: high`, `épico: news-intelligence`, `pontos: 8`, `type: feature`
**Milestone:** Sprint 4 - News Intelligence
**Status:** 🟡 TODO

**Descrição:** Use case `AnalyzeNewsSentimentUseCase`. Integração com modelo de sentimento
(Bedrock, Ollama, ou transformers local). Classificação: VERY_POSITIVE, POSITIVE, NEUTRAL,
NEGATIVE, VERY_NEGATIVE. Score -100 a +100. Atualização de `NewsArticle.sentiment`.

**Critérios de Aceitação:**
- [ ] Processamento assíncrono via fila (RabbitMQ/Kafka/SQS)
- [ ] Suporte a múltiplos modelos (ensemble) com votação
- [ ] Cache de resultados para mesmo texto
- [ ] Métricas: acurácia vs. labels humanos (amostra), latência, throughput
- [ ] Reprocessamento sob demanda (quando modelo atualiza)
- [ ] Logs de auditoria: texto original, modelo usado, resultado, confidence

**Dependências Técnicas:** TASK-011, TASK-017 (modelo) concluídas

---

## 📋 ÉPICO 4: Social Intelligence — Monitoramento de Redes Sociais

### TASK-014: Criar domínio Social — entidades, ports e plataformas

**Labels:** `priority: high`, `épico: social-intelligence`, `pontos: 5`, `type: feature`
**Milestone:** Sprint 5 - Social Intelligence
**Status:** 🟡 TODO

**Descrição:** Entidades: `SocialMention` (platform, postId, authorId, content, timestamp,
sentiment, engagement, relevance, relatedCryptos), `SocialSource` (platform, account, config,
enabled). Ports: `SocialMentionRepository`, `SocialSourceRepository`, `SocialCollectorPort`.

**Critérios de Aceitação:**
- [ ] Platforms: X_TWITTER, REDDIT, TELEGRAM, DISCORD, YOUTUBE
- [ ] Value objects: `EngagementMetrics` (likes, retweets, replies, views), `AuthorProfile`
- [ ] Detecção de influenciadores (followers, engagement rate, historical accuracy)
- [ ] Deduplicação e spam detection

**Dependências Técnicas:** TASK-002, TASK-003 concluídas

---

### TASK-015: Implementar coletores sociais — X API v2, Reddit API, Pushshift

**Labels:** `priority: high`, `épico: social-intelligence`, `pontos: 8`, `type: feature`
**Milestone:** Sprint 5 - Social Intelligence
**Status:** 🟡 TODO

**Descrição:** Adapters para X (Twitter API v2 filtered stream), Reddit (Pushshift + PRAW),
Telegram (Bot API). Busca por símbolos ($BTC, #BTC, "Bitcoin"), contas oficiais, palavras-chave.
Respeito a rate limits e ToS.

**Critérios de Aceitação:**
- [ ] X: filtered stream com regras dinâmicas, full-archive search para histórico
- [ ] Reddit: subreddits relevantes (r/Bitcoin, r/CryptoCurrency, r/ethereum, etc.)
- [ ] Filtro de idioma (PT, EN prioritários)
- [ ] Armazenamento bruto + processado
- [ ] Métricas: volume de menções/hora, crescimento anômalo (z-score > 3)
- [ ] Alertas de spike de menções negativas/positivas

**Dependências Técnicas:** TASK-014 concluída

---

### TASK-016: Análise de sentimento social e detecção de anomalias

**Labels:** `priority: high`, `épico: social-intelligence`, `pontos: 8`, `type: feature`
**Milestone:** Sprint 5 - Social Intelligence
**Status:** 🟡 TODO

**Descrição:** Use case `AnalyzeSocialSentimentUseCase`. Agregação por janela temporal (5m, 1h, 24h):
volume, sentimento médio, tópicos (LDA/bertopic), influenciadores ativos. Detecção de anomalias:
spike de volume, mudança brusca de sentimento, coordenação inautêntica (bot detection heurístico).

**Critérios de Aceitação:**
- [ ] Agregação incremental (não reprocessar tudo)
- [ ] Correlação com market data (TASK-009): preço + volume + sentimento social
- [ ] Alertas: "BTC: menções negativas +400% em 1h, preço -5%, volume +200%"
- [ ] Dashboard de social sentiment por crypto
- [ ] Métricas de qualidade: precision/recall do bot detection

**Dependências Técnicas:** TASK-014, TASK-015, TASK-017 concluídas

---

## 📋 ÉPICO 5: AI Engine — Modelos, Previsão e Inteligência Evolutiva

### TASK-017: Configurar infraestrutura de ML — Registry, Feature Store, Pipeline

**Labels:** `priority: high`, `épico: ai-engine`, `pontos: 8`, `type: infra`
**Milestone:** Sprint 6 - AI Engine
**Status:** 🟡 TODO

**Descrição:** Infraestrutura base para ML: MLflow/Model Registry para versionamento,
Feature Store (Feast ou tabelas PostgreSQL), Pipeline de treino (Airflow/Prefect ou Spring Batch).
Integração Bedrock (Claude, Titan) e Ollama local.

**Critérios de Aceitação:**
- [ ] MLflow tracking server (local docker, prod AWS)
- [ ] Model Registry: staging → production promotion gates
- [ ] Feature Store: features de mercado, técnico, sentimento, macro
- [ ] Pipeline: data preparation → training → evaluation → registration
- [ ] Suporte a modelos: XGBoost, LightGBM, Random Forest, LSTM, Transformer
- [ ] Artefatos versionados no S3/MinIO

**Dependências Técnicas:** TASK-003 concluída

---

### TASK-018: Implementar modelos de previsão — classificação de tendência e probabilidade

**Labels:** `priority: high`, `épico: ai-engine`, `pontos: 13`, `type: feature`
**Milestone:** Sprint 6 - AI Engine
**Status:** 🟡 TODO

**Descrição:** Modelos supervisionados para: (1) Classificação de tendência 24h/7d: BEARISH/NEUTRAL/BULLISH,
(2) Probabilidade de valorização > X% em horizonte H, (3) Estimativa de risco (VaR, CVaR),
(4) Detecção de anomalia (isolation forest). Features: OHLCV, indicadores, sentimento notícias/social,
eventos macro, on-chain metrics. Treino walk-forward, validação out-of-sample.

**Critérios de Aceitação:**
- [ ] Dataset: features + labels (retorno futuro) sem look-ahead bias
- [ ] Modelos baseline: XGBoost, Random Forest, Logistic Regression
- [ ] Métricas: accuracy, precision, recall, F1, AUC-ROC, Brier score (calibração)
- [ ] Backtesting de previsões: PnL teórico, Sharpe, max drawdown
- [ ] Model cards documentando: data, features, hiperparâmetros, métricas, limitações
- [ ] A/B testing framework para comparar versões

**Dependências Técnicas:** TASK-009, TASK-013, TASK-016, TASK-017 concluídas

---

### TASK-019: Implementar análise contextual e geração de relatórios IA

**Labels:** `priority: high`, `épico: ai-engine`, `pontos: 8`, `type: feature`
**Milestone:** Sprint 6 - AI Engine
**Status:** 🟡 TODO

**Descrição:** Use case `GenerateMarketAnalysisUseCase`. LLM (Bedrock/Ollama) recebe contexto
estruturado: preços, indicadores, notícias recentes, sentimento social, eventos, previsões dos
modelos ML. Gera análise em linguagem natural: tendência, riscos, oportunidades, probabilidades,
justificativa. Armazenamento em `AIAnalysis` com versão do modelo.

**Critérios de Aceitação:**
- [ ] Prompt engineering com few-shot examples validados
- [ ] Structured output (JSON schema) para parsing confiável
- [ ] Template de análise: Resumo → Contexto → Indicadores → Externos → Previsão → Recomendação
- [ ] Probabilidades sempre como estimativas (nunca certezas)
- [ ] Rastreabilidade: cada análise linka aos dados/fatos usados
- [ ] Avaliação humana contínua (feedback loop)

**Dependências Técnicas:** TASK-018, TASK-017 concluídas

---

### TASK-020: Inteligência evolutiva — registro, avaliação e promoção de modelos

**Labels:** `priority: high`, `épico: ai-engine`, `pontos: 8`, `type: feature`
**Milestone:** Sprint 7 - Evolução Inteligente
**Status:** 🟡 TODO

**Descrição:** Sistema de aprendizado contínuo: (1) Registrar previsões e decisões com contexto,
(2) Comparar previsão vs. realizado (horizonte 1h, 4h, 24h, 7d), (3) Avaliar estratégias por
regime de mercado (bull, bear, sideways, high vol), (4) Retreino agendado + validação champion/challenger,
(5) Rollback automático se nova versão degrada métricas-chave.

**Critérios de Aceitação:**
- [ ] Tabelas: `ModelPrediction`, `ModelEvaluation`, `StrategyPerformance`, `MarketRegime`
- [ ] Métricas por regime: win rate, avg win/loss, profit factor, expectancy
- [ ] Pipeline de retreino: weekly/monthly com dados até T-1
- [ ] Critérios de promoção: AUC > champion + 1%, Brier score melhor, estabilidade cross-regime
- [ ] Rollback: 1-click para versão anterior
- [ ] Auditoria completa: quem promoveu, quando, métricas antes/depois

**Dependências Técnicas:** TASK-018, TASK-019 concluídas

---

## 📋 ÉPICO 6: Calendar & Events — Eventos Macroeconômicos e On-Chain

### TASK-021: Criar domínio Events — calendário, impacto e correlação histórica

**Labels:** `priority: medium`, `épico: events`, `pontos: 5`, `type: feature`
**Milestone:** Sprint 7 - Events
**Status:** 🟡 TODO

**Descrição:** Entidades: `MarketEvent` (type, title, description, scheduledAt, actualAt,
relatedCryptos, estimatedImpact, actualImpact, source), `EventSource` (FRED, ECB, FOMC,
CryptoCalendar, BlockchainExplorer). Ports: `EventRepository`, `EventCollectorPort`.

**Critérios de Aceitação:**
- [ ] Tipos: RATE_DECISION, CPI, NFP, GDP, ETF_APPROVAL, HALVING, FORK, REGULATORY, EXCHANGE_EVENT
- [ ] Coleta automática: FRED API, calendários crypto, blockchain explorers
- [ ] Correlação histórica: buscar eventos similares ±7 dias e analisar retorno anormal
- [ ] Score de impacto estimado (0-100) baseado em histórico
- [ ] Alertas: evento em 24h/1h com impacto esperado

**Dependências Técnicas:** TASK-003, TASK-007 concluídas

---

## 📋 ÉPICO 7: Strategy Engine — Gestão e Execução de Estratégias

### TASK-022: Criar domínio Strategy — entidades, parâmetros e validação

**Labels:** `priority: high`, `épico: strategy-engine`, `pontos: 8`, `type: feature`
**Milestone:** Sprint 8 - Strategy Engine
**Status:** 🟡 TODO

**Descrição:** Entidades: `TradingStrategy` (name, description, version, status, parameters,
indicators, rules, riskParams), `StrategyParameter` (name, type, default, min, max, description),
`Signal` (strategy, symbol, action, strength, timestamp, metadata). Ports: `StrategyRepository`,
`SignalRepository`, `StrategyValidatorPort`.

**Critérios de Aceitação:**
- [ ] Tipos de estratégia: TREND_FOLLOWING, MEAN_REVERSION, BREAKOUT, SENTIMENT_DRIVEN, HYBRID
- [ ] Parâmetros tipados: numeric, boolean, enum, expression (SpEL/Janino)
- [ ] Regras como expressões avaliáveis: "RSI < 30 AND MACD_SIGNAL > MACD_HIST"
- [ ] Versionamento semântico de estratégias
- [ ] Validação: parâmetros no range, regras sintaticamente corretas, sem look-ahead

**Dependências Técnicas:** TASK-007, TASK-009 concluídas

---

### TASK-023: Implementar motor de sinais — avaliação de regras em tempo real

**Labels:** `priority: high`, `épico: strategy-engine`, `pontos: 8`, `type: feature`
**Milestone:** Sprint 8 - Strategy Engine
**Status:** 🟡 TODO

**Descrição:** Use case `EvaluateStrategiesUseCase` disparado a cada candle (1m/5m/15m). Para cada
estratégia ativa: carrega parâmetros, calcula indicadores necessários, avalia regras, emite `Signal`
se condições atendidas. Deduplicação de sinais (mesma estratégia/símbolo/direção em janela).

**Critérios de Aceitação:**
- [ ] Avaliação assíncrona, não bloqueante
- [ ] Cache de indicadores calculados (TASK-009)
- [ ] Filtros: regime de mercado, volatilidade, liquidez, horário
- [ ] Métricas: sinais gerados, taxa de conversão em operação, latência
- [ ] Testes: replay histórico de sinais vs. backtest

**Dependências Técnicas:** TASK-022, TASK-009 concluídas

---

## 📋 ÉPICO 8: Risk Management — Gestão de Risco Rigorosa

### TASK-024: Criar domínio Risk — limites, regras e validação pré-trade

**Labels:** `priority: high`, `épico: risk-management`, `pontos: 8`, `type: feature`
**Milestone:** Sprint 9 - Risk Management
**Status:** 🟡 TODO

**Descrição:** Entidades: `RiskLimit` (type, value, scope: PORTFOLIO/ASSET/OPERATION), `RiskCheckResult`
(passed, violatedLimits, warnings). Ports: `RiskLimitRepository`, `PortfolioRiskPort`,
`PreTradeRiskValidatorPort`. Regras: Stop Loss, Take Profit, Max Daily Loss, Max Exposure/Asset,
Max Capital/Trade, Max Concurrent Trades, Volatility Circuit Breaker, Correlation Limit.

**Critérios de Aceitação:**
- [ ] Limites configuráveis por perfil de risco (CONSERVATIVE, MODERATE, AGGRESSIVE)
- [ ] Validação pré-trade síncrona (< 10ms): rejeita se qualquer limite violado
- [ ] Stop Loss / Take Profit: fixo, trailing, baseado em ATR
- [ ] Daily Loss Limit: bloqueio automático de novas posições se atingido
- [ ] Circuit Breaker: volatilidade intraday > 3σ bloqueia novas entradas
- [ ] Auditoria: todo check logado com detalhes

**Dependências Técnicas:** TASK-005 (portfolio), TASK-007 concluídas

---

### TASK-025: Implementar monitoramento de risco em tempo real

**Labels:** `priority: high`, `épico: risk-management`, `pontos: 5`, `type: feature`
**Milestone:** Sprint 9 - Risk Management
**Status:** 🟡 TODO

**Descrição:** Use case `MonitorPortfolioRiskUseCase` rodando a cada 10s-1min. Cálculo: PnL não
realizado, exposição por ativo/setor, VaR paramétrico/histórico, drawdown atual, correlação
entre posições. Alertas: breach de limite, margem baixa, concentração excessiva.

**Critérios de Aceitação:**
- [ ] Dashboard de risco em tempo real
- [ ] Auto-liquidação opcional se limite crítico violado (configurável)
- [ ] Relatórios de risco: diário, semanal, mensal
- [ ] Stress testing: cenários históricos (Crash 2020, Terra/Luna, FTX)

**Dependências Técnicas:** TASK-024 concluída

---

## 📋 ÉPICO 9: Trading Engine — Paper Trading e Execução Simulada

### TASK-026: Criar domínio Trading — operações, portfolio e execução simulada

**Labels:** `priority: high`, `épico: trading-engine`, `pontos: 8`, `type: feature`
**Milestone:** Sprint 10 - Trading Engine
**Status:** 🟡 TODO

**Descrição:** Entidades: `PaperPortfolio` (initialCapital, availableCash, positions, totalEquity,
realizedPnl, unrealizedPnl), `PaperPosition` (symbol, side, quantity, entryPrice, currentPrice,
stopLoss, takeProfit, fees, pnl), `TradingOperation` (id, portfolio, symbol, type, side, price,
quantity, fees, slippage, status, openedAt, closedAt, strategy, aiAnalysisId). Ports:
`PaperPortfolioRepository`, `OrderExecutionPort` (simulado), `MarketDataSnapshotPort`.

**Critérios de Aceitação:**
- [ ] Simulação realista: spread (bid/ask), slippage (modelo linear + raiz quadrada do volume),
  comissão (maker/taker), funding rate (perp)
- [ ] Tipos de ordem: MARKET, LIMIT, STOP_MARKET, STOP_LIMIT, TRAILING_STOP
- [ ] Gestão de margem (se futuros): inicial, manutenção, liquidação
- [ ] Reconciliação diária com preços de fechamento
- [ ] Métricas: Sharpe, Sortino, Calmar, Max DD, Win Rate, Profit Factor, Expectancy

**Dependências Técnicas:** TASK-024, TASK-010 (market data snapshot) concluídas

---

### TASK-027: Implementar motor de execução paper trading — fluxo completo

**Labels:** `priority: high`, `épico: trading-engine`, `pontos: 8`, `type: feature`
**Milestone:** Sprint 10 - Trading Engine
**Status:** 🟡 TODO

**Descrição:** Use case `ExecutePaperTradeUseCase`: recebe `Signal` → valida risco (TASK-024)
→ calcula tamanho posição (Kelly, fixed fractional, risk parity) → simula execução com
slippage/spread → cria `TradingOperation` e `PaperPosition` → monitora SL/TP → fecha posição.
Suporte a múltiplas estratégias simultâneas por portfolio.

**Critérios de Aceitação:**
- [ ] Sizing: % do capital, risco fixo por trade, Kelly criterion
- [ ] Partial fills, rejects, expirations simulados
- [ ] Trailing stop update a cada tick favorável
- [ ] Fechamento: SL/TP hit, sinal contrário, fim de sessão, risk limit
- [ ] Journal: toda operação com justificativa (signal + AI analysis + risk check)
- [ ] Múltiplos portfolios isolados para comparar estratégias

**Dependências Técnicas:** TASK-026, TASK-024 concluídas

---

## 📋 ÉPICO 10: Backtesting — Validação Histórica de Estratégias

### TASK-028: Criar framework de backtesting — engine, dados, métricas

**Labels:** `priority: high`, `épico: backtesting`, `pontos: 13`, `type: feature`
**Milestone:** Sprint 11 - Backtesting
**Status:** 🟡 TODO

**Descrição:** Engine de backtesting event-driven: replay de dados históricos (MarketData +
Indicators + News/Social/Events) tick-a-tick ou candle-a-candle. Prevenção rigorosa de look-ahead
bias: apenas dados ≤ timestamp atual disponíveis. Custos: spread, slippage, comissão, funding.

**Critérios de Aceitação:**
- [ ] Data loader: carrega hypertables TimescaleDB por símbolo/intervalo/período
- [ ] Event loop: para cada timestamp, disponibiliza snapshot → estratégia avalia → executa
- [ ] Walk-forward optimization: treino janela W, teste janela T, roll forward
- [ ] Métricas completas: retorno total, CAGR, vol, Sharpe, Sortino, Calmar, Max DD, DD duração,
  Win Rate, Avg Win/Loss, Profit Factor, Expectancy, Ulcer Index
- [ ] Benchmark: Buy & Hold, Equal Weight
- [ ] Relatórios: equity curve, drawdown chart, monthly returns heatmap, trade list
- [ ] Validação out-of-sample obrigatória antes de aprovar estratégia

**Dependências Técnicas:** TASK-007, TASK-009, TASK-022, TASK-026 concluídas

---

### TASK-029: Interface de backtesting — configuração, execução, comparação

**Labels:** `priority: medium`, `épico: backtesting`, `pontos: 5`, `type: feature`
**Milestone:** Sprint 11 - Backtesting
**Status:** 🟡 TODO

**Descrição:** API e UI para: selecionar símbolo(s), período, estratégia, parâmetros, custos.
Executar backtest (async, job com progresso). Comparar múltiplos runs (parâmetros, estratégias).
Exportar resultados (JSON, CSV, PDF report).

**Critérios de Aceitação:**
- [ ] Formulário: símbolo, data início/fim, intervalo, estratégia, parâmetros, custos
- [ ] Job assíncrono com WebSocket progress
- [ ] Tabela comparativa: métricas lado a lado, destacar melhor
- [ ] Visualização: equity curve overlay, drawdown overlay
- [ ] Persistência de runs para auditoria
- [ ] API para integração com otimização automática (optuna/hyperopt)

**Dependências Técnicas:** TASK-028 concluída

---

## 📋 ÉPICO 11: Portfolio & Reporting — Gestão e Relatórios

### TASK-030: Implementar gestão de portfolio — multi-estratégia, rebalanceamento

**Labels:** `priority: high`, `épico: portfolio`, `pontos: 8`, `type: feature`
**Milestone:** Sprint 12 - Portfolio
**Status:** 🟡 TODO

**Descrição:** Portfolio agrega múltiplas estratégias com alocação de capital por estratégia.
Rebalanceamento: periódico (diário/semanal), baseado em risco (risk parity), ou gatilho (drawdown).
Relatórios: performance por estratégia, atribuição de retorno, risk metrics.

**Critérios de Aceitação:**
- [ ] Alocação: % fixo, risk parity, Kelly, voltatilidade alvo
- [ ] Rebalance: ordens de ajuste simuladas, custos considerados
- [ ] Atribuição: Brinson-Hood-Beebower (alocação + seleção + interação)
- [ ] Benchmark personalizável
- [ ] API: `GET /api/portfolio/{id}/performance`, `GET /api/portfolio/{id}/attribution`

**Dependências Técnicas:** TASK-027 concluída

---

### TASK-031: Relatórios automatizados e dashboards

**Labels:** `priority: medium`, `épico: reporting`, `pontos: 5`, `type: feature`
**Milestone:** Sprint 12 - Portfolio
**Status:** 🟡 TODO

**Descrição:** Geração de relatórios: diário (EOD), semanal, mensal. Formatos: PDF, HTML, JSON.
Dashboards: equity curve, posições atuais, risk metrics, strategy breakdown, AI analysis history.
Agendamento via Spring Batch / Quartz.

**Critérios de Aceitação:**
- [ ] Template de relatório configurável
- [ ] Envio por email (SES/SMTP) e/ou webhook
- [ ] Dashboard dados via REST + WebSocket updates
- [ ] Exportação para Excel/CSV

**Dependências Técnicas:** TASK-030 concluída

---

## 📋 ÉPICO 12: Notification & Alerts — Alertas Inteligentes

### TASK-032: Sistema de notificações — canais, templates, preferências

**Labels:** `priority: medium`, `épico: notification`, `pontos: 5`, `type: feature`
**Milestone:** Sprint 13 - Notification
**Status:** 🟡 TODO

**Descrição:** Canais: Email (SES), Telegram Bot, Discord Webhook, Push (Firebase), Webhook
genérico. Templates por evento: sinal gerado, operação aberta/fechada, SL/TP hit, risk alert,
AI analysis, daily summary. Preferências por usuário/portfolio.

**Critérios de Aceitação:**
- [ ] Fila de notificações com retry e DLQ
- [ ] Deduplicação: não spammar mesmo alerta em janela curta
- [ ] Rate limiting por canal
- [ ] Templates Thymeleaf/Handlebars com variáveis
- [ ] Log de entrega: status, timestamp, response

**Dependências Técnicas:** TASK-005 concluída

---

## 📋 ÉPICO 13: Infraestrutura AWS — Deploy, Observabilidade, Segurança (Free-tier First)

### TASK-039: SAM/CloudFormation — stacks por ambiente, recursos (Free-tier Optimized)

**Labels:** `priority: high`, `épico: infra-aws`, `pontos: 8`, `type: infra`
**Milestone:** Sprint 17 - Infra AWS
**Status:** 🟡 TODO

**Descrição:** Template SAM otimizado para free-tier AWS. Exceção: **RDS PostgreSQL + TimescaleDB** (pago, necessário). Demais serviços no free-tier ou self-hosted via EC2 t3.micro (free-tier): ECS Fargate → EC2 t3.micro + Docker Compose; ElastiCache Redis → Redis local Docker; ALB → Nginx/Traefik no EC2; API Gateway WebSocket → WebSocket direto no EC2; EventBridge → Spring Scheduler/Quartz; SNS/SQS → RabbitMQ local; S3 → MinIO local (ou S3 5GB free); Secrets Manager → .env / Parameter Store (free); CloudWatch → Prometheus + Grafana local; WAF → Nginx rate limit + Cloudflare free.

**Critérios de Aceitação:**
- [ ] `infra/template.yaml` com RDS PostgreSQL + TimescaleDB (único recurso pago obrigatório)
- [ ] EC2 t3.micro (free-tier 750h/mês) para backend + Docker Compose
- [ ] Nginx/Traefik como reverse proxy + TLS (Let's Encrypt)
- [ ] Redis, RabbitMQ, MinIO, Prometheus, Grafana via Docker Compose
- [ ] `infra/samconfig.toml` dev/staging/prod
- [ ] Parâmetros: Environment, AdminEmail, Domain, CertificateArn, DBInstanceClass (db.t3.micro free-tier)
- [ ] `sam validate --lint` passa
- [ ] Custo estimado: ~$15-25/mês (apenas RDS db.t3.micro Multi-AZ off)

**Dependências Técnicas:** TASK-001, TASK-003 concluídas

---

### TASK-040: Observabilidade — logs estruturados, métricas, tracing, alertas (Free-tier)

**Labels:** `priority: high`, `épico: infra-aws`, `pontos: 5`, `type: infra`
**Milestone:** Sprint 17 - Infra AWS
**Status:** 🟡 TODO

**Descrição:** OpenTelemetry instrumentation (Spring Boot 3 + Micrometer). Logs JSON com correlationId. Métricas: Prometheus endpoint + Grafana (self-hosted no EC2). Alertas: Alertmanager + ntfy.sh/Telegram/email SMTP (grátis). Sem CloudWatch pago, sem X-Ray pago.

**Critérios de Aceitação:**
- [ ] `otel-javaagent` configurado exportando para Prometheus (localhost:9090)
- [ ] Logs: correlationId, traceId, spanId, service.name, environment (stdout → Loki/Promtail)
- [ ] Métricas: Prometheus + Grafana dashboards (business + technical)
- [ ] Traces: Tempo local ou Jaeger local (Docker)
- [ ] Alertas: Alertmanager → ntfy.sh/Telegram Bot/email (grátis)
- [ ] Dashboard único no Grafana: error rate, latency p99, queue lag, daily loss limit

**Dependências Técnicas:** TASK-039 concluída

---

### TASK-041: Segurança — WAF, Secrets, IAM, criptografia, compliance (Free-tier)

**Labels:** `priority: high`, `épico: infra-aws`, `pontos: 5`, `type: infra`
**Milestone:** Sprint 17 - Infra AWS
**Status:** 🟡 TODO

**Descrição:** Segurança maximizando free-tier: WAF → Nginx rate limit + Cloudflare free (DDoS, WAF básico); Secrets → AWS Parameter Store (free tier 10k params) ou .env no EC2; IAM: roles least privilege, sem Access Analyzer pago; KMS → 1 CMK (free tier 20k requests/mês); VPC: default VPC + security groups (grátis); VPC Flow Logs → CloudWatch Logs free tier (5GB); Security Hub → desabilitado (pago).

**Critérios de Aceitação:**
- [ ] Nginx rate limiting (req/min por IP/usuario) + Cloudflare free WAF
- [ ] Secrets: Parameter Store (SecureString) para DB password, API keys, JWT secret
- [ ] IAM: roles por serviço, condition keys, boundaries (grátis)
- [ ] KMS: 1 CMK por ambiente (free tier)
- [ ] VPC Flow Logs → CloudWatch Logs (5GB free)
- [ ] Security groups restritivos, sem NAT Gateway (usar VPC endpoints S3/Dynamo se necessário)

**Dependências Técnicas:** TASK-039 concluída


---

## 📋 ÉPICO 15: Preparação para Trading Real (Futuro)

### TASK-042: Integração com corretoras — adapters de execução real

**Labels:** `priority: low`, `épico: live-trading-prep`, `pontos: 13`, `type: feature`
**Milestone:** Fase 8 - Preparação Trading Real (Futuro)
**Status:** 🟡 TODO

**Descrição:** Adapters `BinanceExecutionAdapter`, `BybitExecutionAdapter`, `CoinbaseExecutionAdapter`
implementando `OrderExecutionPort` (mesma interface do paper trading). Funcionalidades: spot,
futures, margin. Rate limits, assinaturas, websocket para order updates. **Desabilitado por padrão.**

**Critérios de Aceitação:**
- [ ] Interface idêntica ao paper trading (swap por config)
- [ ] Credenciais em Secrets Manager, permissões read/trade apenas (no withdrawal)
- [ ] Sandbox/testnet obrigatório antes de prod
- [ ] Reconciliação de posições a cada 1min
- [ ] Kill switch: variável de ambiente `LIVE_TRADING_ENABLED=false` por padrão
- [ ] Auditoria completa: cada ordem com request/response assinado

**Dependências Técnicas:** TASK-027, TASK-024 concluídas

---

### TASK-043: Governança e aprovação para live trading

**Labels:** `priority: low`, `épico: live-trading-prep`, `pontos: 5`, `type: feature`
**Milestone:** Fase 8 - Preparação Trading Real (Futuro)
**Status:** 🟡 TODO

**Descrição:** Fluxo de aprovação: estratégia deve passar backtesting (out-of-sample) + paper
trading (mínimo 30 dias) + risk review + aprovação manual do admin. Checklist obrigatório.
Rollback automático se métricas degradam.

**Critérios de Aceitação:**
- [ ] Workflow: strategy → backtest report → paper trading report → risk review → admin approval
- [ ] Checklist: max DD < X%, Sharpe > Y, win rate > Z%, no risk limit breaches
- [ ] Live trading só habilitado com feature flag + assinatura digital do admin
- [ ] Circuit breaker: auto-disable se daily loss > limite ou drawdown > limite

**Dependências Técnicas:** TASK-042, TASK-028, TASK-027 concluídas

---

## 📋 RESUMO DO BACKLOG

**Total de Tasks:** 37
**Estimativa Total:** ~320 pontos

### Por Prioridade:
- **Alta:** 26 tasks
- **Média:** 7 tasks
- **Baixa:** 4 tasks

### Por Épico:
1. Fundação (Setup, Identity): 6 tasks (41 pts)
2. Market Data: 4 tasks (29 pts)
3. News Intelligence: 3 tasks (21 pts)
4. Social Intelligence: 3 tasks (21 pts)
5. AI Engine: 4 tasks (37 pts)
6. Events: 1 task (5 pts)
7. Strategy Engine: 2 tasks (16 pts)
8. Risk Management: 2 tasks (13 pts)
9. Trading Engine (Paper): 2 tasks (16 pts)
10. Backtesting: 2 tasks (18 pts)
11. Portfolio & Reporting: 2 tasks (13 pts)
12. Notification: 1 task (5 pts)
13. Infra AWS (Free-tier First): 3 tasks (18 pts)
14. Live Trading Prep (Future): 2 tasks (18 pts)

> **Nota:** Épico Frontend Angular (6 tasks, 45 pts) removido do escopo atual — será desenvolvido em fase posterior. Épico Infra AWS reescrito com estratégia free-tier-first (RDS PostgreSQL + TimescaleDB como única exceção paga obrigatória).

---

## 🎯 ORDEM SUGERIDA DE EXECUÇÃO (SPRINTS)

**Sprint 1-2 - Fundação & Identity** (TASK-001 a TASK-006)
- Setup multi-module, hexagonal, DB, CI/CD, Auth/JWT, Security

**Sprint 3 - Market Data** (TASK-007 a TASK-010)
- Domínio, coleta Binance WS/REST, indicadores, API/WS

**Sprint 4 - News Intelligence** (TASK-011 a TASK-013)
- Domínio, coletores, sentimento

**Sprint 5 - Social Intelligence** (TASK-014 a TASK-016)
- Domínio, coletores X/Reddit, sentimento + anomalias

**Sprint 6 - AI Engine (Core)** (TASK-017 a TASK-019)
- ML infra, modelos previsão, análise contextual LLM

**Sprint 7 - Events & Evolução** (TASK-020, TASK-021)
- Inteligência evolutiva, calendário eventos

**Sprint 8-9 - Strategy & Risk** (TASK-022 a TASK-025)
- Estratégias, sinais, risk management

**Sprint 10 - Trading Engine (Paper)** (TASK-026, TASK-027)
- Portfolio paper, execução simulada completa

**Sprint 11 - Backtesting** (TASK-028, TASK-029)
- Engine event-driven, validação histórica

**Sprint 12 - Portfolio & Reporting** (TASK-030, TASK-031)
- Multi-estratégia, relatórios

**Sprint 13 - Notification** (TASK-032)
- Alertas multi-canal

**Sprint 17 - Infra AWS (Free-tier First)** (TASK-039 a TASK-041)
- EC2 t3.micro + Docker Compose, RDS (único pago), Prometheus/Grafana, Nginx + Cloudflare free

**Fase 8 - Live Trading Prep** (TASK-042, TASK-043)
- Adapters corretoras, governança (futuro, desabilitado)

---

## 📝 NOTAS ARQUITETURAIS

### Stack Tecnológica
- **Backend:** Spring Boot 3.3+, Java 21, Maven multi-module
- **Frontend:** (removido do escopo atual — será desenvolvido em fase posterior)
- **Database:** PostgreSQL 16 + TimescaleDB (hypertables) via RDS (free tier db.t3.micro), Redis 7 (Docker local)
- **Messaging:** RabbitMQ (Docker local, grátis)
- **ML:** MLflow (Docker local), XGBoost/LightGBM, Ollama (local, grátis) / Bedrock (opcional pago)
- **Infra:** AWS EC2 t3.micro (free-tier 750h/mês) + Docker Compose, RDS PostgreSQL (único recurso pago obrigatório), Nginx + Cloudflare free
- **Observability:** OpenTelemetry, Prometheus + Grafana (Docker local), Alertmanager + ntfy.sh (grátis)

### Arquitetura Hexagonal por Módulo
```
module/
├── domain/
│   ├── entity/           # Entidades ricas, invariantes
│   ├── valueobject/      # VOs imutáveis
│   ├── port/             # Interfaces (driven ports)
│   └── service/          # Serviços de domínio puros
├── application/
│   ├── usecase/          # Casos de uso (orchestration)
│   ├── port/             # Interfaces (driver ports)
│   └── service/          # Serviços de aplicação
└── infra/
    ├── adapter/
    │   ├── in/           # Drivers: REST, WebSocket, CLI, Scheduler
    │   └── out/          # Driven: JPA, External API, Messaging, Cache
    └── config/           # Configuration, beans
```

### Princípios
- **Paper trading only** — live trading desabilitado por feature flag
- **Rastreabilidade total** — toda decisão linka: signal → AI analysis → market data → execution
- **Look-ahead bias prevention** — backtesting engine event-driven com snapshot temporal
- **Model governance** — versionamento, champion/challenger, rollback 1-click
- **Risk first** — validação pré-trade síncrona, circuit breakers, kill switches
- **Evolução contínua** — pipeline de retreino, avaliação por regime, feedback loop humano

---

**Documento versão 1.0** — Baseado no Prompt Master "Crypto Intelligence Simulator" e estrutura de referência FIAP Fase 4.