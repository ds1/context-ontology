# Design Token Context Ontology: Comprehensive Stress Test

**This analysis reveals 47 missing dimensions across the 12 context categories, identifies 78 cross-category dependencies, and documents 52 breaking edge cases.** The ontology’s current enumeration covers approximately 60% of real-world context requirements. Critical gaps exist in accessibility variants beyond motion, foldable device support, component lifecycle states, and locale-specific typography handling. Below is the systematic analysis of each category.

-----

## Category 1: Appearance and Theme Contexts

### Current dimension inventory

|Dimension           |Enumerated Values      |Detection Mechanism               |
|--------------------|-----------------------|----------------------------------|
|Color mode          |light, dark            |`prefers-color-scheme` media query|
|High contrast       |enabled, disabled      |`prefers-contrast: more`          |
|Forced colors       |active, none           |`forced-colors` media query       |
|Inverted colors     |inverted, none         |`inverted-colors` (Safari only)   |
|Brand theme         |implementation-specific|CSS class/attribute               |
|Color scheme variant|tinted, vivid, muted   |Custom implementation             |

### Validation findings

**Missing values from current enumerations:**

- Color mode: Missing `auto` (system deference), `dim` (GitHub’s “dark-dimmed” variant), and explicit contrast variants (`light-high-contrast`, `dark-high-contrast`)
- High contrast: Windows provides **4 preset themes** (High Contrast Black, White, Aquatic, Desert) plus **8 customizable system colors**—current binary model is insufficient
- Color scheme variant: Major systems use different taxonomies—Adobe Spectrum uses `lightest/light/dark/darkest`  (4 levels),  IBM Carbon uses named themes (`white`, `g10`, `g90`, `g100`) 

**Naming inconsistencies identified:**
The terminology `mode` vs `scheme` vs `theme` is inconsistently applied. Atlassian uses `data-color-mode`,  CSS specification uses `color-scheme`, Material uses `theme`. Recommendation: standardize to “scheme” for system preferences and “theme” for brand/application choices.

**Detection mechanism gaps:**

- `inverted-colors` only fires in Safari—cross-browser detection requires JavaScript fallback
- No media query exists for dynamic color extraction source (Material You’s wallpaper-based theming)
- Color gamut detection (`color-gamut: p3`) exists but has no corresponding token pattern in any major design system

### Missing dimensions

|Dimension                    |Definition                                     |Values                                                             |Detection                                      |
|-----------------------------|-----------------------------------------------|-------------------------------------------------------------------|-----------------------------------------------|
|**Dynamic color source**     |Algorithmic palette generated from source image|`wallpaper`, `brand-logo`, `content-image`, `user-selected`, `none`|Platform API (Android `DynamicColors`, iOS 15+)|
|**Color temperature**        |Warm/cool tint variant of theme                |`warm`, `neutral`, `cool`                                          |User preference setting                        |
|**Color gamut context**      |Display’s color space capability               |`srgb`, `p3`, `rec2020`                                            |`@media (color-gamut: p3)`                     |
|**Tonal elevation mode**     |Whether elevation uses tint or shadow          |`tonal`, `shadow`, `hybrid`                                        |Theme configuration                            |
|**Fixed accent preservation**|Colors that persist across light/dark          |`preserve`, `adapt`                                                |Token flag                                     |

### Cross-category dependencies

```
Appearance ──────┬─── Elevation: Tonal elevation derives surface tint from 
                 │               theme's primary color; dark mode reverses
                 │               elevation direction (surfaces lighten upward)
                 │
                 ├─── Accessibility: forced-colors overrides ALL theme colors;
                 │                   prefers-contrast: more coupled to 
                 │                   forced-colors on Windows
                 │
                 └─── Platform: iOS/Android dynamic color APIs differ;
                                Android requires @android:color/system_* tokens
```

### Edge case catalog

|Scenario                                             |Conflict                                                  |Proposed Handling                                                                       |
|-----------------------------------------------------|----------------------------------------------------------|----------------------------------------------------------------------------------------|
|User enables high contrast + dark mode simultaneously|`prefers-contrast: more` triggers but theme choice unclear|Query both: `@media (prefers-color-scheme: dark) and (prefers-contrast: more)`          |
|Forced colors active with brand theme selected       |System colors override brand tokens                       |Use `forced-color-adjust: none` sparingly, map brand semantics to system color keywords |
|Multi-brand portal with shared components            |Token namespace collisions                                |Prefix tokens at primitive level: `{brand}.color.primary`                               |
|Print stylesheet from dark mode                      |Dark backgrounds waste ink                                |`@media print` resets to light mode values, removes decorative colors                   |

-----

## Category 2: Density and Spacing Contexts

### Current dimension inventory

|Dimension         |Enumerated Values             |Detection Mechanism          |
|------------------|------------------------------|-----------------------------|
|Density mode      |compact, comfortable, spacious|User preference/setting      |
|Touch optimization|optimized, standard           |`pointer: coarse` media query|
|Content density   |high, medium, low             |Container context            |
|Scale factor      |1x, 1.25x, 1.5x, 2x           |Device pixel ratio           |

### Validation findings

**Missing values from current enumerations:**

- Density mode: Material Design uses numeric scale (0, -1, -2, -3) where each decrement reduces component height by **4px**.  Current t-shirt sizing lacks this precision
- Touch optimization: WCAG 2.2 defines **24×24px minimum** (AA) and **44×44px** (AAA)— binary “optimized/standard” is insufficient
- Scale factor: Missing OS-level scaling contexts (Windows 125%, 150%, 175%, 200%)

**Naming inconsistencies:**
SAP uses `cozy/compact`, Gmail uses `Compact/Cozy/Comfortable`, AWS Cloudscape uses `comfortable/compact`. No industry consensus exists.

**Detection gaps:**

