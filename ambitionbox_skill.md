---
name: AmbitionBox Design System Generator
description: A comprehensive set of instructions and rules for generating, updating, and validating UI components and design tokens for the AmbitionBox Design System in Figma.
---

# AmbitionBox Design System Rules & Prompts

When instructed to create or update a component in Figma for the AmbitionBox Design System, you **MUST** strictly adhere to the following architectural rules, naming conventions, and construction workflows.

## 1. Design Token Architecture
Tokens are organized into a strict 3-tier hierarchy across distinct Figma Collections:

1.  **Primitives (`Numbers/*`, `Colors/*`, `Typography/*`):**
    *   **Rule:** Components must *never* reference primitives directly.
    *   **Numbers:** Defined in a base-2/4/8 scale (e.g., `Numbers/Solids/4`, `Numbers/Solids/6`, `Numbers/Solids/16`). Raw integer geometry.
    *   **Pills/Full Round:** Use `Border Radius/Full` (9999px) for pill shapes (e.g., Chips, Badges).

2.  **Semantics (`Spacing/*`, `Icon Size/*`, `Border Radius/*`, `Text Color/*`):**
    *   **Rule:** Aliased exclusively to Primitives. Represents intent (e.g., `Spacing/M` aliases to `Numbers/Solids/12`).
    *   **CRITICAL FALLBACK RULE:** You MUST always attempt to select a token from the `Semantics` collection for a component property. If a suitable semantic token is not available, you MUST ask the designer whether to add a new token at the Semantic level or if it's acceptable to pick directly from Primitives. Do not make this decision autonomously.
    *   **Typography:** The primary text size for interactive UI elements (Buttons, Chips, Labels) is `Label M` (14px font size, 18px line height). Use `Typography/Font Size/Label M` and `Typography/Line Height/Label M`.
    *   **Icon Sizes:** Standard icon sizes scale from XS (12px) to XL (32px). `Icon Size/S` (16px) is standard for inline UI elements accompanying 14px text.

3.  **Components (`Button/*`, `Chip/*`, `Card/*`):**
    *   **Rule:** Aliased exclusively to Semantics. Represents specific component traits.
    *   **Format:** `[ComponentName]/[Type?]/[Property]/[State?]` (e.g., `Chip/Primary/Bg/Selected Hover`, `Button/Primary/Text/Default`).
    *   **Layout Tokens:** Define specific paddings (`Padding-Vertical`, `Padding-Horizontal`), gaps (`Gap`), and border radii for the component. Tie these to Spatial Semantics (e.g., `Chip/Gap` -> `Spacing/XS`).

## 2. Interaction States & Theming
When generating stateful components (like Buttons, Chips, List Items), ensure variants exist for all applicable interaction states:

*   **Default:** Base resting state.
*   **Hover:** Slightly darker/tinted background (e.g., using `Yellow/20` instead of `Yellow/10`).
*   **Pressed:** Visibly depressed or darker than hover.
*   **Disabled:** Must use `Surface/Disabled` background and `Text Color/Disabled` text. Borders should typically be removed or set to a disabled color.
*   **Selected:** Active/Toggled state.
*   **Selected Hover:** The state when a user hovers over an already selected component (Critical for complex forms/toggles). E.g., `Chip/Primary/Bg/Selected Hover`.

## 3. Figma Component Construction
When building `COMPONENT_SET` node trees in Figma via API/MCP, follow this construction pattern:

1.  **Use Auto-Layout (`FRAME`):** All components must be built using Auto-Layout.
2.  **Variant Properties:** Group logic by axes like `Type` (Primary, Secondary), `Size` (Large, Medium, Small), and `State` (Default, Hover, etc.).
3.  **Boolean Properties:** If an element is optional (e.g., "Leading Icon", "Trailing Icon"), create a Boolean component property defaulting to `false`. Bind the node's `visible` attribute to this property.
4.  **Text Properties:** Bind text node contents (e.g., "Label") to a Text component property.
5.  **Instance Swap Properties (Icons):**
    *   **Rule:** DO NOT create random placeholder shapes.
    *   **Rule:** ALWAYS use the existing global `Placeholder Icon` component (ID: `61:767`) for icon slots.
    *   Create an Instance Swap component property on the Component Set with the default value set to `61:767`.
    *   Bind the placeholder instance's `mainComponent` to this swap property.
6.  **Binding Fill to Icons:** To ensure icons inherit the correct state color (Text Color), bind the appropriate Component token (e.g., `Chip/Primary/Text/Disabled`) **to the fill of the nested vector/shape layer inside the placeholder instance**, NOT to the background of the instance frame itself.
7.  **Dimensions:** Bind icon instance `width` and `height` to the appropriate component token (e.g., `Button/Size/Medium/Icon Size` which routes to `Icon Size/M`). Avoid arbitrary stretches.

## 4. Visual Organization & Documentation Grid
When programmatically generating a component set, arrange the variants into a legible, labeled 2D documentation grid:
*   **Rule:** You MUST use the `figma_arrange_component_set` MCP tool to automatically organize the component set.
*   The tool recreates the set with proper Figma integration, applies a purple dashed border, and arranges variants in a labeled grid (columns and row labels based on properties like Type, Size, and State).
*   It creates a white container with a title, row/column labels, and the component set itself, providing clear visual documentation for designers reviewing the file.
*   Ensure sufficient padding (e.g., 24px gap) between variants.

## Prompt Example for Future Agents:

> **System Prompt Wrapper for Task Kickoff:**
> "You are building a new UI Component [Component Name] for the AmbitionBox Design System.
> 1. Start by auditing the `Primitives` and `Semantics` collections for necessary raw geometries and scale steps.
> 2. Create the `[Component Name]/*` component-level tokens in the `Components` collection, aliasing them to the `Semantics` collection. Include properties for Layout (Padding-V, Padding-H, Gap, Radius) and Colors (Bg, Text, Border across applicable interaction states). Note: Use `Label M` for standard typography and `Icon Size/S` for inline icons unless specified otherwise.
> 3. Generate the Figma Component Set using the API. Ensure it uses Auto-Layout, Boolean properties (default false for icons), Instance Swaps (default to `61:767` Placeholder Icon), and Text properties.
> 4. Bind the tokens to the nodes. Explicitly ensure icon instance fills (the inner shape element) are bound to the component's text color token.
> 5. Use the `figma_arrange_component_set` tool to generate a visual documentation grid with row/column labels.
> 6. Capture a screenshot via MCP of the arranged set for visual validation."
