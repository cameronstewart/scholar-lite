# Schema Storage & LLM Content Generation: Architecture Guide

## Your Question

**"Should there be a database that stores type rules, example data, metadata, and predefined lists (for LLM content generation)?"**

**Short Answer:** It depends on your use case. Let's explore both approaches.

---

## Current Approach: Schemas in Code

### What You Have Now (Astro)

```typescript
// src/content.config.ts
const publications = defineCollection({
  schema: z.object({
    title: z.string(),
    authors: z.array(z.string()),
    year: z.number(),
    venue: z.string(),
    type: z.enum(['paper', 'book', 'patent', 'software']), // ← Predefined list
    // ... more fields
  }),
});
```

**Characteristics:**
- ✅ Schema is in **code** (TypeScript file)
- ✅ Predefined lists (enums) are **hardcoded**
- ✅ Validated at **build time**
- ✅ Content stored as **files** (Markdown)

---

## Alternative: Schema in Database

### Database-Driven Content Model

```javascript
// Database: Content Schema Table
┌─────────────────────────────────────────────────────────┐
│ content_schemas                                         │
├─────────────┬─────────────┬──────────────┬─────────────┤
│ collection  │ field_name  │ field_type   │ required    │
├─────────────┼─────────────┼──────────────┼─────────────┤
│ publication │ title       │ string       │ true        │
│ publication │ authors     │ array        │ true        │
│ publication │ year        │ number       │ true        │
│ publication │ venue       │ string       │ true        │
│ publication │ type        │ enum         │ true        │
└─────────────┴─────────────┴──────────────┴─────────────┘

// Database: Enum Values Table
┌─────────────────────────────────────────────────────────┐
│ enum_values                                             │
├─────────────┬─────────────┬───────────────────────────┤
│ field       │ value       │ display_label             │
├─────────────┼─────────────┼───────────────────────────┤
│ pub.type    │ paper       │ Conference/Journal Paper  │
│ pub.type    │ book        │ Book Chapter              │
│ pub.type    │ patent      │ Patent                    │
│ pub.type    │ software    │ Software Copyright        │
│ team.role   │ professor   │ Professor                 │
│ team.role   │ phd         │ PhD Student               │
└─────────────┴─────────────┴───────────────────────────┘

// Database: Content Table
┌─────────────────────────────────────────────────────────┐
│ publications                                            │
├──────┬─────────────────┬──────────────┬──────────┬────┤
│ id   │ title           │ authors      │ year     │... │
├──────┼─────────────────┼──────────────┼──────────┼────┤
│ 1    │ Diffusion...    │ ["Emily"...] │ 2024     │... │
│ 2    │ Attention...    │ ["David"...] │ 2023     │... │
└──────┴─────────────────┴──────────────┴──────────┴────┘
```

---

## Comparison: Code vs Database Schemas

### Approach 1: Schemas in Code (Current)

**Structure:**
```
Project
├── src/content.config.ts      # Schema definitions (code)
├── src/content/
│   └── publications/
│       ├── paper-1.md         # Content (files)
│       └── paper-2.md
```

**Pros:**
- ✅ Version controlled (Git tracks schema changes)
- ✅ Type-safe (TypeScript checks at compile time)
- ✅ Fast (no database queries)
- ✅ Portable (just files)
- ✅ Works offline
- ✅ Easy to review (see schema in IDE)
- ✅ No database infrastructure needed

**Cons:**
- ❌ Can't change schema without redeploying code
- ❌ Can't add new field types dynamically
- ❌ Harder for non-developers to modify schema
- ❌ Can't share schema across multiple projects easily

---

### Approach 2: Schemas in Database

**Structure:**
```
Project
├── src/api/
│   └── schema-service.ts      # Fetch schemas from DB
├── database/
│   ├── content_schemas        # Schema definitions (DB)
│   ├── enum_values            # Predefined lists (DB)
│   └── publications           # Content (DB)
```

**Pros:**
- ✅ Dynamic schema changes (no redeployment)
- ✅ Schema shared across multiple apps
- ✅ Non-developers can modify schema (admin UI)
- ✅ Audit trail (who changed schema when)
- ✅ Programmatic schema generation
- ✅ Better for LLM integration (LLM queries DB for schema)

