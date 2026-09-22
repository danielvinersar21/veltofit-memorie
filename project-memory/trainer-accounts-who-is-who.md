---
name: trainer-accounts-who-is-who
description: "Cine e fiecare cont de antrenor din baza de producție — prieteni invitați personal, înscrieri organice, conturi de test ale owner-ului"
metadata: 
  node_type: memory
  type: project
  originSessionId: f6937a95-9895-4af4-a554-c0ce612b3895
  modified: 2026-09-20T08:39:14.173Z
---

Verificat în baza live pe 2026-09-20. Owner-ul a lămurit asta de cel puțin două
ori înainte și de fiecare dată s-a pierdut, fiindcă nimeni n-a scris-o.
**Nu re-deriva apartenența conturilor din emailuri — e aici.**

## Prieteni, invitați personal de owner (NU sunt organici)

- **Bogdan Sularea** (`bogdansularea0@gmail.com`, cont din 29 iul 2026)
- **Radu Pădurean** (`radupadurean01@gmail.com`, cont din 29 iul 2026, 2 clienți activi)

Owner-ul a vorbit cu ei personal și le-a zis să-și facă cont ca să vadă cum e.
De asta sunt `billing_exempt = true` — au acces gratuit pe termen nelimitat,
deliberat. **Nu-i număra niciodată ca tracțiune organică** și nu trage concluzii
despre retenție din faptul că n-au mai revenit.

## Înscrieri organice (oameni care au venit singuri)

- **Crăciun Emanuel Alexandru** (`craciunemanuel25@gmail.com`, 10 sep 2026)
- **Dumitru Andrei** (`avdn18@yahoo.com`, 14 sep 2026)

Ăștia doi sunt singurii veniți din afară. Sunt pe **Free** — 2 locuri, fără card,
**fără trial, și asta e prin design**, nu o defecțiune: vezi
[[free-has-no-trial]]. Nu le expiră nimic; nu există nicio prăpastie în
octombrie. Concluzia asta a fost trasă deja săptămâna trecută și s-a pierdut —
un agent a re-derivat-o greșit pe 20 sep și a raportat un termen fals.

## Familie

- **Iunia** (`iuniavinersar@gmail.com`, 2 sep 2026) — nu e organică.

## Conturile owner-ului (test)

`daniel.vinersar+2@yahoo.com`, `daniel.vinersar@yahoo.ro`,
`daniel.vinersar+test@yahoo.com` (ăsta e plus/active cu card — **el se taxează
cu 99 RON pe 8 oct**), `daniel.vinersar.ux+antrenor1@gmail.com` (canceled),
`daniel.vinersar.ux+testtrainer@gmail.com`.

## Notă despre `trial_expira` pe conturile scutite

Cele 4 conturi scutite din valul vechi (grandfathered) au `trial_ends_at` =
2026-09-20 în bază. E date
moartă: `trainer_seat_limit()` verifică `billing_exempt` **înaintea** ramurii de
trial și întoarce 2147483647. Nu te speria de data aia.

## Contul de control al owner-ului

Pe **20 sep 2026** `daniel.vinersar.ux+testtrainer@gmail.com` (numit „dasdsa") a
fost trecut pe `billing_exempt = true` — deliberat, ca owner-ul să aibă un cont
în exact același regim ca Bogdan și Radu și să vadă ce văd ei pe ecranul de
facturare. **Nu e o anomalie, nu-l „repara".** Se dă înapoi cu un `update ... set
billing_exempt = false`.

## ⚠️ Regula când dai acces gratuit cuiva care deja plătea

Punerea steagului `billing_exempt = true` **NU anulează abonamentul din Stripe**.
Nimic nu-l anulează automat — webhook-ul nu atinge niciodată `billing_exempt`
(deliberat, vezi antetul din `src/app/api/billing/webhook/route.ts`), iar
Checkout refuză un cont scutit cu 400. Deci singurul drum către „scutit + abonament
viu" e un UPDATE manual făcut de owner, exact în direcția asta.

**Deci: anulează-i întâi abonamentul în Stripe, apoi pune steagul.** Altfel omul
e debitat în continuare în timp ce aplicația îi spune că are acces gratuit.

Din 20 sep 2026 ecranul de facturare nu mai minte în cazul ăsta și îi păstrează
butonul spre portal — dar ecranul doar nu mai înrăutățește situația; banii tot
pleacă până anulezi abonamentul.
