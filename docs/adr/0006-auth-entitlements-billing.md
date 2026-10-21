# ADR 0006: Auth, entitlements a billing

- **Status:** Navrženo
- **Datum:** 2026-10-21

## Kontext

SaaS s platbami výhradně přes in-app subscriptions (App Store, Google Play). Nic není zdarma (free tier jen rezervován). Po expiraci se stáhnutý referenční obsah maže. Login username/password; bez PIN/biometrie v MVP, ale s rezervou. Offline po prvním downloadu i s expired login. Notes lokální, nikdy sync/purge.

## Rozhodnutí

### Auth

- Cognito User Pool (**Lite**), username + password.
- Mobile: native form, `USER_SRP_AUTH` (aws-amplify v6 auth), tokeny v `expo-secure-store`.
- Admin: Hosted UI + PKCE.
- Skupiny `mechanic` a `admin` z `cognito:groups`.
- Prostor pro budoucí device lock (PIN/biometrie) — mimo MVP.

### Entitlements

- Tvar: `{ plan, features[], status: active|grace|expired, validUntil, source: appstore|googleplay|manual }`.
- **Nejsou** v tokenu; server-side `core.requireEntitlement(feature)` s cache **60 s**.
- `GET /v1/me/entitlements`.
- Features enum v `core`; aktuálně jen `sync`.

### Billing

- `POST /v1/billing/verify` — discriminated union:
  - `{ platform: 'appstore', signedTransactionInfo }`
  - `{ platform: 'googleplay', productId, purchaseToken }`
  - ověření přes App Store Server API / Google Play Developer API
- `POST /v1/billing/appstore/notifications` — App Store Server Notifications V2 + JWS
- `POST /v1/billing/googleplay/rtdn` — Pub/Sub push + OIDC
- Vazba uživatele: `appAccountToken` / `obfuscatedAccountId` odvozené z Cognito `sub`
- Store credentials v Secrets Manager
- Klient dokončí/acknowledge nákup **až po** verify
- Admin: store entitlements read-only; manuální editovatelné

### Purge

- `core.shouldPurgeReferenceContent(e, lastRefreshAt, now, offlineTolerance = 72h)` → `true` když:
  - `status === expired`, nebo
  - `now > validUntil` a aplikace neobnovila entitlements v toleranci
- **Grace nikdy nepurgeuje**
- Maže se jen `kind: reference`, nikdy notes

## Důsledky

- Paywall je vynucen serverem (manifest 403) i klientem (purge).
- Offline tolerance 72 h je otevřený předpoklad (čeká potvrzení vlastníka).
- Free tier není implementován; API/model počítá s rezervou.

## Zvažované alternativy

- Entitlements v JWT claims — odmítnuto; těžká invalidace, cache 60 s stačí.
- Vlastní platební brána — odmítnuto; jen store subscriptions.
- PIN/biometrie v MVP — odmítnuto; rezervováno.

## Předpoklady

- Offline tolerance **72 h** čeká na potvrzení vlastníka.
- Free tier je pouze rezervován.
- Po expiraci subscription se referenční obsah maže z zařízení.