**Cons:**
- ❌ Requires database infrastructure
- ❌ Network latency (must fetch schema)
- ❌ More complex architecture
- ❌ Harder to version control schema changes
- ❌ No compile-time type safety
- ❌ Doesn't work offline

---

## The Hybrid Approach (Best of Both Worlds)

### Architecture: Code + Database

```
Schema Storage (Database):
- Field definitions
- Validation rules
- Enum values
- Example data
- LLM prompts

↓ Build Time ↓

Generate Code:
- TypeScript types
- Zod schemas
- Type-safe queries

↓ Runtime ↓

Static Site:
- Markdown files
- Type-checked content
- Fast builds
```

**Implementation:**

```typescript
// 1. Store schema in database (Postgres/SQLite/JSON file)
{
  "collections": {
    "publications": {
      "fields": {
        "title": {
          "type": "string",
          "required": true,
          "validation": {
            "minLength": 5,
            "maxLength": 200
          },
          "llm_prompt": "Generate an academic paper title (5-200 chars)"
        },
        "type": {
          "type": "enum",
          "required": true,
          "values": ["paper", "book", "patent", "software"],
          "labels": {
            "paper": "Conference/Journal Paper",
            "book": "Book Chapter",
            "patent": "Patent",
            "software": "Software Copyright"
          },
          "llm_prompt": "Select one: paper, book, patent, or software"
        },
        "year": {
          "type": "number",
          "required": true,
          "validation": {
            "min": 1900,
            "max": 2100
          },
          "llm_prompt": "Publication year as a 4-digit number (1900-2100)"
        }
      },
      "examples": [
        {
          "title": "High-Fidelity Image Generation with Latent Diffusion Models",
          "type": "paper",
          "year": 2024,
          "authors": ["Emily Chen", "David Wang"],
          "venue": "NeurIPS 2024"
        }
      ]
    }
  }
}

// 2. Build-time script: Generate TypeScript from schema
// scripts/generate-schemas.ts
import { schemaDB } from './schema-db';

function generateZodSchema(collection: string) {
  const schema = schemaDB.getCollection(collection);

  let code = `const ${collection} = defineCollection({\n`;
  code += `  schema: z.object({\n`;

  for (const [field, config] of Object.entries(schema.fields)) {
    if (config.type === 'string') {
      code += `    ${field}: z.string()`;
      if (config.validation?.minLength) {
        code += `.min(${config.validation.minLength})`;
      }
      if (config.validation?.maxLength) {
        code += `.max(${config.validation.maxLength})`;
      }
    } else if (config.type === 'enum') {
      code += `    ${field}: z.enum([${config.values.map(v => `'${v}'`).join(', ')}])`;
    } else if (config.type === 'number') {
      code += `    ${field}: z.number()`;
      if (config.validation?.min) {
        code += `.min(${config.validation.min})`;
      }
      if (config.validation?.max) {
        code += `.max(${config.validation.max})`;
      }
    }

    if (!config.required) {
      code += `.optional()`;
    }

    code += `,\n`;
  }

  code += `  })\n});`;

  return code;
}

// Output: src/content.config.ts (auto-generated)
const publications = defineCollection({
  schema: z.object({
    title: z.string().min(5).max(200),
    type: z.enum(['paper', 'book', 'patent', 'software']),
    year: z.number().min(1900).max(2100),
    // ... auto-generated from database
  })
});

// 3. LLM can query schema for content generation
async function generatePublication(userPrompt: string) {
  const schema = await schemaDB.getCollection('publications');
  const examples = schema.examples;

  const llmPrompt = `
You are generating academic publication metadata.

Schema:
${JSON.stringify(schema.fields, null, 2)}

Examples:
${JSON.stringify(examples, null, 2)}

User request: ${userPrompt}

Generate valid publication metadata as JSON:
  `;

  const result = await llm.generate(llmPrompt);

  // Validate against schema before saving
  const validated = publicationSchema.parse(JSON.parse(result));

  return validated;
}
```

---

## Predefined Lists (Enums): Where to Store?

### Option 1: Hardcoded in Schema (Simple)

```typescript
// src/content.config.ts
type: z.enum(['paper', 'book', 'patent', 'software'])
```

**Best for:**
- ✅ Fixed lists that rarely change
- ✅ Small number of values (< 20)
- ✅ Type safety is critical

