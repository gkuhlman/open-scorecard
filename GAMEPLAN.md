# Scorecard Engine — Architecture Gameplan

## Overview

A config-driven, open-source baseball scorecard generator built with Next.js and React-PDF. Users define their ideal scorecard in a JSON config file, and the engine renders it as a pixel-perfect PDF. Configs are shareable, composable, and version-controlled.

---

## Stack

| Layer | Technology | Why |
|---|---|---|
| Framework | Next.js (App Router) | API routes, env vars, SSR, easy Vercel deploy |
| PDF Engine | `@react-pdf/renderer` | React components → PDF. Flexbox layout via Yoga. No browser print quirks |
| Preview | `@react-pdf/renderer` `PDFViewer` | Live in-browser preview while editing configs |
| Schema Validation | Zod | Validate configs at load time with clear error messages |
| Email | Resend (via API route) | Send rendered PDFs as attachments |
| Config Format | JSON | Zero-dependency parsing, great tooling, easy to generate |

---

## Project Structure

```
scorecard-engine/
├── app/
│   ├── page.tsx                    # Landing / config selector / live preview
│   ├── api/
│   │   ├── render/route.ts         # POST config → returns PDF buffer
│   │   └── send/route.ts           # POST config + email → Resend delivery
│   └── preview/
│       └── page.tsx                # PDFViewer with hot-reload on config change
│
├── components/
│   └── pdf/                        # All React-PDF components
│       ├── Scorecard.tsx           # Root document wrapper
│       ├── GameHeader.tsx          # Date, teams, venue, weather, umpires
│       ├── HalfInning.tsx          # Section container (top/bottom)
│       ├── BattingGrid.tsx         # The main table: players × innings
│       ├── AtBatCell.tsx           # Diamond + outcomes + count
│       ├── Diamond.tsx             # SVG-like diamond (React-PDF Svg primitives)
│       ├── CountTracker.tsx        # Ball/strike boxes
│       ├── OutcomeLabels.tsx       # 1B 2B 3B HR BB row
│       ├── StatColumns.tsx         # Configurable stat cols (AB, R, H, RBI, etc.)
│       ├── PitcherLog.tsx          # Pitcher tracking table
│       ├── Scoreboard.tsx          # Inning-by-inning linescore
│       └── GameNotes.tsx           # Lined notes area
│
├── lib/
│   ├── schema.ts                   # Zod schema for config validation
│   ├── config-loader.ts            # Load + validate + merge with defaults
│   ├── theme.ts                    # Resolve config theme → React-PDF StyleSheet
│   └── defaults.ts                 # Default config values (fallback for every field)
│
├── presets/
│   ├── classic-blue.json           # Traditional scorecard (what we built)
│   ├── minimal.json                # Stripped down — diamond + stats only
│   ├── analytics.json              # Extra stat cols: OBP, SLG, pitch count
│   └── retro-cream.json            # Warm tones, serif fonts, vintage feel
│
├── docs/
│   ├── CONFIG.md                   # Full schema reference
│   └── CONTRIBUTING.md             # How to add presets, components, fields
│
├── .env.local                      # RESEND_API_KEY
└── package.json
```

---

## Component Tree

```
<Document>
  <Page size="LETTER" orientation="landscape">
    <Scorecard config={config}>

      <GameHeader />
        → date, teams, venue, weather, umpires
        → controlled by config.header.fields[]

      <HalfInning side="away" label={config.sections.away.label}>
        <BattingGrid
          players={config.grid.rows}
          innings={config.grid.innings}
          statColumns={config.grid.statColumns}
        >
          <AtBatCell>
            <OutcomeLabels items={config.cell.outcomes} />
            <Diamond />
            <CountTracker
              balls={config.cell.count.balls}
              strikes={config.cell.count.strikes}
            />
          </AtBatCell>
        </BattingGrid>
        <SectionFooter>
          <PitcherLog rows={config.pitchers.rows} stats={config.pitchers.stats} />
          <GameNotes lines={config.notes.lines} />
        </SectionFooter>
      </HalfInning>

      <HalfInning side="home" label={config.sections.home.label}>
        <BattingGrid ... />
        <SectionFooter>
          <PitcherLog ... />
          <Scoreboard innings={config.grid.innings} />
        </SectionFooter>
      </HalfInning>

    </Scorecard>
  </Page>
</Document>
```

