# Find the Brand — Project Handoff

> **Status (May 2026):** Shipped. v1.0.0 live on the App Store; v1.0.1 in App Review with security hardening + UX polish. The rest of this doc is the original scaffolding-phase handoff — keep for design history but everything in "Next steps" below was implemented long ago. See **[CLAUDE.md](./CLAUDE.md)** for current-state orientation.

> **Original purpose**: handoff from the design/scaffolding phase (done in Claude.ai chat) to ongoing development in **Claude Code**.

---

## TL;DR for Claude Code

You're inheriting a **React Native + Expo SDK 52** mobile app called "Find the Brand" — a luxury collectibles scanner. The UI is fully scaffolded with mock data. The work ahead is wiring up real functionality: camera, geolocation, AI brand identification, persistent storage, and pricing APIs.

```bash
cd find-the-brand
rm -rf node_modules package-lock.json   # clean install required
npm install
npm run web        # quickest preview
npm run ios        # physical device / simulator via Expo Go
```

If you see `(0, _expoModulesCore.registerWebModule) is not a function` again, it's the SDK 51/52 mismatch — see "Known issues" below.

---

## 1. Product concept

A mobile app where a user points their camera at a luxury item (watch, handbag, sneaker), the app identifies the brand and product, and returns:
- Brand + product name
- Retail (new) price estimate
- Pre-owned market price estimate
- Links to reputable resale stores
- Authentication confidence score
- Geo-tagged scan history

**Target audience**: Collectors and connoisseurs (not thrift flippers — that market is saturated). Positioning is "private collector's vault" not "profit calculator".

**Three screens**, bottom tab navigation:
1. **Scan** — camera + viewfinder + result card
2. **History** (labelled "Archive" in nav) — featured carousel + infinite list + detail modal
3. **Dashboard** (labelled "Stats" in nav) — total stats + geo map + donut chart by brand

---

## 2. Competitive landscape (from web research done earlier)

This space is crowded. Knowing what exists helps you position features:

| App | Focus | Note |
|---|---|---|
| Rebag's Clair AI | Luxury handbags only, top 50 brands | Closest direct competitor for "luxury" framing |
| Legit Check: AI Scanner | Sneakers, designer goods | Same scan→identify→value flow |
| Thriftly: Profit Identifier | Resellers, has analytics dashboard | Almost identical screen list |
| ThriftAI / Revalue | Thrift flippers, clothing | Generic "what's it worth on eBay" UX |
| Underpriced AI | General resale | Runs on Claude Opus, 96% accuracy claim |
| Google Lens / eBay Visual Search | Free baselines | The free benchmark every product gets compared to |

**Differentiation strategy** the user agreed to:
- Not "what's this worth on eBay" (commodity)
- Yes "catalogue your collection / track provenance / monitor market for pieces you own"
- The collector-vault aesthetic IS the moat — every competitor uses generic flipper UI

---

## 3. Design system (current — lime on charcoal)

The user iterated through TWO design directions in the chat. **The current design (final)** is modeled after a screenshot they uploaded — a hiring app with electric lime green on near-black, sans-serif, modern fintech vibe. The earlier "luxury vault" design (gold + serif + ornate) was discarded.

### Color tokens (`theme/theme.js`)

| Token | Hex | Usage |
|---|---|---|
| `bg` | `#0E0E0E` | App background (warm-leaning charcoal, NOT pure black) |
| `bgElevated` | `#1C1C1E` | Cards, icon button containers |
| `bgPill` | `#2A2A2C` | Inline pills, tags |
| `bgInput` | `#1A1A1C` | Form inputs |
| `lime` | `#C5F94D` | **Primary accent** — buttons, highlights, hero card bg |
| `limeBright` | `#D4FF66` | Hover/highlight |
| `limeDeep` | `#A8DD2D` | Pressed/muted |
| `textPrimary` | `#FFFFFF` | Headings, body |
| `textSecondary` | `#8E8E93` | Subtitles, meta |
| `textTertiary` | `#5A5A5E` | Captions, hints |
| `textOnLime` | `#0A0A0A` | Text/icons on lime backgrounds |
| `border` | `rgba(255,255,255,0.08)` | Subtle dividers |
| `borderStrong` | `rgba(255,255,255,0.15)` | Outline pills/chips |
| `accentBlue/peach/lavender/mint/rose/teal` | Pastels | Avatar bgs, donut segments |

