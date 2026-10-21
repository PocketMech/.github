# ADR 0003: API a AWS stack

- **Status:** Navrženo
- **Datum:** 2026-10-21

## Kontext

Potřebujeme REST API s jasným kontraktem, serverless AWS a bezpečné CI publish schémat bez Cognito M2M.

## Rozhodnutí

- REST, OpenAPI **3.1**, prefix `/v1`.
- Kontrakt žije v `core`; píše ho API architekt, schvaluje systémový architekt.
- API Gateway **HTTP API**, Lambda **nodejs24.x**, CDK v TypeScriptu (v `api`).
- Zápisy nesou `baseVersion`; nesoulad → **409**.
- Chyby: RFC **9457** `problem+json`, `type` ve tvaru `urn:pocketmech:problem:*`.
- Stránkování: opaque cursor, `limit` max **100**.
- CI publish schémat: GitHub Actions **OIDC** → IAM role; IAM (SigV4) route; **bez** Cognito M2M.

## Důsledky

- Klienti i admin sdílí jeden OpenAPI kontrakt z `core`.
- Optimistic concurrency přes `baseVersion`/`409`.
- CI nepotřebuje Cognito machine uživatele.

## Zvažované alternativy

- Cognito M2M pro CI — odmítnuto; OIDC + IAM je jednodušší a bezpečnější.
- GraphQL / AppSync jako primární API — odmítnuto (viz ADR 0005).
- REST bez problem+json — odmítnuto; nejednotné chybové tělo.

## Předpoklady

- Jedna HTTP API a jeden Cognito authorizer (ADR 0001).
- Region a data model dle ADR 0002.