**Examples:**
- Publication types (paper, book, patent)
- Team roles (professor, PhD, postdoc)
- Badge types (gold, blue, red)

---

### Option 2: Database/Config File (Flexible)

```json
// config/enums.json
{
  "publication_types": [
    { "value": "paper", "label": "Conference/Journal Paper" },
    { "value": "book", "label": "Book Chapter" },
    { "value": "patent", "label": "Patent" },
    { "value": "software", "label": "Software Copyright" }
  ],
  "team_roles": [
    { "value": "professor", "label": "Professor" },
    { "value": "phd", "label": "PhD Student" },
    { "value": "postdoc", "label": "Postdoctoral Researcher" }
  ],
  "research_areas": [
    { "value": "cv", "label": "Computer Vision" },
    { "value": "nlp", "label": "Natural Language Processing" },
    { "value": "rl", "label": "Reinforcement Learning" }
  ]
}
```

**Best for:**
- ✅ Lists that change occasionally
- ✅ Need labels/translations
- ✅ Want to manage via UI
- ✅ Shared across projects

**Examples:**
- Conference venues (grows over time)
- Research areas (new fields emerge)
- Countries (occasionally new ones)

---

### Option 3: External API (Dynamic)

```typescript
// Fetch from external source
const conferences = await fetch('https://dblp.org/api/conferences')
const venues = conferences.map(c => c.name)

// Use in validation
venue: z.enum(venues)
```

**Best for:**
- ✅ Large lists (1000+ values)
- ✅ Frequently updated
- ✅ Authoritative source exists

**Examples:**
- Conference venues (DBLP database)
- Journal names (Crossref API)
- Author names (ORCID)
- Countries (ISO 3166 standard)

---

## LLM Content Generation: Architecture

### Use Case: Generate Publications with LLM

**Scenario:** User says "Add a paper about diffusion models published at NeurIPS 2024"

---

### Approach 1: LLM Generates Against Schema

```typescript
// 1. Load schema (from code or database)
const schema = {
  title: { type: 'string', required: true, minLength: 5, maxLength: 200 },
  authors: { type: 'array', required: true, items: { type: 'string' } },
  year: { type: 'number', required: true, min: 1900, max: 2100 },
  venue: { type: 'string', required: true },
  type: { type: 'enum', values: ['paper', 'book', 'patent', 'software'] },
};

// 2. Provide schema + examples to LLM
const prompt = `
Generate publication metadata matching this schema:
${JSON.stringify(schema, null, 2)}

Example:
{
  "title": "High-Fidelity Image Generation with Latent Diffusion Models",
  "authors": ["Emily Chen", "David Wang", "Michael Brown"],
  "year": 2024,
  "venue": "NeurIPS 2024",
  "type": "paper"
}

User request: "Add a paper about diffusion models published at NeurIPS 2024"

Generate valid JSON:
`;

// 3. LLM generates
const llmOutput = await llm.generate(prompt);
// {
//   "title": "Denoising Diffusion Probabilistic Models for Image Synthesis",
//   "authors": ["Generated Author"],
//   "year": 2024,
//   "venue": "NeurIPS 2024",
//   "type": "paper"
// }

// 4. Validate against schema
try {
  const validated = publicationSchema.parse(JSON.parse(llmOutput));
  // ✅ Valid! Save it
  await savePublication(validated);
} catch (error) {
  // ❌ Invalid! Ask LLM to fix
  console.error('Validation failed:', error);
  // Retry with error feedback...
}
```

---

### Approach 2: LLM Queries Schema Database

**Better for dynamic schemas:**

```typescript
// schema-service.ts
class SchemaService {
  async getSchema(collection: string) {
    // Fetch from database
    return await db.query(`
      SELECT field_name, field_type, required, validation_rules
      FROM content_schemas
      WHERE collection = $1
    `, [collection]);
  }

  async getEnumValues(field: string) {
    return await db.query(`
      SELECT value, display_label, description
      FROM enum_values
      WHERE field = $1
    `, [field]);
  }

  async getExamples(collection: string, limit = 3) {
    return await db.query(`
      SELECT *
      FROM ${collection}
      ORDER BY created_at DESC
      LIMIT $1
    `, [limit]);
  }

  async buildLLMPrompt(collection: string, userRequest: string) {
    const schema = await this.getSchema(collection);
    const examples = await this.getExamples(collection);

    // Build enum information
    const enums = {};
    for (const field of schema.filter(f => f.field_type === 'enum')) {
      enums[field.field_name] = await this.getEnumValues(field.field_name);
    }

    return `