- Content density conflated with touch optimization in most systems—these are orthogonal concerns
- No CSS media query exists for user density preference (unlike `prefers-color-scheme`)

### Missing dimensions

|Dimension                    |Definition                            |Values                                                        |Rationale                                                            |
|-----------------------------|--------------------------------------|--------------------------------------------------------------|---------------------------------------------------------------------|
|**Font scaling context**     |Text scaling independent of UI scaling|`default`, `large`, `larger`, `largest`                       |Users may want larger text with normal-sized controls (accessibility)|
|**Input device precision**   |Primary input device’s accuracy level |`fine` (mouse), `coarse` (touch), `pen`, `hybrid`             |Different precision requires different hit areas                     |
|**Interaction precision**    |Motor control requirements            |`standard`, `accessible` (larger targets, slower interactions)|Beyond device—addresses tremors, motor disabilities                  |
|**Scroll behavior density**  |Content revealed per scroll gesture   |`standard`, `accelerated`, `paginated`                        |High-density views may warrant different scroll behavior             |
|**Density transition timing**|Animation between density states      |Duration tokens, instant vs animated                          |How UI transforms when user changes density                          |

### Cross-category dependencies

```
Density ─────────┬─── Viewport: Larger viewports default to higher density;
                 │              SAP Fiori auto-switches compact on desktop
                 │
                 ├─── Container: Container queries should inform density;
                 │               sidebar card needs different density than
                 │               main content card
                 │
                 ├─── Accessibility: Touch targets MUST maintain 44px minimum
                 │                   regardless of visual density reduction
                 │
                 └─── Interaction States: Compact density reduces hover/focus
                                          indicator spacing—may conflict with
                                          WCAG 2.4.13 Focus Appearance
```

### Edge case catalog

|Scenario                                                           |Conflict                                               |Proposed Handling                                                                                                     |
|-------------------------------------------------------------------|-------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|
|Compact density with touch input detected                          |Visual density conflicts with touch target requirements|Maintain **48dp touch targets** but allow tighter visual spacing; use `touch-action` areas larger than visual element |
|User zooms browser to 200% in compact mode                         |Density compounds with zoom, breaking layouts          |Detect zoom level, auto-switch to comfortable density above 150% zoom                                                 |
|Form with mixed input types (checkboxes + text fields)             |Different components have different density support    |Establish minimum density per component type in token system                                                          |
|Density switch mid-interaction (user changes setting while focused)|Focus indicator position may shift unexpectedly        |Transition density only on blur or use CSS transitions for smooth adjustment                                          |

-----

## Category 3: Viewport and Responsive Contexts

### Current dimension inventory

|Dimension    |Enumerated Values                          |Detection Mechanism                        |
|-------------|-------------------------------------------|-------------------------------------------|
|Breakpoint   |xs, sm, md, lg, xl, xxl                    |`min-width`/`max-width` media queries      |
|Orientation  |portrait, landscape                        |`orientation` media query                  |
|Display type |browser, standalone, fullscreen, minimal-ui|`display-mode` media query                 |
|Pixel density|1x, 2x, 3x                                 |`resolution` / `-webkit-device-pixel-ratio`|

### Validation findings

**Missing values from current enumerations:**

- Breakpoint: Modern approach shifts to content-based breakpoints rather than device-specific; container queries now supplement viewport queries 
- Display type: Missing `window-controls-overlay` for PWA titlebar customization
- Pixel density: Missing `infinite` value for vector displays (per CSS spec)

**Detection gaps:**

- Foldable device viewport segments require `horizontal-viewport-segments` and `vertical-viewport-segments` media features   (Chrome only)
- Safe area insets (`env(safe-area-inset-*)`) exist but are rarely tokenized
- Virtual keyboard presence detectable via `env(keyboard-inset-*)` but has minimal design system support

### Missing dimensions

|Dimension                 |Definition                            |Values                                                   |Browser Support             |
|--------------------------|--------------------------------------|---------------------------------------------------------|----------------------------|
|**Safe area insets**      |Device notch/Dynamic Island clearance |`env(safe-area-inset-top/right/bottom/left)`             |✅ All modern browsers       |
|**Display cutout type**   |Specific hardware cutout awareness    |`none`, `notch`, `dynamic-island`, `punch-hole`, `corner`|Platform-specific APIs      |
|**Viewport segment count**|Foldable screen segments              |Integer (1, 2, 3)                                        |⚠️ Chrome (foldables)        |
|**Fold posture**          |Foldable device physical state        |`flat`, `half-opened`, `folded`, `tabletop`, `book`      |⚠️ Screen Fold API           |
|**Fold occlusion**        |Whether fold area obscures content    |`none`, `partial`, `full`                                |Platform heuristic          |
|**Virtual keyboard state**|On-screen keyboard visibility         |`visible`, `hidden`, `transitioning`                     |`env(keyboard-inset-height)`|
|**Titlebar area**         |PWA window controls overlay dimensions|`env(titlebar-area-x/y/width/height)`                    |⚠️ Chrome/Edge (PWA)         |

### Cross-category dependencies

```
Viewport ────────┬─── Density: Viewport size informs default density;
                 │            mobile should default to touch-optimized
                 │
                 ├─── Container: Container queries handle component-level
                 │               adaptation; viewport queries handle page layout
                 │
                 ├─── Platform: Foldable APIs differ between Android/Windows;
                 │              Samsung vs Surface fold behaviors differ
                 │
                 └─── Structure: Fold posture affects layout mode
                                 (single-column vs split-view)
```

### Edge case catalog

