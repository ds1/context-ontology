# Podcast Talking Points: Design Tokens as Context Coordinates

*Preparation guide for discussing the Design Token Context Ontology*

---

## The One-Liner Thesis

> **"Design tokens aren't values—they're policies. The token doesn't say 'blue-500'; it says 'given where you are, here's what blue means.'"**

Alternative framings:
- "We've been treating tokens as nouns when they're actually verbs."
- "A token is a question-answering machine, not a lookup table."
- "Context isn't a modifier—it's the coordinate system tokens live in."

---

## Three Core Claims (The "Remember These" List)

### 1. Tokens Are Policies, Not Values
**Soundbite:** "A token doesn't store a value. It stores the *rules* for determining a value."

**Expand if needed:** When you write `color.surface.default`, you're not retrieving `#ffffff`. You're asking: "What should this surface look like *here*, *now*, for *this user*?" The answer depends on dozens of factors—theme, contrast needs, elevation, interaction state. The token encodes the decision logic, not the decision.

### 2. Context Is a Coordinate System
**Soundbite:** "Every UI element exists at a coordinate—a point in an N-dimensional space of conditions."

**Expand if needed:** We already think this way intuitively. "Dark mode on mobile with high contrast" is a point in space. The problem is we treat these dimensions as independent switches instead of as a unified coordinate. When you formalize context as coordinates, resolution becomes geometric: find the most specific rule that covers this point.

### 3. Resolution Is the Core Artifact
**Soundbite:** "The resolution algorithm is more important than the tokens themselves."

**Expand if needed:** Tokens are easy. Any team can define `spacing.4 = 16px`. The hard part is: what happens when `density: compact` AND `viewport: mobile` AND `touch: true`? The resolution algorithm—specificity, precedence, inheritance—is where design systems actually succeed or fail.

---

## Five Key Insights (Deep Cuts)

### Insight 1: The Category Error
**The problem:** We call "light mode" and "hover state" the same thing—"variants." But they're categorically different.

**The insight:** Light/dark is an *environmental condition* (the context you're in). Hover is an *interaction state* (what you're doing). Cramming both into "variants" hides crucial structure.

**The quote:** "CSS figured this out—pseudo-classes and media queries are different things. Our token systems regressed."

---

### Insight 2: Agent-Relative Resolution
**The problem:** We assume a token resolves the same way for everyone.

**The insight:** Resolution depends on the *intersection* of environment AND agent capabilities. A hover state only exists for devices that can hover. A P3 color only exists for displays that support it.

**The quote:** "A chair affords sitting *for a human*. A token affords meaning *for a capable device and user*."

---

### Insight 3: Volatility Drives Architecture
**The problem:** We treat all context dimensions the same at runtime.

**The insight:** Some dimensions never change (platform). Some change rarely (theme preference). Some change constantly (hover state). Smart systems resolve stable dimensions at build time, volatile ones at runtime.

**The quote:** "Volatility isn't metadata—it's the staging strategy for your entire resolution pipeline."

---

### Insight 4: Context Types Are Named Subspaces
**The problem:** We create "mobile dark mode" as a theme variant, duplicating logic.

**The insight:** "Mobile dark mode" is a *named region* in the coordinate space—a shortcut that sets `viewport: mobile` AND `theme: dark`. It's not a separate thing; it's a coordinate preset with optional policy overlays.

**The quote:** "A context type is a named policy for a region of the coordinate space."

---

### Insight 5: Tokens Encode Behavior, Not Appearance
**The problem:** We think of tokens as "what color is this?"

**The insight:** Tokens answer "what should happen when a user encounters this element in this context?" Even a "static" background color is a behavioral response to the act of looking.

**The quote:** "The token isn't a color—it's the answer to a question that hasn't been asked yet."

---

## How This Fits the Standard Token Taxonomy

Most design systems use a three-tier token architecture:

| Tier | Name | Example | Role |
|------|------|---------|------|
| **Tier 1** | Primitive | `blue.500` → `#007BFF` | Raw values. The palette. Context-agnostic. |
| **Tier 2** | Semantic | `color.primary.action` → `{blue.500}` | Meaning and intent. Where theming happens. |
| **Tier 3** | Component | `button.bg.default` → `{color.primary.action}` | Usage. Scoped to specific UI elements. |

**Soundbite:** "The coordinate model doesn't replace this taxonomy—it formalizes what happens inside Tier 2."

**Expand if needed:** The semantic tier is where theming happens. When you switch to dark mode, `color.primary.action` maps to `{blue.300}` instead of `{blue.500}`. But most systems treat this as a simple switch. The coordinate model says: that "switch" is one axis in an N-dimensional space. Add contrast preference, viewport, density, interaction state—each is another axis. Tier 2 becomes a resolution layer, not a lookup table.

