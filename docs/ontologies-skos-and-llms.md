# Where Do Ontologies and SKOS Fit In?

## The Hierarchy of Knowledge Representation

```
Simple → Complex
│
├─ Enums (simplest)
│  └─ ["paper", "book", "patent"]
│
├─ Controlled Vocabularies
│  └─ Term + Synonyms + Scope notes
│
├─ SKOS (Simple Knowledge Organization System)
│  └─ Concepts + Relationships + Labels in multiple languages
│
├─ Taxonomies
│  └─ Hierarchical classifications (parent/child)
│
├─ Thesauri
│  └─ Taxonomy + Related terms + Scope notes
│
└─ Ontologies (most complex)
   └─ Formal logic + Reasoning + Axioms + Inference
```

---

## What Each Is

### 1. Enums (What You Have Now)

**Just a list:**

```typescript
type: z.enum(['paper', 'book', 'patent', 'software'])
```

**That's it.** No relationships, no definitions.

---

### 2. Controlled Vocabularies

**Terms with metadata:**

```
Term: "Professor"
Synonyms: ["Faculty", "Instructor"]
Scope Note: "An academic teaching at university level"
```

**Still simple.** Just metadata about terms.

---

### 3. SKOS (Simple Knowledge Organization System)

**Formalized vocabulary with relationships:**

```turtle
# SKOS in RDF/Turtle format
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .

:professor a skos:Concept ;
    skos:prefLabel "Professor"@en ;
    skos:prefLabel "教授"@zh ;           # Chinese
    skos:prefLabel "Profesor"@es ;       # Spanish
    skos:altLabel "Faculty"@en ;         # Synonym
    skos:altLabel "Prof"@en ;
    skos:broader :academic-staff ;       # Parent concept
    skos:narrower :assistant-professor ; # Child concept
    skos:related :researcher ;           # Related concept
    skos:definition "An academic teaching and researching at university level"@en .
```

**What SKOS adds:**
- ✅ Multilingual labels (prefLabel)
- ✅ Synonyms (altLabel)
- ✅ Hierarchies (broader/narrower)
- ✅ Related terms (related)
- ✅ Definitions (definition)
- ✅ Machine-readable (RDF)

---

### 4. Ontologies (OWL - Web Ontology Language)

**Formal logic and reasoning:**

```turtle
# OWL Ontology
@prefix owl: <http://www.w3.org/2002/07/owl#> .

:Professor a owl:Class ;
    rdfs:subClassOf :Person ;
    rdfs:subClassOf [
        a owl:Restriction ;
        owl:onProperty :hasPublication ;
        owl:minCardinality 1     # Professors must have ≥1 publication
    ] .

:PhDStudent a owl:Class ;
    rdfs:subClassOf :Person ;
    owl:disjointWith :Professor .  # Can't be both simultaneously

:hasAdvisor a owl:ObjectProperty ;
    rdfs:domain :PhDStudent ;
    rdfs:range :Professor ;
    owl:inverseOf :advises .

# Reasoning/Inference:
# IF: Alice :hasAdvisor Bob
# THEN: Bob :advises Alice  (inferred automatically)
```

**What Ontologies add:**
- ✅ Formal logic
- ✅ Constraints and rules
- ✅ Automatic reasoning/inference
- ✅ Consistency checking
- ✅ Complex relationships

---

## Comparison Table

| Feature | Enum | SKOS | Ontology (OWL) |
|---------|------|------|----------------|
| **Complexity** | Very Simple | Medium | Very Complex |
| **Relationships** | None | Yes | Yes (+ logic) |
| **Multilingual** | No | Yes | Yes |
| **Synonyms** | No | Yes | Yes |
| **Reasoning** | No | No | Yes |
| **Format** | Code/JSON | RDF | OWL/RDF |
| **Learning Curve** | 5 min | Days | Weeks |

---

## For Your Use Case (Scholar Lite)

### Current: Enums
```typescript
role: z.enum(['Professor', 'PhD Student', 'Postdoc'])
```

**Problems:**
- ❌ English only (you have 8 languages!)
- ❌ No synonyms documented
- ❌ No definitions

---

### Recommended: SKOS-Inspired JSON

**Not full SKOS (overkill), but borrow the good parts:**

```json
{
  "id": "professor",
  "prefLabel": {
    "en": "Professor",
    "zh": "教授",
    "es": "Profesor",
    "fr": "Professeur",
    "de": "Professor",
    "ja": "教授",
    "ko": "교수",
    "ar": "أستاذ"
  },
  "altLabels": {
    "en": ["Faculty", "Prof"]
  },
  "definition": {
    "en": "Senior academic staff member"
  },
  "broader": "academic-staff"
}
```

**Benefits:**
- ✅ All 8 languages
- ✅ Synonyms documented
- ✅ Still just JSON (no RDF complexity)
- ✅ Version controlled
- ✅ Human-readable

---

### Not Recommended: Full SKOS or OWL

**Why not:**
- ❌ Overkill for 35 items
- ❌ Requires RDF/triplestore infrastructure
- ❌ Complex tooling (reasoners, SPARQL)
- ❌ Steep learning curve
- ❌ No reasoning needed for your use case

---

## When to Use Each

### Enums
**Use when:**
- Single language
- Simple list
- No relationships

**Your case:** Too limiting (8 languages)

---

### SKOS-Inspired JSON
**Use when:**
- **Multilingual labels needed** ← You!
- Want to document synonyms
- Simple hierarchies
- Don't need reasoning

**Your case:** **Recommended** ✅

---

### Full SKOS (RDF)
**Use when:**
- Publishing vocabularies for others
- Need formal standard
- Integrating with Linked Data

**Your case:** Overkill

---

### OWL Ontologies
**Use when:**
- Need automatic reasoning
- Complex constraints
- Knowledge graph
- Scientific domain modeling

**Your case:** Way overkill for 35 items

---

## Summary

**SKOS and Ontologies = Formal knowledge representation**

**Do you need them?**
- Full SKOS: ❌ No (too complex)
- OWL Ontology: ❌ Definitely no (massive overkill)
- SKOS-inspired JSON: ✅ Yes (for 8 languages)

**Key insight:** Use the **concepts** (multilingual labels, synonyms) without the **complexity** (RDF, reasoners, triplestores).

For your 35-item academic site with 8 languages, a lightweight SKOS-inspired JSON approach gives you 80% of the benefit with 20% of the complexity.
