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

**30 sep, `ee5aae5` + `9d1f651`: cardurile din Cheltuieli.** Primul card = „Cost total" = plătit până azi:
abonamente lună de lună de la `started_on` (altfel ziua `created_at`, București) până la `cancelled_on`,
+ plăți o dată active + campanii pornite. Owner-ul NU vrea să completeze date de start („data nu e
importantă") — de-asta fallback-ul pe created_at. Data anulării e acum obligatorie. Cardul
„Plăți o dată · <luna>" a fost scos la cererea lui.
Owner-ul spune „fă direct commit și push" pentru munca din admin — nu-l mai întreba la fiecare pas mic.
