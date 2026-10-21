# ADR 0005: Delta content sync

- **Status:** Navrženo
- **Datum:** 2026-10-21

## Kontext

MVP vyžaduje download-only delta sync referenčního obsahu, odolný vůči změnám schématu, s offline provozem po prvním stažení. User notes se nesyncují.

## Rozhodnutí

- Každá publikace zvyšuje `dataVersion` a produkuje:
  - **delta** z předchozí verze (upserty + tombstones `{id, deletedAt}`)
  - **full** snapshot
- Delty se drží pro posledních **30** verzí; first install / starší base → full snapshot.
- Manifest: `dataVersion`, `schemas {<collection>: N}`, `minClientSchema {<collection>: N}`, per bundle `sha256`, `size`, `type` `full|delta`, `baseDataVersion`, signed URL.
- Klient **vždy** volá manifest; server rozhoduje:
  - **200** → bundles
  - **403** `urn:pocketmech:problem:entitlement-required` → paywall
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
