# 13_EXPRESSION_GUIDE.md
<!-- repo: n8n | version: 1.122.0 | generated: 2025-12-02 -->
<!-- tags: expressions, syntax, functions, variables, javascript -->

## Contents
- [Expression Basics](#expression-basics)
- [Built-in Variables](#built-in-variables)
- [String Methods](#string-methods)
- [Array Methods](#array-methods)
- [Number Methods](#number-methods)
- [Date Methods](#date-methods)
- [Object Methods](#object-methods)
- [Conditional Expressions](#conditional-expressions)
- [JMESPath Queries](#jmespath-queries)
- [Common Patterns](#common-patterns)

---

## Expression Basics
<!-- chunk: 13-basics | keywords: expression, syntax, basics | source: packages/workflow/src/Expression*.ts -->

### Syntax
All expressions use double curly braces with an equals prefix:

```javascript
={{ expression }}
```

### Expression Types

| Type | Example | Description |
|------|---------|-------------|
| Variable access | `={{ $json.field }}` | Access data field |
| Method call | `={{ $json.name.toUpperCase() }}` | Call method |
| Arithmetic | `={{ $json.price * 1.1 }}` | Math operations |
| Concatenation | `={{ "Hello " + $json.name }}` | String joining |
| Ternary | `={{ $json.age > 18 ? "adult" : "minor" }}` | Conditional |

### JavaScript Support
Expressions support standard JavaScript:
- Operators: `+`, `-`, `*`, `/`, `%`, `**`
- Comparison: `===`, `!==`, `>`, `<`, `>=`, `<=`
- Logical: `&&`, `||`, `!`
- Optional chaining: `?.`
- Nullish coalescing: `??`

---

## Built-in Variables
<!-- chunk: 13-variables | keywords: variables, json, input, node, workflow | source: packages/workflow/src/WorkflowDataProxy.ts -->

### $json - Current Item Data
Access the JSON data of the current item being processed.

```javascript
{{ $json }}                    // Entire item
{{ $json.fieldName }}          // Specific field
{{ $json.user.email }}         // Nested access
{{ $json.items[0] }}           // Array access
{{ $json.items[0].name }}      // Nested array access
```

### $input - Input Data Methods
Access input data with utility methods.

```javascript
{{ $input.first() }}           // First input item
{{ $input.last() }}            // Last input item
{{ $input.item }}              // Current item
{{ $input.all() }}             // All items as array
{{ $input.first().json }}      // First item's JSON
{{ $input.all().length }}      // Item count
```

### $node - Other Node Data
Access output from other nodes in the workflow.

```javascript
{{ $node["Node Name"].json }}              // Node output
{{ $node["Node Name"].json.field }}        // Specific field
{{ $node["HTTP Request"].json.data }}      // HTTP response
{{ $node["Previous"].first().json }}       // First item from node
```

### $workflow - Workflow Metadata

```javascript
{{ $workflow.id }}             // Workflow ID
{{ $workflow.name }}           // Workflow name
{{ $workflow.active }}         // Is workflow active
```

### $env - Environment Variables

```javascript
{{ $env.API_KEY }}             // Access env variable
{{ $env.DATABASE_URL }}        // Database connection
{{ $env.NODE_ENV }}            // Environment name
```

### $now - Current DateTime

```javascript
{{ $now }}                     // Current DateTime object
{{ $now.toISO() }}             // ISO string
{{ $now.toFormat('yyyy-MM-dd') }}  // Formatted date
```

### $today - Today's Date

```javascript
{{ $today }}                   // Today at midnight
{{ $today.toISODate() }}       // YYYY-MM-DD format
```

### $execution - Execution Info

```javascript
{{ $execution.id }}            // Execution ID
{{ $execution.mode }}          // Execution mode
{{ $execution.resumeUrl }}     // Resume URL (for wait nodes)
```

### $runIndex / $itemIndex

```javascript
{{ $runIndex }}                // Current run index (0-based)
{{ $itemIndex }}               // Current item index (0-based)
```

### $vars - Workflow Variables

```javascript
{{ $vars.myVariable }}         // Access workflow variable
{{ $vars.apiEndpoint }}        // Custom variable
```

---

## String Methods
<!-- chunk: 13-string | keywords: string, text, methods | source: packages/workflow/src/Extensions/StringExtensions.ts -->

### Case Conversion
```javascript
{{ "hello world".toUpperCase() }}      // "HELLO WORLD"
{{ "HELLO WORLD".toLowerCase() }}      // "hello world"
{{ "hello world".toTitleCase() }}      // "Hello World"
{{ "Hello World".toSnakeCase() }}      // "hello_world"
{{ "hello world".toSentenceCase() }}   // "Hello world"
```

### Validation
```javascript
{{ "test@email.com".isEmail() }}       // true
{{ "https://n8n.io".isUrl() }}         // true
{{ "example.com".isDomain() }}         // true
{{ "12345".isNumeric() }}              // true
{{ "".isEmpty() }}                     // true
{{ "hello".isNotEmpty() }}             // true
```

### Extraction
```javascript
{{ "Contact: test@email.com".extractEmail() }}     // "test@email.com"
{{ "Visit https://n8n.io".extractUrl() }}          // "https://n8n.io"
{{ "test@example.com".extractDomain() }}           // "example.com"
{{ "https://n8n.io/path".extractUrlPath() }}       // "/path"
```

### Encoding
```javascript
{{ "hello".base64Encode() }}           // "aGVsbG8="
{{ "aGVsbG8=".base64Decode() }}        // "hello"
{{ "hello world".urlEncode() }}        // "hello%20world"
{{ "hello%20world".urlDecode() }}      // "hello world"
{{ "password".hash("sha256") }}        // SHA-256 hash
{{ "password".hash("md5") }}           // MD5 hash
```

### Transformation
```javascript
{{ "<p>Hello</p>".removeTags() }}      // "Hello"
{{ "**bold**".removeMarkdown() }}      // "bold"
{{ "café".replaceSpecialChars() }}     // "cafe"
{{ '{"a":1}'.parseJson() }}            // { a: 1 }
{{ "hello".quote() }}                  // '"hello"'
```

### Conversion
```javascript
{{ "42".toNumber() }}                  // 42
{{ "42".toInt() }}                     // 42
{{ "3.14".toFloat() }}                 // 3.14
{{ "true".toBoolean() }}               // true
{{ "2024-01-15".toDateTime() }}        // DateTime object
```

---

## Array Methods
<!-- chunk: 13-array | keywords: array, list, methods | source: packages/workflow/src/Extensions/ArrayExtensions.ts -->

### Access
```javascript
{{ [1, 2, 3].first() }}                // 1
{{ [1, 2, 3].last() }}                 // 3
{{ [1, 2, 3].randomItem() }}           // Random element
{{ [1, 2, 3].length }}                 // 3
```

### Aggregation
```javascript
{{ [1, 2, 3, 4, 5].sum() }}            // 15
{{ [1, 2, 3, 4, 5].average() }}        // 3
{{ [1, 2, 3, 4, 5].min() }}            // 1
{{ [1, 2, 3, 4, 5].max() }}            // 5
```

### Transformation
```javascript
{{ [1, 1, 2, 2, 3].unique() }}         // [1, 2, 3]
{{ [1, 2, 3].chunk(2) }}               // [[1, 2], [3]]
{{ [null, 1, "", 2].compact() }}       // [1, 2]
{{ [1, 2].append(3, 4) }}              // [1, 2, 3, 4]
{{ [1, 2].merge([3, 4]) }}             // [1, 2, 3, 4]
```

### Set Operations
```javascript
{{ [1, 2, 3].union([3, 4, 5]) }}               // [1, 2, 3, 4, 5]
{{ [1, 2, 3].intersection([2, 3, 4]) }}        // [2, 3]
{{ [1, 2, 3].difference([2, 3, 4]) }}          // [1]
```

### Object Array Operations
```javascript
// Extract field from objects
{{ [{name: "John"}, {name: "Jane"}].pluck("name") }}
// ["John", "Jane"]

// Rename keys in objects
{{ [{old: 1}].renameKeys("old", "new") }}
// [{new: 1}]

// Convert to key-value object
{{ [{k: "a", v: 1}, {k: "b", v: 2}].smartJoin("k", "v") }}
// {a: 1, b: 2}
```

### Validation
```javascript
{{ [].isEmpty() }}                     // true
{{ [1, 2, 3].isNotEmpty() }}           // true
```

---

## Number Methods
<!-- chunk: 13-number | keywords: number, math, methods | source: packages/workflow/src/Extensions/NumberExtensions.ts -->

### Rounding
```javascript
{{ (3.7).floor() }}                    // 3
{{ (3.2).ceil() }}                     // 4
{{ (3.456).round(2) }}                 // 3.46
{{ (-5).abs() }}                       // 5
```

### Validation
```javascript
{{ (42).isInteger() }}                 // true
{{ (4).isEven() }}                     // true
{{ (5).isOdd() }}                      // true
```

### Formatting
```javascript
{{ (1234567.89).format() }}            // "1,234,567.89"
{{ (1234567.89).format("de-DE") }}     // "1.234.567,89"
```

### Conversion
```javascript
{{ (1708695471).toDateTime("s") }}     // DateTime from Unix seconds
{{ (1708695471000).toDateTime("ms") }} // DateTime from milliseconds
{{ (0).toBoolean() }}                  // false
{{ (1).toBoolean() }}                  // true
```

---

## Date Methods
<!-- chunk: 13-date | keywords: date, time, datetime, luxon | source: packages/workflow/src/Extensions/DateExtensions.ts -->

### Current Date/Time
```javascript
{{ $now }}                             // Current DateTime
{{ $now.toISO() }}                     // "2024-01-15T10:30:00.000Z"
{{ $now.toISODate() }}                 // "2024-01-15"
{{ $now.toISOTime() }}                 // "10:30:00.000"
```

### Formatting
```javascript
{{ $now.toFormat("yyyy-MM-dd") }}      // "2024-01-15"
{{ $now.toFormat("dd/MM/yyyy") }}      // "15/01/2024"
{{ $now.toFormat("HH:mm:ss") }}        // "10:30:00"
{{ $now.toFormat("MMMM d, yyyy") }}    // "January 15, 2024"
{{ $now.toFormat("cccc") }}            // "Monday"
```

### Format Tokens
| Token | Output | Example |
|-------|--------|---------|
| `yyyy` | 4-digit year | 2024 |
| `yy` | 2-digit year | 24 |
| `MM` | 2-digit month | 01 |
| `M` | 1-2 digit month | 1 |
| `MMMM` | Full month name | January |
| `MMM` | Short month | Jan |
| `dd` | 2-digit day | 15 |
| `d` | 1-2 digit day | 15 |
| `cccc` | Full weekday | Monday |
| `ccc` | Short weekday | Mon |
| `HH` | 24-hour hour | 14 |
| `hh` | 12-hour hour | 02 |
| `mm` | Minutes | 30 |
| `ss` | Seconds | 45 |
| `a` | AM/PM | PM |

### Arithmetic
```javascript
{{ $now.plus(7, "days") }}             // Add 7 days
{{ $now.minus(1, "month") }}           // Subtract 1 month
{{ $now.plus({days: 7, hours: 2}) }}   // Add 7 days and 2 hours
{{ $now.minus({weeks: 2}) }}           // Subtract 2 weeks
```

### Time Units
- `years`, `months`, `weeks`, `days`
- `hours`, `minutes`, `seconds`, `milliseconds`

### Extraction
```javascript
{{ $now.year }}                        // 2024
{{ $now.month }}                       // 1 (January)
{{ $now.day }}                         // 15
{{ $now.hour }}                        // 10
{{ $now.minute }}                      // 30
{{ $now.second }}                      // 0
{{ $now.weekday }}                     // 1 (Monday)
```

### Boundaries
```javascript
{{ $now.startOf("day") }}              // Start of today
{{ $now.endOf("day") }}                // End of today
{{ $now.startOf("month") }}            // First of month
{{ $now.endOf("month") }}              // Last of month
{{ $now.startOf("week") }}             // Start of week
```

### Comparison
```javascript
{{ $now.diff(otherDate, "days") }}     // Days between
{{ $now.diffNow("hours") }}            // Hours from now
{{ date1 < date2 }}                    // Compare dates
{{ $now.hasSame(otherDate, "day") }}   // Same day?
```

### Validation
```javascript
{{ $now.isWeekend }}                   // Is Saturday/Sunday
{{ $now.isInDST }}                     // Is in daylight saving
```

### Conversion
```javascript
{{ "2024-01-15".toDateTime() }}        // Parse ISO string
{{ "01/15/2024".toDateTime("MM/dd/yyyy") }}  // Custom format
{{ (1705312200).toDateTime("s") }}     // From Unix seconds
```

---

## Object Methods
<!-- chunk: 13-object | keywords: object, json, methods | source: packages/workflow/src/Extensions/ObjectExtensions.ts -->

### Access
```javascript
{{ $json.keys() }}                     // Array of keys
{{ $json.values() }}                   // Array of values
{{ $json.hasField("name") }}           // true/false
```

### Transformation
```javascript
{{ $json.removeField("password") }}    // Remove field
{{ {a: null, b: 2}.compact() }}        // {b: 2}
{{ {name: "John", age: 30}.urlEncode() }}  // "name=John&age=30"
```

### Validation
```javascript
{{ {}.isEmpty() }}                     // true
{{ {a: 1}.isNotEmpty() }}              // true
```

### Conversion
```javascript
{{ $json.toJsonString() }}             // JSON string
```

---

## Conditional Expressions
<!-- chunk: 13-conditionals | keywords: conditional, ternary, if | source: packages/workflow/src/Expression.ts -->

### Ternary Operator
```javascript
{{ condition ? valueIfTrue : valueIfFalse }}

// Examples:
{{ $json.age >= 18 ? "Adult" : "Minor" }}
{{ $json.status === "active" ? "✓" : "✗" }}
{{ $json.score > 80 ? "Pass" : "Fail" }}
```

### Nested Ternary
```javascript
{{
  $json.score >= 90 ? "A" :
  $json.score >= 80 ? "B" :
  $json.score >= 70 ? "C" :
  $json.score >= 60 ? "D" : "F"
}}
```

### Logical Operators
```javascript
{{ $json.isActive && $json.isVerified }}     // AND
{{ $json.isAdmin || $json.isModerator }}     // OR
{{ !$json.isDeleted }}                       // NOT
```

### Optional Chaining
```javascript
{{ $json.user?.email }}                // Safe nested access
{{ $json.items?.[0]?.name }}           // Safe array access
{{ $node["Optional"]?.json?.data }}    // Safe node access
```

### Nullish Coalescing
```javascript
{{ $json.name ?? "Unknown" }}          // Default if null/undefined
{{ $json.count ?? 0 }}                 // Default number
{{ $json.items ?? [] }}                // Default array
```

### Combined Patterns
```javascript
{{ $json.user?.name ?? "Guest" }}      // Safe access with default
{{ ($json.price ?? 0) * 1.1 }}         // Default then calculate
```

---

## JMESPath Queries
<!-- chunk: 13-jmespath | keywords: jmespath, query, filter | source: packages/workflow/src/Expression.ts -->

n8n supports JMESPath for complex data queries:

### Basic Queries
```javascript
{{ $jmespath($json, "users[*].name") }}        // All user names
{{ $jmespath($json, "users[0]") }}             // First user
{{ $jmespath($json, "users[-1]") }}            // Last user
```

### Filtering
```javascript
{{ $jmespath($json, "users[?age > `18`]") }}           // Adults
{{ $jmespath($json, "items[?status == 'active']") }}   // Active items
{{ $jmespath($json, "products[?price < `100`]") }}     // Cheap products
```

### Projections
```javascript
// Select specific fields
{{ $jmespath($json, "users[*].{name: name, email: email}") }}

// Flatten nested arrays
{{ $jmespath($json, "orders[*].items[]") }}
```

### Sorting
```javascript
{{ $jmespath($json, "sort_by(users, &name)") }}        // Sort by name
{{ $jmespath($json, "reverse(sort_by(users, &age))") }} // Sort desc
```

### Aggregation
```javascript
{{ $jmespath($json, "length(users)") }}                // Count
{{ $jmespath($json, "max_by(products, &price)") }}     // Most expensive
{{ $jmespath($json, "min_by(products, &price)") }}     // Cheapest
```

---

## Common Patterns
<!-- chunk: 13-patterns | keywords: patterns, examples, common | source: analysis -->

### Data Transformation

```javascript
// Combine first and last name
{{ $json.firstName + " " + $json.lastName }}

// Format currency
{{ "$" + $json.amount.toFixed(2) }}

// Create email from name
{{ $json.firstName.toLowerCase() + "@company.com" }}

// Format phone number
{{ $json.phone.replace(/(\d{3})(\d{3})(\d{4})/, "($1) $2-$3") }}
```

### Date Handling

```javascript
// Format date for display
{{ $json.createdAt.toDateTime().toFormat("MMMM d, yyyy") }}

// Calculate days until deadline
{{ $json.deadline.toDateTime().diff($now, "days").days }}

// Check if date is in the past
{{ $json.expiresAt.toDateTime() < $now }}

// Get relative time
{{ $json.timestamp.toDateTime().toRelative() }}
```

### Array Processing

```javascript
// Join array to string
{{ $json.tags.join(", ") }}

// Filter and count
{{ $json.items.filter(i => i.active).length }}

// Get unique values
{{ $json.categories.unique().join(", ") }}

// Sum prices
{{ $json.orderItems.map(i => i.price * i.qty).sum() }}
```

### Conditional Logic

```javascript
// Status badge
{{ $json.status === "active" ? "🟢 Active" : "🔴 Inactive" }}

// Null-safe access with default
{{ $json.user?.displayName ?? $json.user?.email ?? "Anonymous" }}

// Multiple conditions
{{ $json.amount > 1000 && $json.isVerified ? "Approved" : "Review Required" }}

// Format based on type
{{ typeof $json.value === "number" ? $json.value.toFixed(2) : $json.value }}
```

### Node Data Access

```javascript
// Get data from specific node
{{ $node["HTTP Request"].json.data.id }}

// Check if node executed
{{ $node["Optional Step"]?.json != null }}

// Merge data from multiple nodes
{{
  {
    user: $node["Get User"].json,
    orders: $node["Get Orders"].json.items,
    total: $node["Calculate"].json.total
  }
}}
```

### Error Prevention

```javascript
// Safe number parsing
{{ parseInt($json.count) || 0 }}

// Safe string operations
{{ ($json.name || "").toUpperCase() }}

// Safe array access
{{ ($json.items || [])[0] || {} }}

// Validate before use
{{ $json.email?.includes("@") ? $json.email : null }}
```

---

## Quick Reference
<!-- chunk: 13-quick-ref | keywords: quick, reference, cheatsheet | source: analysis -->

### Variables
| Variable | Description |
|----------|-------------|
| `$json` | Current item data |
| `$input` | Input methods |
| `$node["Name"]` | Other node data |
| `$workflow` | Workflow metadata |
| `$env` | Environment variables |
| `$now` | Current DateTime |
| `$today` | Today's date |
| `$execution` | Execution info |
| `$itemIndex` | Current item index |
| `$runIndex` | Current run index |
| `$vars` | Workflow variables |

### Common Methods
| Method | Type | Description |
|--------|------|-------------|
| `.toUpperCase()` | String | Convert to uppercase |
| `.toLowerCase()` | String | Convert to lowercase |
| `.trim()` | String | Remove whitespace |
| `.split(",")` | String | Split to array |
| `.replace(a, b)` | String | Replace text |
| `.first()` | Array | Get first item |
| `.last()` | Array | Get last item |
| `.length` | Array/String | Get length |
| `.filter(fn)` | Array | Filter items |
| `.map(fn)` | Array | Transform items |
| `.join(",")` | Array | Join to string |
| `.round(2)` | Number | Round to decimals |
| `.toFormat(fmt)` | DateTime | Format date |
| `.plus(n, unit)` | DateTime | Add time |
| `.minus(n, unit)` | DateTime | Subtract time |

→ Workflow JSON: [[12_WORKFLOW_JSON_SCHEMA]]
→ Node Reference: [[13_NODE_REFERENCE]]
→ Templates: [[16_WORKFLOW_TEMPLATES]]
