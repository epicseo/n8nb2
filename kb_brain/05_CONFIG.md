# 05_CONFIG.md
<!-- repo: n8n | branch: claude/github-to-kb-converter-01JAurUearApTk4RAu9NPFAg | commit: 3c0e809e | generated: 2025-11-28 -->
<!-- tags: configuration, environment, variables, deployment, docker -->

## Contents
- [Global Configuration](#global-configuration)
- [Database Configuration](#database-configuration)
- [Execution Configuration](#execution-configuration)
- [Security Configuration](#security-configuration)
- [Logging Configuration](#logging-configuration)
- [Queue/Scaling Configuration](#queuescaling-configuration)
- [Build Configuration](#build-configuration)

---

## Global Configuration
<!-- chunk: 05-global | keywords: global, server, port, host | source: packages/@n8n/config/src/ -->

### Server Settings

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `N8N_HOST` | string | `localhost` | Host name n8n can be reached |
| `N8N_PORT` | number | `5678` | HTTP port n8n listens on |
| `N8N_LISTEN_ADDRESS` | string | `::` | IP address to listen on |
| `N8N_PROTOCOL` | enum | `http` | Protocol: `http` or `https` |
| `N8N_PATH` | string | `/` | Path n8n is deployed to |
| `N8N_EDITOR_BASE_URL` | string | | Public URL for editor |
| `N8N_PROXY_HOPS` | number | `0` | Number of reverse proxies |

### SSL Settings

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `N8N_SSL_KEY` | string | | SSL key file path |
| `N8N_SSL_CERT` | string | | SSL certificate file path |

### General Settings

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `GENERIC_TIMEZONE` | string | `America/New_York` | Default timezone |
| `N8N_DEFAULT_LOCALE` | string | `en` | Default UI locale |
| `N8N_GRACEFUL_SHUTDOWN_TIMEOUT` | number | `30` | Shutdown grace period (seconds) |
| `N8N_DISABLE_UI` | boolean | `false` | Disable frontend UI |
| `N8N_ENCRYPTION_KEY` | string | | Encryption key for credentials |

---

## Database Configuration
<!-- chunk: 05-database | keywords: database, sqlite, postgres, mysql | source: packages/@n8n/config/src/configs/database.config.ts -->

### Database Type

| Variable | Type | Default | Options |
|----------|------|---------|---------|
| `DB_TYPE` | enum | `sqlite` | `sqlite`, `postgresdb`, `mysqldb`, `mariadb` |
| `DB_TABLE_PREFIX` | string | | Table name prefix |
| `DB_PING_INTERVAL_SECONDS` | number | `2` | Database ping interval |

### SQLite Configuration

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `DB_SQLITE_DATABASE` | string | `database.sqlite` | Database file name |
| `DB_SQLITE_POOL_SIZE` | number | `0` | Pool size (0=disabled) |
| `DB_SQLITE_ENABLE_WAL` | boolean | `poolSize > 1` | Enable WAL mode |
| `DB_SQLITE_VACUUM_ON_STARTUP` | boolean | `false` | Run VACUUM on startup |

### PostgreSQL Configuration

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `DB_POSTGRESDB_HOST` | string | `localhost` | Database host |
| `DB_POSTGRESDB_PORT` | number | `5432` | Database port |
| `DB_POSTGRESDB_DATABASE` | string | `n8n` | Database name |
| `DB_POSTGRESDB_USER` | string | `postgres` | Username |
| `DB_POSTGRESDB_PASSWORD` | string | | Password |
| `DB_POSTGRESDB_SCHEMA` | string | `public` | Schema |
| `DB_POSTGRESDB_POOL_SIZE` | number | `2` | Connection pool size |
| `DB_POSTGRESDB_CONNECTION_TIMEOUT` | number | `20000` | Connection timeout (ms) |
| `DB_POSTGRESDB_SSL_ENABLED` | boolean | `false` | Enable SSL |
| `DB_POSTGRESDB_SSL_CA` | string | | SSL CA certificate |
| `DB_POSTGRESDB_SSL_REJECT_UNAUTHORIZED` | boolean | `true` | Reject unauthorized SSL |

### MySQL Configuration

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `DB_MYSQLDB_HOST` | string | `localhost` | Database host |
| `DB_MYSQLDB_PORT` | number | `3306` | Database port |
| `DB_MYSQLDB_DATABASE` | string | `n8n` | Database name |
| `DB_MYSQLDB_USER` | string | `root` | Username |
| `DB_MYSQLDB_PASSWORD` | string | | Password |
| `DB_MYSQLDB_POOL_SIZE` | number | `10` | Pool size |

### Database Logging

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `DB_LOGGING_ENABLED` | boolean | `false` | Enable query logging |
| `DB_LOGGING_OPTIONS` | enum | `error` | `query`, `error`, `schema`, `warn`, `info`, `all` |
| `DB_LOGGING_MAX_EXECUTION_TIME` | number | `0` | Log queries exceeding time (ms) |

---

## Execution Configuration
<!-- chunk: 05-execution | keywords: execution, timeout, save, prune | source: packages/@n8n/config/src/configs/executions.config.ts -->

### Execution Mode

| Variable | Type | Default | Options |
|----------|------|---------|---------|
| `EXECUTIONS_MODE` | enum | `regular` | `regular`, `queue` |

### Timeouts

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `EXECUTIONS_TIMEOUT` | number | `-1` | Execution timeout (seconds, -1=unlimited) |
| `EXECUTIONS_TIMEOUT_MAX` | number | `3600` | Max execution timeout (seconds) |

### Data Retention

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `EXECUTIONS_DATA_PRUNE` | boolean | `true` | Enable execution pruning |
| `EXECUTIONS_DATA_MAX_AGE` | number | `336` | Max age for soft-deletion (hours) |
| `EXECUTIONS_DATA_PRUNE_MAX_COUNT` | number | `10000` | Max finished executions to keep |
| `EXECUTIONS_DATA_HARD_DELETE_BUFFER` | number | `1` | Hard-delete buffer (hours) |

### Saving Behavior

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `EXECUTIONS_DATA_SAVE_ON_ERROR` | enum | `all` | Save on error: `all`, `none` |
| `EXECUTIONS_DATA_SAVE_ON_SUCCESS` | enum | `all` | Save on success: `all`, `none` |
| `EXECUTIONS_DATA_SAVE_ON_PROGRESS` | boolean | `false` | Save progress on each node |
| `EXECUTIONS_DATA_SAVE_MANUAL_EXECUTIONS` | boolean | `true` | Save manual executions |

### Concurrency

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `N8N_CONCURRENCY_PRODUCTION_LIMIT` | number | `-1` | Production concurrency (-1=unlimited) |
| `N8N_CONCURRENCY_EVALUATION_LIMIT` | number | `-1` | Evaluation concurrency |

---

## Security Configuration
<!-- chunk: 05-security | keywords: security, auth, mfa, sso | source: packages/@n8n/config/src/configs/ -->

### Authentication

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `N8N_SECURE_COOKIE` | boolean | `true` | Secure flag on auth cookie |
| `N8N_SAMESITE_COOKIE` | enum | `lax` | SameSite: `strict`, `lax`, `none` |
| `N8N_USER_MANAGEMENT_JWT_SECRET` | string | | JWT secret |
| `N8N_USER_MANAGEMENT_JWT_DURATION_HOURS` | number | `168` | JWT expiration (hours) |
| `N8N_MFA_ENABLED` | boolean | `true` | Enable MFA support |

### File Security

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `N8N_RESTRICT_FILE_ACCESS_TO` | string | | Allowed file access directories |
| `N8N_BLOCK_FILE_ACCESS_TO_N8N_FILES` | boolean | `true` | Block access to n8n files |

### Content Security

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `N8N_CONTENT_SECURITY_POLICY` | string | `{}` | CSP headers (JSON) |
| `N8N_CONTENT_SECURITY_POLICY_REPORT_ONLY` | boolean | `false` | CSP Report-Only mode |

### SSO Configuration

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `N8N_SSO_JUST_IN_TIME_PROVISIONING` | boolean | `true` | Create users on SSO login |
| `N8N_SSO_REDIRECT_LOGIN_TO_SSO` | boolean | `true` | Redirect login to SSO |
| `N8N_SSO_SAML_LOGIN_ENABLED` | boolean | `false` | Enable SAML SSO |
| `N8N_SSO_OIDC_LOGIN_ENABLED` | boolean | `false` | Enable OIDC SSO |
| `N8N_SSO_LDAP_LOGIN_ENABLED` | boolean | `false` | Enable LDAP SSO |

### SMTP Email

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `N8N_EMAIL_MODE` | enum | `smtp` | Email mode |
| `N8N_SMTP_HOST` | string | | SMTP server host |
| `N8N_SMTP_PORT` | number | `465` | SMTP port |
| `N8N_SMTP_USER` | string | | SMTP username |
| `N8N_SMTP_PASS` | string | | SMTP password |
| `N8N_SMTP_SSL` | boolean | `true` | Use SSL |
| `N8N_SMTP_SENDER` | string | | Sender email address |

---

## Logging Configuration
<!-- chunk: 05-logging | keywords: logging, log, level, output | source: packages/@n8n/config/src/configs/logging.config.ts -->

### Log Settings

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `N8N_LOG_LEVEL` | enum | `info` | `error`, `warn`, `info`, `debug`, `silent` |
| `N8N_LOG_OUTPUT` | string | `console` | Output: `console`, `file`, or both |
| `N8N_LOG_FORMAT` | enum | `text` | Format: `text`, `json` |

### File Logging

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `N8N_LOG_FILE_LOCATION` | string | `logs/n8n.log` | Log file path |
| `N8N_LOG_FILE_SIZE_MAX` | number | `16` | Max file size (MiB) |
| `N8N_LOG_FILE_COUNT_MAX` | number | `100` | Max files to keep |

### Log Scopes

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `N8N_LOG_SCOPES` | string | | Filter by scopes (comma-separated) |

**Available Scopes:**
`concurrency`, `external-secrets`, `license`, `multi-main-setup`, `pruning`, `pubsub`, `push`, `redis`, `scaling`, `waiting-executions`, `task-runner`, `task-runner-js`, `task-runner-py`, `workflow-activation`, `insights`, `ssh-client`, `data-table`, `cron`, `community-nodes`, `circuit-breaker`

---

## Queue/Scaling Configuration
<!-- chunk: 05-queue | keywords: queue, redis, scaling, worker | source: packages/@n8n/config/src/configs/scaling-mode.config.ts -->

### Redis Configuration

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `QUEUE_BULL_REDIS_HOST` | string | `localhost` | Redis host |
| `QUEUE_BULL_REDIS_PORT` | number | `6379` | Redis port |
| `QUEUE_BULL_REDIS_DB` | number | `0` | Redis database |
| `QUEUE_BULL_REDIS_PASSWORD` | string | | Redis password |
| `QUEUE_BULL_REDIS_USERNAME` | string | | Redis username |
| `QUEUE_BULL_REDIS_TLS` | boolean | `false` | Enable TLS |
| `N8N_REDIS_KEY_PREFIX` | string | `n8n` | Redis key prefix |

### Worker Configuration

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `QUEUE_WORKER_LOCK_DURATION` | number | `60000` | Worker lock duration (ms) |
| `QUEUE_WORKER_LOCK_RENEW_TIME` | number | `10000` | Lock renewal interval (ms) |
| `QUEUE_WORKER_STALLED_INTERVAL` | number | `30000` | Stalled job check interval (ms) |
| `QUEUE_WORKER_MAX_STALLED_COUNT` | number | `1` | Max stalled job retries |

### Health Checks

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `QUEUE_HEALTH_CHECK_ACTIVE` | boolean | `false` | Enable health check endpoints |
| `QUEUE_HEALTH_CHECK_PORT` | number | `5678` | Health check port |

### Multi-Main Setup

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `N8N_MULTI_MAIN_SETUP_ENABLED` | boolean | `false` | Enable multi-main |
| `N8N_MULTI_MAIN_SETUP_KEY_TTL` | number | `10` | Leader key TTL (seconds) |
| `N8N_MULTI_MAIN_SETUP_CHECK_INTERVAL` | number | `3` | Leader check interval (seconds) |

---

## Build Configuration
<!-- chunk: 05-build | keywords: build, turbo, pnpm, scripts | source: turbo.json, package.json -->

### Turbo Tasks

| Task | Dependencies | Outputs | Purpose |
|------|--------------|---------|---------|
| `build` | `^build` | `dist/**` | Build all packages |
| `typecheck` | `^typecheck`, `^build` | | Type checking |
| `lint` | `^build` | | Run linter |
| `test` | `^build`, `build` | `coverage/**` | Run tests |
| `dev` | | | Development server |

### NPM Scripts

| Script | Command | Purpose |
|--------|---------|---------|
| `pnpm build` | `turbo run build` | Build all packages |
| `pnpm dev` | `turbo run dev --parallel` | Start dev servers |
| `pnpm test` | `turbo run test` | Run all tests |
| `pnpm lint` | `turbo run lint` | Lint code |
| `pnpm typecheck` | `turbo typecheck` | Type checking |
| `pnpm format` | `turbo run format` | Format code |

### Task Runner Configuration

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `N8N_RUNNERS_ENABLED` | boolean | `false` | Enable task runners |
| `N8N_RUNNERS_MODE` | enum | `internal` | `internal` or `external` |
| `N8N_RUNNERS_MAX_CONCURRENCY` | number | `10` | Max concurrent tasks |
| `N8N_RUNNERS_TASK_TIMEOUT` | number | `300` | Task timeout (seconds) |
| `N8N_RUNNERS_MAX_OLD_SPACE_SIZE` | string | | Node.js heap size (MB) |

---

## Endpoints Configuration
<!-- chunk: 05-endpoints | keywords: endpoints, api, webhook | source: packages/@n8n/config/src/configs/endpoints.config.ts -->

### Payload Limits

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `N8N_PAYLOAD_SIZE_MAX` | number | `16` | Max payload size (MiB) |
| `N8N_FORMDATA_FILE_SIZE_MAX` | number | `200` | Max form-data file size (MiB) |

### Endpoint Paths

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `N8N_ENDPOINT_REST` | string | `rest` | REST API path |
| `N8N_ENDPOINT_WEBHOOK` | string | `webhook` | Webhook path |
| `N8N_ENDPOINT_WEBHOOK_TEST` | string | `webhook-test` | Test webhook path |
| `N8N_ENDPOINT_FORM` | string | `form` | Form endpoint path |

### Prometheus Metrics

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `N8N_METRICS` | boolean | `false` | Enable /metrics endpoint |
| `N8N_METRICS_PREFIX` | string | `n8n_` | Metric name prefix |
| `N8N_METRICS_INCLUDE_DEFAULT_METRICS` | boolean | `true` | Include system metrics |
| `N8N_METRICS_INCLUDE_WORKFLOW_ID_LABEL` | boolean | `false` | Include workflow ID label |

→ Debug Guide: [[06_DEBUG]]
→ Quick Reference: [[08_QUICK_REF]]
