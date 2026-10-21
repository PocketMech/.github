# ADR 0008: Přílohy

- **Status:** Navrženo
- **Datum:** 2026-10-21

## Kontext

Obsah PocketMech jsou schématem řízené strukturované záznamy NoSQL (DynamoDB single-table v `eu-central-1`, např. jističe B737/A320). Marek upřesnil, že záznamy mohou volitelně mít přílohy, hlavně fotky. PDF zatím ne, ale seznam typů musí jít rozšířit. Aplikace je offline-first a po konci předplatného se referenční obsah maže.

## Rozhodnutí

### Schéma

- JSON Schema **2020-12** v `core`: pole typu `array` s `items` `$ref` na `AttachmentRef` a klíčem `x-pocketmech-attachment` `{ accept: [image/jpeg, image/png, image/webp, image/heic], maxCount, maxBytes }` (`maxBytes` default **10 MB**).
- Widget v `uiSchema` se jmenuje `attachments`.
- Přidání takového pole je **aditivní** (adaptivní) změna.

### AttachmentRef

V datech záznamu:

```json
{
  "id": "...",
  "sha256": "...",
  "contentType": "...",
  "size": 0,
  "width": 0,
  "height": 0,
  "caption": { "cs": "...", "en": "..." }
}
```

- `caption` je volitelné.
- Binárka **nikdy** nejde do DynamoDB ani do bundlu, takže delta sync se nemění.

### Úložiště

- S3 bucket vlastní `api`, klíč `attachments/<sha256>/{orig,display,thumb}`.
- Přílohy jsou content-addressed a neměnné; výměna znamená nový ref.
- Zpracování Lambdou po uploadu: ověření `sha256` a magic bytes, odstranění EXIF (GPS), varianta **display** WebP max **1600 px** a **thumb** WebP **320 px**.

### Upload

- Jen skupina **admin**.
- `POST /v1/attachments` `{ sha256, contentType, size }` vrátí presigned PUT a `id`.
- `GET /v1/attachments/{id}` vrací stav `pending` | `ready` | `rejected`.
- Zápis nebo publikace záznamu s refem, který není `ready`, skončí chybou **422** (`problem+json`).

### Distribuce

- CloudFront, behavior `attachments/*` s trusted key group.
- Manifest ze `sync-service` nese `attachments { baseUrl, signedQuery, expiresAt, totalBytes }`.
- Jeden krátkodobý podpis s wildcard custom policy; vydává se jen při platném entitlementu sync.
- `totalBytes` je součet variant `display` a `thumb` pro aktuální `dataVersion`.

### Mobil

- Po aplikaci bundlu na pozadí stáhne `display` a `thumb` všech referencovaných příloh do `documentDirectory` (ne do cache).
- Soubory klíčuje podle `sha256`, ověří hash, umožní resume a předem zkontroluje volné místo podle `totalBytes`.
- Soubory, na které už nic neodkazuje, smaže.
- Originál nestahuje.
- Když podpis vyprší, načte nový manifest.
- Fulltext indexuje jen `caption`.
- Purge po konci předplatného maže i přílohy; poznámky uživatele zůstávají.

### Admin

- Widget `attachments` v prohlížeči spočítá `sha256`, nahraje soubor přes presigned PUT, čeká na stav `ready` a zobrazuje náhled, pořadí a odebrání.

### core

- Typ `AttachmentRef`.
- Validace keywordu.
- Čisté funkce `attachmentKey(sha256, variant)` a `collectAttachmentRefs(records, schema)`, které sdílí mobil (stahování a úklid) i server (GC).

### GC v S3

- Maže přílohy, na které neodkazuje žádná z **30** držených publikací a zároveň jsou starší než **90** dní.

## Důsledky

- Nároky na úložiště v telefonu.
- Náklady na S3 a CloudFront.
- Nutnost GC.
- **Otevřená otázka:** limit celkové velikosti offline dat.

## Zvažované alternativy

- Binárky přímo v bundlu — odmítnuto; zvětšuje delty.
- Podepsané URL pro každý soubor v manifestu — odmítnuto; velký manifest.
- CloudFront signed cookies — odmítnuto; v React Native nepohodlné.
- Stahování na vyžádání — odmítnuto; nefunguje offline.

## Předpoklady

- Delta sync a manifest dle ADR 0005.
- Schema-driven UI a aditivní evoluce dle ADR 0004.
- Entitlements a purge referenčního obsahu dle ADR 0006.
- CloudFront trusted key group dle ADR 0001 / 0003.
