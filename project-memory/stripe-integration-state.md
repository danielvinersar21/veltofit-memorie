---
name: stripe-integration-state
description: "START AICI pentru facturare. LIVE MODE E CONFIGURAT COMPLET (6 sep): 7 produse, 14 prețuri, portal, dunning, dispute, cheile în Vercel. ⚠️ DEPĂȘIT: codul E în producție din 2026-09-07 (deploy de 109 commit-uri). Vezi [[billing-payments-ledger]] pentru starea de acum. Start: docs/plans/billing-review-findings-v1.md"
metadata: 
  node_type: memory
  type: project
  originSessionId: 671c6857-3813-43fe-928d-27deea8601cd
  modified: 2026-09-22T14:17:23.599Z
---

## ✅ Audit live, 2026-09-22 (Stripe CLI `--live`, read-only + Dashboard read via Chrome)

OK: account charges/payouts enabled, no requirements due; descriptor VELTOFIT; support@
veltofit.app; tax id set. 7 active products, 14 prices = exactly the app's ladder (Plus 99/990,
Max 195/299/399/499/699/899, annual = 10×), all `velto_*` lookup keys, tax_behavior inclusive.
Live webhook enabled → app.veltofit.app/api/billing/webhook, 4 events, zero undelivered.
Receipts + refund emails OFF, portal invoice history OFF (both deliberate: SOLO issues invoices).
Invoice prefix STRP, footer „TVA 0% – neînregistrat… art. 310". Failed payments: retries, then
cancel the subscription (= Free cap, matches the 1 Sep policy). **Only one live subscription ever:
the owner's test account, Plus, created 8 Sep, CANCELED 9 Sep in trial — no real money, nothing to
cancel.** No live events since 14 Sep.