You are generating ${collection} metadata.

Schema:
${this.formatSchema(schema)}

Valid enum values:
${JSON.stringify(enums, null, 2)}

Recent examples:
${JSON.stringify(examples, null, 2)}

User request: ${userRequest}

Generate valid JSON matching the schema above:
    `;
  }
}

// Usage
const schemaService = new SchemaService();

async function generateContent(collection: string, userRequest: string) {
  const prompt = await schemaService.buildLLMPrompt(collection, userRequest);
  const llmOutput = await llm.generate(prompt);

  // Validate
  const validated = await schemaService.validate(collection, JSON.parse(llmOutput));

  return validated;
}

// Example
const newPub = await generateContent(
  'publications',
  'Add a paper about diffusion models at NeurIPS 2024'
);
```

---

## Controlled Vocabularies for LLM

### Problem: LLMs Make Up Values

**Without controlled vocabulary:**

```typescript
// LLM generates
{
  "type": "research paper"  // ❌ Invalid! Should be "paper"
  "venue": "NeurIPS"        // ⚠️ Inconsistent! Should be "NeurIPS 2024"
  "year": "2024"            // ❌ Wrong type! Should be number 2024
}
```

---

### Solution 1: Strict Enums

```typescript
// Define allowed values
const PUBLICATION_TYPES = ['paper', 'book', 'patent', 'software'];
const TEAM_ROLES = ['Professor', 'PhD Student', 'Postdoc', 'Master Student'];

// In LLM prompt
const prompt = `
Field "type" must be EXACTLY one of: ${PUBLICATION_TYPES.join(', ')}
Field "role" must be EXACTLY one of: ${TEAM_ROLES.join(', ')}

IMPORTANT: Use exact values from the lists above. Do not create new values.
`;
```

---

### Solution 2: Fuzzy Matching + Correction

```typescript
// LLM generates close-enough values
const llmOutput = {
  type: "research paper",  // Close to "paper"
  role: "PhD"              // Close to "PhD Student"
};

// Auto-correct with fuzzy matching
function correctEnum(value: string, allowedValues: string[]) {
  // Exact match
  if (allowedValues.includes(value)) return value;

  // Fuzzy match
  const normalized = value.toLowerCase().trim();
  for (const allowed of allowedValues) {
    if (allowed.toLowerCase().includes(normalized) ||
        normalized.includes(allowed.toLowerCase())) {
      return allowed;
    }
  }

  // No match
  throw new Error(`Invalid value "${value}". Must be one of: ${allowedValues.join(', ')}`);
}

// Apply corrections
corrected = {
  type: correctEnum(llmOutput.type, PUBLICATION_TYPES),  // "research paper" → "paper"
  role: correctEnum(llmOutput.role, TEAM_ROLES)          // "PhD" → "PhD Student"
};
```

---

### Solution 3: Database of Valid Values

```sql
-- Store all valid conference venues
CREATE TABLE conference_venues (
  id SERIAL PRIMARY KEY,
  full_name TEXT NOT NULL,  -- "Conference on Neural Information Processing Systems"
  short_name TEXT NOT NULL, -- "NeurIPS"
  aliases TEXT[],           -- ["NIPS", "Neural IPS"]
  typical_format TEXT       -- "NeurIPS 2024"
);

INSERT INTO conference_venues (full_name, short_name, aliases, typical_format)
VALUES
  ('Conference on Neural Information Processing Systems', 'NeurIPS',
   ARRAY['NIPS', 'Neural IPS'], 'NeurIPS YYYY'),
  ('International Conference on Machine Learning', 'ICML',
   ARRAY['ICML Conference'], 'ICML YYYY');
```

```typescript
// Validate venue against database
async function validateVenue(llmVenue: string): Promise<string> {
  const match = await db.query(`
    SELECT typical_format
    FROM conference_venues
    WHERE
      short_name ILIKE $1 OR
      full_name ILIKE $1 OR
      $1 = ANY(aliases)
  `, [llmVenue]);

  if (match.rows.length === 0) {
    throw new Error(`Unknown venue: ${llmVenue}`);
  }

  return match.rows[0].typical_format.replace('YYYY', extractYear(llmVenue));
}

// LLM says "NeurIPS" → Database corrects to "NeurIPS 2024"
// LLM says "NIPS" (old name) → Database corrects to "NeurIPS 2024"
```

