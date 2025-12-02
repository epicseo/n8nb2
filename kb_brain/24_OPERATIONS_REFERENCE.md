# 24_OPERATIONS_REFERENCE.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: operations, crud, resources, api, integrations -->

## Contents
- [Overview](#overview)
- [CRM Operations](#crm-operations)
- [Communication Operations](#communication-operations)
- [Project Management Operations](#project-management-operations)
- [Database Operations](#database-operations)
- [File Storage Operations](#file-storage-operations)
- [Utility Operations](#utility-operations)

---

## Overview
<!-- chunk: 24-overview | keywords: operations, resources, actions -->

This reference covers ALL operations for major integration nodes.

### Operation Pattern
Most nodes follow CRUD pattern:
- `create` - Create new resource
- `get` - Get single resource by ID
- `getAll` - List resources with pagination
- `update` - Update existing resource
- `delete` - Delete resource

---

## CRM Operations
<!-- chunk: 24-crm | keywords: salesforce, hubspot, pipedrive -->

### Salesforce Operations

**Resources:** Account, Contact, Opportunity, Lead, Case, Task, User, Attachment, Custom Object

| Resource | Operations |
|----------|------------|
| Account | create, get, getAll, update, upsert, delete, addNote, getSummary |
| Contact | create, get, getAll, update, upsert, delete, addNote, addToCampaign |
| Opportunity | create, get, getAll, update, upsert, delete, addNote |
| Lead | create, get, getAll, update, upsert, delete, addNote, addToCampaign, convert |
| Case | create, get, getAll, update, delete, addComment |
| Task | create, get, getAll, update, delete (with recurrence) |

**Key Parameters:**
```json
{
  "resource": "contact",
  "operation": "create",
  "lastName": "Smith",
  "additionalFields": {
    "firstName": "John",
    "email": "john@example.com",
    "accountId": "001xxx"
  }
}
```

### HubSpot Operations

**Resources:** Contact, Company, Deal, Ticket, Engagement, Form, Contact List

| Resource | Operations |
|----------|------------|
| Contact | create, get, getAll, update, delete, listRecent, search |
| Company | create, get, getAll, update, delete |
| Deal | create, get, getAll, update, delete |
| Ticket | create, get, getAll, update, delete |
| Engagement | create (call/email/meeting/note), get, delete |

### Pipedrive Operations

**Resources:** Deal, Lead, Person, Organization, Activity, Note, Product

| Resource | Operations |
|----------|------------|
| Deal | create, get, getAll, update, delete, duplicate |
| Person | create, get, getAll, update, delete |
| Organization | create, get, getAll, update, delete |
| Activity | create, get, getAll, update, delete |
| Note | create, get, getAll, update, delete |

---

## Communication Operations
<!-- chunk: 24-communication | keywords: slack, email, discord, telegram -->

### Slack Operations

**Resources:** Message, Channel, File, Reaction, Star, User, UserGroup

| Resource | Operations |
|----------|------------|
| Message | post, update, delete, getPermalink, search, sendAndWait |
| Channel | create, get, getAll, archive, unarchive, open, close, rename, setPurpose, setTopic, join, leave, invite, kick, member, history, replies |
| File | upload, get, getAll |
| Reaction | add, get, remove |
| Star | add, delete, getAll |
| User | get, getAll, getPresence, updateProfile |
| UserGroup | create, getAll, update, enable, disable |

**Message Types:**
- `text` - Plain text with Markdown
- `block` - Block Kit JSON
- `attachment` - Legacy attachments

### Discord Operations

**Resources:** Message, Channel, Member

| Resource | Operations |
|----------|------------|
| Message | create, get, getAll, update, delete |
| Channel | create, get, getAll, update, delete |
| Member | get, getAll, update, kick |

### Microsoft Teams Operations

**Resources:** Message, Channel, Team, ChannelMessage

| Resource | Operations |
|----------|------------|
| Message | create, get, getAll, update, delete |
| Channel | create, get, getAll, delete |
| Team | get, getAll |

### Telegram Operations

| Operation | Description |
|-----------|-------------|
| sendMessage | Text message (max 4096 chars) |
| sendPhoto | Photo with caption |
| sendAudio | Audio file |
| sendVideo | Video file |
| sendDocument | Any file |
| sendLocation | GPS coordinates |
| sendVenue | Location with address |
| sendContact | Phone contact |
| sendPoll | Poll/quiz |
| editMessage | Edit existing message |
| deleteMessage | Delete message |
| getChat | Get chat info |
| getChatMembers | List members |

### Email Send Operations

| Operation | Parameters |
|-----------|------------|
| send | toEmail, subject, emailBody, ccEmail, bccEmail, attachments, fromEmail, replyTo, isHtml, priority |

---

## Project Management Operations
<!-- chunk: 24-pm | keywords: jira, asana, linear, trello -->

### Jira Operations

**Resources:** Issue, IssueComment, IssueAttachment, User

| Resource | Operations |
|----------|------------|
| Issue | create, get, getAll, update, delete, notify, transitions, changelog |
| IssueComment | add, get, getAll, update, remove |
| IssueAttachment | add, get, getAll, remove |
| User | create, get, delete |

**JQL Examples:**
```
project = PROJ AND status = "In Progress"
assignee = currentUser() AND duedate < 7d
priority in (High, Highest) ORDER BY created DESC
```

### Asana Operations

**Resources:** Task, Project, Portfolio, Team, User

| Resource | Operations |
|----------|------------|
| Task | create, get, getAll, update, delete, addAttachment, addSubtask |
| Project | create, get, getAll, update, delete |
| Portfolio | create, get, getAll, update |

### Linear Operations

**Resources:** Issue, Comment

| Resource | Operations |
|----------|------------|
| Issue | create, get, getAll, update, delete, addLink |
| Comment | add |

**Link Types:** relates_to, duplicates, is_duplicated_by, blocks, is_blocked_by

### ClickUp Operations

**Resources:** Task, List, Folder, Space, Goal, Comment, TimeEntry, Checklist

| Resource | Operations |
|----------|------------|
| Task | create, get, getAll, update, delete, member, setCustomField |
| List | create, get, getAll, update, delete, customFields, member |
| Folder | create, get, getAll, update, delete |
| Goal | create, get, getAll, update, delete |
| Comment | create, getAll, update, delete |
| TimeEntry | create, get, getAll, start, stop, update, delete |

### Trello Operations

**Resources:** Card, List, Board, Member, Checklist, Label, Attachment

| Resource | Operations |
|----------|------------|
| Card | create, get, getAll, update, delete, getAttachments |
| List | create, get, getAll, update, delete, archiveAllCards |
| Board | create, get, update, delete |
| Member | add, get, getAll, remove |

### Notion Operations

**Resources:** Database, Page, Block, User, Comment

| Resource | Operations |
|----------|------------|
| Database | create, get, getAll, update |
| Page | create, get, getAll, update, archive, delete |
| Block | create, getAll, update, delete |

**Block Types:** paragraph, heading_1-3, bulleted_list, numbered_list, toggle, quote, callout, image, video, code, table, divider

---

## Database Operations
<!-- chunk: 24-database | keywords: postgres, mysql, mongodb, redis -->

### PostgreSQL Operations

| Operation | Description |
|-----------|-------------|
| executeQuery | Execute SQL with parameterized queries ($1, $2) |
| insert | Insert rows |
| update | Update rows |
| upsert | Insert or update |
| delete | Delete rows |
| select | Select with filters |

### MySQL Operations

| Operation | Description |
|-----------|-------------|
| executeQuery | Execute SQL with ? placeholders |
| insert | Insert rows |
| update | Update rows |
| delete | Delete rows |

### MongoDB Operations

| Operation | Description |
|-----------|-------------|
| find | Query documents with filters |
| insert | Insert documents |
| update | Update documents |
| delete | Delete documents |
| aggregate | Aggregation pipeline |

**Query Example:**
```json
{ "status": "active", "age": { "$gte": 18 } }
```

### Redis Operations

| Operation | Description |
|-----------|-------------|
| get | Get value by key |
| set | Set key-value |
| delete | Delete key |
| incr | Increment number |
| keys | List keys by pattern |
| push | Push to list |
| pop | Pop from list |
| publish | Publish to channel |
| info | Server info |

---

## File Storage Operations
<!-- chunk: 24-storage | keywords: drive, sheets, s3, dropbox -->

### Google Sheets Operations

| Operation | Description |
|-----------|-------------|
| read | Read sheet data |
| append | Add rows |
| appendOrUpdate | Add or update by key |
| update | Update rows |
| delete | Delete rows |
| clear | Clear range |
| create | Create spreadsheet |

**Key Parameters:**
```json
{
  "operation": "append",
  "documentId": "spreadsheet-id",
  "sheetName": "Sheet1",
  "columns": {
    "mappingMode": "defineBelow",
    "value": {
      "Name": "={{ $json.name }}",
      "Email": "={{ $json.email }}"
    }
  }
}
```

### Google Drive Operations

| Resource | Operations |
|----------|------------|
| File | upload, download, copy, delete, move, share, update |
| Folder | create, delete, share |

### AWS S3 Operations

| Resource | Operations |
|----------|------------|
| Bucket | create, delete, getAll, search |
| File | upload, download, copy, delete, getAll |
| Folder | create, delete, getAll |

### Dropbox Operations

| Operation | Description |
|-----------|-------------|
| upload | Upload file |
| download | Download file |
| copy | Copy file/folder |
| move | Move file/folder |
| delete | Delete file/folder |
| createFolder | Create folder |
| list | List folder contents |
| search | Search files |

### Airtable Operations

| Operation | Description |
|-----------|-------------|
| list | List records with filters |
| get | Get record by ID |
| create | Create record |
| update | Update record |
| upsert | Create or update |
| delete | Delete record |

**Filter Example:**
```
AND({Status} = "Active", {Priority} = "High")
```

---

## Utility Operations
<!-- chunk: 24-utility | keywords: http, webhook, github, stripe -->

### HTTP Request

| Method | Use Case |
|--------|----------|
| GET | Retrieve data |
| POST | Create/submit |
| PUT | Replace |
| PATCH | Update |
| DELETE | Remove |
| HEAD | Headers only |

**Body Types:** json, form-urlencoded, multipart-form-data, raw, binary

### GitHub Operations

**Resources:** Repository, Issue, Pull Request, Release, File, User

| Resource | Operations |
|----------|------------|
| Repository | get, getIssues, getLicense, getProfile, listPopular |
| Issue | create, get, getAll, update, lock |
| PR | create, get, getAll, update, merge |
| Release | create, get, getAll, update, delete |
| File | create, edit, get, delete |

### Stripe Operations

**Resources:** Customer, Charge, Balance, Card, Source, Invoice, PaymentIntent, Subscription, Price, Product, Coupon

| Resource | Operations |
|----------|------------|
| Customer | create, get, getAll, update, delete |
| Charge | create, get, getAll, update |
| Invoice | create, get, getAll, delete, finalize, pay, void |
| PaymentIntent | create, confirm, cancel, capture |
| Subscription | create, get, getAll, update, delete |
| Price | create, get, getAll |
| Product | create, get, getAll, update, delete |

### Twilio Operations

| Resource | Operations |
|----------|------------|
| SMS | send |
| MMS | send (with media) |
| Call | make |

**Parameters:**
```json
{
  "resource": "sms",
  "operation": "send",
  "from": "+1234567890",
  "to": "+0987654321",
  "message": "Hello from n8n!"
}
```

### SendGrid Operations

| Operation | Description |
|-----------|-------------|
| send | Send email |
| sendTemplate | Send with template |

---

## Operations Quick Reference

### Common Optional Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `returnAll` | boolean | Fetch all results |
| `limit` | number | Max results |
| `filters` | object | Filter conditions |
| `sort` | object | Sort order |
| `fields` | string/array | Fields to include |

### Pagination Parameters

| Parameter | Description |
|-----------|-------------|
| `returnAll: true` | Auto-paginate all |
| `limit: 100` | Max per request |
| `offset: 0` | Starting position |
| `cursor` | Cursor token |

### Error Handling

| Option | Description |
|--------|-------------|
| `continueOnFail: true` | Skip errors |
| `retryOnFail: true` | Auto-retry |
| `maxTries: 3` | Retry attempts |

---

*Source: packages/nodes-base/nodes/*/