### Typography
- **Family**: `System` sans-serif (Inter-style on iOS/Android, system on web)
- **Weights**: regular `400`, medium `500`, semibold `600`, bold `700`, heavy `800`
- **Hierarchy method**: bold weight + lime color on focus word, NOT size alone
- **Headlines**: 36px, bold, letter-spacing -1, italic lime accent on one word
- **Body**: 14-16px regular
- **Meta/labels**: 11-13px semibold, occasional `letterSpacing: 0.3` for small caps feel

### Recurring motifs
- **Circular arrow button** — the signature element. Lime circle, rotated arrow icon (`↗`), used as primary CTA throughout. Component: `<ArrowButton>`
- **Title with accent word** — "Identify the *brand*", "Recent for *you*", "Your *collection*" — italicized lime word at the end
- **Top bar layout** — avatar left, notification+search/settings icons right
- **Category chips** — fully rounded pills (`borderRadius: 999`), filled lime when active, outlined dark when not
- **Featured carousel** — horizontal scroll, cards bleed off the right edge to signal "more"
- **Floating pill tab bar** — not edge-to-edge, sits inset with 24px margins, lime active state with text label
- **Bottom-sheet modal** — for detail views, with drag handle at top
- **No gradients on backgrounds** — solid colors only. Earlier vault design had gradients; new design removed them.

### Spacing scale
`xs:4, sm:8, md:12, lg:16, xl:24, xxl:32, xxxl:48`

### Radius scale
`sm:8, md:16, lg:24 (standard cards), xl:32, pill:999`

---

## 4. Project structure

```
find-the-brand/
├── index.js                    # Entry — registerRootComponent
├── App.js                      # Floating pill tab bar with 3 screens
├── app.json                    # Expo config (SDK 52, dark mode, newArchEnabled)
├── package.json                # SDK 52 pinned versions
├── babel.config.js             # babel-preset-expo + reanimated plugin
├── theme/
│   └── theme.js                # All design tokens (colors, fonts, spacing, radius, shadows)
├── components/
│   ├── UI.js                   # Shared primitives — see below
│   └── DonutChart.js           # SVG donut chart, used on Dashboard
├── screens/
│   ├── ScanScreen.js           # Camera placeholder + viewfinder + animated result card
│   ├── HistoryScreen.js        # Featured carousel + infinite list + bottom-sheet modal
│   └── DashboardScreen.js      # Lime hero card + stat grid + geo map + donut chart
└── data/
    └── mockData.js             # 8 sample scans, categories, brand breakdown aggregates
```

### `components/UI.js` exports
| Component | Purpose |
|---|---|
| `<ArrowButton size onPress variant="lime"\|"dark">` | The signature circular CTA |
| `<Chip label active onPress>` | Category pills |
| `<CountBadge count>` | `(26)`-style counter |
| `<IconButton icon onPress hasNotification size>` | Dark circle with outlined icon |
| `<SectionHeader title actionLabel onAction>` | Section header with "See all" link |
| `<Avatar initial color size>` | Circular pastel avatar with letter |

### Mock data shape (`data/mockData.js`)
Each scan:
```js
{
  id, brand, product, category,
  newPrice, secondHandPrice, currency, confidence,
  scannedAt,                          // ISO date
  location: { city, country, lat, lng },
  swatchColor,                        // pastel hex for avatar
  stores: [{ name, url }, ...]
}
```

8 sample items: Rolex Submariner, Hermès Birkin, Patek Nautilus, Louis Vuitton Keepall, Hermès Kelly, Audemars Piguet Royal Oak, Chanel Classic Flap, Rolex Daytona.

Also exports: `categories[]`, `dashboardStats {totalScans, totalValue, countries}`, `brandBreakdown[]` (with percentages and donut palette colors).

---

## 5. Tech stack & dependency notes

### Pinned versions (don't drift without checking)
```json
"expo": "~52.0.46",
"react": "18.3.1",
"react-native": "0.76.9",
"react-dom": "18.3.1",
"react-native-web": "~0.19.13",
"@react-navigation/native": "^7.0.0",
"@react-navigation/bottom-tabs": "^7.0.0",
"react-native-svg": "15.8.0",
"expo-linear-gradient": "~14.0.2",
"@expo/vector-icons": "^14.0.4"
```

