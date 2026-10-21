# ADR 0001: Přehled a hranice komponent

- **Status:** Navrženo
- **Datum:** 2026-10-21

## Kontext

AeroTool je offline-first SaaS: schema-driven strukturované NoSQL záznamy (např. circuit breakery). Záznamy mohou volitelně mít přílohy, hlavně fotky; PDF ne (viz [docs/adr/0008-prilohy.md](0008-prilohy.md)). MVP: offline záznamy + fulltext v React Native (Android/iOS), download-only delta sync, serverless AWS, admin web terminál, schéma a zobrazení z API. Bez multitenancy. EU data residency.

## Rozhodnutí

Repozitáře a vlastnictví:

| Komponenta | Vlastní |
|---|---|
| **api** | DynamoDB tabulka a CDK stack; CRUD referenčních záznamů; servírování schémat; admin operace (batch migrace, entitlements, `POST /v1/bundles` → 202 nebo 409 pokud běží publish, `GET /v1/bundles/{dataVersion}`); billing endpointy; S3 bundle bucket + CloudFront s trusted key group; S3 přílohy (`attachments/<sha256>/{orig,display,thumb}`) + CloudFront behavior `attachments/*` (ADR 0008) |
| **sync-service** | `GET /v1/sync/manifest`; privátní CloudFront signing key v Secrets Manager; krátkodobé signed URL (bundly i wildcard podpis pro přílohy) |
| **core** | Kontrakty, schémata, typy, validace, migrace, entitlements, server helpery, SDK |
| **mobile-app** | Expo klient, lokální SQLite FTS5, offline provoz (včetně stažených příloh) |
| **admin-webterminal** | React + Vite SPA pro správu záznamů (včetně uploadu příloh) |
| **.github** | Systémové ADR, sdílená CI |

Sdílené:

- Jedna API Gateway HTTP API a jeden Cognito authorizer.
- Kontrakty drží `core`.

## Důsledky

- Publish a podepisování URL jsou oddělené (`api` vs `sync-service`).
- Klienti nevolají S3 přímo bez signed URL z manifestu.
- Rozšíření AWS stacku zůstává v `api` CDK, pokud ADR neurčí jinak.

## Zvažované alternativy

- Sync v `api` — odmítnuto; oddělení signing klíče a manifestu snižuje blast radius.
- Více API Gateway — odmítnuto; jedna HTTP API zjednodušuje auth a routing.

## Předpoklady

- Žádná multitenancy.
- Obsah je schema-driven NoSQL záznamy (kolekce se stovkami záznamů) s volitelnými přílohami dle ADR 0008.

## Revize 2026-10-21

- Odstraněno „bez příloh“; záznamy mohou mít přílohy (fotky; PDF ne) dle ADR 0008.
- Do vlastnictví `api` / `sync-service` doplněny S3 a CloudFront pro přílohy.