---

## Recommendation for Your Use Case

### Your Requirements:
1. ✅ Academic portfolio (35 items, growing)
2. ✅ 8 languages
3. ✅ LLM content generation (future)
4. ✅ Type safety
5. ✅ Occasional schema changes

---

### Recommended Architecture: **Hybrid**

```
1. Schema in JSON file (version controlled)
   ├── config/schemas/publications.json
   ├── config/schemas/team.json
   └── config/enums/
       ├── publication-types.json
       ├── team-roles.json
       └── venues.json

2. Build-time code generation
   └── scripts/generate-schemas.ts
       → Generates src/content.config.ts

3. LLM integration
   └── scripts/llm-content-generator.ts
       → Reads schemas from JSON
       → Validates against Zod schemas
```

**Benefits:**
- ✅ Version controlled (Git)
- ✅ Easy to modify (JSON files)
- ✅ Type-safe at build time (generated Zod)
- ✅ LLM can read schemas (JSON)
- ✅ No database infrastructure needed
- ✅ Works offline

---

### Implementation

#### Step 1: Schema Definition (JSON)

```json
// config/schemas/publication.schema.json
{
  "collection": "publications",
  "description": "Research publications (papers, books, patents)",
  "fields": {
    "title": {
      "type": "string",
      "required": true,
      "validation": {
        "minLength": 5,
        "maxLength": 200
      },
      "description": "Publication title",
      "llmHint": "Generate a concise academic paper title (5-200 characters)"
    },
    "authors": {
      "type": "array",
      "items": { "type": "string" },
      "required": true,
      "description": "List of author names",
      "llmHint": "Generate realistic author names (first + last)"
    },
    "year": {
      "type": "number",
      "required": true,
      "validation": {
        "min": 1900,
        "max": 2100
      },
      "description": "Publication year",
      "llmHint": "4-digit year between 1900-2100"
    },
    "venue": {
      "type": "string",
      "required": true,
      "enumRef": "publication-venues",
      "description": "Conference or journal name",
      "llmHint": "Use format 'ConferenceName YYYY' (e.g., 'NeurIPS 2024')"
    },
    "type": {
      "type": "enum",
      "required": true,
      "enumRef": "publication-types",
      "description": "Publication type",
      "llmHint": "Must be exactly one of the allowed values"
    }
  },
  "examples": [
    {
      "title": "High-Fidelity Image Generation with Latent Diffusion Models",
      "authors": ["Emily Chen", "David Wang", "Michael Brown"],
      "year": 2024,
      "venue": "NeurIPS 2024",
      "type": "paper"
    }
  ]
}
```

#### Step 2: Enum Definitions

```json
// config/enums/publication-types.json
{
  "enum": "publication-types",
  "values": [
    {
      "value": "paper",
      "label": "Conference/Journal Paper",
      "description": "Peer-reviewed conference or journal publication"
    },
    {
      "value": "book",
      "label": "Book Chapter",
      "description": "Chapter in an edited book"
    },
    {
      "value": "patent",
      "label": "Patent",
      "description": "Invention patent"
    },
    {
      "value": "software",
      "label": "Software Copyright",
      "description": "Software copyright registration"
    }
  ],
  "llmHint": "Select EXACTLY one value from the list (do not create new values)"
}
```

```json
// config/enums/publication-venues.json
{
  "enum": "publication-venues",
  "allowFreeform": false,
  "suggestedValues": [
    {
      "value": "NeurIPS 2024",
      "aliases": ["NIPS 2024", "Neural IPS 2024"],
      "fullName": "Conference on Neural Information Processing Systems 2024"
    },
    {
      "value": "ICML 2024",
      "aliases": ["International Conference on Machine Learning 2024"],
      "fullName": "International Conference on Machine Learning 2024"
    },
    {
      "value": "CVPR 2024",
      "aliases": ["Computer Vision and Pattern Recognition 2024"],
      "fullName": "IEEE/CVF Conference on Computer Vision and Pattern Recognition 2024"
    }
  ],
  "llmHint": "Use suggested values if available, or format as 'ConferenceName YYYY'"
}
```

