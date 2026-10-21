# ADR 0002: Datový model (DynamoDB single-table)

- **Status:** Navrženo
- **Datum:** 2026-10-21

## Kontext

Referenční obsah (kolekce se stovkami záznamů), schémata a entitlements musí žít v NoSQL s EU residency a podporou delta sync včetně tombstones.

## Rozhodnutí

- DynamoDB **single-table** v regionu **eu-central-1**; vlastníkem tabulky je `api`.
- Záznamy: `PK=COLL#<collection>`, `SK=DOC#<id>`; atributy `data`, `version`, `updatedAt`, `deleted`, `schemaVersion`.
- Schémata: `PK=SCHEMA#<collection>`, `SK=V#<n>`.
- Entitlements: `PK=USER#<sub>`, `SK=ENTITLEMENTS`.
- ID záznamů jsou stabilní napříč verzemi.
- Tombstones se drží alespoň po dobu delta okna.
- GSI zůstávají minimální.

## Důsledky

- Delta sync může spolehlivě emitovat upserty i tombstones.
- Klienti identifikují dokumenty stabilním `id`, ne pozicí ve snapshotu.
- Přibývající GSI vyžadují nové ADR / schválení.

## Zvažované alternativy

- Více tabulek per kolekce — odmítnuto; vyšší provozní režie.
- Soft-delete bez tombstones — odmítnuto; delta by nedokázala mazat na zařízení.

## Předpoklady

- Objem je stovky záznamů na kolekci, ne miliardy řádků.
- EU data residency je závazná.
