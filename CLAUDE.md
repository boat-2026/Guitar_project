# CLAUDE.md

## Project Overview
GearTracker is a React + TypeScript guitar gear management app. It has three features: a gear log, a drag-and-drop signal chain builder, and an AI tone advisor powered by the Claude API. All data persists in localStorage — there is no backend.

## Tech Stack
- React 18+ with Vite and TypeScript
- Tailwind CSS for styling
- @dnd-kit/core and @dnd-kit/sortable for drag-and-drop in the chain builder
- React Router v6 for page routing
- Anthropic Claude API (claude-sonnet-4-20250514) for the tone advisor — called via fetch, no SDK needed
- localStorage for all persistence (gear items, saved chains)
- uuid for generating gear/chain IDs

## Commands
- `npm install` — install dependencies
- `npm run dev` — start dev server
- `npm run build` — production build
- `npm run lint` — run ESLint

## Project Structure
```
src/
  components/
    layout/         — Nav bar, page layout wrapper
    gear/           — GearCard, GearForm, GearList (used on Gear Log page)
    chain/          — ChainCanvas, PedalSlot, GearSidebar (used on Chain Builder page)
    tone/           — ChatMessage, ToneInput (used on Tone Advisor page)
    ui/             — Shared UI primitives (Button, Modal, Input, Select, Badge)
  pages/
    GearLog.tsx     — Main gear inventory view
    ChainBuilder.tsx — Signal chain drag-and-drop builder
    ToneAdvisor.tsx — AI tone chat interface
  hooks/
    useGear.ts      — CRUD operations for gear items in localStorage
    useChains.ts    — CRUD operations for signal chains in localStorage
  types/
    index.ts        — All TypeScript interfaces (GearItem, Guitar, Amp, Pedal, SignalChain)
  utils/
    storage.ts      — localStorage read/write helpers
    toneApi.ts      — Claude API call logic for tone advisor
  App.tsx
  main.tsx
```

## Key Implementation Details

### Data Storage
- All gear and chain data is stored in localStorage under keys: `geartracker_gear` and `geartracker_chains`
- Use JSON.parse/JSON.stringify for serialization
- The useGear and useChains hooks should manage React state synced with localStorage

### Gear Log Page
- Default view at `/`
- Displays all gear as cards in a grid
- Filter tabs: All | Guitars | Amps | Pedals
- Toggle: Owned | Wishlist | All
- Each card shows key info for its category (e.g., pickup config for guitars, effect type for pedals)
- Add button opens a modal form — form fields change based on selected category
- Edit and delete actions on each card

### Chain Builder Page
- Route: `/chains`
- Left sidebar: list of owned gear grouped by category (guitars, pedals, amps)
- Main area: the active signal chain displayed as a horizontal flow
- Guitar slot (leftmost) → Pedal slots (middle, sortable via drag-and-drop) → Amp slot (rightmost)
- Drag pedals from sidebar into the chain. Drag to reorder within the chain. Drag out to remove.
- Guitar and amp are selected via dropdown, not drag-and-drop
- Save button with name input. Saved chains listed in a panel (load, rename, delete).

### Tone Advisor Page
- Route: `/tone`
- Chat-style layout: messages scroll top to bottom, input fixed at bottom
- On submit, build the API request:
  - System prompt includes the user's full owned gear inventory formatted as a structured list
  - User message is their tone question
  - Model: claude-sonnet-4-20250514
  - API key from VITE_ANTHROPIC_API_KEY env var
- Display Claude's response as a chat bubble
- Show a loading indicator while waiting for the response
- Each question is a standalone request (no conversation history)

### Tone Advisor System Prompt Template
```
You are a guitar tone expert. The user will ask how to achieve a specific tone, sound, or style. 

Here is the user's current gear inventory:

GUITARS:
{list each owned guitar with name, brand, model, pickup config, tuning}

AMPS:
{list each owned amp with name, brand, model, wattage, type, channels, built-in effects}

PEDALS:
{list each owned pedal with name, brand, model, effect type, bypass type}

Using ONLY the gear listed above, recommend:
1. Which guitar to use and pickup position
2. Signal chain order for the pedals (only include pedals that are relevant)
3. Suggested amp settings (channel, gain, EQ ballpark)
4. Suggested pedal settings where applicable
5. If there is a critical gap — gear the user does NOT own but would significantly help achieve this tone — mention it briefly at the end as a suggestion.

Be specific and practical. Reference the user's actual gear by name.
```

### Drag-and-Drop Notes
- Use @dnd-kit/core for the DnD context and @dnd-kit/sortable for the pedal chain reordering
- The sidebar gear list uses draggable items
- The pedal chain area is a sortable droppable zone
- Guitar and amp are NOT part of the drag system — use simple select dropdowns for those

### Styling
- Dark theme: dark gray/black backgrounds, lighter text
- Accent color: amber/gold (#F59E0B or similar — ties to guitar/music aesthetic)
- Cards with subtle borders and hover states
- The signal chain should visually feel like a pedalboard — horizontal layout with connection lines or arrows between slots
- Use Tailwind's dark utilities throughout

### Environment Variables
```
VITE_ANTHROPIC_API_KEY=sk-ant-...
```

## Conventions
- Functional components only, use hooks for state
- Keep components small and focused — extract into subcomponents when a file exceeds ~150 lines
- Name files in PascalCase for components, camelCase for hooks/utils
- Use TypeScript strictly — no `any` types
- Error states and loading states should always be handled in the UI
