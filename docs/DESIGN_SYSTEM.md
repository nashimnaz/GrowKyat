# Growkyat — Design System & UI/UX Standards

**Version:** 1.0.0  
**Purpose:** Establish a consistent design language for the entire Growkyat platform.

## 1. Core Principle

Every page, feature, and module in Growkyat must follow one shared design system.

Whether a developer builds the landing page, authentication, dashboard, campaign management, lead management, integrations, or AI tools, the result must feel like part of the same application.

These rules apply to all developers, AI coding assistants, generated code, and future contributors.

**Do not create a new visual style for an individual page or module. Reuse the existing design tokens and components.**

## 2. Overall Visual Direction

Growkyat should have a modern, premium SaaS aesthetic.

The interface should feel:

- Clean, minimal, and professional.
- Spacious, with clear visual hierarchy.
- Bright and lightweight rather than visually heavy.
- Polished through subtle gradients, shadows, and borders.
- Interactive without unnecessary animation.
- Consistent across marketing pages and the application dashboard.

Use white and light surfaces as the primary foundation, with soft gradient lighting and restrained decorative effects where appropriate.

Avoid excessive gradients, strong shadows, unnecessary glass effects, cluttered layouts, oversized decorations, and inconsistent component styling.

## 3. Brand Identity

- Preserve the official Growkyat logo and its proportions.
- Do not recreate, distort, recolour, or replace the logo without explicit approval.
- Use the existing brand identity consistently.
- Keep brand colours configurable through central theme tokens.
- Do not invent new brand colours for individual pages.
- Do not hardcode brand colours throughout components.

The colour palette may evolve in the future. A palette change must be possible centrally without redesigning the entire application.

## 4. Central Design Tokens

All shared visual values must be defined centrally in the frontend theme.

The theme must provide tokens for:

- Primary brand colour.
- Secondary brand colour.
- Accent colour.
- Page background.
- Surface and card backgrounds.
- Primary and secondary text.
- Borders.
- Success, warning, error, and information states.
- Typography.
- Spacing.
- Border radii.
- Shadows.
- Responsive breakpoints.
- Transitions and animation durations.

Suggested implementation location:

`frontend/src/styles/theme.css`

The exact path may be adjusted to match the project structure, but there must be one agreed central theme.

Use semantic names such as `--color-primary`, `--color-background`, and `--color-text-primary` rather than scattering raw hexadecimal values throughout the code.

**Important:** The Markdown file documents the rules; the actual theme tokens must also be implemented in the frontend code.

## 5. Typography

Use one consistent typography system throughout the platform.

Define shared styles for:

- Hero headings.
- Page headings.
- Section headings.
- Card headings.
- Body text.
- Secondary text.
- Labels and helper text.
- Buttons and navigation.

Rules:

- Maintain a clear heading hierarchy.
- Keep body text readable.
- Use consistent font weights and line heights.
- Avoid arbitrary font sizes on individual pages.
- Avoid introducing additional font families without approval.
- Use responsive typography for large headings.

The final font family and exact type scale should be configured centrally.

## 6. Layout and Spacing

Use a consistent spacing scale throughout the application.

Define shared values for:

- Page container widths.
- Section padding.
- Grid gaps.
- Card padding.
- Form spacing.
- Navigation spacing.
- Element gaps.
- Responsive page gutters.

Rules:

- Align content to consistent container boundaries.
- Maintain balanced whitespace.
- Use consistent spacing between similar elements.
- Avoid arbitrary margin and padding values.
- Prefer reusable layout utilities and shared spacing tokens.
- Ensure layouts work across desktop, tablet, and mobile.

## 7. Buttons

All buttons must use shared reusable components.

Required variants:

- Primary.
- Secondary.
- Outline.
- Ghost.
- Destructive.

Each variant must have consistent:

- Height and padding.
- Typography.
- Border radius.
- Colours.
- Hover and focus states.
- Disabled state.
- Loading state.
- Icon alignment and spacing.

Rules:

- Do not create custom button styling for individual pages when an existing variant is suitable.
- Use semantic buttons for actions and links for navigation.
- Avoid inconsistent button heights or radii.
- Ensure keyboard focus is visible.
- Prevent repeated submissions while an operation is processing.

## 8. Forms and Inputs

All forms must use shared form components and consistent validation patterns.

Supported components include:

- Text inputs.
- Email and password inputs.
- Search inputs.
- Text areas.
- Select controls.
- Checkboxes.
- Radio buttons.
- Toggle controls.
- Date and time controls.

Each component must support appropriate labels, focus states, validation feedback, disabled states, and error messages.

Rules:

- Keep labels and field spacing consistent.
- Never rely on placeholder text as the only label.
- Display validation messages close to the relevant field.
- Use consistent input heights, borders, radii, and typography.
- Keep form layouts responsive.
- Make keyboard interaction accessible.

## 9. Cards and Surfaces

Use reusable card and panel components for consistent presentation.

Examples include:

- Feature cards.
- Dashboard summary cards.
- Campaign cards.
- Integration cards.
- AI tool cards.
- Form panels.
- Settings panels.

Cards must use the shared tokens for background, border, radius, padding, and shadow.

Prefer subtle borders and restrained shadows. Avoid giving every card a different appearance.

Use consistent card styles unless a documented component variant is required.

## 10. Navigation

Maintain separate, consistent navigation patterns for the public website and authenticated application.

### Public website

Use a consistent header, navigation links, calls to action, and footer.

### Merchant dashboard

Use a consistent dashboard navigation structure, page header, content area, and active-navigation state.

Rules:

- Keep navigation labels clear and consistent.
- Highlight the active section.
- Use consistent icon sizes and spacing.
- Provide an intentional mobile navigation layout.
- Do not add navigation items for features outside the agreed product scope.
- Do not display non-functional links as if they were working features.

