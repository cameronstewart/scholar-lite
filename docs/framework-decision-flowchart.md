# Framework Decision Flowchart

```
START: Choosing a Static Site Generator
│
├─ Do you need backend/API routes?
│  ├─ YES → Next.js (full-stack)
│  └─ NO → Continue
│
├─ Do you have 10,000+ pages?
│  ├─ YES → Hugo (build speed)
│  └─ NO → Continue
│
├─ Is your team JavaScript-proficient?
│  ├─ NO → Hugo (simpler, Go templates)
│  └─ YES → Continue
│
├─ Do you need type-safe content validation?
│  ├─ YES → Astro (Zod schemas) or Next.js
│  └─ NO → Continue
│
├─ Do you need interactive components?
│  ├─ YES (React only) → Next.js
│  ├─ YES (multi-framework) → Astro
│  └─ NO → Continue
│
├─ Do you need multi-source data (CMS + API + files)?
│  ├─ YES → Gatsby (GraphQL) [declining] or Next.js
│  └─ NO → Continue
│
├─ Maximum simplicity needed?
│  ├─ YES → 11ty or Jekyll
│  └─ NO → Continue
│
└─ Default recommendation:
   ├─ Academic/Portfolio → Astro ✅
   ├─ Blog/Content → Hugo or 11ty
   └─ App/Platform → Next.js
```

---

## Quick Reference: When to Use Each

### 🚀 Astro
**Use When:**
- Building academic portfolios, research labs, personal sites
- Need type safety for content
- Want modern JavaScript/TypeScript
- Interactive components (React, Vue, Svelte)
- Image optimization required
- Team knows modern web dev

**Don't Use When:**
- Need backend API routes (use Next.js)
- Have 10,000+ pages (use Hugo)
- Team doesn't know JavaScript (use Hugo)

---

### ⚡ Hugo
**Use When:**
- Large content sites (1000+ pages)
- Build speed is critical
- Team prefers simple templating
- No JavaScript expertise
- Need multi-output formats (JSON, RSS, HTML)
- Want single binary deployment

**Don't Use When:**
- Need type safety (use Astro)
- Want React components (use Next.js/Astro)
- Require complex JavaScript interactions

---

### ⚛️ Next.js
**Use When:**
- Building full-stack applications
- Need API routes / backend
- Want server-side rendering (SSR)
- E-commerce or authenticated apps
- Incremental static regeneration
- React ecosystem investment

**Don't Use When:**
- Pure static content site (use Astro/Hugo)
- Want multi-framework support (use Astro)
- Need fastest build times (use Hugo)

---

### 🎨 Gatsby
**Use When:**
- Aggregating data from multiple sources
- GraphQL preference
- Complex data transformations
- PWA requirements

**Consider Instead:**
- Astro (better DX, faster builds)
- Next.js (more active development)

**Note:** Declining popularity, maintenance concerns

---

### 📝 Eleventy (11ty)
**Use When:**
- Simple blogs or marketing sites
- Maximum control/flexibility
- Minimal JavaScript footprint
- Learning project
- Template language preference

**Don't Use When:**
- Need type safety (use Astro)
- Large scale (use Hugo)
- Complex components (use Astro/Next.js)

---

### 💎 Jekyll
**Use When:**
- Hosting on GitHub Pages (free)
- Very simple blog
- Ruby familiarity
- Mature theme ecosystem

**Consider Instead:**
- Hugo (faster builds)
- Astro (modern stack)
- 11ty (more flexible)

---

## Scholar Lite Specific Recommendation

```
Your Requirements:
✓ Academic research portfolio
✓ 35 content items (publications, team, patents, etc.)
✓ Type safety needed (prevent content errors)
✓ Modern JavaScript/TypeScript stack
✓ React components already in use
✓ Image optimization required
✓ i18n (8+ languages)
✓ Interactive components (charts, filters)

Recommendation: Astro ✅

Why:
1. Type safety with Zod prevents content errors
2. Already using React components
3. Scales to 1000s of items easily
4. Modern DX (hot reload, TypeScript)
5. Image optimization built-in
6. Multi-framework support (future flexibility)

When to Reconsider:
- Content grows to 10,000+ items → Hugo
- Need backend features → Next.js
- Team changes away from JavaScript → Hugo
```

---

## Migration Complexity Matrix

**From → To (Effort Level)**

|  | Astro | Hugo | Next.js | Gatsby | 11ty |
|--|-------|------|---------|--------|------|
| **Astro** | - | Medium | Medium | Medium | Low |
| **Hugo** | Medium | - | High | High | Medium |
| **Next.js** | Medium | High | - | Medium | Medium |
| **Gatsby** | Medium | High | Medium | - | Medium |
| **11ty** | Low | Medium | Medium | Medium | - |

**Effort Levels:**
- **Low**: Mostly content copy, minimal template changes
- **Medium**: Template rewrite, some content adjustments
- **High**: Major refactoring, schema changes, component rewrite

---

## Performance Comparison

### Build Time (1000 markdown files)

```
Hugo:     ████ 2s
11ty:     ████████████ 8s
Astro:    ████████████████████ 15s
Next.js:  ████████████████████████ 20s
Gatsby:   ████████████████████████████████████████ 60s
```

### Runtime Performance (Lighthouse Score)

```
Astro:    ████████████████████████████ 100
Hugo:     ████████████████████████████ 100
11ty:     ████████████████████████████ 100
Next.js:  ██████████████████████████ 95
Gatsby:   ████████████████████████ 90
```

### Developer Experience (Subjective 1-10)