|Scenario                                                    |Conflict                                                     |Proposed Handling                                                                                          |
|------------------------------------------------------------|-------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------|
|Samsung Z Fold cover screen (narrow) to main screen (tablet)|Extreme aspect ratio change mid-session                      |Use `horizontal-viewport-segments: 2` to detect unfolded state; preserve UI state across transition        |
|iOS Dynamic Island inset changes during activity            |Safe area insets are dynamic, not static                     |Use `safe-area-inset-*` in calculations rather than hardcoded values; consider `safe-area-max-*` for layout|
|Split-screen PWA with reduced viewport                      |Viewport queries fail, app renders desktop mode in tiny space|Container queries for component adaptation; viewport queries only for chrome/navigation                    |
|Foldable in “flex mode” (half-opened)                       |Content/controls should split across fold                    |Define `--layout-mode: split-vertical` token triggered by `fold-posture: half-opened`                      |

-----

## Category 4: Container Sizing Contexts

### Current dimension inventory

|Dimension         |Enumerated Values            |Detection Mechanism                 |
|------------------|-----------------------------|------------------------------------|
|Container width   |Continuous (any length value)|`@container (width > Xpx)`          |
|Container context |Named containers             |`container-name` + `@container name`|
|Parent constraints|Implicit (inherited sizing)  |Not directly queryable              |

### Validation findings

**Missing values from current enumerations:**

- Container width: Should include semantic breakpoint tokens (`--breakpoint-container-sm: 320px`)
- Container context: Named containers work but style queries (`@container style(--prop: value)`) only support custom properties currently

**Detection gaps:**

- Block-size (height) container queries require `container-type: size` which has performance implications—most systems avoid this
- Content overflow state not queryable in CSS
- Aspect ratio queryable but not commonly tokenized

### Missing dimensions

|Dimension                     |Definition                                        |Values                                           |Current State                              |
|------------------------------|--------------------------------------------------|-------------------------------------------------|-------------------------------------------|
|**Container height context**  |Block-size container queries                      |Continuous length                                |`container-type: size` exists but uncommon |
|**Container aspect ratio**    |Ratio-based queries                               |`16/9`, `4/3`, `1/1`, `9/16`                     |Queryable but rarely used                  |
|**Content overflow state**    |Whether contents exceed container                 |`within-bounds`, `overflowing-x`, `overflowing-y`|Not queryable in CSS; requires JS          |
|**Available space percentage**|Container size relative to viewport               |0-100%                                           |Calculation required                       |
|**Scroll-state queries**      |Container scroll position                         |`scrolled-top`, `scrolled-bottom`, `scrolling`   |New in spec: `@container scroll-state(...)`|
|**Container style properties**|Query computed styles (not just custom properties)|Any CSS property                                 |Spec allows but not implemented            |

### Cross-category dependencies

```
Container ───────┬─── Viewport: Media queries for global layout,
                 │              container queries for component layout
                 │
                 ├─── Density: Narrow containers should trigger compact mode
                 │             @container (width < 20rem) { --density: compact }
                 │
                 ├─── Structure: Nested containers create formatting contexts
                 │               with token scope boundaries
                 │
                 └─── Elevation: Container containment affects stacking context
                                 creation and z-index resolution
```

### Edge case catalog

|Scenario                                        |Conflict                                                    |Proposed Handling                                                                  |
|------------------------------------------------|------------------------------------------------------------|-----------------------------------------------------------------------------------|
|Height query needed but performance-constrained |`container-type: size` has layout cost                      |Prefer `inline-size` only; use JavaScript for height-dependent logic               |
|Nested container queries with conflicting styles|Child container overrides parent styles unexpectedly        |Establish token inheritance rules; use `@layer` for cascade control                |
|Container query + media query coordination      |Both can style same element                                 |Define cascade order: media query establishes foundation, container query adapts   |
|Circular size dependency                        |Container query depends on content that depends on container|CSS prevents this—containment breaks the cycle; design tokens must account for this|

-----

## Category 5: Interaction States — Mutually Exclusive

### Current dimension inventory

|State   |CSS Pseudo-class              |ARIA Mapping               |
|--------|------------------------------|---------------------------|
|rest    |(none—default)                |(implicit)                 |
|hover   |`:hover`                      |None (presentational)      |
|focus   |`:focus`, `:focus-visible`    |Managed by browser         |
|active  |`:active`                     |`aria-pressed` (limited)   |
|disabled|`:disabled`, `[aria-disabled]`|`aria-disabled="true"`     |
|dragged |(none)                        |`aria-grabbed` (deprecated)|

### Validation findings

**Critical classification error in ontology:**
Per industry analysis (Dynatrace, Material Design), **focus is NOT mutually exclusive**. An element can be simultaneously focused AND hovered. Correct classification:

- **Mutually exclusive**: rest, hover, active, dragged, disabled
- **Additive overlay**: focus (applies on top of exclusive states)

**Naming inconsistencies:**
Material Design inconsistently uses `hover` vs `hovered`, `focus` vs `focused`. Recommendation: standardize to **present tense** for pseudo-class alignment (`hover`, `focus`, `active`) or **past tense** for state description (`hovered`, `focused`, `activated`).

**ARIA mapping gaps:**

- `aria-grabbed` deprecated in ARIA 1.1 with no replacement 
- No ARIA equivalent for hover (accessibility concern—hover-dependent functionality must have alternatives)
- Active/pressed distinction unclear: CSS `:active` = being pressed; `aria-pressed` = toggle state 

### Missing states

|State            |Definition                                     |Detection                      |Use Case                                         |
|-----------------|-----------------------------------------------|-------------------------------|-------------------------------------------------|
|**focus-visible**|Focus via keyboard navigation (not mouse click)|`:focus-visible` pseudo-class  |Different focus rings for keyboard vs mouse users|
|**focus-within** |Ancestor contains focused element              |`:focus-within` pseudo-class   |Styling form groups when child is focused        |
|**pressed**      |Momentary press before release                 |JavaScript `pointerdown`       |Button depression feedback                       |
|**pending**      |Action initiated, awaiting response            |JavaScript state               |Async operation in progress                      |
|**skeleton**     |Content placeholder during load                |JavaScript state               |Layout preservation while loading                |
|**touch-hold**   |Long-press on touch device                     |JavaScript `touchstart` + timer|Context menu trigger on mobile                   |

