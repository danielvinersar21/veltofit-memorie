---
name: admin-campaigns-page
description: "Pagina de admin „Campanii” (LIVE 30 sep, e9f7c12+bf907d7) — jurnalul boost-urilor/reclamelor; aici se trece fiecare rundă plătită, nu în documente."
metadata:
  node_type: memory
  type: project
  originSessionId: ef45e45e-f2d8-47b2-8021-9dd826882a68
  modified: 2026-09-30T08:43:40.872Z
---

Cerută de owner pe 2026-09-30: „o pagină dedicată pentru ad-uri/boost, ce și când am avut, ce rezultate”.
Înlocuiește documentul Claude Doc din aceeași zi (rămas doar arhivă).

- Tabel `admin_campaigns` (RLS ca admin_expenses), aplicat în prod 30 sep; seed: reclama reel 22–29 sep,
  boost „Cine sunt” 21 sep–1 oct, boost reel planificat 2 oct.
- Owner-ul a ales MIXT: cifrele Meta de mână; automat pe fereastra campaniei: contorul landing
  (doar din 29 sep, marcat „parțial”), conturile noi din `admin_trainer_overview` (fără demo), cost/vizită.
- Carduri sus: cheltuit, vizite Meta, conturi noi UNICE pe reuniunea ferestrelor, cost pe cont nou.
  În prod arată 4 conturi — CONFIRMAT de owner pe 30 sep.
- Capcane: două campanii în aceleași zile → aceleași conturi la amândouă (avertisment pe rând);
  vizitele din Instagram ajung des „fără sursă” (arătate separat).
- Analiza din „Ce am învățat” a campaniilor din septembrie = cea din [[meta-ads-first-campaign]].

**Cum aplici:** când owner-ul aduce cifre noi de la o campanie, le trece EL în pagină (sau îl ghidezi);
nu mai face documente/capturi pentru comparații.

**30 sep, `079f928`: legată de „Cheltuieli".** Campaniile neplanificate apar în grupul Marketing ca plăți
o dată, doar de citit, pe luna de start (ora București). „Campanii" = SINGURUL loc pentru banii de reclame.
Owner-ul a șters din prod rândul manual „Meta Ads — campania din septembrie" (165,10) și a scos cei 300 lei
de pe rândul „Instagram @veltofit.app" (înapoi pe gratuit), ca să nu se dubleze. Nu mai trece bani de
reclame în Cheltuieli.
