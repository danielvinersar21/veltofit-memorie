---
name: test3-seat-wall-results
description: "Peretele de locuri, fluxul „în așteptare\" și refuzurile de la adăugare manuală — rulate în PRODUCȚIE 2026-08-30/31, toate TRECUTE. Ce s-a reparat pe urma lor, ce a rămas deschis, și starea contului de test (scos de pe trial, deliberat)."
metadata: 
  node_type: memory
  type: project
  originSessionId: 671c6857-3813-43fe-928d-27deea8601cd
  modified: 2026-08-30T20:40:29.892Z
---

## Stare curentă a contului de test (2026-08-31)

`daniel.vinersar+test@yahoo.com`, antrenor, `billing_exempt = false`.
**Deliberat SCOS DE PE TRIAL** — `subscription_status = NULL`,
`trial_ends_at = NULL`, `plan_tier = 'free'`, `seat_limit = 2`. Nu-l pune la loc
pe trial fără motiv: așa arată exact un cont Free din modelul nou, și e singura
stare în care se poate verifica randarea la plafon.

Dacă vreodată e nevoie de stările bannerului de trial:
`set subscription_status = 'trialing', trial_ends_at = now() + interval '3 days'`.

Clienții pe cont: Iunia + `+testc5@yahoo.ro` **activi** (plafonul e plin),
`+c1@yahoo.ro` **arhivat**, Ilie `+c2@yahoo.ro` **invitat** (pending fără
consimțământ), `+c3@yahoo.ro` — **legătura ȘTEARSĂ** în testul de refuz pentru
cont existent; contul există, fără legătură cu antrenorul.

`+testc5@yahoo.ro` e un cont REAL creat în producție prin linkul reutilizabil.

## Ce a dovedit testul — TRECUT, tot fluxul

Tot ce a fost construit pe 2026-08-29 și nu fusese văzut niciodată funcționând:

- revendicarea linkului la plafon **nu respinge pe nimeni** — aterizează `pending`
  + `consented_at` completat, iar `trainer_active_client_count` a rămas 2, deci
  **nu consumă loc**;
- clientul vede „Antrenorul tău te activează în curând", nu o fundătură;
- antrenorul vede „În așteptare", butonul Activează dezactivat, textul „Nu ai
  locuri libere";
- arhivare → se eliberează locul → Activează se deblochează → activarea trece;
- clientul primește notificarea de activare.

Distincția `pending` + `consented_at` NULL = „Invitat" vs `pending` +
`consented_at` completat = „În așteptare" funcționează corect pe date reale.

## Patru constatări

**1. ✅ REPARAT 2026-08-31 (`b69e013`). Drawerul „Adaugă client" ascundea linkul
reutilizabil la plafon, pe un motiv care nu mai era valabil de 8 zile.** Comentariul din `add-client-drawer`
(`f8c8886`, 2026-08-22 18:59) zice „a da un link care îl duce peste plafon e mai
rău decât să nu-l arăți". Șapte minute mai târziu, `b764278` a adăugat
`accept_invite_seat_cap.sql`, care face revendicarea să aterizeze în așteptare —
aplicat în prod 2026-08-29. Testul a demonstrat empiric că plafonul nu se
depășește. `ReusableLinkCard` apare într-un **singur** loc în toată aplicația,
deci la plafon antrenorul nu poate ajunge la propriul link, iar în „așteptare"
mai pot ajunge doar cei care aveau linkul dinainte.

**2. 🔴 Notificarea către antrenor la revendicare nu se scrie NICIODATĂ pe calea
cu link.** Detaliile complete în [[notifications-never-worked]] — mecanism nou,
al patrulea defect.

**3. ⚠️ „Vezi planurile" se învârte în cerc.** Bara de plafon → `veltofit.app/#preturi`
→ orice card de preț → `dashboard.veltofit.app/login?mode=signup` → pagina de
login vede sesiune activă → înapoi în dashboard. Zero plată, niciun mesaj.
Confirmat pe producție. Devine mai grav pe modelul nou (Free = 2 din ziua 1, deci
peretele se atinge la al treilea client, nu în ziua 31). E lotul 3 din
`trial-end-and-card-v1.md` §9.