```
Astro:    ████████████████████ 9/10
Next.js:  ██████████████████ 8/10
11ty:     ████████████████ 7/10
Hugo:     ██████████████ 6/10
Gatsby:   ████████████ 5/10
```

---

## Cost Analysis

### Infrastructure Costs (Monthly, Typical Academic Site)

| Framework | Hosting | Build Minutes | Total |
|-----------|---------|---------------|-------|
| Hugo | Free (Netlify/Vercel) | ~10 min/month | $0 |
| Astro | Free (Netlify/Vercel) | ~50 min/month | $0 |
| 11ty | Free (Netlify/Vercel) | ~30 min/month | $0 |
| Next.js | Free tier or $20 | ~100 min/month | $0-20 |
| Gatsby | Free tier | ~200 min/month | $0 |

**Note**: All can be hosted for free on Netlify, Vercel, or Cloudflare Pages

---

## Ecosystem Maturity

### NPM Downloads (Weekly, Dec 2024)

```
Next.js:  6,000,000 ████████████████████████████████████
Gatsby:   500,000   ██████
Astro:    400,000   ████
11ty:     100,000   █
Hugo:     N/A (binary, not NPM)
```

### GitHub Stars

```
Next.js:  128k ████████████████████████████████████
Hugo:     78k  ██████████████████████
Gatsby:   55k  ████████████████
Astro:    48k  ██████████████
11ty:     17k  █████
```

### Community Activity (2024)

```
Next.js:  ████████████████████ Very Active (Vercel-backed)
Astro:    ██████████████████ Very Active (growing)
Hugo:     ████████████ Active (stable)
11ty:     ██████████ Active
Gatsby:   ████ Declining
```

---

## Feature Comparison Checklist

| Feature | Astro | Hugo | Next.js | Gatsby | 11ty |
|---------|-------|------|---------|--------|------|
| TypeScript Native | ✅ | ❌ | ✅ | ✅ | ⚠️ |
| Type-Safe Content | ✅ | ❌ | ⚠️ | ✅ | ❌ |
| Hot Reload | ✅ | ✅ | ✅ | ✅ | ✅ |
| Image Optimization | ✅ | ⚠️ | ✅ | ✅ | ❌ |
| Multi-Framework | ✅ | ❌ | ❌ | ❌ | ❌ |
| API Routes | ❌ | ❌ | ✅ | ❌ | ❌ |
| SSR/ISR | ⚠️ | ❌ | ✅ | ❌ | ❌ |
| GraphQL | ❌ | ❌ | ⚠️ | ✅ | ❌ |
| i18n Built-in | ✅ | ✅ | ✅ | ⚠️ | ⚠️ |
| Markdown | ✅ | ✅ | ✅ | ✅ | ✅ |
| MDX | ✅ | ❌ | ✅ | ✅ | ⚠️ |
| Content Collections | ✅ | ✅ | ❌ | ✅ | ⚠️ |
| RSS Generation | ✅ | ✅ | ⚠️ | ✅ | ⚠️ |
| Sitemap | ✅ | ✅ | ✅ | ✅ | ⚠️ |
| Syntax Highlighting | ✅ | ✅ | ⚠️ | ✅ | ⚠️ |

**Legend:**
- ✅ Built-in / First-class support
- ⚠️ Via plugin / manual setup
- ❌ Not available / not recommended

---

## Learning Resources Time Investment

**Time to Productivity (For JS developers)**

```
11ty:     █████ 1-2 days
Hugo:     ██████████ 3-5 days
Astro:    ███████████████ 1 week
Next.js:  ████████████████████ 1-2 weeks
Gatsby:   ████████████████████████████ 2-3 weeks
```

**For Non-JS developers:**

```
Hugo:     ██████████ 3-5 days
Jekyll:   ███████████████ 1 week
11ty:     ████████████████████ 1-2 weeks
Astro:    ████████████████████████████ 2-3 weeks
Next.js:  ████████████████████████████████ 3-4 weeks
```

---

## Final Recommendation Algorithm

```javascript
function chooseFramework(requirements) {
  const {
    teamSkills,
    contentScale,
    needsBackend,
    needsTypesSafety,
    needsComponents,
    buildSpeedCritical,
  } = requirements;

  // Immediate disqualifiers
  if (needsBackend) return 'Next.js';
  if (contentScale > 10000 && buildSpeedCritical) return 'Hugo';

  // Strong recommendations
  if (needsTypesSafety && needsComponents && teamSkills.includes('JavaScript')) {
    return 'Astro';
  }

  if (!teamSkills.includes('JavaScript') && contentScale > 1000) {
    return 'Hugo';
  }

  // Default recommendations by use case
  const useCase = requirements.useCase;

  const recommendations = {
    'academic-portfolio': 'Astro',
    'blog': contentScale > 1000 ? 'Hugo' : '11ty',
    'documentation': 'Astro or Hugo',
    'marketing-site': '11ty or Astro',
    'app-marketing': 'Astro or Next.js',
    'e-commerce': 'Next.js',
    'saas': 'Next.js',
  };

  return recommendations[useCase] || 'Astro';
}

// Your project
chooseFramework({
  teamSkills: ['JavaScript', 'TypeScript', 'React'],
  contentScale: 35,
  needsBackend: false,
  needsTypesSafety: true,
  needsComponents: true,
  buildSpeedCritical: false,
  useCase: 'academic-portfolio',
});
// → Returns: 'Astro' ✅
```

---

**Quick Decision:** If unsure, start with **Astro** (modern, flexible, great DX) or **Hugo** (simple, fast). You can migrate later if needed using universal content patterns.
