# Scholar-Lite: Add Author Attribution

This skill adds author attribution to news and activities collections in Scholar-Lite based academic websites.

## What This Skill Does

1. Updates `src/content.config.ts` to add `authors` field to news and activities schemas
2. Creates `src/components/AuthorByline.astro` component with smart team member resolution
3. Updates news and activities page templates to display author bylines
4. Provides examples of usage in markdown frontmatter

## When to Use This Skill

Use this skill when:
- You have a Scholar-Lite based academic website
- You want to credit authors of news articles and activity posts
- You want to link internal authors to their team profiles
- You need to credit external collaborators

## Implementation

The skill will:

### 1. Update Content Schema

Add optional authors field to news and activities collections:

```typescript
const news = defineCollection({
  schema: z.object({
    title: z.string(),
    date: z.date(),
    summary: z.string().optional(),
    authors: z.array(z.string()).optional(), // NEW
  }),
});

const activities = defineCollection({
  schema: ({ image }) => z.object({
    title: z.string(),
    date: z.date(),
    cover: image().optional(),
    description: z.string().optional(),
    authors: z.array(z.string()).optional(), // NEW
  }),
});
```

### 2. Create AuthorByline Component

Creates a reusable component that:
- Takes an array of author strings
- Attempts to resolve each as a team member slug
- Links to team profiles if found
- Displays as plain text if not found
- Handles multiple authors with proper formatting

### 3. Update Templates

Integrates the AuthorByline component into:
- News detail pages
- Activities detail pages
- Optionally: listing cards

### 4. Provide Examples

Shows how to use in markdown:

```markdown
---
title: "Research Update"
date: 2025-12-15
authors: ["prof-sarah-johnson", "phd-emily-chen"]
---
```

## Prerequisites

- Scholar-Lite theme (https://github.com/fjd2004711/scholar-lite)
- Team collection with member profiles
- Team member slugs following convention (e.g., `prof-name`, `phd-name`)

## Usage in Articles

### Internal Authors (Team Members)
```markdown
authors: ["prof-sarah-johnson", "phd-wei-zhang"]
```
Renders as: "By Prof. Sarah Johnson and Wei Zhang" (both linked)

### External Authors
```markdown
authors: ["Dr. John Smith (MIT)", "Prof. Jane Doe (Harvard)"]
```
Renders as: "By Dr. John Smith (MIT) and Prof. Jane Doe (Harvard)" (plain text)

### Mixed
```markdown
authors: ["prof-sarah-johnson", "Dr. External Collaborator (Stanford)"]
```
Renders as: "By Prof. Sarah Johnson and Dr. External Collaborator (Stanford)" (first linked, second plain)

## Backward Compatibility

The `authors` field is optional, so existing content without authors continues to work normally.

## Testing

After implementation, test:
1. Article with single internal author → links to profile
2. Article with multiple internal authors → all linked, comma-separated
3. Article with external author → displays as text
4. Article with mixed authors → internal linked, external text
5. Article without authors → no byline shown
6. Invalid team slug → falls back to plain text

## Future Enhancements

This implementation enables:
- Author archive pages
- Filtering by author
- Author statistics
- Search by author

But those are deferred to future iterations.