#### Step 3: Code Generator

```typescript
// scripts/generate-schemas.ts
import fs from 'fs';
import path from 'path';

interface SchemaDefinition {
  collection: string;
  fields: Record<string, FieldDefinition>;
}

interface FieldDefinition {
  type: string;
  required: boolean;
  validation?: any;
  enumRef?: string;
}

function generateZodSchema(schemaPath: string): string {
  const schema: SchemaDefinition = JSON.parse(fs.readFileSync(schemaPath, 'utf-8'));
  const enums = loadEnums();

  let code = `const ${schema.collection} = defineCollection({\n`;
  code += `  loader: glob({ pattern: "**/*.{md,mdx}", base: "./src/content/${schema.collection}" }),\n`;
  code += `  schema: ({ image }) => z.object({\n`;

  for (const [fieldName, fieldDef] of Object.entries(schema.fields)) {
    code += `    ${fieldName}: `;

    switch (fieldDef.type) {
      case 'string':
        code += `z.string()`;
        if (fieldDef.validation?.minLength) {
          code += `.min(${fieldDef.validation.minLength})`;
        }
        if (fieldDef.validation?.maxLength) {
          code += `.max(${fieldDef.validation.maxLength})`;
        }
        break;

      case 'number':
        code += `z.number()`;
        if (fieldDef.validation?.min) {
          code += `.min(${fieldDef.validation.min})`;
        }
        if (fieldDef.validation?.max) {
          code += `.max(${fieldDef.validation.max})`;
        }
        break;

      case 'array':
        code += `z.array(z.string())`;
        break;

      case 'enum':
        if (fieldDef.enumRef) {
          const enumValues = enums[fieldDef.enumRef].values.map(v => v.value);
          code += `z.enum([${enumValues.map(v => `'${v}'`).join(', ')}])`;
        }
        break;

      case 'image':
        code += `image()`;
        break;
    }

    if (!fieldDef.required) {
      code += `.optional()`;
    }

    code += `,\n`;
  }

  code += `  })\n});\n\n`;

  return code;
}

function loadEnums(): Record<string, any> {
  const enumsDir = path.join(__dirname, '../config/enums');
  const enums = {};

  for (const file of fs.readdirSync(enumsDir)) {
    const enumData = JSON.parse(fs.readFileSync(path.join(enumsDir, file), 'utf-8'));
    enums[enumData.enum] = enumData;
  }

  return enums;
}

// Generate all schemas
const schemasDir = path.join(__dirname, '../config/schemas');
let output = `import { defineCollection, z } from 'astro:content';\n`;
output += `import { glob } from 'astro/loaders';\n\n`;
output += `// AUTO-GENERATED - DO NOT EDIT\n`;
output += `// Generated from config/schemas/*.json\n\n`;

for (const file of fs.readdirSync(schemasDir)) {
  output += generateZodSchema(path.join(schemasDir, file));
}

output += `export const collections = {\n`;
const collections = fs.readdirSync(schemasDir).map(f => path.basename(f, '.schema.json'));
output += collections.map(c => `  ${c},`).join('\n');
output += `\n};\n`;

fs.writeFileSync(
  path.join(__dirname, '../src/content.config.ts'),
  output
);

console.log('✅ Generated src/content.config.ts');
```

#### Step 4: LLM Integration

```typescript
// scripts/llm-content-generator.ts
import fs from 'fs';
import path from 'path';

class ContentGenerator {
  private schemas: Map<string, any> = new Map();
  private enums: Map<string, any> = new Map();

  constructor() {
    this.loadSchemas();
    this.loadEnums();
  }

  private loadSchemas() {
    const schemasDir = path.join(__dirname, '../config/schemas');
    for (const file of fs.readdirSync(schemasDir)) {
      const schema = JSON.parse(fs.readFileSync(path.join(schemasDir, file), 'utf-8'));
      this.schemas.set(schema.collection, schema);
    }
  }

  private loadEnums() {
    const enumsDir = path.join(__dirname, '../config/enums');
    for (const file of fs.readdirSync(enumsDir)) {
      const enumData = JSON.parse(fs.readFileSync(path.join(enumsDir, file), 'utf-8'));
      this.enums.set(enumData.enum, enumData);
    }
  }