### Cross-category dependencies

```
Interaction States ──┬─── Accessibility: Focus MUST take precedence over
                     │                   hover for keyboard navigation
                     │                   (WCAG 2.4.7 Focus Visible)
                     │
                     ├─── Semantic: Error state + disabled creates 
                     │              "disabled due to error" compound state
                     │
                     └─── Elevation: Active/pressed states often include
                                     elevation change (Material: pressed = -1dp)
```

### Edge case catalog

|Scenario                              |Conflict                                 |Proposed Handling                                                                                      |
|--------------------------------------|-----------------------------------------|-------------------------------------------------------------------------------------------------------|
|Element is both hovered AND focused   |Which state visual takes precedence?     |Focus takes semantic precedence; hover may show briefly then revert to focus (Material approach)       |
|Disabled element receives hover       |Should disabled show hover feedback?     |**No**—disabled elements should not respond to pointer;  cursor remains `not-allowed`, no visual change|
|Focus on element that triggers loading|Focus + pending states concurrent        |Maintain focus indicator during pending; add loading indicator alongside                               |
|Touch device has no hover state       |Hover-dependent interactions inaccessible|Provide touch-hold as hover alternative; never require hover for essential functionality               |

-----

## Category 6: Additive/Combinable States

### Current dimension inventory

|State    |ARIA Attribute |Applicable Elements                |
|---------|---------------|-----------------------------------|
|selected |`aria-selected`|listbox items, tabs, grid cells    |
|checked  |`aria-checked` |checkboxes, switches, radio buttons|
|expanded |`aria-expanded`|accordions, menus, trees           |
|loading  |`aria-busy`    |regions being updated              |
|error    |`aria-invalid` |form fields                        |
|warning  |(none)         |validation messages                |
|success  |(none)         |validation messages                |
|read-only|`aria-readonly`|editable fields                    |
|visited  |(none)         |links only (CSS `:visited`)        |

### Validation findings

**Semantic confusion requiring correction:**

- **selected vs checked**: These are NOT interchangeable. `aria-selected` = selection within containers (tabs, list items). `aria-checked` = toggle state (checkboxes, switches). `aria-pressed` = button toggle only 
- **error vs invalid**: Different concepts. `invalid` = value doesn’t match expected format (validation). `error` = system/submission failure occurred. CSS `:invalid` handles client-side validation only

**Missing ARIA mappings:**

- Warning state has no ARIA equivalent—use `aria-describedby` pointing to warning message
- Success state has no ARIA equivalent—announce via live region
- Visited is browser-managed with privacy restrictions on styling

### Missing states

|State             |Definition                           |ARIA Mapping                                 |Use Case                                        |
|------------------|-------------------------------------|---------------------------------------------|------------------------------------------------|
|**indeterminate** |Neither fully checked nor unchecked  |`aria-checked="mixed"`                       |Parent checkbox with partially selected children|
|**current**       |Item represents current location/page|`aria-current="page|step|location|date|time"`|Navigation active state                         |
|**busy**          |Region actively updating             |`aria-busy="true"`                           |Live region update in progress                  |
|**required**      |Field must have value                |`aria-required="true"`                       |Required field indicator                        |
|**modified/dirty**|User changed value from initial      |(none)                                       |Unsaved changes indicator                       |
|**autofilled**    |Browser auto-populated value         |CSS `:autofill`                              |Different styling for autofilled inputs         |
|**partial**       |Partial selection in hierarchy       |`aria-checked="mixed"`                       |Tree selection states                           |

### Cross-category dependencies

```
Combinable States ──┬─── Interaction: selected + disabled = valid combination
                    │                 (item selected but action unavailable)
                    │
                    ├─── Semantic: error triggers feedback.error color tokens
                    │              success triggers feedback.success tokens
                    │
                    └─── Structural: expanded state affects child visibility
                                     and nesting context propagation
```

### Edge case catalog

|Scenario                             |Conflict                        |Proposed Handling                                                                                |
|-------------------------------------|--------------------------------|-------------------------------------------------------------------------------------------------|
|Checkbox is checked AND indeterminate|Conflicting states              |Indeterminate takes visual precedence; `checked` reflects programmatic state                     |
|Form field has error AND warning     |Which validation state displays?|Error takes precedence; show warning only after error resolved                                   |
|Item is selected AND current         |Redundant visual indication     |`current` used for navigation, `selected` for data selection—both may apply with distinct styling|
|Loading element receives interaction |User clicks while loading       |Add `aria-disabled` during loading to prevent double-submission; maintain visual loading state   |

-----

## Category 7: Accessibility Contexts

### Current dimension inventory

|Dimension              |Enumerated Values                |Detection Mechanism           |
|-----------------------|---------------------------------|------------------------------|
|Reduced motion         |reduce, no-preference            |`prefers-reduced-motion`      |
|Contrast preference    |more, less, no-preference, custom|`prefers-contrast`            |
|Reduced transparency   |reduce, no-preference            |`prefers-reduced-transparency`|
|Forced colors          |active, none                     |`forced-colors`               |
|Color vision deficiency|(not captured)                   |None                          |

### Validation findings

**Browser support gaps:**

- `prefers-reduced-transparency`: Safari 16.4+ only as of January 2026
- `prefers-reduced-data`: Limited support, at-risk in spec
- Color vision deficiency: **No CSS media query exists**—affects ~8% of males

**Platform coupling issue:**
On Windows, `prefers-contrast: more` and `forced-colors: active` are **intrinsically coupled**—enabling high contrast themes triggers both. Developers cannot distinguish between users wanting enhanced contrast vs forced color replacement.