---

## Config Schema

Every field is optional — anything omitted falls back to `defaults.ts`. This means a minimal config can be just `{ "name": "My Card" }` and you get a fully functional scorecard.

### Top-Level Fields

| Field | Type | Description |
|---|---|---|
| `name` | string | Config display name |
| `version` | string | Semver for the config format |
| `description` | string | What makes this scorecard unique |
| `author` | string | GitHub username or name |
| `theme` | Theme | Colors, fonts, sizing |
| `page` | Page | Paper size, orientation, margins |
| `header` | Header | Game metadata fields |
| `grid` | Grid | Batting grid structure |
| `cell` | Cell | What goes inside each at-bat cell |
| `pitchers` | Pitchers | Pitcher log configuration |
| `scoreboard` | Scoreboard | Linescore toggle and layout |
| `notes` | Notes | Game notes area |
| `sections` | Sections | Labels and visibility for away/home halves |

---

## Example Config: `classic-blue.json`

```json
{
  "name": "Classic Blue",
  "version": "1.0.0",
  "description": "Traditional scorecard with outcome labels, ball/strike tracking, and full stat columns.",
  "author": "gabescorecard",

  "theme": {
    "colors": {
      "primary": "#3a9bd5",
      "primaryLight": "#b8ddf0",
      "primaryFaint": "#eaf5fb",
      "ink": "#2c3e50",
      "background": "#fdfdfd",
      "pageBackground": "#e8e0d6",
      "border": "#a5d4e8",
      "borderLight": "#cde6f2",
      "diamondFill": "#e2f1f8",
      "diamondStroke": "#a5d4e8"
    },
    "fonts": {
      "display": "Barlow Condensed",
      "body": "Barlow"
    },
    "sizing": {
      "inningCellWidth": 64,
      "rowHeight": 66,
      "playerColWidth": 110,
      "posColWidth": 28,
      "statColWidth": 24
    }
  },

  "page": {
    "size": "LETTER",
    "orientation": "landscape",
    "margins": {
      "top": 24,
      "right": 24,
      "bottom": 24,
      "left": 24
    }
  },

  "header": {
    "show": true,
    "fields": [
      { "key": "date", "label": "Date", "width": "20%" },
      { "key": "awayTeam", "label": "Away", "width": "20%" },
      { "key": "homeTeam", "label": "Home", "width": "20%" },
      { "key": "venue", "label": "Venue", "width": "25%" },
      { "key": "weather", "label": "Weather", "width": "15%" }
    ]
  },

  "grid": {
    "rows": 9,
    "innings": 10,
    "statColumns": [
      { "key": "AB", "label": "AB" },
      { "key": "R", "label": "R" },
      { "key": "H", "label": "H" },
      { "key": "RBI", "label": "RBI" }
    ]
  },

  "cell": {
    "outcomes": {
      "show": true,
      "position": "top",
      "items": ["1B", "2B", "3B", "HR", "BB"]
    },
    "diamond": {
      "show": true,
      "style": "filled",
      "maxSize": 42
    },
    "count": {
      "show": true,
      "position": "bottom-right",
      "balls": 3,
      "strikes": 2,
      "layout": "vertical"
    }
  },

  "pitchers": {
    "rows": 5,
    "stats": [
      { "key": "IP", "label": "IP" },
      { "key": "H", "label": "H" },
      { "key": "R", "label": "R" },
      { "key": "ER", "label": "ER" },
      { "key": "BB", "label": "BB" },
      { "key": "K", "label": "K" }
    ]
  },

  "scoreboard": {
    "show": true,
    "totals": ["R", "H", "E"]
  },

  "notes": {
    "show": true,
    "lines": 5
  },

  "sections": {
    "away": {
      "label": "Top / Away Team",
      "footer": ["pitchers", "notes"]
    },
    "home": {
      "label": "Bottom / Home Team",
      "footer": ["pitchers", "scoreboard"]
    }
  }
}
```

