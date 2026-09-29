---
name: import-clienti-etapa-0
description: "Importul de clienți a început 29 sep. Etapa 0 (Arhivat→Inactiv) e LIVRATĂ pe toate trei straturile, inclusiv producție. Plus reîncadrarea de priorități care a ales importul în locul raportului."
metadata: 
  node_type: memory
  type: project
  originSessionId: fdf31a5b-0e5d-471f-92a5-d45ad32eda3a
  modified: 2026-09-29T10:50:23.370Z
---

Owner-ul a ales pe **29 septembrie 2026** să construiască importul din Excel,
contra ordinii pe care o propusesem eu (raportul întâi). **Avea dreptate**, și
motivul pe care l-a dat („am discutat mult despre el") era mai slab decât cifra
care îl susține:

🔑 **Din 4 antrenori înscriși, 1 singur a adăugat vreodată un client.** E cea mai
îngustă strâmtoare din toată pâlnia. Raportul — grămada 1 din
`docs/plans/ce-mai-construim-v1.md` — contează doar pentru cei care au trecut de
intrare; trei din patru nu trec. Țintele de adopție își asumă 30% la pasul ăla;
măsurat e 25%. **Importul atacă exact numărul care blochează totul.**

Eu prioritizasem ce se vede în cod în loc de ce se vede în admin. De reținut ca
tipar, nu doar ca episod.

## Decizii luate pe 29 sep

- **Emailul e OPȚIONAL** la un contact importat (întrebarea 1 din specificație).
  Multe liste de antrenor au doar nume și telefon. Consecința: rândul n-are buton
  de invitație până nu se completează o adresă.
- **Domeniul: 0 + 1 + 2 + 3 + 6**, fără invitații. ≈11,5–15,5 zile.

⚠️ **Cifra de „9,5–12,5 zile" din §7 al specificației e prea mică** — exclude
etapa 0, pe care regulile de ordonare o fac obligatorie înaintea etapei 2.

## Etapa 0 — LIVRATĂ ȘI ÎN PRODUCȚIE (`9c3d5ec` + `1bc6cd7`)

Redenumirea „Arhivat" → „Inactiv" a atins **trei straturi**, iar inventarul din
§5.10 al specificației acoperea doar unul:

1. **`src/`** — 16 fișiere. Nouă forme gramaticale distincte: starea e
   „inactiv/inactivă/inactivi", acțiunea e „dezactivare/dezactivează". **Verbul
   NU e „inactivează".** Patru locuri nu erau în inventar (numerele de linie din
   23 sep alunecaseră — s-a căutat textul, nu linia).
2. **Paginile legale** — politică §5, termeni §7, prin `content-reviewer`. Datele
   „Ultima actualizare" mutate, fără email de notificare (paginile rezervă
   anunțul pentru schimbări semnificative, iar vocabularul nu e una).
3. **Baza de date** — 4 funcții, 8 mesaje, aplicate în producție.

**Ce NU s-a redenumit, deliberat:** `archive_client()`, `reactivate_client()`,
`ClientState.inactive`, `ClientStateFilter.archived`, `seat-wall-archive`, cele 27
de comentarii și `COMMENT ON`, și sufixul `' (arhivat '` pus pe numele unui
EXERCIȚIU orfan (alt obiect, alt sens). Se schimbă ce citește omul.

## 🔴 Capcana care era să lase afară calea mai frecventă

Lista de funcții SQL am construit-o dintr-un `grep | head -12` pe 16 rezultate.
**Am briefat dintr-o listă tăiată**, iar `reactivate_client` era sub tăietură — și
e calea MAI des umblată, fiindcă decizia 8 face din comutatorul Activ/Inactiv ceva
apăsat zilnic, iar refuzul la plafon vine tocmai pe „Activează".

A prins-o agentul care scria migrarea, comparând mesajul prin md5. **N-a lărgit
scopul singur** — a pus constatarea într-un rând de verificare marcat „DESCHIS".
Ăsta e comportamentul corect al unui subagent care găsește ceva în afara briefului.

**Rețeta, scrisă în antetul migrării:** numeri rădăcina GOALĂ (fără `head`, fără
tipar îngust) → clasifici în „mesaj către om" / „COMMENT" / „alt sens" → verifici
pe `pg_proc.prosrc`, nu pe fișiere. *Un tipar îngust întoarce puține linii ȘI PARE
COMPLET.*

Verificarea care ar fi prins-o din prima e acum un rând standard: întreabă baza
dacă a mai rămas vreun șir vechi ÎN ORICE funcție, nu doar în cele atinse.

## Reparat pe drum, nu era redenumire

**„și e reversibilă", necalificat, în Termeni §7.** Dezactivarea e gratuită,
**reactivarea NU** — `reactivate_client()` verifică plafonul și refuză când e plin
(Garda 4). Iar paragraful descria exact cazul în care plafonul E plin: ai coborât
pe Free și ai dezactivat până la limită. Promisiunea era necalificată fix acolo
unde e cel mai probabil să nu se poată ține. Inexactitate moștenită, reparată
fiindcă fraza era oricum atinsă.

## Ce urmează

Etapa 1 (fundația `client_contacts`, 2–3 zile, **neutră** — cu tabela goală
numărătoarea de locuri e identică), apoi 2, 3 și 6. Etapa 6 pleacă **odată** cu 3,
nu după.

⚠️ **Rămâne la owner:** limita de trimitere a emailurilor (Supabase → Auth → Rate
Limits, plus contul Resend). Nu blochează 1–3, blochează etapa 5.

Legat: [[what-justifies-the-money-2026-09-26]], [[adoption-targets-2026-2027]],
[[db-two-databases-migration-order]], [[trainer-accounts-who-is-who]].