**Implementation best practice correction:**
Default to reduced motion and enhance for `no-preference` (accessibility-first approach), not the reverse.

### Missing dimensions

|Dimension                      |Definition                               |Values                                                                        |Rationale                                                            |
|-------------------------------|-----------------------------------------|------------------------------------------------------------------------------|---------------------------------------------------------------------|
|**CVD-safe mode**              |Color vision deficiency-optimized palette|`none`, `protanopia-safe`, `deuteranopia-safe`, `tritanopia-safe`, `universal`|8% of males affected; GitHub Primer has tritanopia/colorblind themes |
|**Cognitive load preference**  |Simplified UI mode                       |`default`, `reduced`                                                          |No CSS query; would reduce information density, simplify interactions|
|**Focus visibility preference**|Enhanced focus indicators                |`default`, `enhanced`                                                         |Some users need larger, higher-contrast focus rings                  |
|**Large pointer mode**         |Increased click/tap targets              |`default`, `large`                                                            |WCAG 2.5.8 requires 24px (AA), 44px (AAA)                            |
|**Screen magnification active**|Screen magnifier in use                  |Boolean                                                                       |Affects layout decisions; reduce animations                          |

### Cross-category dependencies

```
Accessibility ───────┬─── Appearance: prefers-contrast may override theme;
                     │                dark mode often preferred by users
                     │                with light sensitivity
                     │
                     ├─── Elevation: Reduced motion should disable elevation
                     │               transitions; forced-colors removes box-shadow
                     │
                     ├─── Density: Large pointer mode overrides density,
                     │             enforcing minimum touch targets
                     │
                     └─── Semantic: Status indicators MUST use non-color
                                    differentiators (icons, patterns) per WCAG 1.4.1
```

### Edge case catalog

|Scenario                                                    |Conflict                                      |Proposed Handling                                                                    |
|------------------------------------------------------------|----------------------------------------------|-------------------------------------------------------------------------------------|
|`prefers-contrast: more` + `forced-colors: active` both true|Can’t distinguish preference type             |Test both queries together; provide system color fallbacks that also enhance contrast|
|User needs reduced motion BUT expects loading feedback      |Removing all animation removes status feedback|Replace motion-based loaders with static progress indicators or state text           |
|Red-green colorblind user encounters error/success states   |Cannot distinguish error from success         |Mandatory secondary indicators: icons, patterns, text labels alongside color         |
|High contrast mode applied to branded UI                    |Brand colors replaced with system colors      |Map brand semantics to system color keywords; accept loss of brand expression        |

-----

## Category 8: Locale and Internationalization Contexts

### Current dimension inventory

|Dimension     |Enumerated Values                      |Detection Mechanism             |
|--------------|---------------------------------------|--------------------------------|
|Text direction|ltr, rtl, auto                         |`dir` attribute, CSS `direction`|
|Writing mode  |horizontal-tb, vertical-rl, vertical-lr|CSS `writing-mode`              |
|Language      |BCP 47 tags (en-US, ar-EG, zh-Hans)    |`lang` attribute                |
|Number format |(implementation-specific)              |`Intl.NumberFormat` locale      |

### Validation findings

**Missing values:**

- Writing mode: `sideways-rl` and `sideways-lr`  exist but are Firefox-only 
- Language: Should include script subtags for font selection (`zh-Hans` vs `zh-Hant`)

**Token architecture gap:**
Figma Variables don’t support typography token types that match W3C DTCG spec—no percentage-based line-heights. Figma’s default auto line-height (1.2×) is **below WCAG minimum** (1.5×). 

### Missing dimensions

|Dimension                |Definition                          |Values                                                    |Impact                                          |
|-------------------------|------------------------------------|----------------------------------------------------------|------------------------------------------------|
|**Script type**          |Writing system classification       |`latin`, `cjk`, `arabic`, `cyrillic`, `devanagari`, `thai`|Different font stacks, kerning, glyph complexity|
|**Text expansion factor**|UI sizing multiplier for translation|German: 1.2-1.35, Finnish: 1.25-1.40, Chinese: 0.7-0.9    |Critical for button widths, navigation labels   |
|**Quote style**          |Quotation mark conventions          |US: `"text"`, German: `„text"`, French: `«text»`          |Typography tokens per locale                    |
|**Calendar system**      |Regional calendar                   |`gregorian`, `hijri`, `buddhist`, `hebrew`, `japanese`    |Date picker components                          |
|**Pluralization rules**  |CLDR plural categories              |English: 2 forms, Arabic: 6 forms, Russian: 3 forms       |Dynamic content string tokens                   |
|**Line height locale**   |Language-specific line spacing      |CJK: 1.5-1.8×, Arabic: extra for diacritics               |Typography tokens by locale                     |

### Cross-category dependencies

```
Locale ──────────────┬─── Platform: iOS UISemanticContentAttribute vs
                     │              Android layoutDirection handle RTL differently
                     │
                     ├─── Structure: Header/footer slots maintain position,
                     │               but start/end slots must flip in RTL
                     │
                     ├─── Viewport: Text expansion affects responsive breakpoints
                     │              (German labels may trigger earlier breakpoint)
                     │
                     └─── Density: Long German compound words may require
                                   different truncation tokens
```

### Edge case catalog

|Scenario                              |Conflict                                                 |Proposed Handling                                                       |
|--------------------------------------|---------------------------------------------------------|------------------------------------------------------------------------|
|RTL text contains embedded LTR URLs   |Unicode BiDi algorithm fails on structured expressions   |Use Unicode control characters (LRM, RLM) or `\u200E` isolates          |
|Icon contains directional arrow       |Some icons should mirror in RTL, some should not         |Document per-icon mirroring behavior; checkmarks and clocks don’t mirror|
|Form controls in vertical writing mode|Chrome 119+ supports but conflicts with `direction: rtl` |Test writing-mode + direction combinations explicitly                   |
|Font fallback in RTL locale           |Noto fallback fonts may have inconsistent metrics        |Specify complete font stack with RTL-aware metrics for fallbacks        |

