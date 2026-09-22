---
name: rename-veltofit
description: "2026-09-06 — produsul se numește Veltofit, nu Velto. 202 înlocuiri în cod plus 86 în skill-uri, plus produsele din Stripe. Lista cu ce NU se redenumește niciodată, fiindcare fiecare intrare de acolo rupe altceva."
metadata:
  type: project
---

Owner-ul a decis pe 2026-09-06 că produsul se numește **Veltofit** — a schimbat și
logoul. Redenumirea e făcută: 202 înlocuiri în `src/`, 86 în `.claude/`, cele 7
produse din Stripe live, și `PRODUCT_NAME` în `billing-copy.ts`.

## 🔴 Ce NU se redenumește, și ce se rupe dacă o faci

Toate sunt cu literă mică, deci un tipar pe `\bVelto\b` nu le atinge — dar cine
face un `sed` orb pe „velto" le distruge:

| Ce | Unde | Ce se rupe |
|---|---|---|
| prefixul `velto_` din cheile de preț | `price-keys.ts:76` | **plățile**. E contractul cu Stripe: codul cere prețul după `lookup_key`, nu după nume |
| `velto-theme` | `use-theme.ts:7` + scriptul din `layout.tsx` | tema fiecărui utilizator, resetată la următoarea vizită |
| `velto:activeWorkout` | `use-active-session.ts` | **antrenamentul în curs din telefonul oricui e în sală** la momentul deploy-ului |
| `velto_pending_plan` | `pending-plan.ts` | pachetul ales pe landing, pierdut pe drumul spre plată |
| numele fișierelor `velto-small-logo-*.png` | `layout.tsx`, `manifest.json` | 404 la iconuri |
| `name: velto` / `name: velto-social` | frontmatter-ul skill-urilor | skill-urile nu se mai pot invoca; simptomul arată ca și cum n-ar exista |
| `VINERSAR ILIE-DANIEL PFA` | `terms`, `privacy`, `(marketing)` | e **entitatea legală**, numele de pe facturi. Nu e numele produsului |
| `@velto.mock` | seed + `admin_dashboard.sql:140` filtrează după el | perechea trebuie să rămână sincronizată |

## Ce s-a păstrat deliberat cu numele vechi

Trei linii care **citează** ce scria pe ecran la o dată anume („it said 'Velto
Free'"). Un citat editat nu mai e citat. Regula folosită peste tot: **proza se
redenumește, citatele nu.**

## Unde trăiește numele acum

`PRODUCT_NAME` + `planLabel(tier)` în `billing-copy.ts`. Înainte era scris de mână
în patru locuri — antetul cardului de plan, ambele opțiuni din selector, zidul de
locuri. Următoarea redenumire e o linie.

⚠️ **Trebuie să corespundă cu numele produselor din Stripe**, pentru că ăla ajunge
pe FACTURĂ. O factură care spune alt nume decât aplicația e nepotrivire pe un
document despre bani — una dintre cauzele documentate de contestații.

## Capcana care a scăpat de prima dată

Sedul a lovit și `$HOME/Velto/Social/assets` din scripturile de randare social,
lăsându-le să arate spre un folder inexistent. **Eșecul ar fi fost tăcut**:
scripturile cad pe `public/images` când folderul lipsește, deci un carusel s-ar fi
construit din alte capturi, fără eroare. Folderul a fost redenumit în
`~/Veltofit/Social` și lanțul rulat cap-coadă. Vezi [[instagram-reels-animate]].

Și a lovit propoziția care documenta redenumirea: „la redenumirea Veltofit →
Veltofit". Verifică întotdeauna comentariile care descriu chiar schimbarea.
