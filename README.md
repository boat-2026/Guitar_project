# GearTracker

A guitar gear management app with signal chain building and AI-powered tone advice.

## Features

### 1. Gear Log
Track all your guitar gear in one place across three categories:

**Guitars**
- Name, brand, model
- Pickup configuration (SSS, HSS, HH, P90, single, humbucker, etc.)
- Default tuning
- Notes (freeform — setup specs, feel, preferences)
- Owned vs. Wishlist flag

**Amps**
- Name, brand, model
- Wattage
- Type (tube, solid-state, modeling, hybrid)
- Channel count
- Built-in effects (reverb, tremolo, etc.)
- Notes
- Owned vs. Wishlist flag

**Pedals**
- Name, brand, model
- Effect type (drive/overdrive, distortion, fuzz, delay, reverb, modulation, compressor, tuner, EQ, wah, looper, other)
- Bypass type (true bypass, buffered)
- Power draw in mA
- Mono/stereo
- Notes
- Owned vs. Wishlist flag

Each gear item has a unique ID, created date, and updated date.

### 2. Signal Chain Builder
Visual drag-and-drop interface to build and save signal chains:

- Select a **guitar** as the input (left side)
- Select an **amp** as the output (right side)
- Drag **pedals** from your owned gear into the chain between guitar and amp
- Reorder pedals via drag-and-drop
- Visual connection lines or arrows showing signal flow: Guitar → Pedal 1 → Pedal 2 → ... → Amp
- Save chains with a name (e.g., "Blues Jam," "Sunday Clean," "Full Board")
- Load, edit, rename, and delete saved chains
- Only owned gear (not wishlist) is available for chain building

### 3. AI Tone Advisor
Ask Claude how to achieve a specific tone using your gear:

- Text input where user types a question like:
  - "How do I get the John Mayer Gravity live tone?"
  - "What's the best chain for SRV Texas Flood?"
  - "How do I get a good ambient worship tone?"
- The app sends the user's full owned gear list as context along with the question to the Claude API
- Claude responds with:
  - A recommended signal chain using the user's actual gear
  - Suggested settings (pickup position, pedal knob positions, amp settings)
  - Gaps — gear the user doesn't own but would help (ties back to wishlist)
- Conversation is displayed in a chat-style interface
- Each tone request is independent (no multi-turn memory needed)

## Tech Stack

- **Frontend:** React (Vite + TypeScript)
- **Styling:** Tailwind CSS
- **Drag-and-drop:** @dnd-kit/core + @dnd-kit/sortable
- **State/Storage:** localStorage for all gear data, chains, and app state (no backend/database)
- **AI Integration:** Anthropic Claude API (claude-sonnet-4-20250514) via direct fetch from the client
- **Routing:** React Router (3 main views: Gear Log, Chain Builder, Tone Advisor)

## Data Models

```typescript
type GearCategory = 'guitar' | 'amp' | 'pedal';
type OwnershipStatus = 'owned' | 'wishlist';

interface GearItemBase {
  id: string;
  category: GearCategory;
  name: string;
  brand: string;
  model: string;
  status: OwnershipStatus;
  notes: string;
  createdAt: string;
  updatedAt: string;
}

interface Guitar extends GearItemBase {
  category: 'guitar';
  pickupConfig: string;     // SSS, HSS, HH, P90, etc.
  tuning: string;           // Standard, Drop D, Open G, etc.
}

interface Amp extends GearItemBase {
  category: 'amp';
  wattage: number;
  ampType: string;          // tube, solid-state, modeling, hybrid
  channels: number;
  builtInEffects: string[];
}

interface Pedal extends GearItemBase {
  category: 'pedal';
  effectType: string;       // drive, delay, reverb, modulation, etc.
  bypassType: string;       // true bypass, buffered
  powerDraw: number;        // mA
  stereo: boolean;
}

type GearItem = Guitar | Amp | Pedal;

interface SignalChain {
  id: string;
  name: string;
  guitarId: string;
  ampId: string;
  pedalIds: string[];       // ordered array
  createdAt: string;
  updatedAt: string;
}
```

## Pages / Views

1. **Gear Log** (`/`) — Default view. Table/card list of all gear with filters by category and ownership status. Add, edit, delete gear items. 
2. **Chain Builder** (`/chains`) — Visual chain builder. Sidebar with owned gear, main area for the active chain. List of saved chains.
3. **Tone Advisor** (`/tone`) — Chat-style interface. Input box at bottom, responses above. Gear context sent automatically.

## UI Notes

- Clean, dark-themed UI (gear/music app vibe)
- Responsive but desktop-first (this is a desk/workshop tool)
- Gear items shown as cards with category-specific icons
- Signal chain shows left-to-right flow with clear connection indicators
- Tone advisor uses a simple chat bubble layout

## Getting Started

```bash
npm install
npm run dev
```

Requires a Claude API key set as `VITE_ANTHROPIC_API_KEY` in a `.env` file for the Tone Advisor feature.