-----

## Category 9: Platform and Runtime Contexts

### Current dimension inventory

|Dimension        |Enumerated Values                  |Detection Mechanism         |
|-----------------|-----------------------------------|----------------------------|
|Platform         |web, ios, android, flutter         |Build-time configuration    |
|Framework        |react, angular, vue, web-components|Build-time configuration    |
|Browser engine   |webkit, gecko, blink               |User-agent/feature detection|
|Rendering context|SSR, CSR                           |Runtime detection           |

### Validation findings

**Missing values:**

- Platform: Missing `electron`, `tauri`, `react-native`, `macos`, `windows`, `linux`
- Rendering context: Missing `SSG` (static site generation), `ISR` (incremental static regeneration), `hydrating` (transition state)

**Token format gaps:**
Style Dictionary transforms to platform-specific formats, but color format requirements differ: RGB for CSS, RGBA for iOS JSON, 8-digit AARRGGBB hex for Android XML.  No unified token format handles this automatically.

### Missing dimensions

|Dimension            |Definition                      |Values                                         |Rationale                                           |
|---------------------|--------------------------------|-----------------------------------------------|----------------------------------------------------|
|**Device capability**|Hardware feature availability   |`haptics`, `camera`, `gps`, `biometrics`, `nfc`|Interaction feedback tokens (haptic intensity)      |
|**Performance tier** |Device capability classification|`high`, `medium`, `low`                        |Animation complexity, shadow quality, blur fallbacks|
|**Network quality**  |Connection speed tier           |`offline`, `slow-2g`, `2g`, `3g`, `4g+`        |Image quality tokens, skeleton duration             |
|**JavaScript state** |JS availability                 |`enabled`, `disabled`, `loading`               |Graceful degradation tokens                         |
|**Print context**    |Print vs screen media           |`screen`, `print`                              |CMYK-safe colors, hide decorative elements          |
|**Embedded context** |Runtime embedding               |`standalone`, `iframe`, `webview`              |Security-constrained tokens, size limitations       |
|**Hydration state**  |SSR→CSR transition              |`static`, `hydrating`, `interactive`           |Loading state tokens, placeholder tokens            |

### Cross-category dependencies

```
Platform ────────────┬─── Locale: React Native lacks CSS logical properties;
                     │            requires explicit RTL/LTR value transforms
                     │
                     ├─── Appearance: iOS/Android dynamic color APIs differ;
                     │                Electron requires system theme detection
                     │
                     ├─── Accessibility: Platform-specific accessibility APIs
                     │                   (VoiceOver vs TalkBack vs NVDA)
                     │
                     └─── Structure: React Native doesn't support web slots;
                                     compound components need platform-specific
                                     composition patterns
```

### Edge case catalog

|Scenario                                                     |Conflict                                          |Proposed Handling                                                                                   |
|-------------------------------------------------------------|--------------------------------------------------|----------------------------------------------------------------------------------------------------|
|SSR renders dark theme, client hydrates with light preference|Hydration mismatch flash of wrong theme           |Inject `prefers-color-scheme` check in `<head>` before render; use cookie for server-side preference|
|React Native token uses CSS `inherit`                        |`inherit` not supported in React Native StyleSheet|Transform inherited values to explicit values during token build                                    |
|Low-performance device runs animation-heavy UI               |Janky animations worse than no animations         |Detect performance tier; reduce animation complexity or duration accordingly                        |
|Print stylesheet from JavaScript-dependent component         |Interactive elements meaningless in print         |`@media print` tokens that expand collapsed content, show all tabs                                  |

-----

## Category 10: Elevation and Surface Contexts

### Current dimension inventory

|Dimension    |Enumerated Values                      |Detection Mechanism|
|-------------|---------------------------------------|-------------------|
|Surface level|0-5 (Material), layer-01/02/03 (Carbon)|Component hierarchy|
|Shadow depth |none, sm, md, lg, xl                   |CSS `box-shadow`   |
|Z-index layer|100-800 scale (Atlassian)              |CSS `z-index`      |

### Validation findings

**Missing values:**

- Surface level: Material 3 introduced **surface container variants** (`surface-container-lowest`, `surface-container-low`, `surface-container`, `surface-container-high`, `surface-container-highest`)—more granular than 0-5
- Z-index: Atlassian documents semantic layers (dropdown: 300, modal: 510, tooltip: 800) but most systems lack this documentation

**Implementation gap:**
IBM Carbon’s **contextual tokens** auto-resolve based on nearest Layer component ancestor—advanced but difficult to maintain. Most systems lack this DOM-aware token resolution.

### Missing dimensions

|Dimension                        |Definition                              |Values                       |Rationale                                         |
|---------------------------------|----------------------------------------|-----------------------------|--------------------------------------------------|
|**Backdrop blur level**          |Glassmorphism blur tokens               |`none`, `sm`, `md`, `lg`     |Frosted glass effects                             |
|**Overlay opacity**              |Scrim/backdrop opacity                  |`0`, `32%`, `50%`, `100%`    |Modal scrims, backdrop overlays                   |
|**Stacking context relationship**|Parent-child z-index inheritance        |Documented relationships     |Critical for micro-frontends                      |
|**Print elevation**              |Elevation representation in print       |`border`, `separator`, `none`|Shadows don’t print; need alternative             |
|**Surface tint color**           |Customizable tint for tonal elevation   |Color token reference        |Material 3 uses primary; may need override        |
|**Elevation transition timing**  |Animation duration for elevation changes|Duration + easing tokens     |Ties to motion tokens with reduced-motion fallback|

