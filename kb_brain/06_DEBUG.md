# 06_DEBUG.md
<!-- repo: n8n | branch: claude/github-to-kb-converter-01JAurUearApTk4RAu9NPFAg | commit: 3c0e809e | generated: 2025-11-28 -->
<!-- tags: debug, troubleshooting, errors, logging, health -->

## Contents
- [Error Types](#error-types)
- [Logging Configuration](#logging-configuration)
- [Debug Commands](#debug-commands)
- [Common Issues](#common-issues)
- [Health Checks](#health-checks)

---

## Error Types
<!-- chunk: 06-errors | keywords: error, exception, http, status | source: packages/cli/src/errors/ -->

### HTTP Response Errors

| Error | Status | Description | Resolution |
|-------|--------|-------------|------------|
| `BadRequestError` | 400 | Invalid request parameters | Check request format and parameters |
| `UnauthenticatedError` | 401 | Missing/invalid authentication | Re-authenticate with valid credentials |
| `AuthError` | 401 | Authentication failed | Verify username/password or API key |
| `ForbiddenError` | 403 | Access denied to resource | Contact admin for required permissions |
| `InvalidMfaCodeError` | 403 | Invalid MFA code | Verify MFA code and retry |
| `NotFoundError` | 404 | Resource not found | Verify resource ID and existence |
| `WebhookNotFoundError` | 404 | Webhook doesn't exist | Check webhook configuration and method |
| `ConflictError` | 409 | Resource conflict | Resolve conflict (duplicate name, etc.) |
| `ContentTooLargeError` | 413 | Payload too large | Reduce payload or increase limits |
| `TooManyRequestsError` | 429 | Rate limit exceeded | Implement retry with backoff |
| `InternalServerError` | 500 | Server error | Check server logs and retry |
| `ServiceUnavailableError` | 503 | Service unavailable | Check database connectivity |

→ API Errors: [[02_CORE_API#error-handling]]

---

### Execution Errors

| Error | Type | Cause | Resolution |
|-------|------|-------|------------|
| `NodeCrashedError` | NodeOperationError | Out-of-memory in node | Reduce batch size; split into loops |
| `WorkflowCrashedError` | WorkflowOperationError | Total memory exhausted | Increase heap: `--max-old-space-size` |
| `ExecutionNotFoundError` | UnexpectedError | Execution pruned/missing | Check execution retention settings |
| `WorkflowHasIssuesError` | WorkflowOperationError | Validation issues | Fix workflow errors before execution |

---

### Task Runner Errors

| Error | Type | Cause | Resolution |
|-------|------|-------|------------|
| `TaskRunnerDisconnectedError` | UnexpectedError | Runner crashed/disconnected | Restart runner; reduce batch sizes |
| `TaskRunnerOomError` | UserError | Runner out of memory | Increase `N8N_RUNNERS_MAX_OLD_SPACE_SIZE` |
| `TaskRunnerFailedHeartbeatError` | UserError | Runner unresponsive | Optimize script; increase heartbeat |
| `TaskRequestTimeoutError` | OperationalError | Task request timed out | Increase `N8N_RUNNERS_TASK_REQUEST_TIMEOUT` |
| `MissingRequirementsError` | UserError | Python/venv not available | Install Python 3; set up venv |

---

### Credential Errors

| Error | Type | Cause | Resolution |
|-------|------|-------|------------|
| `CredentialNotFoundError` | UserError | Credential deleted/inaccessible | Create new credential |
| `CredentialsOverwritesAlreadySetError` | Error | Duplicate credential override | Remove duplicate config |

---

### Other Errors

| Error | Type | Cause | Resolution |
|-------|------|-------|------------|
| `FeatureNotLicensedError` | UserError | License doesn't support feature | Upgrade license |
| `SubworkflowPolicyDenialError` | WorkflowOperationError | Cross-project restriction | Request read access |
| `NonJsonBodyError` | UserError | Request body not JSON | Set Content-Type: application/json |

---

## Logging Configuration
<!-- chunk: 06-logging | keywords: logging, log, debug, scopes | source: packages/@n8n/config/src/configs/logging.config.ts -->

### Log Levels

| Level | Priority | Description |
|-------|----------|-------------|
| `error` | Highest | Only errors |
| `warn` | High | Errors + warnings |
| `info` | Medium | Standard production |
| `debug` | Low | Development/troubleshooting |
| `silent` | Lowest | No logs |

### Configuration

```bash
# Development: Debug with file output
N8N_LOG_LEVEL=debug
N8N_LOG_OUTPUT=console,file
N8N_LOG_FORMAT=text

# Production: JSON format
N8N_LOG_LEVEL=info
N8N_LOG_OUTPUT=console
N8N_LOG_FORMAT=json

# Troubleshooting specific feature
N8N_LOG_LEVEL=debug
N8N_LOG_SCOPES=task-runner,scaling
```

→ Full logging config: [[05_CONFIG#logging-configuration]]

### Log Scopes

Filter logs by feature area using `N8N_LOG_SCOPES`:

| Scope | Feature Area |
|-------|--------------|
| `concurrency` | Concurrency management |
| `task-runner` | General task runner |
| `task-runner-js` | JavaScript runner |
| `task-runner-py` | Python runner |
| `workflow-activation` | Workflow activation |
| `scaling` | Scaling operations |
| `redis` | Redis connections |
| `pubsub` | Pub/sub messaging |
| `pruning` | Data pruning |
| `waiting-executions` | Waiting workflows |
| `circuit-breaker` | Circuit breaker |
| `community-nodes` | Community nodes |
| `cron` | Cron scheduling |

---

## Debug Commands
<!-- chunk: 06-commands | keywords: debug, cli, commands, test | source: .vscode/launch.json, package.json -->

### CLI Commands

```bash
# View server info
n8n info

# Test database connection
n8n db:validate

# Export workflow for inspection
n8n export:workflow --id=<workflow-id>

# Execute workflow from CLI
n8n execute --id=<workflow-id>

# Test credentials
n8n credentials:test --id=<credential-id>

# View license info
n8n license:info
```

### Node.js Debug Flags

```bash
# Enable debugging on port 9229
node --inspect app.js

# Enable debugging with breakpoint at startup
node --inspect-brk app.js

# Debug on custom host:port
node --inspect=0.0.0.0:9230 app.js
```

### Memory Debugging

```bash
# Set max heap size (for OOM issues)
NODE_OPTIONS="--max-old-space-size=4096" npm start

# For frontend builds (needs more memory)
NODE_OPTIONS="--max-old-space-size=8192"
```

### Test Commands

```bash
# Run all tests
pnpm test

# Run unit tests only
pnpm test:unit

# Run integration tests
pnpm test:integration

# Watch mode
pnpm test:dev

# Specific package tests
cd packages/cli && pnpm test
```

### Database Debug

```bash
# Enable query logging
DB_LOGGING_ENABLED=true
DB_LOGGING_OPTIONS=query
DB_LOGGING_MAX_EXECUTION_TIME=100  # Log slow queries > 100ms
```

---

## Common Issues
<!-- chunk: 06-issues | keywords: issues, problems, troubleshooting | source: analysis -->

### Memory Issues

| Issue | Symptoms | Resolution |
|-------|----------|------------|
| Node Crashes | `NodeCrashedError` | Reduce batch size; use loops |
| Workflow Crashes | `WorkflowCrashedError` | Increase heap: `--max-old-space-size=8192` |
| Code Node OOM | `TaskRunnerOomError` | Increase `N8N_RUNNERS_MAX_OLD_SPACE_SIZE` |
| Slow Execution | Long response times | Monitor memory; optimize queries |

### Database Issues

| Issue | Symptoms | Resolution |
|-------|----------|------------|
| Connection Failed | `ServiceUnavailableError` | Check DB connection string/credentials |
| Migration Timeout | Startup hangs | Increase timeout; check disk space |
| Query Timeout | Slow queries | Enable `DB_LOGGING_OPTIONS=query` |
| Locking (SQLite) | Database locked | Use PostgreSQL for production |

### Execution Issues

| Issue | Symptoms | Resolution |
|-------|----------|------------|
| Not Starting | Stuck in "running" | Check task runner; restart service |
| Timeout | `TaskRequestTimeoutError` | Increase `N8N_RUNNERS_TASK_REQUEST_TIMEOUT` |
| Credential Missing | `CredentialNotFoundError` | Recreate or update credential reference |
| Webhook Fails | Webhook not triggering | Verify path/method configuration |

### Authentication Issues

| Issue | Symptoms | Resolution |
|-------|----------|------------|
| Login Fails | `UnauthenticatedError` | Verify username/password |
| MFA Issues | `InvalidMfaCodeError` | Check time sync; regenerate TOTP |
| Permission Denied | `ForbiddenError` | Request required role from admin |

### Configuration Issues

| Issue | Symptoms | Resolution |
|-------|----------|------------|
| Runner Mode Conflict | `MissingAuthTokenError` | Set `N8N_RUNNERS_AUTH_TOKEN` |
| Invalid Config | Startup errors | Validate env var syntax |
| Cache Failure | `UncacheableValueError` | Remove circular refs from data |

---

## Health Checks
<!-- chunk: 06-health | keywords: health, readiness, liveness | source: packages/cli/src/ -->

### Endpoints

| Endpoint | Status | Description |
|----------|--------|-------------|
| `GET /healthz` | 200 | Basic liveness (server running) |
| `GET /healthz/readiness` | 200/503 | Readiness (DB + migrations) |

### Usage

```bash
# Basic health check
curl http://localhost:5678/healthz
# Response: {"status":"ok"}

# Readiness check (includes DB)
curl http://localhost:5678/healthz/readiness
# Success: {"status":"ok"} (HTTP 200)
# Failure: {"status":"error"} (HTTP 503)
```

### Kubernetes Example

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 5678
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /healthz/readiness
    port: 5678
  initialDelaySeconds: 30
  periodSeconds: 10
```

---

## Log Analysis
<!-- chunk: 06-log-analysis | keywords: logs, analysis, grep | source: analysis -->

### Common Log Queries

```bash
# Watch errors in log file
tail -f ~/.n8n/logs/n8n.log | grep -i error

# JSON log parsing
tail -f ~/.n8n/logs/n8n.log | jq '.level, .message'

# Count errors by type
grep -o '"name":"[^"]*Error' ~/.n8n/logs/n8n.log | sort | uniq -c

# Filter by scope
grep 'task-runner' ~/.n8n/logs/n8n.log
```

### Recommended Logging Setups

**Development:**
```bash
N8N_LOG_LEVEL=debug
N8N_LOG_OUTPUT=console
N8N_LOG_FORMAT=text
```

**Production:**
```bash
N8N_LOG_LEVEL=warn
N8N_LOG_OUTPUT=console,file
N8N_LOG_FORMAT=json
N8N_LOG_FILE_LOCATION=/var/log/n8n/n8n.log
```

**Troubleshooting:**
```bash
N8N_LOG_LEVEL=debug
N8N_LOG_OUTPUT=console,file
N8N_LOG_SCOPES=task-runner,scaling,waiting-executions
```

→ Configuration: [[05_CONFIG]]
→ Patterns: [[04_PATTERNS#error-handling-pattern]]
