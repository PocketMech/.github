# ADR 0005: Delta content sync

- **Status:** Navrženo
- **Datum:** 2026-10-21

## Kontext

MVP vyžaduje download-only delta sync referenčního obsahu, odolný vůči změnám schématu, s offline provozem po prvním stažení. User notes se nesyncují.

## Rozhodnutí

- Každá publikace zvyšuje `dataVersion` a produkuje:
  - **delta** z předchozí verze (upserty + tombstones `{id, deletedAt}`)
  - **full** snapshot
- **`dataVersion`:**
  - na **kořeni manifestu** = cílová verze publikace
  - u **každého balíčku** = verze, kterou daný balíček vytváří
  - u **delta** balíčku navíc `baseDataVersion` (verze, ze které delta vychází)
- Každý balíček má vlastní `sha256` a `size`; pole `type` `full|delta`; signed URL.
- Delty se drží pro posledních **30 publikací**; first install / klient starší než okno → jen **full**.
- Tombstones mají TTL **až po vypadnutí z okna 30 publikací** (bez pevné meze 90 dní).
- Klient volá `GET /v1/sync/manifest?dataVersion=&schemas=coll:N,...` (aktuální lokální `dataVersion` a schema verze per kolekce).
- Manifest nese mimo jiné: kořenové `dataVersion`, `schemas {<collection>: N}`, `minClientSchema {<collection>: N}`, seznam balíčků, a dle ADR 0008 také `attachments { baseUrl, signedQuery, expiresAt, totalBytes }`.
- Server rozhoduje:
  - **200** → bundles (delta pokud `baseDataVersion` sedí a je v okně; jinak full)
  - **403** `urn:pocketmech:problem:entitlement-required` → paywall (rozšíření viz ADR 0006)
- Když base nesedí (nebo delta není k dispozici), klient **načte nový manifest** a poté stáhne **full**.
- Mobile ověří `sha256` a aplikuje bundle **transakčně** do SQLite FTS5 (včetně indexu).
- `GET /v1/sync/manifest` vlastní `sync-service` (ADR 0001).
- AppSync **zamítnut**.

### Mimo MVP (rezervováno)

- Sync user dat (`clientId` + `mutationId` rezervovány)
- QR handover
- Audit log

## Důsledky

- Sync je již v MVP delta a odolný vůči schema changes (spolu s ADR 0004).
- Po expiraci subscription se referenční obsah maže na zařízení (ADR 0006); notes zůstávají.
- Bez entitlements není download.

## Zvažované alternativy

- AppSync — odmítnuto.
- Pouze full snapshot vždy — odmítnuto; zbytečný přenos.
- Sync user notes — odmítnuto vlastníkem; notes zůstávají lokální.

## Předpoklady

- CloudFront trusted key group + signing key v Secrets Manager (ADR 0001).
- Tombstones dle ADR 0002.
- Žádný audit log v MVP.
- Přílohy v manifestu dle ADR 0008.

## Revize 2026-10-21

- `dataVersion` na kořeni i u balíčku; delta má `baseDataVersion`; každý balíček vlastní `sha256`/`size`.
- Query `?dataVersion=&schemas=coll:N,...`; delty pro posledních 30 publikací, starší klient jen full.
- Nesedící base → nový manifest a pak full; tombstones TTL = okno 30 publikací (ne 90 dní).
- Manifest nese `attachments { baseUrl, signedQuery, expiresAt, totalBytes }` (ADR 0008).