**4. ⚠️ Testul 4 din `launch-readiness-v1.md` nu e rulabil așa cum e scris.** Cere
să tastezi adrese „când n-ai locuri", dar la plafon formularul nu e randat deloc.
`SEAT_CAP_REFUSAL` e practic inaccesibil din interfață — apărare pe server,
acoperită doar de teste unitare. Testul trebuie rescris: refuzurile pe stări se
rulează **cu loc liber**.

## Decizia din spatele reparației ✅ CONSTRUITĂ 2026-08-31

În drawer, la plafon: **formularul manual rămâne ascuns** (serverul chiar refuză,
nimic bun nu poate ieși), dar **cardul de link revine**, cu o propoziție de genul
„Poți trimite linkul și acum. Cine intră nu e respins — apare la tine ca «În
așteptare» și îl activezi când eliberezi un loc."
Un singur fișier, plus o linie de copy. Risc asumat: se poate aduna o coadă de
oameni care așteaptă la nesfârșit dacă antrenorul nu eliberează niciodată un loc.

## Ce s-a bifat pe 2026-08-31

- **Runda de refuzuri la adăugare manuală — TRECUT.** Toate șase mesajele, cu
  loc liber. Inclusiv cel netestat vreodată: refuzul pentru un email cu cont
  Velto existent, atins pe drumul ocolit din §D2 (ștergi legătura, retastezi
  adresa) — **zero legături create**, verificat în bază. Și lista de permise
  `isLinkableRole` a refuzat corect un cont de admin.
- **Modificarea din drawer — CONSTRUITĂ ȘI COMISĂ** (`b69e013`). Cardul de link
  revine la plafon cu o propoziție explicativă deasupra; formularul manual
  rămâne ascuns. Pe lângă: butonul de copiere din refuz a fost scos la cererea
  owner-ului (textul spunea „link-ul de mai sus", iar cardul chiar e mai sus),
  împreună cu tot plumbingul lui — `reusableInviteToken()` din rută,
  `inviteToken` din răspuns, `inviteUrl` din tipul refuzului.
- **Două defecte de contrast reparate în același panou**: bordura era
  `--error-20` (invizibilă peste propria tentă) și textul `--error` pe
  `--error-10` (~3.4:1, sub AA pentru 13px, interzis explicit în
  `design-tokens.md`).
- **Constatările scrise în `launch-readiness-v1.md`**: testul 3 marcat trecut,
  testul 4 rescris (nu era rulabil — cerea tastare la plafon, unde formularul nu
  există), §B7 precizat cu bucla verificată, §E cu notificarea care nu pleacă.

## De unde continuăm

**Lotul 1 din `trial-end-and-card-v1.md` §9 — Stripe + baza.** Împărțit:
owner creează produsele și prețurile în Stripe după specificația din
`pricing-monetization-v1.md`; în cod se scriu migrarea care oprește triggerul din
a mai acorda trial (§B3 din launch-readiness) și webhook-ul care scrie
`seat_limit` din Price-ul cumpărat.

Rămase din testele manuale, independente: testul 9 (15 pagini de profil,
20 min), 6 (telefon real) și 7 (tehnicile de serii în sală).

## Documente deja actualizate în sesiunea asta

- `trial-end-and-card-v1.md` — §7.5 (trialul se face în Stripe + gaura din
  triggerul de provisioning), §7.6 (funcțiile de roadmap: Free vs plătit),
  §9 (lista completă de făcut, pe 3 loturi)
- `pricing-monetization-v1.md` — principiile 2 și 4 rescrise
- `launch-readiness-v1.md` — B1 marcat rezolvat, B3 nou (triggerul)

Vezi și [[backlog-launch-blockers]], [[launch-readiness-plan]],
[[billing-phase1-schema]], [[backlog-trainer-writes-silent-noop]].