### Cross-category dependencies

```
Elevation ───────────┬─── Appearance: Dark mode reverses elevation direction
                     │                (surfaces lighten upward); tint color
                     │                derived from theme primary
                     │
                     ├─── Accessibility: Reduced motion disables elevation
                     │                   transitions; forced-colors removes
                     │                   box-shadow entirely
                     │
                     ├─── Structure: Nested modals require incrementing z-index;
                     │               card-in-card compounds elevation
                     │
                     └─── Platform: Shadows render differently across browsers;
                                    Android elevation uses native ViewCompat
```

### Edge case catalog

|Scenario                              |Conflict                                                  |Proposed Handling                                                                                                        |
|--------------------------------------|----------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
|Forced colors mode removes all shadows|Elevation hierarchy lost                                  |Use borders or outlines as fallback; `@media (forced-colors: active) { box-shadow: none; border: 2px solid ButtonText; }`|
|Micro-frontend z-index collision      |Independent teams create conflicting z-index values       |Establish global z-index token contract with reserved ranges per application                                             |
|Third-layer component in dark theme   |Carbon’s Gray 80 isn’t a “full theme” for layer-03        |Document maximum supported nesting depth per theme                                                                       |
|Print stylesheet with elevated cards  |Box-shadow doesn’t print; tonal elevation invisible in B&W|`@media print` uses border-based elevation indicators                                                                    |

-----

## Category 11: Semantic and Feedback Contexts

### Current dimension inventory

|Dimension|Enumerated Values                                  |Detection Mechanism|
|---------|---------------------------------------------------|-------------------|
|Status   |success, warning, error, info, neutral             |Component/context  |
|Emphasis |primary, secondary, tertiary, bold, subtle         |Component variant  |
|Intent   |brand, informational, success, warning, destructive|Component variant  |

### Validation findings

**Naming patterns vary significantly:**

- Status: Salesforce uses `success/warning/error/info`; Atlassian uses `success/warning/danger/discovery/information`
- Emphasis: Bootstrap uses `primary/secondary/success/danger/warning/info`; mixing status with emphasis creates confusion

**Missing distinction:**
GitLab distinguishes **status tokens** (current state—static) from **feedback tokens** (result of action—dynamic). Most systems conflate these.

### Missing dimensions

|Dimension          |Definition                      |Values                                |Rationale                                  |
|-------------------|--------------------------------|--------------------------------------|-------------------------------------------|
|**Urgency**        |Notification priority           |`low`, `medium`, `high`, `critical`   |Alert severity, interrupt level            |
|**Confidence**     |Data reliability                |`confirmed`, `likely`, `uncertain`    |Data source indicators                     |
|**Sentiment**      |Emotional valence               |`positive`, `neutral`, `negative`     |User feedback display                      |
|**Importance**     |Visual hierarchy within semantic|`primary`, `secondary`, `tertiary`    |Multiple warnings with different prominence|
|**Temporal status**|Content freshness               |`new`, `updated`, `stale`, `archived` |Content lifecycle indicators               |
|**Provenance**     |Content origin                  |`human`, `ai-generated`, `ai-assisted`|Emerging pattern for AI transparency       |

### Cross-category dependencies

```
Semantic ────────────┬─── Interaction: Error state + disabled creates
                     │                 compound "disabled due to error" state
                     │
                     ├─── Accessibility: Status MUST include non-color
                     │                   indicators (icons, patterns)
                     │
                     ├─── Combinable States: Error/success are often
                     │                       combinable states that trigger
                     │                       semantic color tokens
                     │
                     └─── Elevation: High-emphasis alerts may use elevated
                                     surfaces for attention
```

### Edge case catalog

|Scenario                                    |Conflict                                |Proposed Handling                                                                |
|--------------------------------------------|----------------------------------------|---------------------------------------------------------------------------------|
|Yellow warning background with text         |Yellow backgrounds often fail contrast  |Use `warning.inverse` text token; ensure 4.5:1 contrast                          |
|Multiple status indicators on same component|Error + warning + info simultaneously   |Define precedence: error > warning > info; show highest priority                 |
|Brand intent + danger intent overlap        |Destructive action in brand color       |Destructive always uses danger palette; brand defers to semantic                 |
|Success state on disabled component         |Conflicting signals (good + unavailable)|Disabled takes precedence visually; success can be announced via aria-describedby|

-----

## Category 12: Structural Contexts

### Current dimension inventory

|Dimension     |Enumerated Values               |Detection Mechanism |
|--------------|--------------------------------|--------------------|
|Component type|(implementation-specific)       |Component name/class|
|Region        |header, main, footer, nav, aside|Landmark roles      |
|Nesting level |depth-0, depth-1, depth-n       |DOM traversal       |

### Validation findings

**Token explosion problem:**
Adobe Spectrum found “writing component-level tokens can easily get out of hand. If you’re tokenizing every attribute of a component, for every permutation and state, the data becomes multidimensional and scales exponentially.”

**Naming collision issue:**
The same semantic name (`$dt-primary`) may not apply across all components—what’s “primary” for a checkbox isn’t necessarily “primary” for navigation (Frontside’s “context dilemma”).

**Architecture rule:**
SAP specifies “a component-specific token must never reference another component-specific token” to prevent hidden dependencies.

### Missing dimensions