**The diagram:**
```
Tier 1 (Primitives):     blue.300, blue.500, blue.600
                              ↑
Tier 2 (Semantic):       color.primary.action = f(coordinate)
                              ↑
                         Resolution Algorithm
                              ↑
                         Context Coordinate: { colorScheme: dark, contrast: high, ... }
                              ↑
Tier 3 (Component):      button.bg.default → {color.primary.action}
```

**What this adds:**
1. **Compound conditions** — Handle `dark AND high-contrast AND mobile`, not just single switches
2. **Explicit resolution rules** — Specificity and precedence when multiple contexts apply
3. **Volatility awareness** — Distinguish "resolve once" from "resolve constantly"

---

## Memorable Analogies

### The GPS Analogy
"Your GPS doesn't store 'turn left.' It stores the rules for determining what to do given your current position, destination, and traffic conditions. Tokens should work the same way—context in, decision out."

### The CSS Specificity Analogy
"CSS already solved this. When two rules match, the more specific one wins. When specificity ties, source order breaks it. We reinvent this badly in every design system. The coordinate model makes specificity explicit and generalizable."

### The Affordance Analogy (from Gibson)
"A door handle doesn't 'have' affordance. It affords *pulling* for creatures with hands. Similarly, a hover state doesn't 'exist'—it's an affordance for devices with pointers. Tokens mediate between environment and capability."

### The Projection Analogy
"Figma collections are like looking at a 3D object's shadow. You see a 2D shape that's useful but lossy. The shadow (Theme collection with light/dark modes) is a projection of the full coordinate space. Useful for implementation, but don't mistake it for the source of truth."

---

## Anticipated Questions & Responses

### "Isn't this overengineered for most teams?"

**Response:** "The complexity already exists—it's just hidden in one-off conditionals, duplicated tokens, and tribal knowledge about 'how dark mode works with high contrast.' The coordinate model makes existing complexity visible and manageable. You don't add complexity; you surface it."

### "How does this work with Figma/existing tools?"

**Response:** "Figma collections are projections of the coordinate space. Each collection captures one dimension—Theme captures colorScheme, Density captures spacing scale. The tools work fine; they just can't represent the full model. That's okay—you build to the projection, but you *design* to the coordinate space."

### "What about performance? Evaluating all these dimensions sounds expensive."

**Response:** "Volatility classification solves this. Platform and locale? Resolve once at build time. Theme and density? Resolve at app init. Hover state? That's the only thing evaluating at 60fps. Most coordinates are nearly static; only the volatile tail matters for runtime."

### "Where do I start?"

**Response:** "Start by naming your dimensions explicitly. Most teams have implicit dimensions scattered across code—`if (isMobile)`, `if (darkMode)`, `if (isHovering)`. Inventory those. Then ask: what's the precedence when they conflict? That question alone will surface your real architecture."

---

## Calls to Action

### For Design System Teams
> "Audit your token definitions. For every token that has 'variants,' ask: is this a different *value*, or is it a different *context*? If it's context, you've found a dimension."

### For Tool Builders
> "Build resolution into your tools, not just storage. A token file format that can't express 'this value when dark AND high-contrast' is incomplete. The format should encode the coordinate predicates."

### For the Industry
> "We need a standard resolution algorithm. The W3C Design Tokens spec defines format but punts on resolution. That's like defining JSON without defining how to query it. Resolution is where interoperability actually matters."

### For Researchers
> "Apply this to component APIs. If tokens are policies over coordinates, components are policies over tokens. The same formalism—predicates, specificity, resolution—might unify token systems and component prop APIs."

---

## Quick Reference Card

| Concept | One-Liner |
|---------|-----------|
| **Token** | A policy, not a value |
| **Context** | A coordinate in N-dimensional space |
| **Resolution** | Policy evaluation at a coordinate |
| **Specificity** | More dimensions constrained = more specific |
| **Precedence** | Accessibility > Appearance > Density > ... |
| **Volatility** | How often a dimension changes; drives when to evaluate |
| **Context Type** | Named coordinate region with policy overlay |
| **Projection** | Lossy but useful 2D view of the full space |

---

## The Closing Statement

> "For twenty years, we've been building design systems like they're databases—store values, retrieve values. But design decisions aren't data; they're *policies*. The context coordinate model treats them that way. It's not a new idea; it's the idea CSS has had all along, applied systematically to tokens. The question isn't whether your system needs this. The question is whether you want the complexity visible or hidden."

---

## Appendix: If You Only Remember Three Things

1. **Tokens are policies.** They encode decision logic, not decisions.

2. **Context is coordinates.** Every element lives at a point in condition-space.

3. **Resolution is the hard part.** Specificity, precedence, inheritance—get these right and the rest follows.
