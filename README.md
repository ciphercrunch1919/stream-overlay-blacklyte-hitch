# BLACKLYTE Stream Overlay for hitchcat

Browser source overlay for OBS / Streamlabs. Streamers add it once with their code in the URL. This code is based off of the ADVANCED.GG stream-overlay github code: https://github.com/59adf2f/stream-overlay. All credit to them. The ADVANCED team controls campaigns, products, timing, and can completely redesign the overlay at any time without streamers changing anything. This code will not include those changes as its altered to fit a blacklyte stream overlay for hitchcat's stream. 

## Repo Structure

```
stream-overlay/
  overlay.html      ← browser source (the entire look/feel lives here)
  config.json       ← campaigns, timing, products, defaults
  README.md
```

---

## For Streamers

### One-Time Setup
1. Add a **Browser Source** in OBS or Streamlabs
2. URL: `https://advancedgg.github.io/stream-overlay/overlay.html?code=YOURCODE`
3. Dimensions: **520 x 240**
4. Check **"Refresh browser when scene becomes active"**
5. Done. Never touch it again.

### URL Parameters (yours to customize)

| Param | Default | What it does |
|-------|---------|-------------|
| `code` | `HITCH` | Your discount code |
| `percent` | `10` | Override discount % |
| `theme` | cyan | `green`, `purple`, `gold` |
| `position` | `bl` | `bl`, `br`, `tl`, `tr` |
| `always` | `0` | `1` = stays on screen permanently |
| `test` | `0` | `1` = rapid cycle for testing |

### Examples
```
?code=POKEAIM
?code=POKEAIM&theme=purple&position=br
?code=NINJA&theme=gold&always=1
?code=TEST&test=1
```

---

## For the ADVANCED Team

### Architecture

```
Streamer's OBS
    │
    │  loads URL with ?code=POKEAIM&theme=purple
    │
    ▼
overlay.html  (GitHub Pages)
    │
    │  fetches every 60s
    │
    ▼
config.json   (GitHub Pages)
    │
    │  reads campaign, products, timing
    │
    ▼
Shopify CDN   (jar + shaker images)
```

**Three things you can update independently:**

1. **config.json** — change campaigns, swap product images, adjust timing. Every overlay picks it up within 60 seconds.
2. **overlay.html** — completely redesign the look, layout, animations, everything. Streamers get the new design on their next scene activation or OBS refresh.
3. **Both at once** — push a full rebrand in one commit.

### The URL Param Contract

Any version of `overlay.html` (current or future redesign) **MUST** read these URL params:

```
?code=STRING       → streamer's discount code (injected into display)
&percent=NUMBER    → override discount % (optional)
&theme=STRING      → color theme: green | purple | gold (optional)
&position=STRING   → screen position: bl | br | tl | tr (optional)
&always=1          → keep on screen permanently (optional)
&test=1            → rapid cycle for testing (optional)
```

This is the contract with streamers. Their URLs never change. You change everything server-side.

### The Config Contract

Any version of `overlay.html` **MUST** read these `config.json` fields:

```
campaign.active              → bool: show campaign
campaign.label               → string: tag ("NEW DROP", "LIMITED TIME")
campaign.title               → string: headline
campaign.subtitle            → string: secondary text
campaign.override_discount   → bool: replace default discount display
campaign.override_message    → string: replacement text ({code} and {percent} are placeholders)
campaign.override_code       → string: force a code for ALL streamers ("" = use their personal code)
campaign.override_percent    → string: force a % ("" = use default)
campaign.start / campaign.end → ISO timestamps for scheduling (null = no bounds)
defaults.percent             → default discount %
defaults.site_tag            → bottom tag text
shakers[]                    → array of shaker image URLs
jars[]                       → array of jar image URLs
timing.interval_minutes      → minutes between popups (1-30)
timing.duration_seconds      → seconds visible (5-60)
```

### Pushing a Campaign

Edit `config.json`:

```json
{
  "campaign": {
    "active": true,
    "label": "NEW DROP",
    "title": "Strawberry Lychee Energy",
    "subtitle": "Available now at advanced.gg",
    "override_discount": true,
    "override_message": "NEW Strawberry Lychee Energy — Use code {code} at checkout"
  }
}
```

When `override_discount` is `true`, the default "Use code X for 10% off" is completely replaced by `override_message`. The `{code}` placeholder auto-fills with whatever the streamer set in their URL.

### Killing a Campaign

```json
{ "campaign": { "active": false } }
```

### Forcing a Sitewide Code

```json
{
  "campaign": {
    "active": true,
    "override_code": "EASTERBOGO",
    "override_discount": true,
    "override_message": "BOGO is LIVE — Use code {code} at advanced.gg"
  }
}
```

This overrides every streamer's personal code with `EASTERBOGO`.

### Swapping Product Images

Just edit the `chairs` and `desks` arrays. Use Shopify CDN URLs:
* need to update so I don't have to use shopify cdn urls

CURRENT: 
```json
{
  "chairs": [
    "https://cdn.shopify.com/s/files/.../new-shaker.png"
  ],
  "desks": [
    "https://cdn.shopify.com/s/files/.../new-jar.png"
  ]
}
```

Each slide-in picks one random jar + one random shaker, never repeating back-to-back.

### Full Redesign

Replace `overlay.html` with an entirely new file. New CSS, new layout, new animations, whatever you want. Just make sure the new version:

1. Reads the URL params listed above
2. Fetches and reads `config.json` with the fields listed above
3. Polls `config.json` periodically (currently 60s)

Streamer URLs stay the same. They get the new design automatically.

### Campaign Presets (copy/paste)

**Strawberry Lychee Launch (April 17)**
```json
"campaign": {
  "active": true, "label": "NEW DROP",
  "title": "Strawberry Lychee Energy",
  "subtitle": "Available now at advanced.gg",
  "override_discount": true,
  "override_message": "NEW Strawberry Lychee Energy — Use code {code} at checkout",
  "start": "2026-04-17T00:00:00Z", "end": "2026-04-20T23:59:59Z"
}
```

**Easter BOGO**
```json
"campaign": {
  "active": true, "label": "LIMITED TIME",
  "title": "Easter BOGO",
  "subtitle": "Buy one get one free on all energy sticks",
  "override_discount": true, "override_code": "EASTERBOGO",
  "override_message": "BOGO is LIVE — Use code {code} at advanced.gg"
}
```

**Mexican Hot Chocolate (April 29)**
```json
"campaign": {
  "active": true, "label": "NEW FLAVOR",
  "title": "Mexican Hot Chocolate",
  "subtitle": "Limited edition — while supplies last",
  "override_discount": true,
  "override_message": "Mexican Hot Chocolate is HERE — Code {code} for {percent}% off",
  "start": "2026-04-29T00:00:00Z"
}
```

---

## GitHub Pages Setup

1. Create repo `stream-overlay` under `advancedgg` org
2. Push files to `main`
3. Settings → Pages → Deploy from `main`, root folder
4. Live at: `https://advancedgg.github.io/stream-overlay/overlay.html`
5. Optional: CNAME → `overlay.advanced.gg`
