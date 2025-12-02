# 05_CREDENTIALS.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: credentials, authentication, oauth, api-key, security -->

## Contents
- [Overview](#overview)
- [Authentication Patterns](#authentication-patterns)
- [Credential JSON Structure](#credential-json-structure)
- [API Key Credentials](#api-key-credentials)
- [OAuth2 Credentials](#oauth2-credentials)
- [Database Credentials](#database-credentials)
- [Cloud Provider Credentials](#cloud-provider-credentials)
- [Service Credentials](#service-credentials)
- [Credential Mapping](#credential-mapping)

---

## Overview
<!-- chunk: 05-overview | keywords: credentials, count, authentication -->

### Statistics
- **Total Credential Types:** 389
- **OAuth2 Credentials:** 120+
- **API Key Credentials:** 100+
- **Database Credentials:** 30+
- **Basic Auth:** 30+

### Authentication Pattern Distribution
| Pattern | Count | Examples |
|---------|-------|----------|
| OAuth2 | 120+ | Google, Slack, GitHub, Salesforce |
| API Key (Header) | 100+ | OpenAI, Stripe, Airtable |
| API Key (Query) | 20+ | Legacy APIs |
| Basic Auth | 30+ | Jira, Jenkins |
| OAuth1 | 5 | Twitter/X |
| JWT | 10+ | Google Service Account |
| AWS Signature | 5 | AWS services |

---

## Authentication Patterns
<!-- chunk: 05-auth | keywords: oauth2, apikey, basic, jwt -->

### API Key in Header (Most Common)
```json
{
  "authenticate": {
    "type": "generic",
    "properties": {
      "headers": {
        "Authorization": "Bearer {{$credentials.apiKey}}"
      }
    }
  }
}
```
**Used by:** OpenAI, Anthropic, Stripe, SendGrid, Airtable

### API Key in Query String
```json
{
  "authenticate": {
    "type": "generic",
    "properties": {
      "qs": { "api_key": "{{$credentials.apiKey}}" }
    }
  }
}
```

### OAuth2 Authorization Code
```json
{
  "authenticate": {
    "type": "oauth2",
    "properties": {
      "tokenType": "Bearer",
      "accessTokenUrl": "https://oauth.service.com/token",
      "authUrl": "https://oauth.service.com/authorize"
    }
  }
}
```
**Used by:** Google, Slack, GitHub, Microsoft, Salesforce

### Basic HTTP Authentication
```json
{
  "authenticate": {
    "type": "generic",
    "properties": {
      "headers": {
        "Authorization": "Basic {{Buffer.from($credentials.user + ':' + $credentials.password).toString('base64')}}"
      }
    }
  }
}
```
**Used by:** Jira, Jenkins, HTTP services

### AWS Signature V4
```json
{
  "authenticate": {
    "type": "awsSigv4",
    "properties": {
      "region": "{{$credentials.region}}",
      "service": "s3"
    }
  }
}
```
**Used by:** AWS S3, DynamoDB, Lambda, SQS, SNS

---

## Credential JSON Structure
<!-- chunk: 05-structure | keywords: format, interface, required -->

### In Workflow JSON
```json
{
  "nodes": [{
    "name": "Slack",
    "type": "n8n-nodes-base.slack",
    "credentials": {
      "slackOAuth2Api": {
        "id": "credential-uuid",
        "name": "Slack Production"
      }
    }
  }]
}
```

### Credential Definition Interface
```typescript
interface ICredentialType {
  name: string;           // Internal identifier (camelCase)
  displayName: string;    // User-visible name
  properties: INodeProperties[];
  extends?: string[];     // Parent credential types
  authenticate?: {
    type: 'generic' | 'oauth2' | 'jwt' | 'awsSigv4';
    properties: object;
  };
  test?: { request: IHttpRequestOptions };
}
```

---

## API Key Credentials
<!-- chunk: 05-apikey | keywords: openai, anthropic, stripe -->

### OpenAI API
**Name:** `openAiApi`

```json
{
  "name": "openAiApi",
  "properties": [
    { "name": "apiKey", "type": "string", "typeOptions": { "password": true }, "required": true },
    { "name": "organizationId", "type": "string" },
    { "name": "url", "type": "string", "default": "https://api.openai.com/v1" }
  ]
}
```

### Anthropic API
**Name:** `anthropicApi`

```json
{
  "name": "anthropicApi",
  "properties": [
    { "name": "apiKey", "type": "string", "typeOptions": { "password": true }, "required": true }
  ]
}
```

### Stripe API
**Name:** `stripeApi`

```json
{
  "name": "stripeApi",
  "properties": [
    { "name": "secretKey", "type": "string", "typeOptions": { "password": true }, "required": true }
  ]
}
```

### Airtable API
**Name:** `airtableApi`

```json
{
  "name": "airtableApi",
  "properties": [
    { "name": "apiKey", "type": "string", "typeOptions": { "password": true }, "required": true }
  ]
}
```

### SendGrid API
**Name:** `sendGridApi`

```json
{
  "name": "sendGridApi",
  "properties": [
    { "name": "apiKey", "type": "string", "typeOptions": { "password": true }, "required": true }
  ]
}
```

### Pinecone API
**Name:** `pineconeApi`

```json
{
  "name": "pineconeApi",
  "properties": [
    { "name": "apiKey", "type": "string", "typeOptions": { "password": true }, "required": true },
    { "name": "environment", "type": "string", "required": true }
  ]
}
```

---

## OAuth2 Credentials
<!-- chunk: 05-oauth2 | keywords: oauth2, google, slack, github -->

### Generic OAuth2
**Name:** `oAuth2Api` (base for extending)

```json
{
  "name": "oAuth2Api",
  "properties": [
    { "name": "grantType", "type": "options", "options": [
      { "name": "Authorization Code", "value": "authorizationCode" },
      { "name": "Client Credentials", "value": "clientCredentials" },
      { "name": "PKCE", "value": "pkce" }
    ]},
    { "name": "authUrl", "type": "string", "required": true },
    { "name": "accessTokenUrl", "type": "string", "required": true },
    { "name": "clientId", "type": "string", "required": true },
    { "name": "clientSecret", "type": "string", "typeOptions": { "password": true }, "required": true },
    { "name": "scope", "type": "string" }
  ]
}
```

### Slack OAuth2
**Name:** `slackOAuth2Api`

```json
{
  "name": "slackOAuth2Api",
  "extends": ["oAuth2Api"],
  "properties": [
    { "name": "authUrl", "default": "https://slack.com/oauth/v2/authorize" },
    { "name": "accessTokenUrl", "default": "https://slack.com/api/oauth.v2.access" },
    { "name": "scope", "default": "channels:read channels:write chat:write users:read" }
  ]
}
```

### Google OAuth2
**Name:** `googleOAuth2Api`

```json
{
  "name": "googleOAuth2Api",
  "extends": ["oAuth2Api"],
  "properties": [
    { "name": "authUrl", "default": "https://accounts.google.com/o/oauth2/v2/auth" },
    { "name": "accessTokenUrl", "default": "https://oauth2.googleapis.com/token" },
    { "name": "authQueryParameters", "default": "access_type=offline&prompt=consent" }
  ]
}
```

### Google Sheets OAuth2
**Name:** `googleSheetsOAuth2Api`

```json
{
  "name": "googleSheetsOAuth2Api",
  "extends": ["googleOAuth2Api"],
  "properties": [
    { "name": "scope", "default": "https://www.googleapis.com/auth/drive.file https://www.googleapis.com/auth/spreadsheets" }
  ]
}
```

### GitHub OAuth2
**Name:** `githubOAuth2Api`

```json
{
  "name": "githubOAuth2Api",
  "extends": ["oAuth2Api"],
  "properties": [
    { "name": "server", "default": "https://github.com" },
    { "name": "authUrl", "default": "={{$self.server}}/login/oauth/authorize" },
    { "name": "accessTokenUrl", "default": "={{$self.server}}/login/oauth/access_token" },
    { "name": "scope", "default": "repo admin:repo_hook user gist" }
  ]
}
```

### Microsoft OAuth2
**Name:** `microsoftOAuth2Api`

```json
{
  "name": "microsoftOAuth2Api",
  "extends": ["oAuth2Api"],
  "properties": [
    { "name": "authUrl", "default": "https://login.microsoftonline.com/common/oauth2/v2.0/authorize" },
    { "name": "accessTokenUrl", "default": "https://login.microsoftonline.com/common/oauth2/v2.0/token" }
  ]
}
```

### Salesforce OAuth2
**Name:** `salesforceOAuth2Api`

```json
{
  "name": "salesforceOAuth2Api",
  "extends": ["oAuth2Api"],
  "properties": [
    { "name": "environment", "type": "options", "options": [
      { "name": "Production", "value": "production" },
      { "name": "Sandbox", "value": "sandbox" }
    ]},
    { "name": "authUrl", "default": "={{$self.environment === 'sandbox' ? 'https://test.salesforce.com' : 'https://login.salesforce.com'}}/services/oauth2/authorize" }
  ]
}
```

### HubSpot OAuth2
**Name:** `hubspotOAuth2Api`

```json
{
  "name": "hubspotOAuth2Api",
  "extends": ["oAuth2Api"],
  "properties": [
    { "name": "authUrl", "default": "https://app.hubspot.com/oauth/authorize" },
    { "name": "accessTokenUrl", "default": "https://api.hubapi.com/oauth/v1/token" }
  ]
}
```

---

## Database Credentials
<!-- chunk: 05-database | keywords: postgres, mysql, mongodb, redis -->

### PostgreSQL
**Name:** `postgres`

```json
{
  "name": "postgres",
  "properties": [
    { "name": "host", "default": "localhost", "required": true },
    { "name": "database", "required": true },
    { "name": "user", "default": "postgres", "required": true },
    { "name": "password", "typeOptions": { "password": true }, "required": true },
    { "name": "port", "type": "number", "default": 5432 },
    { "name": "ssl", "type": "options", "options": [
      { "name": "Disable", "value": "disable" },
      { "name": "Require", "value": "require" },
      { "name": "Verify (Full)", "value": "verify-full" }
    ]}
  ]
}
```

### MySQL
**Name:** `mySql`

```json
{
  "name": "mySql",
  "properties": [
    { "name": "host", "default": "localhost", "required": true },
    { "name": "database", "required": true },
    { "name": "user", "default": "root", "required": true },
    { "name": "password", "typeOptions": { "password": true } },
    { "name": "port", "type": "number", "default": 3306 },
    { "name": "ssl", "type": "boolean", "default": false }
  ]
}
```

### MongoDB
**Name:** `mongoDb`

```json
{
  "name": "mongoDb",
  "properties": [
    { "name": "configurationType", "type": "options", "options": [
      { "name": "Connection String", "value": "connectionString" },
      { "name": "Values", "value": "values" }
    ]},
    { "name": "connectionString", "typeOptions": { "password": true }, "placeholder": "mongodb://user:password@localhost:27017" },
    { "name": "host", "default": "localhost" },
    { "name": "database", "required": true },
    { "name": "user", "type": "string" },
    { "name": "password", "typeOptions": { "password": true } },
    { "name": "port", "type": "number", "default": 27017 }
  ]
}
```

### Redis
**Name:** `redis`

```json
{
  "name": "redis",
  "properties": [
    { "name": "host", "default": "localhost", "required": true },
    { "name": "port", "type": "number", "default": 6379 },
    { "name": "password", "typeOptions": { "password": true } },
    { "name": "database", "type": "number", "default": 0 },
    { "name": "ssl", "type": "boolean", "default": false }
  ]
}
```

---

## Cloud Provider Credentials
<!-- chunk: 05-cloud | keywords: aws, gcp, azure -->

### AWS
**Name:** `aws`

```json
{
  "name": "aws",
  "properties": [
    { "name": "region", "type": "options", "options": [
      { "name": "US East (N. Virginia)", "value": "us-east-1" },
      { "name": "US West (Oregon)", "value": "us-west-2" },
      { "name": "EU (Ireland)", "value": "eu-west-1" },
      { "name": "EU (Frankfurt)", "value": "eu-central-1" },
      { "name": "Asia Pacific (Tokyo)", "value": "ap-northeast-1" }
    ], "required": true },
    { "name": "accessKeyId", "required": true },
    { "name": "secretAccessKey", "typeOptions": { "password": true }, "required": true },
    { "name": "temporaryCredentials", "type": "boolean", "default": false },
    { "name": "sessionToken", "typeOptions": { "password": true } }
  ]
}
```

### Google Service Account
**Name:** `googleApi`

```json
{
  "name": "googleApi",
  "properties": [
    { "name": "email", "required": true, "placeholder": "service@project.iam.gserviceaccount.com" },
    { "name": "privateKey", "typeOptions": { "password": true }, "required": true },
    { "name": "delegatedEmail", "description": "Email for G Suite domain-wide delegation" }
  ]
}
```

### Azure OpenAI
**Name:** `azureOpenAiApi`

```json
{
  "name": "azureOpenAiApi",
  "properties": [
    { "name": "apiKey", "typeOptions": { "password": true }, "required": true },
    { "name": "resourceName", "required": true },
    { "name": "apiVersion", "default": "2024-02-15-preview" }
  ]
}
```

---

## Service Credentials
<!-- chunk: 05-services | keywords: smtp, telegram, twilio, jira -->

### SMTP
**Name:** `smtp`

```json
{
  "name": "smtp",
  "properties": [
    { "name": "host", "required": true, "placeholder": "smtp.gmail.com" },
    { "name": "port", "type": "number", "default": 465, "required": true },
    { "name": "user", "required": true },
    { "name": "password", "typeOptions": { "password": true }, "required": true },
    { "name": "secure", "type": "boolean", "default": true }
  ]
}
```

### Telegram API
**Name:** `telegramApi`

```json
{
  "name": "telegramApi",
  "properties": [
    { "name": "accessToken", "typeOptions": { "password": true }, "required": true, "description": "Bot token from @BotFather" }
  ]
}
```

### Twilio API
**Name:** `twilioApi`

```json
{
  "name": "twilioApi",
  "properties": [
    { "name": "accountSid", "required": true },
    { "name": "authToken", "typeOptions": { "password": true }, "required": true }
  ]
}
```

### Jira API
**Name:** `jiraSoftwareCloudApi`

```json
{
  "name": "jiraSoftwareCloudApi",
  "properties": [
    { "name": "email", "required": true },
    { "name": "apiToken", "typeOptions": { "password": true }, "required": true },
    { "name": "domain", "required": true, "placeholder": "your-domain.atlassian.net" }
  ]
}
```

### Linear API
**Name:** `linearApi`

```json
{
  "name": "linearApi",
  "properties": [
    { "name": "apiKey", "typeOptions": { "password": true }, "required": true }
  ]
}
```

### HTTP Basic Auth
**Name:** `httpBasicAuth`

```json
{
  "name": "httpBasicAuth",
  "properties": [
    { "name": "user", "required": true },
    { "name": "password", "typeOptions": { "password": true }, "required": true }
  ]
}
```

### HTTP Header Auth
**Name:** `httpHeaderAuth`

```json
{
  "name": "httpHeaderAuth",
  "properties": [
    { "name": "name", "default": "Authorization", "required": true },
    { "name": "value", "typeOptions": { "password": true }, "required": true }
  ]
}
```

---

## Credential Mapping
<!-- chunk: 05-mapping | keywords: mapping, nodes, usage -->

### Service to Credential Quick Reference

| Service | Credential Name | Auth Type |
|---------|-----------------|-----------|
| OpenAI | `openAiApi` | API Key |
| Anthropic | `anthropicApi` | API Key |
| Slack | `slackOAuth2Api` | OAuth2 |
| Google Sheets | `googleSheetsOAuth2Api` | OAuth2 |
| Google Drive | `googleDriveOAuth2Api` | OAuth2 |
| GitHub | `githubOAuth2Api` | OAuth2 |
| Jira | `jiraSoftwareCloudApi` | Basic Auth |
| Salesforce | `salesforceOAuth2Api` | OAuth2 |
| HubSpot | `hubspotOAuth2Api` | OAuth2 |
| Airtable | `airtableApi` | API Key |
| Stripe | `stripeApi` | API Key |
| PostgreSQL | `postgres` | Password |
| MySQL | `mySql` | Password |
| MongoDB | `mongoDb` | Password |
| Redis | `redis` | Password |
| AWS | `aws` | Access Key |
| Telegram | `telegramApi` | Bot Token |
| Twilio | `twilioApi` | SID/Token |
| SendGrid | `sendGridApi` | API Key |
| SMTP | `smtp` | Password |
| Pinecone | `pineconeApi` | API Key |
| Qdrant | `qdrantApi` | API Key |
| Linear | `linearApi` | API Key |
| Notion | `notionApi` | API Key |

### Multi-Credential Nodes
Some nodes support multiple credential types:

```json
// Using OAuth2
{
  "name": "Google Sheets",
  "credentials": {
    "googleSheetsOAuth2Api": { "id": "1", "name": "Google OAuth" }
  }
}

// Using Service Account
{
  "name": "Google Sheets",
  "credentials": {
    "googleApi": { "id": "2", "name": "Google Service Account" }
  }
}
```

---

*Source: packages/nodes-base/credentials/*
