# Context Coordinate Model and the Three-Tier Token Taxonomy

*How the coordinate model fits within established token architecture*

---

## The Standard Three-Tier Taxonomy

The document "Design Systems: Single Source of Truth" (Section 3.2) describes the industry-standard token taxonomy:

| Tier | Name | Example | Role |
|------|------|---------|------|
| **Tier 1** | Primitive | `blue.500` → `#007BFF` | Raw values. The palette. Context-agnostic. |
| **Tier 2** | Semantic | `color.primary.action` → `{blue.500}` | Meaning and intent. Where theming happens. |
| **Tier 3** | Component | `button.bg.default` → `{color.primary.action}` | Usage. Scoped to specific UI elements. |

The document identifies Tier 2 (Semantic) as "the crux of truth" because it enables theming:

> "In 'Dark Mode,' the token `color.primary.action` might map to `{blue.300}` instead of `{blue.500}`. The component code does not change; it still points to `color.primary.action`."

---

## Where the Context Coordinate Model Fits

The coordinate model doesn't replace this taxonomy—it **formalizes what happens inside Tier 2**.

The document treats context as a simple switch: light mode vs dark mode. The coordinate model says: that "switch" is actually one axis (`colorScheme`) in an N-dimensional space. When you add high contrast, that's another axis (`contrast`). When you add mobile viewport, density preference, interaction state—each is another axis.

---

## Tier 2 Becomes the Resolution Layer

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

The document says Tier 2 "defines the *meaning* of values." The coordinate model makes that precise: meaning is a **function** from context coordinates to primitive values, not a static mapping with variant switches.

---

## What the Coordinate Model Adds

### 1. Compound Conditions

The document only shows single-dimension switching (light/dark). The coordinate model handles `dark AND high-contrast AND mobile`.

### 2. Explicit Resolution Rules

The document says theming "maps" values but doesn't specify what happens when multiple contexts apply. The coordinate model provides specificity and precedence.

### 3. Volatility Awareness

The document treats all context switches the same. The coordinate model distinguishes platform (resolve once) from hover state (resolve constantly).

---

## Summary

The three-tier taxonomy remains correct. The coordinate model is a deeper specification of how Tier 2 actually works when context is multidimensional.
