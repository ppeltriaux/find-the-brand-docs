# Find the Brand

iOS app for identifying luxury and branded items via camera. Point camera at item → AI identifies it → app surfaces brand, product, retail + pre-owned prices, official store + resale links, and tags the location on your map. Pro tier unlocks authentication checks, item story, barcode, and text scanning.

Live on the App Store (May 2026). Built with **React Native + Expo SDK 54** (New Architecture).

## Design system

| Token | Value | Use |
|---|---|---|
| `bg` | `#0E0E0E` | App background |
| `bgElevated` | `#1C1C1E` | Cards, icon buttons |
| `bgPill` | `#2A2A2C` | Inline pills, tags |
| `lime` | `#C5F94D` | Primary accent — buttons, highlights |
| `textPrimary` | `#FFFFFF` | Headings, body |
| `textSecondary` | `#8E8E93` | Subtitles, meta |

## Run locally

```bash
npm install
npm run ios        # iOS simulator (Mac only)
npm start          # Expo Go via QR
```

## Architecture

- **App** — Expo iOS app. Camera, barcode/text scan, location, push notifications, IAP via RevenueCat.
- **Proxy** (`/proxy`) — Node Express on a Raspberry Pi. Brokers Anthropic vision calls, eBay pricing, RC entitlement verification, push delivery. Shared-secret auth + per-device monthly AI cap + RC-verified Pro for paid endpoints + Apple App Attest (phase 1, log-only).
- **Native module** (`/modules/app-attest`) — Swift wrapper around `DCAppAttestService` for cryptographically proving requests originate from a real device running our real app.
- **Admin dashboard** — separate HTTP listener (`proxy/dashboard.js`, port 4000) on the Pi's LAN-only firewall. Summary cards (clickable to drill down), charts, admin actions (delete watchlist subscriptions), and a backups section that auto-snapshots `data.json` daily and lets you download any of the last 30 days.
- **Data** — AsyncStorage for scans/profile. `Documents/scans/` for persisted photos. Server stores device tokens, watchlist subscriptions, inbox items, AI-call quotas, RC entitlement cache, and App Attest public keys.

## Project structure

```
find-the-brand/
├── App.js                  # Floating pill tab bar
├── theme/theme.js          # Lime-on-charcoal token system
├── components/             # ArrowButton, Chip, Avatar, DonutChart, etc.
├── screens/                # Scan, History, Dashboard, Profile, Onboarding
├── lib/                    # identify, authenticate, provenance, watchlist,
│                           # purchases (RC), photo storage, image resize,
│                           # device id, scan quota, proxy fetch, appAttest, etc.
├── modules/app-attest/     # Local Expo Module — Apple DCAppAttestService wrapper
├── proxy/                  # Express server + state.json + cron + dashboard
│   ├── server.js
│   ├── appAttest.js        # Apple cert chain + assertion verification
│   ├── dashboard.js        # Internal admin dashboard listener
│   └── public/dashboard.html
└── ios/                    # Bare native iOS project (prebuild output)
```

See **[CLAUDE.md](./CLAUDE.md)** for session-orientation context and **[HANDOFF.md](./HANDOFF.md)** for design history.
