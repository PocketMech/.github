# ADR 0007: Distribuce balíčku core

- **Status:** Navrženo
- **Datum:** 2026-10-21

## Kontext

Kontrakty, schémata, validace, migrace a server helpery musí sdílet `mobile-app`, `api`, `sync-service` a `admin-webterminal` bez monorepa. Mobile běží na Hermes (žádný `eval`).

## Rozhodnutí

- npm balíček **`@pocketmech/core`** na **GitHub Packages**
- Versioning: **Changesets** (semver)
- Dual **ESM/CJS**
- Subpaths:
  - `./contract`
  - `./schema`
  - `./types`
  - `./validation` — standalone precompiled Ajv, **bez eval**, pro Hermes
  - `./migrations`
  - `./manifest`
  - `./entitlements`
  - `./server`
  - `./sdk`
- Oddělené repozitáře; **žádný monorepo**

## Důsledky

- Spotřebitelé pinují semver range; breaking změny jdou přes major.
- Validace funguje v React Native / Hermes.
- Publikační pipeline `core` je nezávislá na `api` deployi (schémata do Dynamo/bundle přes CI dle ADR 0003/0004).

## Zvažované alternativy

- Monorepo (pnpm/nx) — odmítnuto vlastníkem.
- Kopírování kontraktů mezi repy — odmítnuto; drift.
- Ajv s codegen/eval v runtime — odmítnuto kvůli Hermes.

## Předpoklady

- GitHub Packages je dostupné všem komponentovým repům.
- Systémový architekt schvaluje kontrakty v `core` (ADR 0000).