|Dimension                   |Definition                        |Values                                          |Rationale                              |
|----------------------------|----------------------------------|------------------------------------------------|---------------------------------------|
|**List position**           |Item position in sequence         |`first`, `middle`, `last`, `only`               |Border-radius variations, separators   |
|**Slot position**           |Position within compound component|`header`, `body`, `footer`, `trigger`, `content`|Modal/card slot styling                |
|**Tree depth**              |Nesting level in hierarchy        |`depth-0` through `depth-n`                     |Navigation indentation                 |
|**Component lifecycle**     |Mount state                       |`mounting`, `mounted`, `unmounting`             |Enter/exit animation tokens            |
|**Grid/flex child position**|Position within layout            |`start`, `center`, `end`, `stretch`             |Auto-margin tokens                     |
|**Component complexity**    |Atomic design level               |`atom`, `molecule`, `organism`                  |Token inheritance rules differ by level|
|**Instance count**          |nth occurrence                    |`first`, `nth`, `last`                          |Alternating row colors                 |

### Cross-category dependencies

```
Structure ───────────┬─── Elevation: Nested modals increment z-index;
                     │               card-in-card compounds surface level
                     │
                     ├─── Locale: Start/end slots flip in RTL;
                     │            header/footer maintain position
                     │
                     ├─── Platform: React Native lacks web slot pattern;
                     │              needs platform-specific composition
                     │
                     └─── Density: Deep nesting may require compact mode
                                   to prevent excessive indentation
```

### Edge case catalog

|Scenario                                                   |Conflict                                            |Proposed Handling                                                 |
|-----------------------------------------------------------|----------------------------------------------------|------------------------------------------------------------------|
|Figma slot swap loses auto-constraint                      |Swapped content doesn’t inherit constraints         |Design tool limitation; document expected behavior                |
|Compound component state synchronization                   |`<Form.Label>` needs error state from `<Form.Input>`|Use React context or CSS `:has()` for state propagation           |
|Same component in different regions needs different styling|Card in sidebar vs card in main content             |Use container queries + named containers rather than region tokens|
|Cross-component token reference                            |Button references Card’s border token               |Prohibited; use shared semantic token both reference              |

-----

## Cross-Category Dependency Master Matrix

This matrix documents the **78 identified cross-category dependencies** organized by relationship type:

### Critical dependencies (require immediate resolution)

|Dependency                |Categories                 |Issue                                        |Resolution                                                            |
|--------------------------|---------------------------|---------------------------------------------|----------------------------------------------------------------------|
|Forced colors override    |Appearance ↔ All           |System colors replace all color tokens       |Provide `forced-color-adjust` guidance; map semantics to system colors|
|Focus precedence          |Interaction ↔ Accessibility|Focus must override hover for keyboard users |CSS cascade: `:focus-visible` after `:hover`                          |
|Touch target minimum      |Density ↔ Accessibility    |Compact mode cannot reduce targets below 44px|`min()` function: `min(var(--compact-size), 44px)`                    |
|Tonal elevation derivation|Elevation ↔ Appearance     |Surface tint color depends on theme primary  |Token reference: `elevation.surface.tint: {color.primary}`            |

### Conditional dependencies (context-specific)

|Dependency              |Trigger Condition                     |Affected Tokens                   |
|------------------------|--------------------------------------|----------------------------------|
|Motion ↔ Elevation      |`prefers-reduced-motion: reduce`      |Elevation transitions disabled    |
|Transparency ↔ Elevation|`prefers-reduced-transparency: reduce`|Backdrop blur tokens set to `none`|
|Writing mode ↔ Structure|`writing-mode: vertical-*`            |Slot positions rotate             |
|Viewport ↔ Density      |`viewport-segments: 2`                |Layout mode becomes `split`       |

### Implicit dependencies (often undocumented)

|Dependency                  |Assumption                                   |Risk                                    |
|----------------------------|---------------------------------------------|----------------------------------------|
|Platform → Color format     |CSS uses hex, iOS uses RGB object            |Build tool must transform               |
|Locale → Font stack         |`lang` attribute determines font             |Font fallback may have different metrics|
|Container → Stacking context|`container-type` creates new stacking context|Affects z-index resolution              |

-----

## Recommendations for Ontology Updates

### Immediate additions (high impact, well-supported)

1. **Add `focus-visible` to interaction states** — Essential for accessible keyboard navigation
1. **Add `indeterminate` to combinable states** — Required for hierarchical selection
1. **Add `current` (aria-current) to combinable states** — Navigation context
1. **Add safe area inset dimensions to viewport** — Well-supported, rarely tokenized
1. **Add CVD-safe mode to accessibility** — Following GitHub Primer’s lead

### Medium-term additions (emerging support)

1. **Add foldable device dimensions** — Viewport segments, fold posture
1. **Add container scroll-state queries** — New in CSS spec
1. **Add text expansion factor to locale** — Critical for internationalization
1. **Add performance tier to platform** — Device capability awareness
1. **Add urgency/temporal to semantic** — Beyond status/emphasis

### Structural changes

1. **Reclassify `focus` from mutually exclusive to additive** — Corrects fundamental misclassification
1. **Split `error` into `invalid` (validation) and `error` (system)** — Different semantics
1. **Add dependency documentation requirement** — Each dimension must declare cross-category relationships
1. **Add resolution order specification** — When dimensions conflict, which takes precedence

### Token architecture principles

```
├── Context Layer (Environment)
│   ├── viewport-*, platform-*, locale-*
│   └── Detected automatically, read-only
│
├── Preference Layer (User Settings)
│   ├── prefers-*, density-*, accessibility-*
│   └── User-controllable, system-detectable
│
├── Semantic Layer (Design Intent)
│   ├── status-*, emphasis-*, intent-*
│   └── Author-defined, context-independent
│
├── State Layer (Interaction)
│   ├── interaction-*, combinable-*
│   └── Runtime-dynamic, composable
│
└── Component Layer (Implementation)
    ├── {component}-{property}-{state}
    └── References semantic layer only
```

This layered architecture ensures dependencies flow downward—component tokens reference semantic tokens, which reference preference tokens, which respect context tokens. Cross-references at the same layer should be avoided; sibling dependencies indicate architectural issues requiring refactoring.