  async generate(collection: string, userPrompt: string): Promise<any> {
    const schema = this.schemas.get(collection);
    if (!schema) {
      throw new Error(`Unknown collection: ${collection}`);
    }

    const prompt = this.buildPrompt(schema, userPrompt);
    const llmOutput = await this.callLLM(prompt);
    const validated = await this.validate(collection, llmOutput);

    return validated;
  }

  private buildPrompt(schema: any, userRequest: string): string {
    let prompt = `You are generating ${schema.collection} metadata.\n\n`;

    prompt += `Schema:\n`;
    for (const [field, def] of Object.entries(schema.fields)) {
      prompt += `- ${field} (${def.type}${def.required ? ', required' : ', optional'}): ${def.description}\n`;
      if (def.llmHint) {
        prompt += `  Hint: ${def.llmHint}\n`;
      }
      if (def.enumRef) {
        const enumData = this.enums.get(def.enumRef);
        if (enumData) {
          prompt += `  Allowed values: ${enumData.values.map(v => v.value).join(', ')}\n`;
          if (enumData.llmHint) {
            prompt += `  ${enumData.llmHint}\n`;
          }
        }
      }
    }

    prompt += `\nExamples:\n`;
    for (const example of schema.examples || []) {
      prompt += JSON.stringify(example, null, 2) + '\n\n';
    }

    prompt += `User request: ${userRequest}\n\n`;
    prompt += `Generate valid JSON matching the schema above:\n`;

    return prompt;
  }

  private async callLLM(prompt: string): Promise<any> {
    // Call your LLM API (OpenAI, Anthropic, etc.)
    // For demo purposes, just return mock data
    console.log('LLM Prompt:\n', prompt);

    // In real implementation:
    // const response = await openai.chat.completions.create({
    //   model: 'gpt-4',
    //   messages: [{ role: 'user', content: prompt }],
    //   response_format: { type: 'json_object' }
    // });
    // return JSON.parse(response.choices[0].message.content);

    return {
      title: "Denoising Diffusion Probabilistic Models",
      authors: ["Jonathan Ho", "Ajay Jain", "Pieter Abbeel"],
      year: 2024,
      venue: "NeurIPS 2024",
      type: "paper"
    };
  }

  private async validate(collection: string, data: any): Promise<any> {
    // Import the generated Zod schema and validate
    const { collections } = await import('../src/content.config');
    const schema = collections[collection].schema({ image: () => z.any() });

    try {
      return schema.parse(data);
    } catch (error) {
      console.error('Validation failed:', error);
      throw error;
    }
  }
}

// Usage
const generator = new ContentGenerator();

const newPublication = await generator.generate(
  'publications',
  'Add a paper about diffusion models published at NeurIPS 2024'
);

console.log('Generated publication:', newPublication);
```

---

## Summary: Your Best Approach

### **Hybrid: JSON Schema + Generated Code**

```
config/
├── schemas/
│   ├── publication.schema.json  ← Define schema here
│   ├── team.schema.json
│   └── ...
├── enums/
│   ├── publication-types.json   ← Define allowed values
│   ├── team-roles.json
│   └── venues.json
└── examples/
    └── publication-examples.json  ← Example data for LLM

scripts/
├── generate-schemas.ts           ← Generates Zod schemas
└── llm-content-generator.ts      ← LLM integration

src/
└── content.config.ts             ← AUTO-GENERATED (don't edit)

package.json:
  "scripts": {
    "generate:schemas": "tsx scripts/generate-schemas.ts",
    "prebuild": "npm run generate:schemas"
  }
```

### Workflow

1. **Define schema** in JSON (human-readable, version-controlled)
2. **Run generator** to create TypeScript types
3. **LLM reads** JSON schemas to generate content
4. **Validate** LLM output against Zod schemas
5. **Build** site with type-safe content

### Benefits

✅ Version controlled (Git tracks schema changes)
✅ Type-safe (Zod validation at build time)
✅ LLM-friendly (JSON schemas easy to parse)
✅ No database needed
✅ Flexible (easy to modify schemas)
✅ Portable (just JSON files)

---

## Next Steps

Want me to implement this hybrid approach for your project? I can:

1. Create JSON schema definitions for your 9 collections
2. Build the code generator script
3. Set up LLM content generation
4. Add controlled vocabularies for common fields

Let me know!
