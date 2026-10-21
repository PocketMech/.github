# ADR 0004: Schema-driven UI a evoluce schémat

- **Status:** Navrženo
- **Datum:** 2026-10-21

## Kontext

UI (mobile i admin) musí být řízené schématem z API. Obsah je strukturovaný; není prostor pro PDF/přílohy. Schéma se musí vyvíjet bez rozbití offline klientů.

## Rozhodnutí

- JSON Schema **2020-12** s rozšířeními:
  - `x-pocketmech-search` `{ weight }`
  - `x-pocketmech-filter`
  - `x-pocketmech-kind` `reference` | `user`
- Tenký `uiSchema`: `{ order, fields{<f>:{widget,label{cs,en},help?,hidden?,readonly?}}, list{columns,defaultSort}, detail{sections} }`.
- Soubory v `core/schema/<collection>/v<N>.json`.
- Zdroj pravdy je **git**; publish přes CI. Admin má schémata **read-only**.
- Publikované verze jsou **immutable** a hash-locked.
- Aditivní změny jsou kompatibilní: schema + uiSchema jdou v bundlech; mobile renderuje genericky a ignoruje neznámá pole.
- Breaking změna: čistá synchronní migrační funkce v `core` + major verze; **upcast při publikaci**.
- Každý záznam nese `schemaVersion`.
- Podpora: aktuální + jedna předchozí breaking verze per kolekce (jen full bundle, **90 dní**).
- Starší klienti: **426** + výzva k update; dál používají stará data.

## Důsledky

- Žádné hand-coded obrazovky per kolekce v MVP.
- Migrace jsou v `core`, ne ad-hoc v Lambda.
- Klienti mimo podporované schema verze musí aktualizovat aplikaci.

## Zvažované alternativy

- Volné JSON bez schématu — odmítnuto; nelze genericky renderovat ani validovat.
- Neomezená zpětná kompatibilita — odmítnuto; náklad na údržbu.

## Předpoklady

- CI publish dle ADR 0003.
- Bundle obsahují schema i uiSchema (ADR 0005).