---

## Variations to Demonstrate Flexibility

### Minimal — just the diamond

```json
{
  "name": "Minimal",
  "cell": {
    "outcomes": { "show": false },
    "count": { "show": false }
  },
  "grid": {
    "statColumns": [
      { "key": "R", "label": "R" },
      { "key": "H", "label": "H" }
    ]
  },
  "notes": { "show": false }
}
```

### Analytics — extra stat columns, pitch count in cell

```json
{
  "name": "Analytics",
  "cell": {
    "outcomes": {
      "items": ["1B", "2B", "3B", "HR", "BB", "K", "HBP"]
    },
    "count": {
      "show": true,
      "balls": 4,
      "strikes": 3
    }
  },
  "grid": {
    "statColumns": [
      { "key": "AB", "label": "AB" },
      { "key": "R", "label": "R" },
      { "key": "H", "label": "H" },
      { "key": "RBI", "label": "RBI" },
      { "key": "BB", "label": "BB" },
      { "key": "K", "label": "K" },
      { "key": "OBP", "label": "OBP" }
    ]
  },
  "pitchers": {
    "stats": [
      { "key": "IP", "label": "IP" },
      { "key": "H", "label": "H" },
      { "key": "R", "label": "R" },
      { "key": "ER", "label": "ER" },
      { "key": "BB", "label": "BB" },
      { "key": "K", "label": "K" },
      { "key": "PC", "label": "PC" }
    ]
  }
}
```

---

## Implementation Order

### Phase 1 — Core renderer (start here)
1. `create-next-app` with TypeScript, App Router
2. Install `@react-pdf/renderer` and Zod
3. Build `defaults.ts` and `schema.ts` — define the full Zod schema from the types above
4. Build `Diamond.tsx` and `AtBatCell.tsx` first — the hardest components
5. Build `BattingGrid.tsx`, `PitcherLog.tsx`, `Scoreboard.tsx`
6. Wire up `Scorecard.tsx` as the root document
7. Create `/preview` page with `PDFViewer` loading `classic-blue.json`
8. Iterate until the PDF output matches the HTML prototype we built

### Phase 2 — Config system
1. Config loader: read JSON, validate with Zod, deep-merge with defaults
2. Preset gallery: list presets, preview thumbnails, download PDF
3. Add 3-4 preset configs to demonstrate range

### Phase 3 — Sharing & delivery
1. Resend integration: API route that renders PDF + emails it
2. GitHub-friendly: presets live in `/presets`, PRs to add new ones
3. README with badges, screenshots, "Create Your Own" instructions

### Phase 4 — Nice to haves
1. YAML support via `js-yaml` loader
2. Web-based config editor (form UI that outputs JSON)
3. Print-at-home instructions (paper size, scaling tips)
4. Notion / Google Sheets import for lineup data pre-fill

---

## Key Decisions

**Why React-PDF over HTML + print CSS?**
HTML print stylesheets are fragile across browsers (as we discovered with Brave). React-PDF uses its own layout engine — same output everywhere. The tradeoff is learning its style API (no CSS, just flexbox objects), but for a structured layout like a scorecard it maps well.

**Why JSON over YAML?**
JSON is the pragmatic default: zero dependencies, native browser parsing, better tooling (JSON Schema, IDE autocomplete). YAML is a nice Phase 4 add for people who prefer editing configs by hand.

**Why Zod?**
Configs will come from users and contributors. Zod gives you runtime validation with readable error messages ("Expected number for rowHeight, got string"). It also generates TypeScript types from the schema, so your components stay in sync with the config format.

**Why not a visual editor from day one?**
The config JSON IS the product for v1. Power users (your target audience for an open-source scorecard tool) will prefer editing JSON directly. A visual editor is a great Phase 4 feature once the schema is stable.
