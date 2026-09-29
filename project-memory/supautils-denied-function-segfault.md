---
name: supautils-denied-function-segfault
description: Pe imaginea Supabase (PG 17.6.1.104, local ȘI prod), un rol anon/authenticated setat dintr-o sesiune postgres care cheamă o funcție fără EXECUTE DĂRÂMĂ serverul (signal 11). Verificările SQL nu au voie să facă asta.
metadata:
  type: project
---

Găsit 2026-09-29 pe baza locală, la `landing_events_verify.sql`. Reproducere minimă:
`BEGIN; SET LOCAL ROLE anon; SELECT * FROM public.admin_trainer_detail(gen_random_uuid()); ROLLBACK;`
→ backend-ul Postgres moare cu segfault, tot serverul intră în recovery.

Cauza: `supautils` cu `hint_roles = 'anon, authenticated, service_role'` (funcția care
adaugă hint-uri „GRANT … TO anon" la erorile de permisiune — aceeași schimbare din `684ab06`).
Pică DOAR dacă: eroarea e pe o FUNCȚIE (pe tabel merge, dă hint), rolul curent e în
hint_roles, și sesiunea a fost deschisă ca `postgres` (SQL editor, `db-local-apply.sh`).
Traficul real prin PostgREST (cheia anon, rol authenticator) NU e afectat — verificat local.

`supabase/.temp/postgres-version` arată prod pe aceeași imagine (17.6.1.104) → **aproape sigur
la fel în producție**. Rularea unei verificări cu o asemenea sondă în SQL editor-ul de prod ar
reporni baza de producție.

**⚠ `docs/database/trainer_signup_source_verify.sql` check 4.4 (linia ~132) face exact asta** —
NU se mai rulează pe prod așa cum e. A trecut pe 21 sep probabil pentru că hint_roles nu exista încă.

**Cum aplici:** în fișierele de verificare, un „rolul X NU poate chema funcția F" se verifică
din catalog (`has_function_privilege('anon', 'public.f(args)', 'EXECUTE')`), niciodată chemând-o.
Chemările permise (inclusiv cele refuzate de garda din corpul funcției, „Not authorized") sunt OK.
Vezi [[db-two-databases-migration-order]].
