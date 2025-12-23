# User Story: Author Attribution for News and Activities

## User Story

**As a** lab administrator or content contributor
**I want** to attribute news articles and activity posts to specific authors
**So that** readers can identify who wrote the content and credit team members appropriately

## Background

Currently, the Scholar-Lite theme supports author tracking for publications, books, patents, and software, but not for news articles and activity posts. This creates an inconsistency in content attribution across the site.

## Personas

### Primary Users
- **Lab Administrator**: Manages website content, needs to credit team members
- **Research Team Members**: Write news and activity posts, want recognition for their contributions
- **Website Visitors**: Want to know who authored content and see author expertise

### Use Cases
1. Lab administrator publishes a news article about a grant award and credits the PI
2. PhD student writes about attending a conference and is credited as author
3. Multiple team members organize an event and are all credited
4. External collaborator contributes a guest post and is credited by name
5. Visitor clicks on an author's name to view their profile and other contributions

## Acceptance Criteria

### Must Have
- [ ] News collection schema includes optional `authors` field (array of strings)
- [ ] Activities collection schema includes optional `authors` field (array of strings)
- [ ] Templates display author byline on news article pages
- [ ] Templates display author byline on activity pages
- [ ] System attempts to resolve author strings as team member slugs
- [ ] If team member found, display name and link to profile
- [ ] If not found, display author string as plain text
- [ ] Authors field is optional (backward compatible with existing content)
- [ ] Multiple authors supported and displayed correctly

### Should Have
- [ ] Author names appear on news/activity listing cards
- [ ] Consistent styling for author bylines across all content types
- [ ] Helper function/component for author resolution and display
- [ ] Graceful handling when team member profile is incomplete

### Could Have
- [ ] Filter news/activities by author
- [ ] Author archive pages showing all content by an author
- [ ] Author hover cards with quick profile preview
- [ ] Search functionality includes author names

### Won't Have (This Iteration)
- Complex access control based on authors
- Author notification systems
- Co-author approval workflows

## Technical Implementation

### 1. Schema Changes (`src/content.config.ts`)

```typescript
const news = defineCollection({
  loader: glob({ pattern: "**/*.{md,mdx}", base: "./src/content/news" }),
  schema: z.object({
    title: z.string(),
    date: z.date(),
    summary: z.string().optional(),
    authors: z.array(z.string()).optional(), // NEW: Can be team slugs or names
  }),
});

const activities = defineCollection({
  loader: glob({ pattern: "**/*.{md,mdx}", base: "./src/content/activities" }),
  schema: ({ image }) => z.object({
    title: z.string(),
    date: z.date(),
    cover: image().optional(),
    description: z.string().optional(),
    summary: z.string().optional(), // NEW: For consistency with news
    authors: z.array(z.string()).optional(), // NEW: Can be team slugs or names
  }),
});
```

### 2. Helper Component/Function

Create `src/components/AuthorByline.astro`:

```typescript
---
import { getCollection } from 'astro:content';

interface Props {
  authors?: string[];
}

const { authors } = Astro.props;

if (!authors || authors.length === 0) {
  return null;
}

// Load team members once
const teamMembers = await getCollection('team');
const teamMemberMap = new Map(
  teamMembers.map(member => [member.id, member])
);

// Resolve each author
const resolvedAuthors = authors.map(authorString => {
  const teamMember = teamMemberMap.get(authorString);
  if (teamMember) {
    return {
      name: teamMember.data.name,
      slug: teamMember.id,
      isTeamMember: true,
    };
  }
  return {
    name: authorString,
    slug: null,
    isTeamMember: false,
  };
});
---

<div class="author-byline">
  <span class="author-label">By</span>
  {resolvedAuthors.map((author, index) => (
    <>
      {author.isTeamMember ? (
        <a href={`/team/${author.slug}`} class="author-link">
          {author.name}
        </a>
      ) : (
        <span class="author-name">{author.name}</span>
      )}
      {index < resolvedAuthors.length - 2 && <span>, </span>}
      {index === resolvedAuthors.length - 2 && <span> and </span>}
    </>
  ))}
</div>

<style>
  .author-byline {
    font-size: 0.875rem;
    color: var(--color-text-secondary);
    margin-bottom: 1rem;
  }

  .author-label {
    font-weight: 500;
    margin-right: 0.25rem;
  }

  .author-link {
    color: var(--color-primary);
    text-decoration: none;
  }

  .author-link:hover {
    text-decoration: underline;
  }

  .author-name {
    font-weight: 500;
  }
</style>
```

### 3. Template Updates

Update news/activities detail pages to include:

```astro
---
import AuthorByline from '@/components/AuthorByline.astro';
// ... other imports

const { entry } = Astro.props;
---

<article>
  <h1>{entry.data.title}</h1>
  <time>{formatDate(entry.data.date)}</time>
  <AuthorByline authors={entry.data.authors} />

  <!-- Article content -->
</article>
```

Update listing cards (optional):

```astro
{entry.data.authors && entry.data.authors.length > 0 && (
  <div class="card-authors">
    <AuthorByline authors={entry.data.authors} />
  </div>
)}
```

### 4. Content Updates

Update existing articles with author information:

```markdown
---
title: "Lab Wins Best Paper Award"
date: 2025-12-15
summary: "Our research was recognized at CVPR 2025"
authors: ["prof-sarah-johnson", "phd-emily-chen"]
---
```

