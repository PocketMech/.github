# ADR 0000: Proces ADR a šablona

- **Status:** Navrženo
- **Datum:** 2026-10-21

## Kontext

PocketMech (aplikace AeroTool) je offline-first SaaS příručka pro mechaniky B737/A320. Systémové rozhodnutí je potřeba dokumentovat jednotně napříč repozitáři (`.github`, `mobile-app`, `core`, `sync-service`, `admin-webterminal`, `api`), vše v TypeScriptu.

## Rozhodnutí

- Systémové ADR žijí v tomto repu (`.github`) pod `docs/adr/`.
- Komponentové ADR žijí v příslušných komponentových repozitářích.
- Systémový architekt schvaluje kontrakty mezi komponentami.
- Formát je krátký MADR-like: Kontext, Rozhodnutí, Důsledky, Zvažované alternativy, Předpoklady; status začíná jako **Navrženo**.
- Číslování je čtyřmístné (`NNNN-slug.md`).

## Důsledky

- Přehled architektury a hranic je v `docs/architecture.md`.
- Změna kontraktu bez schválení systémového architekta není platná.
- Komponentové týmy nerozšiřují systémové ADR mimo vlastní repo.

## Zvažované alternativy

- Monolitický design doc bez ADR — odmítnuto; chybí sledovatelnost rozhodnutí.
- Všechna ADR jen v `core` — odmítnuto; systémové hranice patří do `.github`.

## Předpoklady

- Vlastník produktu potvrzuje závazná rozhodnutí uvedená v ADR 0001+.