### Why these versions
- **SDK 51 → 52 was a forced upgrade** mid-chat. SDK 51 shipped `expo-modules-core` ~1.12 which doesn't export `registerWebModule`. Newer transitive deps from `@expo/vector-icons` and React Navigation expected the function. Result: web build crashed at module load with `(0, _expoModulesCore.registerWebModule) is not a function`. SDK 52 has matching versions everywhere.
- **React Navigation v6 → v7** to align with SDK 52
- **Explicit `index.js`** (not implicit `node_modules/expo/AppEntry.js`) because SDK 52's web bundler is more reliable with explicit `registerRootComponent`
- **Removed `react-native-reanimated`** — it was in package.json but not used anywhere; smaller bundle, fewer web edge cases. The babel config still references its plugin (harmless but you can remove it if you confirm nothing pulls it in transitively)

### Verifying alignment
```bash
npx expo install --check    # warns if anything drifted
npx expo install --fix      # auto-fixes
```

---

## 6. Where things stand & what to build next

### What works
- ✅ Three screens with bottom-tab navigation
- ✅ Full design system applied across all screens
- ✅ Mock data flows through every screen
- ✅ Scan animation: pulsing capture button, scan-line sweep, fade-up result card
- ✅ History: horizontal featured carousel + infinite-scrolling list + bottom-sheet modal with full detail view
- ✅ Dashboard: lime hero card with total value, stat grid, SVG dot-grid map with gold/lime pin markers, donut chart with legend
- ✅ Runs on web, iOS, Android via Expo Go

### What's mocked / TODO

**Camera (highest priority)**
- Currently a static dark placeholder with an animated viewfinder
- Replace with `expo-camera` — `<CameraView>` from SDK 52
- Permission flow: `useCameraPermissions()` hook
- Capture: `cameraRef.current.takePictureAsync()`
- Keep the existing viewfinder/scan-line overlay; just put the live preview behind it
- Web fallback: gracefully degrade to file upload (`<input type="file">`) since `expo-camera` web support is limited

