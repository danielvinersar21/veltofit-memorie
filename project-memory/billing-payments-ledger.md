---
name: billing-payments-ledger
description: "Jurnalul de încasări e VIU în producție din 2026-09-17: migrare aplicată, cod pe main, `invoice.paid` bifat pe endpoint-ul LIVE Stripe. Ecranul /dashboard/admin/payments e lista săptămânală de facturare din SOLO. Configurația Stripe nu există în repo — e doar aici."
metadata:
  node_type: memory
  type: project
---

Consecința deciziei din [[launch-gtm-decision]] că factura o emite PFA-ul din
SOLO, nu Stripe: pentru fiecare încasare cineva trebuie să emită manual un
document în **15 zile calendaristice**, termen scris în Termeni. Până acum nu
exista nicio evidență — se căuta de mână prin Stripe → Payments.

**Ce e livrat, 2026-09-17, tot lanțul:**

| verigă | stare |
|---|---|
| `docs/database/billing_payments_ledger.sql` | ✅ aplicat în prod, verificat în 3 probe |
| cod (webhook + ecran + 4 straturi) | ✅ `main`, commit `42c6a9e`, urcat |
| `invoice.paid` pe endpoint-ul **LIVE** Stripe | ✅ bifat + salvat, confirmat de owner |
| deploy Vercel | ✅ verde |

🔑 **Bifa din Stripe nu există nicăieri în repo.** Endpoint-ul live ascultă acum
PATRU evenimente, nu trei: cele trei de abonament plus `invoice.paid`. Dacă
vreodată cineva reconfigurează endpoint-ul după `billing-operations-v1.md` §D,
verifică să nu fi rămas la trei — tabelul n-ar primi niciun rând, iar ecranul ar
scrie „nu ai nimic de facturat" în timp ce banii intră.

**De ce `invoice.paid` și nu `invoice.payment_succeeded`:** al doilea e strict mai
îngust — o factură achitată fără încercare de plată (transfer, marcată manual) nu
îl declanșează. Descrierea Stripe însăși zice „…or an invoice is marked as paid
out-of-band". Argumentul complet e în `webhook/route.ts`.

**Ce s-a dovedit, nu s-a presupus:** un antrenor obișnuit nu poate citi tabelul
nici direct, nici prin RPC, **nici cu `profiles.role='admin'` pus pe propriul
profil** — poarta e `admin_users`. Relivrarea Stripe nu dublează rândul, factura
de 0 lei e respinsă de bază, iar prima zi de întârziere se vede ca `-1`, nu `0`.

**Adaptive Pricing NU atinge facturarea.** O plată făcută din Croația a arătat
19,42 EUR clientului și a intrat în jurnal ca **99,00 RON** — moneda locală e doar
vitrina, factura rămâne în moneda de integrare. Nu e nimic de convertit în SOLO.

**Rămase deschise, mici, niciuna blocantă:**
- indexul parțial e `WHERE invoiced_at IS NULL`, dar lista cere și
  `emailed_at IS NULL` — un rând emis-dar-netrimis iese din index. Cere propria
  migrare; nu modifica fișierul aplicat.
- numele accesibile se repetă identic pe fiecare rând (cititor de ecran aude un
  șir de butoane cu același nume)
- cele trei tente ale benzii de termen au aceeași deschidere — la scanare rapidă
  nu se disting. Regula „nu doar culoarea" **e** respectată (banda scrie textul).

**Proba finală — MUTATĂ (2026-09-22):** owner-ul își anulează abonamentul de test („nu are
sens să fac o factură manuală către mine"), deci pe 8 octombrie nu mai vine nicio încasare.
Proba se face la **prima încasare de la un antrenor real** — alerta „PRIMA, emite factura în
SOLO" o anunță ([[owner-alerts]]). Pașii rămân: deschide
Stripe → Payments, numără plățile cu sumă > 0 din săptămână, compară cu
`SELECT count(*) FROM public.admin_payments_list(false, true)`. Verifică și
SUMELE, nu doar numărul de rânduri — o factură cu pro-rată nu e prețul de listă.
