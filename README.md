# Scorecard

A config-driven baseball scorecard generator. Fork this repo, edit one JSON file, and get a printable scorecard deployed to GitHub Pages.

**Zero dependencies. Single HTML output. Fully customizable.**

## Get Started in 3 Steps

### 1. Fork this repo
Click the **Fork** button at the top right of this page.

### 2. Edit `scorecard.json`
This is the only file you need to touch. Customize your scorecard by changing colors, grid size, stat columns, and more. Any field you omit falls back to a sensible default — so even `{}` produces a working scorecard.

### 3. Enable GitHub Pages
Go to **Settings → Pages → Source** and select **GitHub Actions**. Push a change to `scorecard.json` and your scorecard will be live at `https://<username>.github.io/scorecard/`.

## How It Works

When you push changes to `scorecard.json`, a GitHub Action runs `node scripts/generate.js` which:

1. Reads your `scorecard.json`
2. Deep-merges it with `defaults.json` (your config wins)
3. Generates a self-contained `dist/index.html`
4. Deploys it to GitHub Pages

## Config Reference

Edit `scorecard.json` to customize your scorecard. Every field is optional — only specify what you want to change.

### Top-Level Fields

| Field | Type | Description |
|---|---|---|
| `name` | string | Display name for the scorecard |
| `description` | string | What makes this scorecard unique |
| `author` | string | Your name or GitHub username |

### Theme

```json
{
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
  }
}
```

### Grid

```json
{
  "grid": {
    "rows": 9,
    "innings": 10,
    "statColumns": [
      { "key": "AB", "label": "AB" },
      { "key": "R", "label": "R" },
      { "key": "H", "label": "H" },
      { "key": "RBI", "label": "RBI" }
    ]
  }
}
```

### Cell (At-Bat)

```json
{
  "cell": {
    "outcomes": {
      "show": true,
      "items": ["1B", "2B", "3B", "HR", "BB"]
    },
    "diamond": {
      "show": true,
      "maxSize": 42
    },
    "count": {
      "show": true,
      "balls": 3,
      "strikes": 2
    }
  }
}
```

### Pitchers

```json
{
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
  }
}
```

### Scoreboard & Notes

```json
{
  "scoreboard": {
    "show": true,
    "totals": ["R", "H", "E"]
  },
  "notes": {
    "show": true,
    "lines": 5
  }
}
```

## Example Configs

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

### Analytics — extra stats and pitch tracking

```json
{
  "name": "Analytics",
  "cell": {
    "outcomes": {
      "items": ["1B", "2B", "3B", "HR", "BB", "K", "HBP"]
    },
    "count": {
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
  }
}
```

## Local Development

```bash
npm run build        # Generates dist/index.html
open dist/index.html # Preview in browser
```

No dependencies to install — just Node.js.

## Printing

Open `dist/index.html` in your browser, hit **Cmd+P** (or **Ctrl+P**), and print. The scorecard includes print-optimized styles that remove shadows and adjust for paper.

For best results:
- Set orientation to **Landscape**
- Set margins to **Minimum** or **None**
- Enable **Background graphics** if you want the shaded stat columns

## License

MIT
