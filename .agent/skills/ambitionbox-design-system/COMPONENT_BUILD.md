---
name: AmbitionBox Component Build Playbook
description: Step-by-step execution rules for building Figma components — designer collaboration, build order, validation, and known gotchas
---

# COMPONENT BUILD PLAYBOOK

This playbook defines **how** to build design system components in Figma. It is the operational companion to `SKILL.md` (which defines the *what* — naming, token architecture, tier rules).

> [!IMPORTANT]
> **Golden Rule:** You are a design system *assistant*, not a decision-maker. Always plan, flag, suggest, and wait for the designer's approval before executing.

---

## 1. DESIGNER COLLABORATION PROTOCOL

### 1.1 Plan-First, Execute-Second

Before touching Figma, **always** present a complete build plan covering:

- [ ] Which component tokens will be created (with their full alias chains: Component → Semantic → Primitive)
- [ ] Which text styles will be used per variant
- [ ] Which states/variants will exist (with the variant property matrix)
- [ ] Which component properties are needed (Text, Boolean, Instance Swap, Variant)
- [ ] Which icons or sub-components are involved

**Format the plan as a clear table or matrix** so the designer can review it at a glance.

### 1.2 Flag Everything That's Missing

If **any** of the following don't exist in the design system, **stop and flag it** before proceeding:

