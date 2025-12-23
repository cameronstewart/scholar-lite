# Feature Request: Add Author Tracking to News and Activities Collections

## Problem

The `news` and `activities` content collections currently do not track authors, while other collections like `publications`, `books`, `patents`, and `softwares` do have author/contributor tracking.

### Current Schema

**News collection** (src/content.config.ts:86-93):
```typescript
const news = defineCollection({
  schema: z.object({
    title: z.string(),
    date: z.date(),
    summary: z.string().optional(),
  }),
});
```

**Activities collection** (src/content.config.ts:140-148):
```typescript
const activities = defineCollection({
  schema: ({ image }) => z.object({
    title: z.string(),
    date: z.date(),
    cover: image().optional(),
    description: z.string().optional(),
  }),
});
```

## Impact

Without author tracking, it's impossible to:
- Credit the author/writer of news articles
- Identify who organized or led activities
- Filter content by author
- Display author information on article pages
- Link articles to team members

## Proposed Solutions

### Option 1: Simple String Author (Quick Fix)
Add an optional author field as a string:

```typescript
const news = defineCollection({
  schema: z.object({
    title: z.string(),
    date: z.date(),
    summary: z.string().optional(),
    author: z.string().optional(), // NEW
  }),
});
```

**Pros:** Simple, quick to implement, flexible
**Cons:** No validation, no automatic linking to team members

### Option 2: Author Array (Consistent with Publications)
Match the pattern used in publications:

```typescript
const news = defineCollection({
  schema: z.object({
    title: z.string(),
    date: z.date(),
    summary: z.string().optional(),
    authors: z.array(z.string()).optional(), // NEW
  }),
});
```

**Pros:** Supports multiple authors, consistent with other collections
**Cons:** Still no automatic linking to team members

### Option 3: Reference Team Members (Advanced)
Link directly to team member entries:

```typescript
const news = defineCollection({
  schema: z.object({
    title: z.string(),
    date: z.date(),
    summary: z.string().optional(),
    authors: z.array(z.string()).optional(), // Team member slugs
  }),
});
```

Then in templates, look up team members by slug to display full profiles.

**Pros:** Strong typing, automatic linking, rich author data
**Cons:** More complex, requires template updates

### Option 4: Hybrid Approach (Recommended)
Allow both team references and free-form names:

```typescript
const news = defineCollection({
  schema: z.object({
    title: z.string(),
    date: z.date(),
    summary: z.string().optional(),
    authors: z.array(z.string()).optional(), // Can be team slugs or names
  }),
});

const activities = defineCollection({
  schema: ({ image }) => z.object({
    title: z.string(),
    date: z.date(),
    cover: image().optional(),
    description: z.string().optional(),
    authors: z.array(z.string()).optional(), // Can be team slugs or names
  }),
});
```

In templates, try to resolve as team member first, fallback to plain text.

**Pros:** Flexible, backward compatible, supports internal and external authors
**Cons:** Requires smart template logic

## Recommendation

I recommend **Option 4 (Hybrid)** for maximum flexibility, with the following implementation:

1. Add `authors: z.array(z.string()).optional()` to both collections
2. Update templates to try resolving as team member slug first
3. Display as plain text if not found in team collection
4. Make it optional to maintain backward compatibility

## Template Changes Needed

- Update news listing pages to display authors
- Update activity listing pages to display organizers/leaders
- Add author byline to individual article pages
- Optionally add filtering/search by author

## Example Usage

```markdown
---
title: "Lab Wins Best Paper Award"
date: 2025-12-15
summary: "Our research was recognized at CVPR 2025"
authors: ["prof-sarah-johnson", "phd-emily-chen"]
---
```

Would render as: "By Prof. Sarah Johnson and Emily Chen" (with links to their profiles)

Or with external author:
```markdown
---
title: "Research Collaboration Announced"
date: 2025-12-15
authors: ["prof-sarah-johnson", "Dr. John Smith (MIT)"]
---
```

Would render as: "By Prof. Sarah Johnson and Dr. John Smith (MIT)" (first linked, second plain text)

## Implementation Steps

1. Update `src/content.config.ts` to add authors field
2. Update news/activities page templates to display authors
3. Add helper function to resolve team member slugs
4. Update existing content to add author information
5. Update documentation

## Priority

**Medium-High** - This is a common feature for academic websites and would improve content attribution and discoverability.