🔴 **Capcană, căzut în ea DE DOUĂ ORI (6 sep și 22 sep): prin API, portalul live arată
`subscription_update.products` = null/0. E FALS.** Dashboard-ul (Settings → Billing →
Customer portal) arată „Customers can switch plans" PORNIT, cu toate cele 7 produse, lunar
și anual, plus linkurile de termeni și confidențialitate. Owner-ul confirmă că schimbarea
pachetului merge. Pe 22 sep am raportat asta ca „gol" din citirea API și a trebuit retras.
**Portalul se verifică DOAR în dashboard, niciodată din API.** Fusese deja tranșat pe
8 sep, pe abonamentul live, în `docs/plans/backlog.md` („Portalul funcționează complet"):
citește acolo înainte să re-raportezi ceva despre portal.

(B) ✅ **Emailurile Stripe către clienți, PORNITE pe 22 sep** (aprobat de owner, salvate și
verificate după reîncărcare): reminder cu 7 zile înainte de finalul trialului; email la plată
cu cardul eșuată; link găzduit de Stripe pentru confirmarea plății (3D Secure), cu remindere
la 3/5/7 zile. Rămân oprite: reînnoire, card care expiră, debit bancar, facturi trimise manual.
„Payment method updates" → ✅ **„Link to a Stripe-hosted page"** (22 sep, aprobat de owner).
Înainte era „Use a mix of both (Legacy)" cu toate linkurile spre landing. ⚠️ Stripe a avertizat
că trecerea e ireversibilă: varianta Legacy nu mai apare deloc în listă. Rămâne „own custom link".

(C) ✅ **Curățenia din Stripe, făcută 22 sep** (aprobată de owner): prețul vechi de 3500 EUR
(`price_1NcqjI…`, produsul arhivat „Monthly", 2023, zero abonamente) → arhivat; live are acum
exact 14 prețuri active. Cele 3 abonamente din test mode → anulate (2 ale contului de test
`8fd89cd9…`, Plus 15 și Max 30; 1 fixture de 15 USD de la `stripe trigger`). Zero active în test.
Webhook-ul de TEST (`we_1UCeqx…`) e **dezactivat**, deci anulările n-au atins baza.
⏳ Rândul din DB al contului de test ține încă abonamentul de test ca activ: SQL-ul de resetare
(ca `applyCancellation` + golește `stripe_customer_id`, care e un client din test mode) i-a fost
dat owner-ului pe 22 sep, cu gardă pe cele două id-uri. ✅ **Rulat de owner pe 22 sep**
(a raportat „am rulat”, fără STOP; rezultatul final nu mi l-a arătat). Dacă mai apare contul
de test cu abonament activ în admin, de aici pornești.

## 🔴 CITEȘTE ASTA ÎNTÂI — corectură din 2026-09-08

Nota asta a spus luni de zile „LIVE MODE CONFIGURAT COMPLET 6 sep … cheile în
Vercel". **Cheile NU erau în Vercel.** Ce fusese configurat pe 6 septembrie erau
produsele, prețurile, portalul și dunning-ul din contul Stripe — nu variabilele
din deployment. Formularea a ascuns diferența, iar producția a rămas pe chei de
TEST până pe 8 septembrie.

Ce a ieșit când owner-ul a văzut eticheta „Sandbox" pe pagina de plată a
aplicației reale:

- `STRIPE_SECRET_KEY` din Vercel = cheie de test, neschimbată vreodată;
- **niciun webhook în live** — `GET /v1/webhook_endpoints --live` întorcea `[]`;
- webhook-ul de **TEST** arăta spre `https://app.veltofit.app/api/billing/webhook`,
  deci evenimentele de test scriau în **baza de producție**. Contul `+test`
  figura acolo cu un abonament Max/active dintr-o plată în modul test.

**Reparat 2026-09-08**, în ordinea asta: endpoint de test dezactivat → endpoint
live creat cu aceleași trei evenimente → ambele chei live puse în Vercel →
**redeploy** (fără el variabilele nu ajung nicăieri, Vercel le coace în build) →
curățarea a două rânduri poluate → testul 14 reluat și trecut.

⚠️ **Endpoint-ul live e fixat pe `api_version: 2022-11-15`**, versiunea implicită
a contului, în timp ce SDK-ul e pe `2026-08-26.dahlia`. Nu strică nimic azi —
codul citește din payload doar `id` și `status`, restul îl recitește de la API —
dar versiunea nu se poate schimba pe un endpoint existent, doar ștergând și
recreând, cu alt signing secret și încă un redeploy.

**Lecția, ca să nu se repete:** „configurat în Stripe" și „configurat în
deployment" sunt două lucruri, iar nota asta le-a numit la fel.

---

**Începe de aici dacă reiei facturarea:**
`docs/plans/billing-review-findings-v1.md` — lista completă, ordonată, cu ce e
reparat și ce nu.

## Unde s-a ajuns

**2026-08-31** — lotul 1 din `trial-end-and-card-v1.md` §9 construit și **dovedit
cap la cap în test mode**, pe `daniel.vinersar+test@yahoo.com`: checkout cu trial
de 30 de zile → webhook → `plan_tier='plus'`, `seat_limit=15`, `trialing`,
registrul revendicat → upgrade prin portal la Max 30 → `seat_limit=30` cu trialul
intact → downgrade cerut → programat, baza neatinsă.

Numărul pe care îl plătește antrenorul călătorește printr-un singur canal:
**lookup key-ul Price-ului** (`velto_<tier>_<seatLimit>_<interval>`). De aceea nu
există niciun Price ID în repo și nicio variabilă de mediu de schimbat la
lansare — cheile sunt identice în test și în live.

**2026-09-01 — cele 4 critice sunt reparate.** A treia trecere a recenzorului
peste tot lotul: **0 blocante**, 4 avertismente, 8 sugestii.

## Ce s-a reparat 2026-09-01

| | |
|---|---|
| **C1** | Toate cele 3 scrieri pe `trainer_subscriptions` cer rândurile înapoi cu `.select()`. Zero rânduri → 400 permanent în webhook (livrarea eșuată e singura alertă a proiectului), log cu ambele id-uri în checkout |
| **C2** | Anularea filtrează pe `stripe_subscription_id`; zero rânduri se dezambiguizează în „rândul a mers mai departe" (200) vs „nu există rând" (400) |
| **C2b** | *Găsit de recenzor la recenzia reparației.* Garda de ordonare era dezactivată când rândul ținea `stripe_subscription_id = NULL` — starea normală **după fiecare anulare**, nu doar înainte de primul abonament. Două drumuri intrau: reîncercarea unui `updated` vechi după anulare (urca locurile la loc ȘI bloca omul la cumpărare cu 409), și checkout abandonat la 3DS (`max/999999` fără un leu plătit) |
| **C3** | Reparat **în SQL, la citire**, nu în webhook — altfel rândul uită ce a cumpărat omul și recuperarea depinde de o livrare viitoare pe exact endpoint-ul defect |
| **C4** | Garda anti-abonament-dublu întreabă Stripe (`subscriptions.list({customer, status:'all', limit:3})`) în loc să citească coloana locală, care se scrie abia de webhook |

## Politica de neplată — decisă de owner 2026-09-01

**Plafonul coboară când Stripe termină de reîncercat, nu când începe.**

    active, trialing, past_due                          → capacitatea cumpărată
    unpaid, incomplete, incomplete_expired, paused,
    canceled                                            → plafonul de Free (2)

`past_due` e intenționat acolo: un card expirat nu blochează instant pe cineva care
plătește de luni de zile, cât timp Stripe încearcă.

⚠️ **Coborârea plafonului NU scoate pe nimeni de pe listă** — `trainer_can_add_client`
oprește doar următoarea adăugare. Antrenorul cu 30 de clienți îi păstrează pe toți.
Ce transformă plafonul în constrângere reală e **zidul de alegere a pachetului**
(`trial-end-and-card-v1.md` §4, punctul 3.5 din lotul 3) — **nu e construit**.
C3 armează declanșatorul, zidul e cel care mușcă. Fără zid, C3 doar coboară un
număr pe care nimeni nu-l simte. Vezi și [[launch-readiness-plan]].

📌 Planul îl descrie ca „plasă de siguranță, nu drumul principal" (§3) — **fraza
aia nu mai e adevărată** după decizia de mai sus: acum fiecare neplatnic ajunge pe
zid, nu doar cine anulează deliberat. De rescris §3 și de urcat zidul în ordine.

## Trei liste de stări, deliberat diferite — nu le unifica

| stare | `LIVE_STATUSES` (webhook: poate lua rândul?) | `BLOCKING_STATUSES` (checkout: ar fi taxare dublă?) | SQL (dă capacitate?) |
|---|:--:|:--:|:--:|
| trialing / active / past_due | da | da | da |
| incomplete | nu | **da** | nu |
| unpaid | nu | **da** | nu |
| paused | nu | **nu** | nu |
| canceled / incomplete_expired | nu | nu | nu |

Recenzorul a verificat fiecare abatere: sunt cele intenționate, nicio scăpare.
`paused` nu e accesibil azi (ruta de checkout nu setează `payment_method_collection`,
deci implicit `always` — cardul se ia mereu).

## Ce e aplicat în prod vs scris

| | |
|---|---|
| ✅ aplicat 31 aug | `billing_free_has_no_trial.sql` — triggerul nu mai acordă trial; registrul se revendică în webhook |
| ✅ aplicat 31 aug | `billing_is_trialing_any_tier.sql` — `is_trialing` nu mai cere `plan_tier='free'` |
| ✅ aplicat 31 aug | `billing_convert_grandfathered.sql` — unealta de conversie. **Niciun cont convertit încă** |
| ✅ **aplicat ȘI VERIFICAT 1 sep** | `billing_lapsed_states_free_cap.sql` — C3. Dovedit pe date reale în prod: `past_due → 50`, `unpaid → 2`, `paused → 2`, `active → 50`; structura și privilegiile confirmate; cele 5 conturi scutite neatinse |

✅ **Codul e comis ȘI pushuit** — 3 sep 2026, ramura **`feat/billing-launch`**
(6 commit-uri noi peste cele 25 de pe `main` local; 31 în total pe GitHub, în
afara lui `origin/main`). Poarta a trecut la push: lint + type-check +
**827 de teste**. Nu mai există într-un singur loc.

🔑 **Regula care a deblocat asta, de reținut:** un push pe RAMURĂ nu atinge
producția — Vercel scoate doar un preview pe URL separat. Doar `git push origin
main` publică. Owner-ul amâna push-ul de teamă că publică o pagină „Facturare"
cu butoane care crapă; teama e validă doar pentru `main`.

🔴 **`origin/main` e tot la `7b092d8`.** Producția rulează cod dinaintea
facturării. Publicarea rămâne o decizie separată, de luat DUPĂ ce cheile Stripe
sunt în Vercel.

🟢 **Live mode e configurat de la 6 sep** — vezi secțiunea de mai jos. Cheile live
sunt în Vercel. Ce lipsește ca să încasezi bani nu mai e Stripe, ci **merge-ul în
`main`**.

## Teste

**582 în 31 de fișiere** (349 dimineața zilei de 1 sep). `src/app/**` n-avea niciun
test înainte; acum are harness PostgREST propriu (`postgrest-fake.ts`, cu teste pe
el însuși), plus suite pentru webhook, checkout, store, pagina de facturare și
`BillingNotice`.

Toate verificate **prin mutație** — fiecare defect pus la loc, pe rând, și
confirmat că suita pică. Două lecții care merită reținute:

1. **Prima suită a trecut verde peste C2b.** `it.each` varia *starea* peste patru
   valori dar ținea *forma rândului* fixă — coloana care conta n-a fost exersată
   niciodată. 101 teste verzi cu defectul între ele. Când scrii teste tabelare,
   întreabă-te care axă NU variază.
2. **Mock-ul de Stripe din testele de checkout n-avea `subscriptions`**, deci toate
   cele 40 treceau pe ramura de rezervă — verde din motivul greșit. Testele
   verifică acum și **cu ce argumente exacte** e chemat Stripe.

## Ce a rămas deschis

- ~~**V1**~~ ✅ reparat 1 sep — **trei** mesaje, împărțite după ce are omul de
  făcut: factură neplătită → plătește și locurile revin; plată nefinalizată →
  reia plata (nu datorează nimic, deci nu-l trimite să caute o factură
  inexistentă); abonament oprit/anulat → pornește altul. Toate spun că **clienții
  rămân**, niciunul nu mai pomenește arhivarea. Peretele obișnuit de pe Free e
  păzit cu `plan_tier !== 'free'`.
  ⚠️ Butonul duce cross-host spre dashboard, iar sesiunea Supabase e per-origine
  (`localStorage`) — deci de pe `app.veltofit.app` **omul trebuie să se
  autentifice încă o dată**. Nu e fundătură (`returnUrl` îl aduce înapoi), dar e
  un pas în plus. De evitat ar cere cookie partajat între subdomenii sau o pagină
  de facturare pe host-ul de app; niciuna nu există.
- ~~**V2**~~ ✅ reparat 1 sep — refuzul se curăță la o citire reușită. Proprietatea
  „supraviețuiește lui `openPortal`" e intactă (portalul nu recitește nimic).
  📌 Descoperit pe drum: `AddClientDrawer` reciteşte starea la **fiecare
  deschidere** și e montat în ambele layout-uri, deci un refuz de pe ecran se
  pierde mai ușor decât pare citind doar pagina de facturare.
- **A1** — nota de anulare arată `base_seat_limit` (15 pentru Plus) în loc de 2.
  C3 îl agravează: plafonul efectiv poate fi acum sub `base_seat_limit`.
- **N1–N8** — vezi fișierul de constatări. Cel mai apăsat: **N1**, antetul
  webhook-ului și `billing-operations-v1.md:207` se contrazic pe dacă Stripe
  reîncearcă un 400.
- **Testele manuale 10–12** din `launch-readiness-v1.md` §4 — semantica de zero
  rânduri a lui PostgREST (presupunerea de sub tot ce s-a reparat), anulare +
  reîncercare întârziată prin `stripe listen` + Resend, checkout abandonat la 3DS.

## Contul de test

`daniel.vinersar+test@yahoo.com` — abonament **Velto Max 30**, trial până pe
2026-09-30, cu **downgrade programat spre Plus pe aceeași dată**. Vezi
[[test3-seat-wall-results]] pentru starea clienților lui.

**Nedovedit încă:** momentul în care downgrade-ul programat chiar se aplică. E
singura ramură care coboară locurile pe drumul normal, și nu se poate observa azi.
Merită un abonament creat special, cu perioadă scurtă, înainte de live.

## Stripe — test mode (istoric, cum s-a ajuns aici)

Două produse (`Velto Plus`, `Velto Max`), 14 prețuri cu lookup key, **fără trial
pe niciun preț**. Portalul: anulare și downgrade la finalul perioadei, „End trials
on subscription updates" OPRIT, Tax ID bifat.

🔴 **Doar 4 prețuri sunt în lista de comutare a portalului**, și e o constrângere,
nu o scăpare: portalul acceptă un singur preț per recurență+monedă **per produs**.
Deci scara de capacitate cere **un produs pe treaptă** — șapte produse, nu două.
Motivul mecanic e în `billing-operations-v1.md` §D2. **Citește-l înainte de a crea
produsele în live mode.**

⚠️ Toată configurația din test se recreează în live. (La prima introducere, cinci
prețuri anuale au ieșit `2.999` în loc de `2.990` — ultimul zero devenit nouă, de
cinci ori la rând. Landing-ul derivă anualul din lunar, deci pagina ar fi promis
altceva decât încasa cardul.)

## 🟢 Stripe LIVE — configurat 6 sep 2026, complet

Făcut prin API cu cheia live restrânsă, nu de mână. **Nimic din ce urmează nu mai
trebuie refăcut.**

| | |
|---|---|
| **7 produse** | `Veltofit Plus 15/30/50`, `Veltofit Max 30/50/100/Nelimitat` — un produs pe treaptă, exact cum cerea constrângerea portalului de mai sus |
| **14 prețuri** | lunar + anual, fiecare cu lookup key `velto_<tier>_<seats>_<interval>`. Sumele verificate una câte una față de landing |
| `tax_behavior` | **`inclusive`** pe toate 14 — owner-ul e PFA **neplătitor de TVA** și facturează mereu prețul întreg. Dacă devine plătitor, ASTA e coloana care se schimbă |
| **Portalul** | anulare la finalul perioadei, downgrade la finalul perioadei, „End trials on subscription updates" OPRIT, Tax ID bifat |
| **Dunning** | 4 reîncercări în 3 săptămâni, apoi abonamentul se **anulează** (nu `unpaid`) |
| **Dispute** | rambursare **imediată**, ales de owner |
| **Cheile** | `STRIPE_SECRET_KEY` (live) + `STRIPE_WEBHOOK_SECRET` (live) puse în Vercel **direct de owner**, fără să treacă prin conversație |

✅ **Tranșat 22 sep:** lista de produse a portalului citește `products: 0` prin API, dar
dashboard-ul le arată pe toate 7, iar owner-ul confirmă că schimbarea pachetului merge.
Crede dashboard-ul, nu API-ul (vezi capcana de sus).

📌 Numele: pachetele s-au redenumit `Velto *` → `Veltofit *` în aceeași zi, în
Stripe și în cod. Ce NU se redenumește niciodată e în [[rename-veltofit]] —
citește-o înainte de orice altă redenumire.

## Cheile

Live: în Vercel (owner-ul le-a pus, nu au trecut prin conversație).
Test: `STRIPE_SECRET_KEY` și `STRIPE_WEBHOOK_SECRET` în `.env.local`.
Nu există cheie publishable, deliberat: sesiunile se creează pe server.
Local, `whsec` vine din `stripe listen` și se schimbă la fiecare rulare —
vezi [[stripe-webhooks-need-cli-locally]], se spune ÎNAINTE de orice test de plată.