- A semantic token (e.g., `Text Color/Brand Hover` doesn't exist)
- A text style (e.g., no underline variant for `Text/Label/M`)
- An icon component (e.g., `arrow_forward` not found)
- A primitive value (e.g., color shade not available in the scale)
- A spacing, border radius, or stroke width token

**Never silently create workarounds.** Never detach a style. Never hard-code a value.

### 1.3 Suggest Best Practices Proactively

Don't just flag problems — **recommend solutions**:

- *"These hover states should use one shade darker from the scale (Brand/70 instead of Brand/60), not just opacity changes"*
- *"This underline needs a system-level text style `Text/Label/M Underline`, not a manual textDecoration override"*
- *"This icon should be wired via Instance Swap, not embedded as a static child"*

### 1.4 Wait for Designer Go-Ahead

- **Never execute until the designer explicitly approves the plan**
- If there are multiple decisions to make, batch them into a clear numbered list
- If the designer says "go ahead" on a plan, execute exactly that plan — no silent additions or changes

### 1.5 Surface Tradeoffs

When there are multiple valid approaches, present them with pros/cons:

```
Option A: Create a new semantic token `Text Color/Brand Hover` → Brand/70
  ✅ Reusable across components  
  ⚠️ Adds one more token to maintain

Option B: Use existing `Text Color/Brand` with opacity change
  ✅ No new tokens  
  ❌ Breaks the established pattern of using darker shades for hover
```

---

## 2. TOKEN RESOLUTION & PRIMITIVE FALLBACK RULES

### 2.1 Always Search Semantics First

When a component needs a token value, **always search the Semantic tier first**. Component tokens should reference Semantics, not Primitives directly.

### 2.2 Primitive Fallback with Flagging

If a suitable semantic token does **not** exist, you **may** reference a Primitive directly as a temporary fallback, but you **MUST**:

1. **Flag it to the designer** with a clear explanation:
   - What semantic token is missing
   - Which primitive you're using as a fallback
   - Your recommendation on whether it should be promoted to a semantic token

2. **Give a preliminary judgement** on whether this needs a semantic token:

| Situation | Judgement | Reasoning |
|-----------|-----------|-----------|
| Value is used by 2+ components | **Should be semantic** | Reuse demands a shared token |
| Value represents a design intent (hover, error, disabled) | **Should be semantic** | Intent-driven values belong at semantic tier |
| Value is truly unique to this one component | **Can stay primitive** | No reuse benefit — component token → primitive is acceptable |
| Value maps to an existing semantic pattern (e.g., there's `Text Color/Brand` but no `Text Color/Brand Hover`) | **Should be semantic** | It's a gap in an existing pattern |

3. **Document the fallback** in your build plan so it's trackable:
   ```
   ⚠️ PRIMITIVE FALLBACK: Link/Primary/Text/Hover → Colors/Brand/70
      Missing semantic: Text Color/Brand Hover
      Recommendation: CREATE at semantic level (hover is a reusable intent)
      Status: Awaiting designer approval
   ```

### 2.3 Never Skip Silently

The absolute worst outcome is a component that works but has an invisible architectural shortcut. Every tier skip must be visible and approved.

---

## 3. ICON SOURCING RULES

### 3.1 Functional Icons (Specific Purpose)

When a component needs an icon with **specific semantic meaning** (checkbox indicator, radio button, toggle, navigation arrow, close button, etc.):

1. **First choice: Google Material Icons** from the `Icons` page → `Google Icons` section (3,480+ icons available)
2. Search for the icon by its Material Design name (e.g., `radio_button_unchecked`, `radio_button_checked`, `check_box`, `check_box_outline_blank`, `close`, `arrow_forward`)
3. If the needed icon is **not found** in Google Icons, **ask the designer** which library to use
4. **Always confirm with the designer** if there's a custom/alternative library they prefer for this specific use case

### 3.2 Generic / Decorative Icons (Placeholder)

When a component uses an icon as a **generic placeholder** that the consumer will swap out (e.g., a button with a trailing icon, a card with a leading icon):

1. Use a **neutral placeholder icon** from Google Icons (e.g., `star`, `add`, `arrow_forward`)
2. The specific icon doesn't matter because the consumer will Instance Swap it
3. Focus on getting the **size and spacing** correct, not the specific icon choice

### 3.3 Decision Matrix

| Icon Type | Example | Source | Instance Swap? |
|-----------|---------|--------|----------------|
| **Functional indicator** | Radio circle, Checkbox, Toggle | Google Icons (specific name) | ✅ Yes — different states swap to different icons |
| **Functional navigation** | Back arrow, Close X, Menu | Google Icons (specific name) | ⚠️ Ask designer — may be fixed or swappable |
| **Generic placeholder** | Button trailing icon, Card icon | Google Icons (any neutral icon) | ✅ Yes — consumer decides the icon |
| **Custom/brand** | Logo, Brand mark | Ask designer | ❌ Typically fixed |

> [!IMPORTANT]
> **Never use a sub-component or icon from a different library without confirming with the designer.** If you find a similar component from another library/page, stop and ask: "I found `Radio Buttons` on the current page, but should we use `radio_button_unchecked` from Google Icons instead?"

---

## 4. PRE-BUILD AUDIT CHECKLIST

Before building any component, verify:

- [ ] **Tokens:** All required semantic tokens exist (colors, spacing, border radius)
- [ ] **Text Styles:** All required text styles exist, including variants (underline, caps)
- [ ] **Icons:** All required icons identified + sourced from correct library (see Section 3)
- [ ] **Icon type classified:** Functional vs. generic — determines Instance Swap requirement
- [ ] **Component Token Naming** follows `[Component].[Variant].[Property].[State]`
- [ ] **States matrix** is complete (Default, Hover, Disabled, etc.)
- [ ] **Platform scope** is clear (Mobile-only? Web-only? Both?)
- [ ] **Documentation sheet** will be created after the build (see Section 5, Step 7)

If any check fails, flag it (see Section 1.2) and wait for designer approval before proceeding.

---

## 5. BUILD ORDER-OF-OPERATIONS

Follow this **exact sequence** to avoid style detachment and binding failures:

### Step 1: Create Component Tokens
Create all component-level variables as aliases to semantic tokens (or flagged primitives).

```
Link/Primary/Text/Default   → {Text Color/Brand}
Link/Primary/Text/Hover     → {Text Color/Brand Hover}
Link/Secondary/Text/Default → {Text Color/Tertiary}
Link/Secondary/Text/Hover   → {Text Color/Tertiary Hover}
Link/Layout/Gap             → {Spacing/2XS}
```

### Step 2: Create Component Frame & Variants
Build the component set structure with all variants defined by properties (Type, Size, State).

### Step 3: Apply Text Styles (CRITICAL ORDER)

> [!CAUTION]
> **This is the #1 source of bugs.** Typography styles must be applied in this exact order:

1. **Load the font first:**
   ```js
   await figma.loadFontAsync(textNode.fontName);
   ```

2. **Apply the text style (async):**
   ```js
   await textNode.setTextStyleIdAsync(styleId);
   ```
   ⚠️ Using `textNode.textStyleId = styleId` (sync) will silently fail in dynamic-page mode.

3. **Then bind fill variables:**
   ```js
   const newFill = figma.variables.setBoundVariableForPaint(
     { type: 'SOLID', color: { r: 0, g: 0, b: 0 } },
     'color',
     variable
   );
   textNode.fills = [newFill];
   ```

**Why this order matters:** Setting fills *before* applying a text style can cause the style to detach, because the fill overrides a property the style was managing. Always: **style first → fills second**.

### Step 4: Wire Instance Swap Properties (MANDATORY for icons)

> [!IMPORTANT]
> **Every icon or indicator sub-component MUST have an Instance Swap property** unless the designer explicitly says otherwise. This is not optional.

For each icon/indicator in the component:
1. **Identify all icon instances** in the component tree
2. **Add Instance Swap property** to the component set for each icon
3. **Set the default component key** (from Google Icons or the correct library)
4. **Verify** the swap works by checking `componentProperties` on the instance
5. If an icon should NOT be swappable, **document why** (e.g., "Close icon is fixed per designer")

### Step 5: Wire Boolean & Text Properties
- Boolean: show/hide elements (e.g., `showTrailingIcon`)
- Text: editable content (e.g., `Label`)

### Step 6: Screenshot & Validate
Capture a screenshot of every variant and run the Post-Build Validation Checklist (Section 7).

### Step 7: Create Documentation Sheet (MANDATORY)

Every component **must** have a documentation/specification sheet created alongside it. This is a Figma Section placed near the component on the same page.

**Required sections in the documentation sheet:**

1. **Component Name** — Title text (heading style)
2. **Description** — 1-2 sentence explanation of what the component is and when to use it
3. **Component Preview** — An instance of the component showing each variant
4. **Component Properties Table** — All properties with their type, values, and defaults:
   ```
   Property    Type       Values
   ─────────────────────────────────────────
   State       VARIANT    Unselected / Selected
   Label       TEXT       "Label" (default)
   Icon        INSTANCE   radio_button_unchecked (default)
   ```
5. **Design Tokens Table** — Token names, alias chains, and resolved values:
   ```
   Token                              Chain                                  Value
   ──────────────────────────────────────────────────────────────────────────────────
   Padded Radio Button/Bg/Default     → Brand_Neutral/10                     #F2F2F6
   Padded Radio Button/Text/Default   → Text Color/Primary → Brand_Neutral/90  #1E223C
   ```
6. **Anatomy** — Visual breakdown showing which part maps to which token (optional but recommended)
7. **Usage Notes** — Do's and Don'ts, edge cases, accessibility notes (optional)

**Layout:** Create as a Figma Section named `[Component Name] — Component Spec` with a white background, ~720px wide.

---

## 6. COMPONENT PROPERTIES REFERENCE

### 5.1 Property Types

| Type | Purpose | Example |
|------|---------|---------|
| **Variant** | Switches between component variants | `Type`: Primary / Secondary |
| **Text** | Editable label content | `Label`: "Link Label" |
| **Boolean** | Show/hide elements | `Show Trailing Icon`: true/false |
| **Instance Swap** | Swap a sub-component | `Trailing Icon`: arrow_forward / chevron_right |

### 5.2 Instance Swap Wiring

To correctly wire an Instance Swap:

1. **Identify the icon/sub-component** in the component tree
2. **Add the Instance Swap property** to the component set:
   ```js
   figma_add_component_property(nodeId, 'Trailing Icon', 'INSTANCE_SWAP', defaultComponentKey)
   ```
3. **Verify** the swap works by checking `componentProperties` on the instance

### 5.3 Text Styles Available in the System

| Style Name | Font Size | Weight | Decoration | Use Case |
|------------|-----------|--------|------------|----------|
| `Text/Display/L` | 48px | Bold | None | Hero headlines |
| `Text/Display/M` | 32px | Bold | None | Marketing headers |
| `Text/Display/S` | 28px | Bold | None | Section headers |
| `Text/Heading/L` | 24px | SemiBold | None | Page titles |
| `Text/Heading/M` | 18px | SemiBold | None | Section titles |
| `Text/Heading/S` | 14px | SemiBold | None | Sub-section titles |
| `Text/Heading/XS` | 12px | SemiBold | None | Small headings |
| `Text/Body/L` | 18px | Regular | None | Large body text |
| `Text/Body/M (Default)` | 16px | Regular | None | Default body text |
| `Text/Body/S` | 14px | Regular | None | Small body text |
| `Text/Body/XS` | 12px | Regular | None | Footnotes |
| `Text/Meta/L` | 14px | Regular | None | Large metadata |
| `Text/Meta/M (normal)` | 12px | Regular | None | Metadata |
| `Text/Meta/M (All Caps)` | 12px | Regular | None | Uppercase metadata |
| `Text/Meta/S (normal)` | 10px | Regular | None | Small metadata |
| `Text/Meta/S (All Caps)` | 10px | Regular | None | Uppercase small metadata |
| `Text/Label/L` | 16px | SemiBold | None | Large labels, buttons |
| `Text/Label/M` | 14px | SemiBold | None | Medium labels, buttons |
| `Text/Label/S` | 12px | SemiBold | None | Small labels |
| `Text/Label/M Underline` | 14px | SemiBold | **Underline** | Medium link labels |
| `Text/Label/S Underline` | 12px | SemiBold | **Underline** | Small link labels |

> [!NOTE]
> If you need a text style that doesn't exist (e.g., `Text/Body/M Underline`), **flag it** and propose creating it at the system level. Never manually set `textDecoration` without a backing style.

---

## 7. POST-BUILD VALIDATION CHECKLIST

After building, run through **every** check:

### Visual Validation
- [ ] Screenshot every variant at 2x scale
- [ ] Hover state colors are **visibly different** from default (darker shade)
- [ ] Underlines appear where expected (hover states, secondary links)
- [ ] Icon sizes are correct and aligned with text
- [ ] Spacing and padding match token values

### Style Attachment Validation
- [ ] Every text node has a **bound text style** (not detached)
- [ ] Every text fill has a **bound variable** (not hardcoded color)
- [ ] Every spacing value references a **token** (not manual pixel value)
- [ ] Opacity is controlled by a token where applicable

### Property Validation
- [ ] Instance Swap toggles correctly between icon options
- [ ] **Every icon has Instance Swap** (or explicit waiver documented)
- [ ] Boolean properties show/hide the correct elements
- [ ] Text properties update the label content
- [ ] Variant properties switch between all states/types/sizes
- [ ] **Documentation sheet created** with properties table, token table, and description

### Token Chain Validation
- [ ] Component tokens → Semantic tokens → Primitive values (no tier skipping)
- [ ] Hover tokens point to **different** values than default tokens
- [ ] No duplicate/stale variables with the same name

---

## 8. KNOWN GOTCHAS & FAILURE MODES

### 7.1 Typography

| Problem | Cause | Fix |
|---------|-------|-----|
| Text style silently not applied | Used sync `textStyleId =` instead of `setTextStyleIdAsync` | Always use async API |
| Text style applied but then lost | Fill variable set after style, causing detachment | Apply style first, fill second |
| Underline missing after rebuild | Text style was manually overridden, not using system style | Use `Text/Label/M Underline` or `S Underline` styles |
| Font load error | Font not loaded before text modification | Always call `loadFontAsync` first |

### 7.2 Instance Swap

| Problem | Cause | Fix |
|---------|-------|-----|
| Instance Swap not applied | Property added but not wired to the icon node | Verify `componentProperties` includes the swap |
| Wrong icon showing | Used node ID instead of component key | Use component key for published components, node ID for local |
| Swap property missing after arrange | Component set was recreated by `combineAsVariants` | Re-add properties after arranging |

### 7.3 Variables & Tokens

| Problem | Cause | Fix |
|---------|-------|-----|
| Hover looks identical to default | Both states point to same semantic token | Create separate hover semantic token (one shade darker) |
| Stale variable IDs | Variable IDs from previous sessions/files | Always re-fetch variable IDs at start of each session |
| Duplicate component tokens | Multiple builds created overlapping tokens | Audit existing tokens before creating new ones |

### 7.4 Git & Deployment

| Problem | Cause | Fix |
|---------|-------|-----|
| Push rejected by GitHub | Secret scanning detected tokens in code | Use `.gitignore` to exclude files with secrets |
| Wrong files committed | `git add .` includes everything | Only stage specific files needed |

---

### 8.5 Icon Sourcing

| Problem | Cause | Fix |
|---------|-------|-----|
| Icon from wrong library | Used a component from current page instead of Google Icons | Always search `Icons` page → `Google Icons` section first |
| Instance Swap missing on icon | Treated icon as static child | Every icon MUST have Instance Swap unless explicitly waived |
| No documentation sheet | Skipped Step 7 | Always create spec sheet after build |

---

## 9. ESCALATION RULES

If you encounter any of these situations, **stop and escalate to the designer immediately**:

1. **Conflicting design decisions** — Two existing patterns contradict each other
2. **Token naming ambiguity** — Unclear whether something is `Primary` vs `Brand`
3. **Cross-component impact** — A change affects other existing components
4. **Platform divergence** — Mobile and web need fundamentally different approaches
5. **Accessibility concerns** — Contrast ratios, focus states, or screen reader issues
6. **Performance risks** — Very large component sets (20+ variants) that may slow Figma

---

**Version:** 1.0  
**Last Updated:** March 2026  
**Companion to:** [SKILL.md](./SKILL.md) (Token Architecture & Naming Conventions)