## 11. Page Structure

Pages serving similar purposes should follow shared layout patterns.

A typical authenticated page should reuse the relevant elements:

1. Dashboard shell and navigation.
2. Page header with title and optional description.
3. Primary action area, where needed.
4. Main content.
5. Loading, empty, success, or error states as applicable.

Marketing pages should use consistent section spacing, heading styles, content widths, and call-to-action components.

Not every page needs the exact same layout, but every page must use the same design language.

## 12. AI and Automation Interfaces

AI-related features must follow the same design system as the rest of Growkyat.

Shared patterns may include:

- Prompt input areas.
- Suggested action chips.
- Generation buttons.
- Preview panels.
- Progress indicators.
- Result cards.
- Confirmation dialogs.
- Error and retry states.

Rules:

- Clearly distinguish user input, AI-generated output, and completed actions.
- Show loading or progress feedback during longer operations.
- Provide understandable error messages.
- Do not claim an operation succeeded until the relevant backend response confirms it.
- Require confirmation for consequential actions where appropriate.
- Keep all AI interfaces consistent with the shared theme.

## 13. Icons and Illustrations

Use one consistent icon library for general interface icons.

Rules:

- Keep icon size, stroke weight, and alignment consistent.
- Prefer simple, clear icons.
- Avoid mixing unrelated icon styles.
- Use official integration assets where appropriate.
- Preserve the official Growkyat logo.
- Ensure decorative illustrations do not interfere with content or interaction.

## 14. Motion and Interactions

Use animation only when it improves usability or visual quality.

Acceptable examples include:

- Subtle hover transitions.
- Small card entrance effects.
- Smooth dropdown and modal transitions.
- Lightweight decorative motion.
- Progress indicators.
- Carefully implemented scrolling effects.

Rules:

- Avoid excessive motion and distracting animations.
- Keep interaction timings consistent.
- Respect reduced-motion preferences.
- Do not let animations delay essential interactions.
- Consider performance on mobile and lower-powered devices.
- Do not add complex animation libraries without a clear need.

## 15. Responsive Design

Every page must support desktop, tablet, and mobile layouts.

Rules:

- Use responsive layouts and shared breakpoints.
- Avoid fixed widths that cause overflow.
- Make navigation usable on smaller screens.
- Adapt grids and cards to the available space.
- Ensure forms and dialogs fit the viewport.
- Keep text readable without unnecessary zooming.
- Provide suitable touch targets.
- Make tables and dense data layouts usable on mobile.
- Prevent decorative elements from obscuring important content.

Responsive behaviour must be tested, not assumed from desktop styling.

## 16. Accessibility

All developers must follow these baseline accessibility rules:

- Use semantic HTML.
- Provide accessible labels for interactive controls.
- Support keyboard navigation.
- Maintain visible focus indicators.
- Use sufficient colour contrast.
- Associate validation errors with relevant form fields.
- Provide alternative text for meaningful images.
- Do not communicate status through colour alone.
- Respect reduced-motion preferences.

Accessibility must be considered during implementation rather than added only at the end.

## 17. Shared Component Architecture

Create a shared component library instead of duplicating UI across pages.

Initial components should include:

- Button.
- Input.
- Select.
- Checkbox.
- Card.
- Badge.
- Modal.
- Tooltip.
- Page header.
- Navigation components.
- Loading state.
- Empty state.
- Error state.
- Confirmation dialog.

Suggested locations:

- `frontend/src/components/ui/` for reusable UI primitives.
- `frontend/src/components/layout/` for shared layout components.
- `frontend/src/styles/` for global styles and theme tokens.

Adjust these paths only if the established project structure requires it.

Before creating a new component, check whether an existing component can be reused or extended.

Shared components must remain generic enough to support multiple modules without introducing module-specific dependencies unnecessarily.

## 18. Rules for AI Coding Tools

When using an AI coding assistant to create or modify Growkyat UI, provide these instructions:

- Read `docs/DESIGN_SYSTEM.md` before implementing UI.
- Inspect the existing theme and shared components before creating new ones.
- Reuse existing components and design tokens.
- Do not introduce new brand colours, fonts, radii, shadows, or spacing scales independently.
- Do not replace the established design with a generic template.
- Do not modify the logo without explicit approval.
- Do not duplicate existing components unnecessarily.
- Do not change unrelated pages or shared styles when implementing a feature.
- Preserve responsive behaviour and accessibility.
- If a required design token or component is missing, extend the shared system consistently.
- If a major visual decision is unclear, ask for clarification rather than inventing a different design direction.

## 19. Changes to the Design System

The shared design system is the single source of truth for UI consistency.

Developers must not independently change global tokens or shared component behaviour in ways that create visual inconsistencies.

For proposed changes:

1. Identify the reason for the change.
2. Check which pages and components are affected.
3. Update the central tokens or shared component.
4. Check affected pages for regressions.
5. Communicate the change to other developers.
6. Update this document when the standard itself changes.

Future brand-colour changes should primarily require updates to the central theme, not individual page rewrites.

## 20. Definition of Done for UI Work

Before considering a UI task complete, verify:

- [ ] Existing design tokens and components were reused.
- [ ] No unnecessary duplicate components were introduced.
- [ ] Typography and spacing are consistent.
- [ ] Buttons and inputs use shared styles.
- [ ] Desktop and mobile layouts work.
- [ ] Loading, empty, error, and disabled states are handled where relevant.
- [ ] Keyboard interaction and focus states work.
- [ ] No unrelated global styling was changed.
- [ ] The implementation matches the intended Growkyat design language.
- [ ] Changes are ready for code review.

---

**Final rule:** Growkyat must look and behave like one unified product, regardless of which developer or AI coding tool implements a page.
