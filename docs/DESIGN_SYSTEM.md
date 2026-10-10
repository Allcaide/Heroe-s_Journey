# Design System

How should the app look?

Related: [PRODUCT.md](PRODUCT.md) (what each screen does).

## UI Philosophy

The application should feel:

- premium;
- polished;
- modern;
- clean;
- friendly;
- fast;
- mobile-first;
- information-dense where appropriate;
- easy to use during a workout;
- visually clear when displaying nutrition data;
- visually close to provided design references.

The app should prioritize clarity over decorative UI.

The Nutrition screen should follow familiar food-tracking patterns.

The Training screen should follow familiar workout-tracking patterns.

Do not unnecessarily redesign established interaction patterns merely to make the application look different.

The unique visual language should primarily come from:

- the John AI experience;
- verification states;
- recovery information;
- the overall design system.

## UI Quality Bar

Use:

- rounded cards;
- clear spacing;
- progress indicators;
- large touch targets;
- simple animations when useful.

During workouts, controls must remain easy to hit and understand quickly.

During food tracking, quantities and verification state must be immediately understandable.

## Implementing from Design References

For any UI-related task with a reference:

- the goal is to reproduce the provided design accurately;
- match the layout, spacing, typography, colours, borders, radius, shadows, alignment and proportions;
- replicate all important visible UI elements;
- do not simplify or redesign a provided reference unless explicitly asked.

When the user provides a screenshot or design reference, treat it as the visual source of truth for that screen and replicate it accurately.

Do not approximate a reference unnecessarily.

At the same time, if no reference exists, prioritize:

- clarity;
- consistency;
- platform conventions;
- accessibility;
- fast interaction;
- the established design system.

## Domain-Specific UI

- Nutrition should feel familiar to users of MyFitnessPal-like trackers.
- Training should feel familiar to users of Strong-like trackers.
- John should have a distinct but integrated identity.
- Barcode verification and estimated-food states must be visually obvious without becoming visually noisy.
- Recovery information must be easy to understand at a glance.

Food verification states (each food row offers quick access to barcode verification and quantity/weight adjustment):

| State | Visual |
|---|---|
| Verified | Subtle green circular indicator containing the barcode icon |
| John-estimated | Barcode icon with a subtle visual indicator representing John |

## Styling

Use NativeWind/Tailwind CSS classes for styling wherever practical. (NativeWind is not installed yet — see [CLAUDE.md — Current Project State](../CLAUDE.md#current-project-state).)

Before implementing NativeWind-related code:

1. Check the exact version installed in `package.json`.
2. Follow the syntax and setup supported by that exact version.
3. Do not copy APIs or configuration from another NativeWind version.
4. Do not upgrade NativeWind without explicit approval.

Avoid large inline styles unless required.

Use `StyleSheet` or inline styles only when React Native-specific behaviour, dynamic styling, or a platform-specific API genuinely makes NativeWind unsuitable.

Prefer consistent reusable styling patterns.

If the project already has shared utilities in `global.css`, reuse them.

If a styling pattern is genuinely repeated across the application and would benefit from a shared utility, suggest or create the appropriate shared utility rather than duplicating it everywhere.

Do not create abstractions for a single isolated style without a real benefit.

## Responsive UI

The application must work across the iPhone sizes the project supports.

Do not hard-code dimensions unnecessarily.

Prefer:

- flexbox;
- relative sizing;
- safe-area-aware layouts;
- platform conventions;
- responsive spacing.

Touch targets should be sufficiently large for use during workouts.

## Accessibility

Prefer accessible React Native components and semantics where appropriate.

Ensure that:

- important icons have accessible labels;
- buttons have meaningful labels;
- colour is not the only way to communicate state;
- text remains readable;
- workout controls are usable while moving or under poor lighting.

For verification state, the green icon should not be the only indication of verification; the underlying semantic state should also be accessible.

## Images

### Centralized image imports

Before using any image asset:

1. Check whether `constants/images.ts` exists.
2. If it does not exist, create it.
3. Import and export all app images from `constants/images.ts`.
4. Use images through the centralized `images` object from components and screens.

Example:

```ts
import mascot from "@/assets/images/mascot.png";
import mascotLogo from "@/assets/images/mascot-logo.png";

export const images = {
  mascot,
  mascotLogo,
};
```

Use images like this:

```tsx
<Image source={images.mascot} />
```

Do not require/import image assets directly inside screens or components unless there is a strong reason, and do not repeatedly import the same image directly in unrelated screens.

### Image generation

If image generation is explicitly requested or enabled:

- generate assets consistent with the application's established design system;
- when a UI reference is provided, match its style, composition, colours and visual language closely;
- do not introduce an unrelated visual style;
- keep generated assets organised under `assets/`.

Use clear naming such as:

```text
assets/images/
  john-avatar.png
  onboarding-illustration.png
  recovery-icon.png
```

Generated assets should then be integrated through the application's centralized image system.
