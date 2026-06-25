<div align="center">

<img src="docs/banner.png" alt="Souq — the map-first marketplace for the Gulf" width="100%" />

# 🛒 Souq

### The map-first marketplace for the Gulf.

*Discover shops, freelancers and independent sellers around you — on a living map, not a list.*

[![Status](https://img.shields.io/badge/status-Production%20MVP-3FCF8E?style=for-the-badge)](#-development-status)
[![Platform](https://img.shields.io/badge/platform-Android%20%C2%B7%20iOS-0B1E27?style=for-the-badge)](#-tech-stack)
![React Native](https://img.shields.io/badge/React_Native-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![Supabase](https://img.shields.io/badge/Supabase-3FCF8E?style=for-the-badge&logo=supabase&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)

</div>

> **Public portfolio repository.** This repo documents Souq's architecture, design, and roadmap. The application source is proprietary and kept private. For a code walkthrough under NDA, reach out: **arasghorbani9090@gmail.com**.

---

## 📖 Overview

Across the Gulf, an enormous amount of commerce happens informally — through WhatsApp, Instagram pages, word of mouth, and physical shops you only find by walking past them. There's no single place to **discover what's for sale around you**.

**Souq** turns the map itself into the marketplace. Open the app and you see real storefronts as profile-image markers pinned to where they actually are. Tap one to view its products, ratings, and hours, then message the seller directly on WhatsApp. Sellers — whether a registered shop, a freelancer, or someone selling from home — get a storefront in minutes, no website required.

**Who it's for**
- 🛍 **Buyers** — find shops, services, and items near them, filtered by category and radius.
- 🏪 **Shops** — a discoverable storefront with products, hours, and one-tap contact.
- 🧑‍🔧 **Service providers & independent sellers** — a presence on the map without building a website.

---

## ✨ Features

- 🗺 **Map-first discovery** — native MapLibre map where every seller is a circular shop-photo marker, clustered at city zoom and revealed individually as you zoom in.
- 🔍 **Geolocation product search** — search products and shops within an adjustable radius; "My Areas" lets buyers pin the neighborhoods they care about.
- 🏬 **Storefronts** — each seller gets a cover, logo, rating, working hours, and a product grid — a shareable mini-shop.
- 💬 **WhatsApp-native contact** — buyers reach sellers through the channel Gulf commerce already runs on; one tap, pre-filled message.
- 🧭 **Open-now & directions** — live open/closed status from business hours, plus one-tap directions.
- 👤 **Three account types** — shops, service providers, and independent sellers, each surfaced with the right badges and UX.
- 📱 **Fast onboarding** — phone + OTP, designed so a seller goes from install to a live listing in minutes.
- ⚡ **Native performance** — built on the New Architecture (Fabric/Hermes) for smooth map interaction on real devices.

---

## 🏗 Architecture

```mermaid
flowchart TD
    subgraph Client["📱 Mobile App — React Native / Expo"]
        UI["Map · Storefronts · Search"]
        LOC["Geolocation & Radius"]
        IMG["Image pipeline / thumbs"]
    end

    subgraph Edge["☁️ Supabase Platform"]
        AUTH["Auth — Phone + OTP"]
        REST["PostgREST API / RPC"]
        STORE["Storage — logos · covers · products"]
    end

    subgraph Data["🗄 PostgreSQL + PostGIS"]
        SHOPS[("shops")]
        PRODUCTS[("products")]
        HOURS[("business_hours")]
        GEO["PostGIS spatial index"]
    end

    subgraph External["🔌 Integrations"]
        WA["WhatsApp deep links"]
        MAPS["Map tiles / directions"]
    end

    UI --> REST
    LOC --> REST
    UI --> AUTH
    IMG --> STORE
    REST --> SHOPS
    REST --> PRODUCTS
    REST --> HOURS
    SHOPS --> GEO
    REST -. spatial RPC .-> GEO
    UI --> WA
    UI --> MAPS
```

### Map rendering pipeline

```mermaid
sequenceDiagram
    participant U as Buyer
    participant M as Map View
    participant C as Clustering (client)
    participant API as Supabase RPC
    participant DB as PostGIS

    U->>M: Pan / zoom map
    M->>API: shops_in_area(bounds)
    API->>DB: spatial query (radius + area)
    DB-->>API: shops + ratings + hours
    API-->>M: GeoJSON feature set
    M->>C: cluster by zoom-dependent grid
    C-->>M: clusters + shop-photo markers
    U->>M: Tap marker
    M-->>U: Storefront sheet → products → WhatsApp
```

---

## 🧱 Tech Stack

| Layer | Technology |
| --- | --- |
| **Mobile** | React Native, Expo (SDK 56), TypeScript, New Architecture (Fabric + Hermes) |
| **Maps** | MapLibre (native), client-side geo-grid clustering, custom photo markers |
| **Backend** | Supabase — PostgREST, Auth (phone/OTP), Storage, Edge functions |
| **Database** | PostgreSQL + PostGIS (spatial indexing, radius & area queries, RPC) |
| **Integrations** | WhatsApp deep linking, native maps / directions |
| **Tooling** | EAS / local Gradle release builds, signed Android APK distribution |

---

## 📂 Folder Structure

> Representative structure — illustrative of organization, not a source dump.

```
souq/
├── app/                      # React Native / Expo application
│   ├── src/
│   │   ├── screens/          # Map, Search, Storefront, Create Listing, Onboarding
│   │   ├── components/       # MapShopSheet, StorefrontModal, ProductCard, Icon
│   │   ├── data/             # shops, products, storefront — typed data access
│   │   ├── map/              # map modes, styles, clustering helpers
│   │   ├── lib/              # geolocation, image URLs, maps/deeplinks
│   │   └── theme/            # design tokens
│   └── android/              # native Android project (release builds)
├── supabase/
│   ├── migrations/           # schema, spatial RPCs, marketplace data model
│   └── seed/                 # data generation for demo ecosystems
└── docs/                     # architecture & design notes
```

---

## 🖼 Screenshots

> Placeholders — drop real captures into `docs/screenshots/` and they'll render here.

| Map discovery | Storefront | Product search |
| --- | --- | --- |
| ![Map](docs/screenshots/map.png) | ![Storefront](docs/screenshots/storefront.png) | ![Search](docs/screenshots/search.png) |

| Create listing | Onboarding | Cluster zoom |
| --- | --- | --- |
| ![Create](docs/screenshots/create.png) | ![Onboarding](docs/screenshots/onboarding.png) | ![Clusters](docs/screenshots/clusters.png) |

---

## 🗺 Roadmap

```mermaid
gantt
    title Souq Roadmap
    dateFormat  YYYY-MM
    axisFormat  %b %y
    section Foundation
    Map-first MVP (native)        :done,    f1, 2026-01, 2026-04
    Storefronts + WhatsApp        :done,    f2, 2026-03, 2026-05
    Marketplace data model        :done,    f3, 2026-05, 2026-06
    section Growth
    Seller self-onboarding        :active,  g1, 2026-06, 2026-08
    Categories + filters v2       :         g2, 2026-07, 2026-09
    In-app saved areas & alerts   :         g3, 2026-08, 2026-10
    section Scale
    iOS release                   :         s1, 2026-09, 2026-11
    Payments / escrow pilot       :         s2, 2026-10, 2027-01
    Seller analytics dashboard    :         s3, 2026-11, 2027-02
```

- [x] Native map-first MVP on Android (real-device verified)
- [x] Storefronts, ratings, business hours, open-now
- [x] WhatsApp-native contact + directions
- [x] Geolocation product search + radius / My Areas
- [ ] Frictionless seller self-onboarding (phone/OTP → live listing)
- [ ] Category taxonomy & advanced filters
- [ ] iOS release
- [ ] Payments / escrow pilot
- [ ] Seller analytics

---

## 📈 Development Status

🟢 **Production MVP** — running on native Android, tested on real devices, backed by a live PostGIS database with a full marketplace data model. Actively building seller onboarding and preparing iOS.

---

## 🤝 For Partners

Souq is founder-built and shipping. If you're an **investor, accelerator, technical co-founder, or engineer** interested in Gulf-region commerce infrastructure:

📧 **arasghorbani9090@gmail.com** · 🔗 [LinkedIn](https://www.linkedin.com/in/aras-ghorbani-ab1a7b62)

<div align="center"><sub>© Souq. Architecture & docs are public; application source is proprietary.</sub></div>

## Source Code

The production source code is maintained in a private repository. The portfolio repository demonstrates the architecture, features, and technical design. Source code can be shared for employment or collaboration opportunities under appropriate confidentiality terms.