For external authors:

```markdown
---
title: "Research Collaboration Announced"
date: 2025-12-15
authors: ["prof-sarah-johnson", "Dr. John Smith (MIT)"]
---
```

## Examples

### Example 1: Internal Authors Only
```markdown
---
title: "New GPU Cluster Installed"
date: 2025-08-20
authors: ["prof-sarah-johnson"]
---
```
**Renders as:** "By Prof. Sarah Johnson" (linked to profile)

### Example 2: Multiple Internal Authors
```markdown
---
title: "Lab Attends NeurIPS 2025"
date: 2025-12-10
authors: ["phd-emily-chen", "phd-wei-zhang", "master-alex-morgan"]
---
```
**Renders as:** "By Emily Chen, Wei Zhang and Alex Morgan" (all linked)

### Example 3: Mixed Internal and External
```markdown
---
title: "MIT Collaboration Announced"
date: 2025-12-15
authors: ["prof-sarah-johnson", "Prof. Michael Brown (MIT)"]
---
```
**Renders as:** "By Prof. Sarah Johnson and Prof. Michael Brown (MIT)" (first linked, second plain text)

### Example 4: External Author Only
```markdown
---
title: "Guest Post: Future of AI"
date: 2025-11-01
authors: ["Dr. Jane Doe (Stanford University)"]
---
```
**Renders as:** "By Dr. Jane Doe (Stanford University)" (plain text)

### Example 5: No Authors (Backward Compatible)
```markdown
---
title: "Lab Update"
date: 2025-10-15
---
```
**Renders as:** (No author byline displayed)

## Definition of Done

- [ ] Schema updated in `src/content.config.ts`
- [ ] AuthorByline component created and tested
- [ ] News detail template updated to display authors
- [ ] Activities detail template updated to display authors
- [ ] All 10 new articles updated with appropriate author attribution
- [ ] Existing sample articles updated with authors (at least 2-3 examples)
- [ ] Manual testing confirms:
  - Internal authors link to profiles correctly
  - External authors display as plain text
  - Multiple authors format correctly (commas and "and")
  - Missing authors field doesn't break pages
- [ ] Code committed and pushed to branch
- [ ] Documentation updated with author field usage

## Testing Scenarios

### Test Case 1: Single Internal Author
- **Given**: A news article with `authors: ["prof-sarah-johnson"]`
- **When**: Page is rendered
- **Then**: Displays "By Prof. Sarah Johnson" with link to `/team/prof-sarah-johnson`

### Test Case 2: Multiple Internal Authors
- **Given**: A news article with `authors: ["phd-emily-chen", "phd-wei-zhang"]`
- **When**: Page is rendered
- **Then**: Displays "By Emily Chen and Wei Zhang" with both names linked

### Test Case 3: External Author
- **Given**: A news article with `authors: ["Dr. John Smith (MIT)"]`
- **When**: Page is rendered
- **Then**: Displays "By Dr. John Smith (MIT)" without link

### Test Case 4: Mixed Authors
- **Given**: A news article with `authors: ["prof-sarah-johnson", "Dr. External (Harvard)"]`
- **When**: Page is rendered
- **Then**: First name is linked, second is plain text

### Test Case 5: No Authors
- **Given**: A news article with no `authors` field
- **When**: Page is rendered
- **Then**: No author byline is displayed, page renders normally

### Test Case 6: Invalid Team Slug
- **Given**: A news article with `authors: ["nonexistent-person"]`
- **When**: Page is rendered
- **Then**: Displays "By nonexistent-person" as plain text (graceful fallback)

### Test Case 7: Three or More Authors
- **Given**: A news article with `authors: ["person1", "person2", "person3"]`
- **When**: Page is rendered
- **Then**: Displays "By Person1, Person2 and Person3" with proper comma separation

## Success Metrics

- All new content includes author attribution
- Team members can be easily credited for their contributions
- External collaborators can be credited appropriately
- No breaking changes to existing content
- Author links generate additional profile page views (measure after deployment)

## Dependencies

- Existing team collection and profiles
- Team member slugs must match convention (e.g., `prof-sarah-johnson`)

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Team slug doesn't match | Author appears as plain text | Document slug naming convention clearly |
| Performance impact from loading all team members | Slower page loads | Use Astro's static generation, cache team member map |
| Inconsistent author naming | Confusing display | Provide clear examples in documentation |
| Breaking existing content | Site errors | Make field optional, add fallback handling |

## Future Enhancements

- Author archive pages (`/authors/[slug]`)
- RSS feeds filtered by author
- Author statistics (number of posts, dates active)
- Co-author networking visualization
- Author search and autocomplete in admin interface

## Questions & Assumptions

**Questions:**
- Should we display author photos/avatars alongside names?
- Should author bylines appear on listing cards or only detail pages?
- Do we need author filtering in the UI?

**Assumptions:**
- Team member slugs follow consistent naming pattern
- Most content will be authored by internal team members
- External authors will be clearly marked with institution
- Authors field will be manually maintained (no auto-population)

## Priority

**HIGH** - Improves content attribution and is a common requirement for academic websites.

## Estimated Effort

- Schema changes: 15 minutes
- Component development: 1-2 hours
- Template integration: 1 hour
- Content updates: 30 minutes
- Testing: 1 hour
- Documentation: 30 minutes

**Total: ~4-5 hours**