**Geolocation**
- Currently each scan has a hardcoded location
- Add `expo-location` — request permission, capture lat/lng on scan
- Reverse-geocode to city/country (Nominatim or `expo-location`'s built-in `reverseGeocodeAsync`)

**Brand identification (the AI core)**
- The Anthropic API is callable from React Native. Use Claude with vision:
  ```js
  // POST to https://api.anthropic.com/v1/messages
  // model: "claude-opus-4-7" or "claude-sonnet-4-6"
  // Send image as base64 with media_type "image/jpeg"
  // Prompt for brand, product, category, confidence
  ```
- Recommend Sonnet 4.6 for cost/speed unless accuracy demands Opus
- Structure the response as JSON: instruct Claude to return `{brand, product, category, confidence}` only
- Parse, then enrich with pricing (next item)
- **Never put the API key in the app bundle.** Stand up a thin proxy server (Node/Cloudflare Worker) that holds the key and the app calls that.

**Pricing data**
- No good free API for luxury resale prices. Options:
  1. Scrape Chrono24 / Vestiaire Collective / The RealReal — fragile, ToS-risky
  2. Partner with one (Rebag has a Clair API for handbags)
  3. Maintain your own price database, periodic crawl
  4. eBay Browse API (`/buy/browse/v1/item_summary/search`) for sold-listing comps — free with developer account, decent for many categories
- For MVP, eBay sold-listings is the pragmatic choice

**Persistent storage**
- All scans currently live in memory; lost on refresh
- Use `AsyncStorage` from `@react-native-async-storage/async-storage` for simple key-value
- For richer querying (filter by brand, sort by date, etc.) consider `expo-sqlite`
- On web, both work via fallbacks

**Auth & cloud sync (later)**
- For multi-device, scans need a backend. Supabase or Firebase are quickest paths
- Auth: passkeys / Sign in with Apple as primary, email/password optional

**Real maps**
- Dashboard's "geography" section is currently an SVG dot-grid with sin-wave continent shapes. Looks intentional but limited.
- Upgrade to `react-native-maps` (uses Apple Maps on iOS, Google on Android) with custom dark style + lime markers
- Or keep stylised version as the dashboard summary, deep-link to a real map screen

**App icon, splash, assets**
- Currently no icons or splash images, just background colors
- Generate icon set, splash screens, adaptive icons (Android)

### Nice-to-haves
- Haptic feedback on capture and successful scan (`expo-haptics`)
- Share scan as image (`react-native-view-shot` + native share sheet)
- Push notifications for "watch this brand" alerts (`expo-notifications`)
- Pinch-to-zoom on the detail modal hero (`react-native-gesture-handler` with `Pinchable` view)
- Pull-to-refresh on history list

### Things to NOT do (per chat history)
- Don't add `react-native-reanimated` back unless something genuinely needs it
- Don't reintroduce gradients on backgrounds (the lime design is solid colors)
- Don't make this a thrift-flipper app — that market is saturated; stay in collector territory
- Don't bundle the Anthropic API key in the app

---

## 7. Known issues / gotchas

### `registerWebModule` error
If you see `(0, _expoModulesCore.registerWebModule) is not a function`:
1. Confirm `package.json` has `expo: "~52.0.46"`, not 51
2. Wipe and reinstall: `rm -rf node_modules package-lock.json && npm install`
3. Run `npx expo install --check` and fix any version warnings

### Web preview stretches across full screen
The layout is designed for ~390px-wide phone viewports. Use Chrome DevTools → toggle device toolbar (`Cmd+Shift+M` / `Ctrl+Shift+M`) → pick "iPhone 14 Pro". The app still works at desktop width but looks strange because the floating tab bar gets very wide.

### Mac-only: iOS simulator
`npm run ios` requires Xcode. On Linux/Windows, use Expo Go on a physical iPhone via QR code (`npm start` shows it).

### Brace-expansion in `mkdir`
Earlier in the chat, `mkdir -p /home/claude/find-the-brand/{a,b,c}` failed in this shell because brace expansion wasn't enabled — it created a literal directory named `{a,b,c}`. Use space-separated paths instead: `mkdir -p path/a path/b path/c`. Just a heads-up if you do shell-level scaffolding.

---

## 8. Two design iterations in chat history

For context, the chat went through two complete design rebuilds. If the user references "the gold version" or "the vault design" in future requests, that was the FIRST iteration:

| Iteration 1 (discarded) | Iteration 2 (current) |
|---|---|
| Gold `#C9A961` accents | Lime `#C5F94D` accents |
| Georgia italic serif typography | System sans-serif typography |
| "Vault / Maison / Lot N°" copy | "Identify / Recent / Collection" copy |
| Diamond markers, ornate dividers | Pill chips, arrow buttons |
| Gradient backgrounds | Solid backgrounds |
| Auction-house aesthetic | Modern fintech aesthetic |

The user uploaded a screenshot of a Gen-Z hiring app and asked to apply that design language. The current code reflects that.

If they ask to revert to the vault aesthetic, all those tokens and patterns are documented above in §3 by contrast — recreatable but not in current code.

---

## 9. Suggested first session in Claude Code

1. **Verify the project runs**: `cd find-the-brand && rm -rf node_modules package-lock.json && npm install && npm run web`. Confirm all three screens render correctly in mobile viewport.
2. **Wire up real camera**: replace the placeholder in `screens/ScanScreen.js` with `expo-camera`. Keep the existing viewfinder overlay layered on top.
3. **Wire up geolocation**: `expo-location` to capture coordinates on each scan, replacing the hardcoded `location` field.
4. **Stand up an API proxy**: a tiny Cloudflare Worker or Vercel serverless function that accepts an image, calls Anthropic with vision, returns structured JSON. Then wire `handleCapture` in `ScanScreen.js` to hit it instead of randomly picking from `mockScans`.
5. **Add persistence**: `AsyncStorage` for scan history so it survives app restarts.

After step 4 the app is genuinely functional end-to-end with a real AI brand identifier. Steps 1-4 are roughly 1-2 sessions of focused work.

---

## 10. User's stated preferences (from chat)

- System administration manager by background
- Prefers **bullet points and tables** in responses
- Favorite editor: **vim**
- Prefers **direct communication**, minimal preamble
- Wants **artifacts/files** delivered, not just inline content

So when responding in Claude Code, keep prose tight, use tables/bullets where they help, and produce real files rather than describing what files would contain.
