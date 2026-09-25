---
name: db-two-databases-migration-order
description: "Din 25 sep 2026 sunt DOUĂ baze (producție + locală în Docker) și nimic nu le sincronizează. Ordinea migrărilor: local întâi, apoi prod, apoi `npm run db:schema`. Owner-ul a cerut să nu-l întrebi dacă a rulat pașii — rulează-i tu."
metadata:
  type: project
---

**Pe 2026-09-25 a apărut o a doua bază de date.** Producția (`veltofit-db`,
eu-central-2, PostgreSQL 17.6) și una locală, Supabase în Docker. **Nimic nu le
ține sincronizate.** O migrare aplicată doar într-una le desparte, iar simptomul
apare în altă parte decât cauza.

S-a întâmplat în aceeași zi: o migrare a plecat direct în producție dintr-o a
doua sesiune, locala a rămas în urmă, și o pagină întreagă a picat local fără ca
nimeni să înțeleagă de ce.

## Ordinea

```
1. Scrii migrarea                  docs/database/
2. npm run db:local:apply <file>   ← LOCAL întâi
3. Testezi în aplicație            npm run dev, pe localhost
4. Aplici pe PRODUCȚIE             pre-flight întâi, ghidat
5. npm run db:schema               regenerezi SCHEMA.sql ȘI ÎL COMIȚI
```

**De ce local întâi:** dacă o migrare urmează să pice, să pice pe o bază de care
nu depinde nimeni.

🔑 **Localul NU înlocuiește pre-flight-ul, și e important de înțeles de ce:** o
bază locală construită din repo va fi de acord cu **repo-ul**, nu cu producția —
deci nu poate spune niciodată că repo-ul greșește. Exact asta s-a întâmplat de
două ori pe 24-25 sep: dumpul vechi n-avea `plans.training_days`, iar
`delete_test_user.sql` afirma fals că cheile nu cascadează. Local prinde erorile
grosolane; nepotrivirile cu baza reală le prinde doar pre-flight-ul.

## Ce e unde

- **`docs/database/SCHEMA.sql`** — structura REALĂ a producției. Fotografie
  completă: chei străine cu `ON DELETE`, CHECK, politici RLS, funcții,
  declanșatoare, drepturi, comentarii. **Ăsta e adevărul.** Antetul are
  instrucțiunile; reperul „SFARSIT ANTET" îl folosește scriptul ca să nu-l piardă.
- Restul din `docs/database/` sunt **migrări** — intenții, nu stare.
- **`for_debug_db.sql` e GOLIT.** Era un dump parțial care tăia clauzele
  `ON DELETE`. A indus în eroare de două ori. Nu-l scoate din istoric.

## Capcane verificate, ca să nu se re-descopere

- **Conexiunea directă `db.<ref>.supabase.co` e doar pe IPv6** și nu se poate
  atinge dintr-un container Docker obișnuit. `supabase db dump` își aranjează
  singur containerul și merge. Pentru `psql` de mână:
  `aws-1-eu-central-2.pooler.supabase.com:5432`, user `postgres.<ref>`.
- **`SUPABASE_ACCESS_TOKEN` din `.env.local` EXPIRĂ.** Eșuează cu „Unauthorized"
  și simptomul nu spune de ce — a pierdut o jumătate de oră pe 25 sep. Unul nou
  la supabase.com/dashboard/account/tokens. **Nu unul fără expirare:** tokenul
  poate crea și ȘTERGE proiecte și citi cheile de API.
- **Parola bazei ≠ parola contului Supabase.** Resetarea ei nu atinge nimic din
  aplicație (aia folosește cheile de API), doar conexiunile directe.
- **Dumpul E stabil** — două rulări consecutive dau fișiere identice. Deci o
  diferență la regenerare înseamnă o schimbare reală în producție.

## Cum se aplică

🔑 **Owner-ul a cerut explicit (25 sep): *„eu sigur o să uit și n-o să știu să le
fac."*** Deci **nu-l întreba dacă a rulat pașii — rulează-i tu**, și spune-i la
ce pas ești. Pasul 4 rămâne al lui (aplicarea pe producție se face de mână, cu
pre-flight), restul le faci tu.

Regula e scrisă și în `CLAUDE.md` §7, care se citește la fiecare sesiune.
Related: [[feedback-guide-manual-tests-step-by-step]], [[feedback-never-start-dev-server]].
