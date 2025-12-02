# 16_CREDENTIALS_REFERENCE.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: credentials, authentication, oauth, api-key, security -->

## Contents
- [Overview](#overview)
- [Authentication Patterns](#authentication-patterns)
- [Credential JSON Structure](#credential-json-structure)
- [API Key Credentials](#api-key-credentials)
- [OAuth2 Credentials](#oauth2-credentials)
- [OAuth1 Credentials](#oauth1-credentials)
- [Basic Auth Credentials](#basic-auth-credentials)
- [Database Credentials](#database-credentials)
- [Cloud Provider Credentials](#cloud-provider-credentials)
- [Common Service Credentials](#common-service-credentials)
- [Credential to Node Mapping](#credential-to-node-mapping)

---

## Overview
<!-- chunk: 16-overview | keywords: credentials, count, authentication -->

### Statistics
- **Total Credential Types:** 389
- **Credentials with Active Nodes:** 350
- **Unique Nodes Using Credentials:** 247

### Authentication Pattern Distribution
| Pattern | Count | Examples |
|---------|-------|----------|
| OAuth2 | 120+ | Google, Slack, GitHub, Salesforce |
| API Key (Header) | 100+ | OpenAI, Stripe, Airtable |
| API Key (Query) | 20+ | HubSpot (legacy), Mailchimp |
| Basic Auth | 30+ | HTTP services, LDAP |
| OAuth1 | 5 | Twitter/X |
| JWT | 10+ | Auth0, Google Service Account |
| AWS Signature | 5 | AWS services |
| Custom | 50+ | Database connections, SSH |

---

## Authentication Patterns
<!-- chunk: 16-auth-patterns | keywords: oauth2, apikey, basic, jwt -->

### Pattern 1: API Key in Header
Most common pattern - API key sent in Authorization header.

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

### Pattern 2: API Key in Query String
API key appended to URL parameters.

```json
{
  "authenticate": {
    "type": "generic",
    "properties": {
      "qs": {
        "api_key": "{{$credentials.apiKey}}"
      }
    }
  }
}
```

**Used by:** HubSpot (legacy), some legacy APIs

### Pattern 3: OAuth2 Authorization Code
Standard OAuth2 flow with redirect.

```json
{
  "authenticate": {
    "type": "oauth2",
    "properties": {
      "tokenType": "Bearer",
      "accessTokenUrl": "https://oauth.service.com/token",
      "authUrl": "https://oauth.service.com/authorize",
      "scope": "read write"
    }
  }
}
```

**Used by:** Google, Slack, GitHub, Microsoft, Salesforce

### Pattern 4: Basic HTTP Authentication
Username and password encoded in header.

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

**Used by:** Jira, Jenkins, basic HTTP services

### Pattern 5: JWT Service Account
Private key signs JWT for authentication.

```json
{
  "authenticate": {
    "type": "jwt",
    "properties": {
      "algorithm": "RS256",
      "iss": "{{$credentials.email}}",
      "sub": "{{$credentials.email}}",
      "aud": "https://oauth2.googleapis.com/token"
    }
  }
}
```

**Used by:** Google Service Account, Firebase, GCP

### Pattern 6: AWS Signature V4
Complex signing algorithm for AWS services.

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
<!-- chunk: 16-json-structure | keywords: format, interface, required -->

### In Workflow JSON
Credentials are referenced by ID and name:

```json
{
  "nodes": [
    {
      "name": "Slack",
      "type": "n8n-nodes-base.slack",
      "credentials": {
        "slackOAuth2Api": {
          "id": "credential-uuid-here",
          "name": "Slack Production"
        }
      }
    }
  ]
}
```

### Credential Definition Interface
```typescript
interface ICredentialType {
  name: string;           // Internal identifier (camelCase)
  displayName: string;    // User-visible name
  documentationUrl?: string;
  icon?: string;
  properties: INodeProperties[];
  extends?: string[];     // Parent credential types
  authenticate?: {
    type: 'generic' | 'oauth2' | 'jwt' | 'awsSigv4' | 'custom';
    properties: object;
  };
  test?: {
    request: IHttpRequestOptions;
  };
}
```

### INodeProperties for Credentials
```typescript
interface INodeProperties {
  displayName: string;
  name: string;
  type: 'string' | 'number' | 'boolean' | 'options' | 'hidden';
  default: any;
  required?: boolean;
  typeOptions?: {
    password?: boolean;    // Mask input
    expirable?: boolean;   // Auto-refresh token
  };
  displayOptions?: {
    show?: { [key: string]: any[] };
    hide?: { [key: string]: any[] };
  };
}
```

---

## API Key Credentials
<!-- chunk: 16-apikey | keywords: apikey, token, header -->

### OpenAI API
**Internal Name:** `openAiApi`
**Used By:** OpenAI, LangChain OpenAI nodes

```json
{
  "name": "openAiApi",
  "displayName": "OpenAI",
  "properties": [
    {
      "displayName": "API Key",
      "name": "apiKey",
      "type": "string",
      "typeOptions": { "password": true },
      "required": true
    },
    {
      "displayName": "Organization ID",
      "name": "organizationId",
      "type": "string",
      "description": "Optional - only if you belong to multiple organizations"
    },
    {
      "displayName": "Base URL",
      "name": "url",
      "type": "string",
      "default": "https://api.openai.com/v1"
    }
  ]
}
```

### Anthropic API
**Internal Name:** `anthropicApi`
**Used By:** Anthropic, LangChain Anthropic nodes

```json
{
  "name": "anthropicApi",
  "displayName": "Anthropic",
  "properties": [
    {
      "displayName": "API Key",
      "name": "apiKey",
      "type": "string",
      "typeOptions": { "password": true },
      "required": true
    }
  ]
}
```

### Stripe API
**Internal Name:** `stripeApi`
**Used By:** Stripe node

```json
{
  "name": "stripeApi",
  "displayName": "Stripe API",
  "properties": [
    {
      "displayName": "Secret API Key",
      "name": "secretKey",
      "type": "string",
      "typeOptions": { "password": true },
      "required": true
    }
  ]
}
```

### Airtable API
**Internal Name:** `airtableApi`
**Used By:** Airtable node, Airtable Trigger

```json
{
  "name": "airtableApi",
  "displayName": "Airtable Personal Access Token API",
  "properties": [
    {
      "displayName": "API Key / Access Token",
      "name": "apiKey",
      "type": "string",
      "typeOptions": { "password": true },
      "required": true
    }
  ]
}
```

### SendGrid API
**Internal Name:** `sendGridApi`
**Used By:** SendGrid node

```json
{
  "name": "sendGridApi",
  "displayName": "SendGrid API",
  "properties": [
    {
      "displayName": "API Key",
      "name": "apiKey",
      "type": "string",
      "typeOptions": { "password": true },
      "required": true
    }
  ]
}
```

---

## OAuth2 Credentials
<!-- chunk: 16-oauth2 | keywords: oauth2, authorization, token -->

### Generic OAuth2
**Internal Name:** `oAuth2Api`
**Base for extending**

```json
{
  "name": "oAuth2Api",
  "displayName": "OAuth2 API",
  "properties": [
    {
      "displayName": "Grant Type",
      "name": "grantType",
      "type": "options",
      "options": [
        { "name": "Authorization Code", "value": "authorizationCode" },
        { "name": "Client Credentials", "value": "clientCredentials" },
        { "name": "PKCE", "value": "pkce" }
      ],
      "default": "authorizationCode"
    },
    {
      "displayName": "Authorization URL",
      "name": "authUrl",
      "type": "string",
      "required": true
    },
    {
      "displayName": "Access Token URL",
      "name": "accessTokenUrl",
      "type": "string",
      "required": true
    },
    {
      "displayName": "Client ID",
      "name": "clientId",
      "type": "string",
      "required": true
    },
    {
      "displayName": "Client Secret",
      "name": "clientSecret",
      "type": "string",
      "typeOptions": { "password": true },
      "required": true
    },
    {
      "displayName": "Scope",
      "name": "scope",
      "type": "string"
    },
    {
      "displayName": "Auth URI Query Parameters",
      "name": "authQueryParameters",
      "type": "string"
    },
    {
      "displayName": "Authentication",
      "name": "authentication",
      "type": "options",
      "options": [
        { "name": "Header", "value": "header" },
        { "name": "Body", "value": "body" }
      ],
      "default": "header"
    }
  ]
}
```

### Slack OAuth2
**Internal Name:** `slackOAuth2Api`
**Used By:** Slack node, Slack Trigger

```json
{
  "name": "slackOAuth2Api",
  "displayName": "Slack OAuth2 API",
  "extends": ["oAuth2Api"],
  "properties": [
    {
      "displayName": "Authorization URL",
      "name": "authUrl",
      "type": "hidden",
      "default": "https://slack.com/oauth/v2/authorize"
    },
    {
      "displayName": "Access Token URL",
      "name": "accessTokenUrl",
      "type": "hidden",
      "default": "https://slack.com/api/oauth.v2.access"
    },
    {
      "displayName": "Scope",
      "name": "scope",
      "type": "hidden",
      "default": "channels:read channels:write chat:write files:read files:write groups:read groups:write im:read im:write mpim:read mpim:write reactions:read reactions:write stars:read stars:write users:read users:read.email usergroups:read usergroups:write"
    }
  ]
}
```

### Google OAuth2
**Internal Name:** `googleOAuth2Api`
**Base for Google services**

```json
{
  "name": "googleOAuth2Api",
  "displayName": "Google OAuth2 API",
  "extends": ["oAuth2Api"],
  "properties": [
    {
      "displayName": "Authorization URL",
      "name": "authUrl",
      "type": "hidden",
      "default": "https://accounts.google.com/o/oauth2/v2/auth"
    },
    {
      "displayName": "Access Token URL",
      "name": "accessTokenUrl",
      "type": "hidden",
      "default": "https://oauth2.googleapis.com/token"
    },
    {
      "displayName": "Auth URI Query Parameters",
      "name": "authQueryParameters",
      "type": "hidden",
      "default": "access_type=offline&prompt=consent"
    }
  ]
}
```

### Google Sheets OAuth2
**Internal Name:** `googleSheetsOAuth2Api`

```json
{
  "name": "googleSheetsOAuth2Api",
  "displayName": "Google Sheets OAuth2 API",
  "extends": ["googleOAuth2Api"],
  "properties": [
    {
      "displayName": "Scope",
      "name": "scope",
      "type": "hidden",
      "default": "https://www.googleapis.com/auth/drive.file https://www.googleapis.com/auth/spreadsheets"
    }
  ]
}
```

### GitHub OAuth2
**Internal Name:** `githubOAuth2Api`
**Used By:** GitHub node, GitHub Trigger

```json
{
  "name": "githubOAuth2Api",
  "displayName": "GitHub OAuth2 API",
  "extends": ["oAuth2Api"],
  "properties": [
    {
      "displayName": "Github Server",
      "name": "server",
      "type": "string",
      "default": "https://github.com",
      "description": "For GitHub Enterprise, use your server URL"
    },
    {
      "displayName": "Authorization URL",
      "name": "authUrl",
      "type": "hidden",
      "default": "={{$self.server}}/login/oauth/authorize"
    },
    {
      "displayName": "Access Token URL",
      "name": "accessTokenUrl",
      "type": "hidden",
      "default": "={{$self.server}}/login/oauth/access_token"
    },
    {
      "displayName": "Scope",
      "name": "scope",
      "type": "hidden",
      "default": "repo admin:repo_hook admin:org admin:org_hook user gist"
    }
  ]
}
```

### Microsoft OAuth2
**Internal Name:** `microsoftOAuth2Api`

```json
{
  "name": "microsoftOAuth2Api",
  "displayName": "Microsoft OAuth2 API",
  "extends": ["oAuth2Api"],
  "properties": [
    {
      "displayName": "Authorization URL",
      "name": "authUrl",
      "type": "hidden",
      "default": "https://login.microsoftonline.com/common/oauth2/v2.0/authorize"
    },
    {
      "displayName": "Access Token URL",
      "name": "accessTokenUrl",
      "type": "hidden",
      "default": "https://login.microsoftonline.com/common/oauth2/v2.0/token"
    }
  ]
}
```

### Salesforce OAuth2
**Internal Name:** `salesforceOAuth2Api`
**Used By:** Salesforce node, Salesforce Trigger

```json
{
  "name": "salesforceOAuth2Api",
  "displayName": "Salesforce OAuth2 API",
  "extends": ["oAuth2Api"],
  "properties": [
    {
      "displayName": "Environment",
      "name": "environment",
      "type": "options",
      "options": [
        { "name": "Production", "value": "production" },
        { "name": "Sandbox", "value": "sandbox" }
      ],
      "default": "production"
    },
    {
      "displayName": "Authorization URL",
      "name": "authUrl",
      "type": "hidden",
      "default": "={{$self.environment === 'sandbox' ? 'https://test.salesforce.com' : 'https://login.salesforce.com'}}/services/oauth2/authorize"
    },
    {
      "displayName": "Access Token URL",
      "name": "accessTokenUrl",
      "type": "hidden",
      "default": "={{$self.environment === 'sandbox' ? 'https://test.salesforce.com' : 'https://login.salesforce.com'}}/services/oauth2/token"
    }
  ]
}
```

---

## OAuth1 Credentials
<!-- chunk: 16-oauth1 | keywords: oauth1, twitter, signature -->

### Twitter OAuth1
**Internal Name:** `twitterOAuth1Api`
**Used By:** Twitter/X node

```json
{
  "name": "twitterOAuth1Api",
  "displayName": "X OAuth API",
  "extends": ["oAuth1Api"],
  "properties": [
    {
      "displayName": "Request Token URL",
      "name": "requestTokenUrl",
      "type": "hidden",
      "default": "https://api.twitter.com/oauth/request_token"
    },
    {
      "displayName": "Authorization URL",
      "name": "authUrl",
      "type": "hidden",
      "default": "https://api.twitter.com/oauth/authorize"
    },
    {
      "displayName": "Access Token URL",
      "name": "accessTokenUrl",
      "type": "hidden",
      "default": "https://api.twitter.com/oauth/access_token"
    },
    {
      "displayName": "Signature Method",
      "name": "signatureMethod",
      "type": "hidden",
      "default": "HMAC-SHA1"
    }
  ]
}
```

---

## Basic Auth Credentials
<!-- chunk: 16-basic | keywords: basic, username, password -->

### HTTP Basic Auth
**Internal Name:** `httpBasicAuth`
**Used By:** HTTP Request node, various services

```json
{
  "name": "httpBasicAuth",
  "displayName": "Basic Auth",
  "properties": [
    {
      "displayName": "User",
      "name": "user",
      "type": "string",
      "required": true
    },
    {
      "displayName": "Password",
      "name": "password",
      "type": "string",
      "typeOptions": { "password": true },
      "required": true
    }
  ]
}
```

### Jira API
**Internal Name:** `jiraSoftwareCloudApi`
**Used By:** Jira node, Jira Trigger

```json
{
  "name": "jiraSoftwareCloudApi",
  "displayName": "Jira Software Cloud API",
  "properties": [
    {
      "displayName": "Email",
      "name": "email",
      "type": "string",
      "required": true
    },
    {
      "displayName": "API Token",
      "name": "apiToken",
      "type": "string",
      "typeOptions": { "password": true },
      "required": true,
      "description": "Get your API token from https://id.atlassian.com/manage-profile/security/api-tokens"
    },
    {
      "displayName": "Domain",
      "name": "domain",
      "type": "string",
      "required": true,
      "placeholder": "your-domain.atlassian.net"
    }
  ]
}
```

---

## Database Credentials
<!-- chunk: 16-database | keywords: postgres, mysql, mongodb, redis -->

### PostgreSQL
**Internal Name:** `postgres`
**Used By:** PostgreSQL node

```json
{
  "name": "postgres",
  "displayName": "Postgres",
  "properties": [
    {
      "displayName": "Host",
      "name": "host",
      "type": "string",
      "required": true,
      "default": "localhost"
    },
    {
      "displayName": "Database",
      "name": "database",
      "type": "string",
      "required": true
    },
    {
      "displayName": "User",
      "name": "user",
      "type": "string",
      "required": true,
      "default": "postgres"
    },
    {
      "displayName": "Password",
      "name": "password",
      "type": "string",
      "typeOptions": { "password": true },
      "required": true
    },
    {
      "displayName": "Port",
      "name": "port",
      "type": "number",
      "default": 5432
    },
    {
      "displayName": "SSL",
      "name": "ssl",
      "type": "options",
      "options": [
        { "name": "Disable", "value": "disable" },
        { "name": "Allow", "value": "allow" },
        { "name": "Require", "value": "require" },
        { "name": "Verify (Not Full)", "value": "verify" },
        { "name": "Verify (Full)", "value": "verify-full" }
      ],
      "default": "disable"
    }
  ]
}
```

### MySQL
**Internal Name:** `mySql`
**Used By:** MySQL node

```json
{
  "name": "mySql",
  "displayName": "MySQL",
  "properties": [
    {
      "displayName": "Host",
      "name": "host",
      "type": "string",
      "required": true,
      "default": "localhost"
    },
    {
      "displayName": "Database",
      "name": "database",
      "type": "string",
      "required": true
    },
    {
      "displayName": "User",
      "name": "user",
      "type": "string",
      "required": true,
      "default": "root"
    },
    {
      "displayName": "Password",
      "name": "password",
      "type": "string",
      "typeOptions": { "password": true }
    },
    {
      "displayName": "Port",
      "name": "port",
      "type": "number",
      "default": 3306
    },
    {
      "displayName": "SSL",
      "name": "ssl",
      "type": "boolean",
      "default": false
    }
  ]
}
```

### MongoDB
**Internal Name:** `mongoDb`
**Used By:** MongoDB node

```json
{
  "name": "mongoDb",
  "displayName": "MongoDB",
  "properties": [
    {
      "displayName": "Configuration Type",
      "name": "configurationType",
      "type": "options",
      "options": [
        { "name": "Connection String", "value": "connectionString" },
        { "name": "Values", "value": "values" }
      ],
      "default": "values"
    },
    {
      "displayName": "Connection String",
      "name": "connectionString",
      "type": "string",
      "typeOptions": { "password": true },
      "displayOptions": { "show": { "configurationType": ["connectionString"] } },
      "placeholder": "mongodb://user:password@localhost:27017"
    },
    {
      "displayName": "Host",
      "name": "host",
      "type": "string",
      "default": "localhost",
      "displayOptions": { "show": { "configurationType": ["values"] } }
    },
    {
      "displayName": "Database",
      "name": "database",
      "type": "string",
      "required": true
    },
    {
      "displayName": "User",
      "name": "user",
      "type": "string",
      "displayOptions": { "show": { "configurationType": ["values"] } }
    },
    {
      "displayName": "Password",
      "name": "password",
      "type": "string",
      "typeOptions": { "password": true },
      "displayOptions": { "show": { "configurationType": ["values"] } }
    },
    {
      "displayName": "Port",
      "name": "port",
      "type": "number",
      "default": 27017,
      "displayOptions": { "show": { "configurationType": ["values"] } }
    }
  ]
}
```

### Redis
**Internal Name:** `redis`
**Used By:** Redis node, Redis Trigger

```json
{
  "name": "redis",
  "displayName": "Redis",
  "properties": [
    {
      "displayName": "Host",
      "name": "host",
      "type": "string",
      "required": true,
      "default": "localhost"
    },
    {
      "displayName": "Port",
      "name": "port",
      "type": "number",
      "default": 6379
    },
    {
      "displayName": "Password",
      "name": "password",
      "type": "string",
      "typeOptions": { "password": true }
    },
    {
      "displayName": "Database Number",
      "name": "database",
      "type": "number",
      "default": 0
    },
    {
      "displayName": "SSL",
      "name": "ssl",
      "type": "boolean",
      "default": false
    }
  ]
}
```

---

## Cloud Provider Credentials
<!-- chunk: 16-cloud | keywords: aws, gcp, azure -->

### AWS
**Internal Name:** `aws`
**Used By:** AWS S3, DynamoDB, Lambda, SQS, SNS, etc.

```json
{
  "name": "aws",
  "displayName": "AWS",
  "properties": [
    {
      "displayName": "Region",
      "name": "region",
      "type": "options",
      "options": [
        { "name": "US East (N. Virginia)", "value": "us-east-1" },
        { "name": "US East (Ohio)", "value": "us-east-2" },
        { "name": "US West (N. California)", "value": "us-west-1" },
        { "name": "US West (Oregon)", "value": "us-west-2" },
        { "name": "EU (Ireland)", "value": "eu-west-1" },
        { "name": "EU (London)", "value": "eu-west-2" },
        { "name": "EU (Frankfurt)", "value": "eu-central-1" },
        { "name": "Asia Pacific (Tokyo)", "value": "ap-northeast-1" },
        { "name": "Asia Pacific (Singapore)", "value": "ap-southeast-1" }
      ],
      "required": true
    },
    {
      "displayName": "Access Key ID",
      "name": "accessKeyId",
      "type": "string",
      "required": true
    },
    {
      "displayName": "Secret Access Key",
      "name": "secretAccessKey",
      "type": "string",
      "typeOptions": { "password": true },
      "required": true
    },
    {
      "displayName": "Use Temporary Credentials",
      "name": "temporaryCredentials",
      "type": "boolean",
      "default": false
    },
    {
      "displayName": "Session Token",
      "name": "sessionToken",
      "type": "string",
      "typeOptions": { "password": true },
      "displayOptions": { "show": { "temporaryCredentials": [true] } }
    }
  ]
}
```

### Google Service Account
**Internal Name:** `googleApi`
**Used By:** Google Drive, Sheets, Calendar, etc.

```json
{
  "name": "googleApi",
  "displayName": "Google Service Account API",
  "properties": [
    {
      "displayName": "Service Account Email",
      "name": "email",
      "type": "string",
      "required": true,
      "placeholder": "service@project.iam.gserviceaccount.com"
    },
    {
      "displayName": "Private Key",
      "name": "privateKey",
      "type": "string",
      "typeOptions": { "password": true },
      "required": true,
      "description": "Private key from JSON key file"
    },
    {
      "displayName": "Impersonate User",
      "name": "delegatedEmail",
      "type": "string",
      "description": "Email of user to impersonate (for G Suite domain-wide delegation)"
    }
  ]
}
```

---

## Common Service Credentials
<!-- chunk: 16-services | keywords: smtp, telegram, twilio -->

### SMTP
**Internal Name:** `smtp`
**Used By:** Email Send node

```json
{
  "name": "smtp",
  "displayName": "SMTP",
  "properties": [
    {
      "displayName": "Host",
      "name": "host",
      "type": "string",
      "required": true,
      "placeholder": "smtp.gmail.com"
    },
    {
      "displayName": "Port",
      "name": "port",
      "type": "number",
      "required": true,
      "default": 465
    },
    {
      "displayName": "User",
      "name": "user",
      "type": "string",
      "required": true
    },
    {
      "displayName": "Password",
      "name": "password",
      "type": "string",
      "typeOptions": { "password": true },
      "required": true
    },
    {
      "displayName": "Secure",
      "name": "secure",
      "type": "boolean",
      "default": true,
      "description": "Use SSL/TLS"
    }
  ]
}
```

### Telegram API
**Internal Name:** `telegramApi`
**Used By:** Telegram node, Telegram Trigger

```json
{
  "name": "telegramApi",
  "displayName": "Telegram API",
  "properties": [
    {
      "displayName": "Access Token",
      "name": "accessToken",
      "type": "string",
      "typeOptions": { "password": true },
      "required": true,
      "description": "Bot token from @BotFather"
    }
  ]
}
```

### Twilio API
**Internal Name:** `twilioApi`
**Used By:** Twilio node, Twilio Trigger

```json
{
  "name": "twilioApi",
  "displayName": "Twilio API",
  "properties": [
    {
      "displayName": "Account SID",
      "name": "accountSid",
      "type": "string",
      "required": true
    },
    {
      "displayName": "Auth Token",
      "name": "authToken",
      "type": "string",
      "typeOptions": { "password": true },
      "required": true
    }
  ]
}
```

---

## Credential to Node Mapping
<!-- chunk: 16-mapping | keywords: mapping, nodes, usage -->

### High-Usage Credentials (Used by Multiple Nodes)

| Credential | Nodes Using It |
|------------|----------------|
| `googleApi` | Google Drive, Sheets, Calendar, Docs, Slides, Gmail, BigQuery |
| `aws` | S3, DynamoDB, Lambda, SQS, SNS, Comprehend, Rekognition, SES |
| `slackOAuth2Api` | Slack, n8n (notifications) |
| `githubApi` | GitHub, n8n (community nodes) |
| `httpBasicAuth` | HTTP Request, GraphQL |
| `httpDigestAuth` | HTTP Request, GraphQL |
| `httpHeaderAuth` | HTTP Request, GraphQL |

### Service-Specific Credentials

| Service | Credential Name | Auth Type |
|---------|-----------------|-----------|
| OpenAI | `openAiApi` | API Key |
| Anthropic | `anthropicApi` | API Key |
| Slack | `slackOAuth2Api` | OAuth2 |
| Google Sheets | `googleSheetsOAuth2Api` | OAuth2 |
| GitHub | `githubOAuth2Api` | OAuth2 |
| Jira | `jiraSoftwareCloudApi` | Basic Auth |
| Salesforce | `salesforceOAuth2Api` | OAuth2 |
| HubSpot | `hubspotOAuth2Api` | OAuth2 |
| Airtable | `airtableApi` | API Key |
| Stripe | `stripeApi` | API Key |
| PostgreSQL | `postgres` | Password |
| MySQL | `mySql` | Password |
| MongoDB | `mongoDb` | Password/Connection String |
| Redis | `redis` | Password |
| AWS | `aws` | Access Key |
| Telegram | `telegramApi` | Bot Token |
| Twilio | `twilioApi` | Account SID/Auth Token |
| SendGrid | `sendGridApi` | API Key |
| SMTP | `smtp` | Password |

---

## Quick Reference

### Creating Credentials in Workflow JSON
```json
{
  "nodes": [
    {
      "name": "HTTP Request",
      "type": "n8n-nodes-base.httpRequest",
      "credentials": {
        "httpBasicAuth": {
          "id": "1",
          "name": "My Basic Auth"
        }
      }
    }
  ]
}
```

### Multi-Credential Node Example
Some nodes support multiple credential types:

```json
{
  "name": "Google Sheets",
  "type": "n8n-nodes-base.googleSheets",
  "credentials": {
    "googleSheetsOAuth2Api": {
      "id": "2",
      "name": "Google Sheets OAuth2"
    }
  }
}
```

Or using Service Account:
```json
{
  "name": "Google Sheets",
  "type": "n8n-nodes-base.googleSheets",
  "credentials": {
    "googleApi": {
      "id": "3",
      "name": "Google Service Account"
    }
  }
}
```

---

*Generated from n8n v1.122.0 - packages/nodes-base/credentials*
