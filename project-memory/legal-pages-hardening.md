---
name: legal-pages-hardening
description: "2026-09-06 — cele trei pagini legale, aduse de la un set care descria un produs ce nu încasa bani, la 14 secțiuni cu anexă de prelucrare. Ce s-a decis, ce a rămas, și tiparul care le-a stricat de CINCI ori — a cincea oară pe footerul landing-ului, pe 8 septembrie."
metadata:
  type: project
---

Owner-ul a spus explicit că **nu vorbește cu un avocat deocamdată** și a cerut să
verificăm împreună. Ce s-a făcut e verificarea faptelor — dacă textul se potrivește
cu produsul și dacă bifează elementele cerute. Nu e opinie juridică, iar asta i s-a
spus o dată, clar.

## 🔴 Tiparul care le-a stricat, de CINCI ori

**Corectura se face acolo unde a fost observată, iar vecinii păstrează povestea
veche.** S-a întâmplat la: notele de sănătate (§3 reparat, §4 și §6 nu), procesatorul
de plăți, notele private, și la rescrierea finală a §3.

**A cincea, pe 8 septembrie: footerul landing-ului.** Sigla SOL fusese scoasă din
termeni pe 6 septembrie, dar `(marketing)/page.tsx` a trimis mai departe la
`ec.europa.eu/consumers/odr/` încă două zile — pe o pagină publică, live.
Reparat în `9366abc`, cu un comentariu în cod care spune de ce, ca a șasea oară
să nu mai fie. Ideea trăiește în TREI locuri, nu în două: termeni, politică,
footerul de pe landing.

Când schimbi o afirmație într-o pagină legală, **caută-o peste tot unde trăiește
ideea**, nu doar unde ai văzut-o. Și antetul de comentarii al fișierului contează:
cel din `privacy/page.tsx` devenise o a doua sursă de adevăr care nu mai era de
acord cu pagina de sub el.

## 🔑 Capcana de metodă, plătită pe 8 septembrie

Citirea landing-ului „ca text" s-a făcut extrăgând literalii de string din TSX.
**Asta sare peste `href`, `alt` și `aria-label` — adică peste fiecare link din
pagină**, exact clasa în care trăia blocantul. Nimic din text nu zicea „SOL";
doar atributele. Un agent care a deschis fișierul întreg (`content-reviewer`) a
găsit-o din prima.

Deci: verificarea unei pagini publice nu e completă dacă a citit doar textul
vizibil. Ori `grep` separat pe atribute, ori pune pe cineva să citească fișierul.

## Ce s-a găsit citind ca text, nu ca diff

- **Politica nega procesatorul de plăți** („Nu avem integrat niciun procesator")
  în timp ce Stripe era integrat. Ar fi devenit falsă exact la deploy.
- **Termenii nu pomeneau deloc că vinzi ceva** — nici preț, nici reînnoire, nici
  anulare, nici rambursare. Singura mențiune de bani era despre banii dintre
  antrenor și clientul lui.
- **Datele de facturare lipseau complet** din politică, deși checkout-ul
  colectează adresă și CUI.
- **§8 promitea ștergerea tuturor datelor** la cerere — ceea ce legea contabilă
  interzice pentru facturi.
- Termenii nu erau **linkuiți nicăieri la înregistrare**, deși spuneau „prin
  crearea contului îi accepți".

## 🔑 Trei lucruri de reținut, care nu se deduc

**Platforma europeană SOL e DESFIINȚATĂ.** Închisă pe 20 iulie 2025, cu datele
șterse; nu mai primea sesizări din 20 martie. Regulamentul 2024/3228 a abrogat
Regulamentul 524/2013. Adăugat din reflex și scos în aceeași zi. Concurentul
românesc (i-am-pt.com) încă îl are.

⚠️ **Formularea „comercianții sunt obligați să scoată linkul" era prea tare** —
corectată pe 8 septembrie, după ce owner-ul a întrebat de ce. Ce s-a abrogat e
**obligația de a-l afișa**; niciun articol nu cere ștergerea. Motivul real, și
suficient: linkul trimite clientul către un remediu care nu mai există, ceea ce
poate fi practică comercială înșelătoare. Când citezi asta, cită-l așa.

**Sigla ANPC nu pleacă odată cu ea.** SAL e sistemul ROMÂNESC, dintr-o altă
directivă, neatins de abrogare — obligațiile naționale de informare au rămas
peste tot în UE după închiderea platformei europene. Linkul ei trimite la
`anpc.ro/ce-este-sal/`, aceeași adresă ca din termeni, nu la pagina de start.

**Anexa de prelucrare NU cere contract semnat separat.** Forma electronică se
califică, iar o secțiune din termeni, acceptată odată cu ei, ține locul contractului.
E §13 acum. Document semnat separat trebuie doar când vinzi unei SĂLI, care are
juridic propriu.

**Contractul nu se face prin Stripe.** Owner-ul credea asta. Stripe procesează banii;
contractul se încheie la crearea contului, prin acceptarea termenilor — de-aia
factura o emite PFA-ul, cu numele legal pe ea.

## Ce fac concurenții (verificat, nu presupus)

- **Trainerize**: face împărțirea explicit — „the customer determines the purposes
  and means of processing. ABC acts as a processor". Scris pentru SĂLI, nu pentru
  antrenori individuali.
- **I AM PT** (PFA, Alba Iulia, același stadiu): **nu o face deloc**, se declară
  operator unic, zero mențiuni de art. 28. Exact poziția în care era Velto.

Concluzia dată owner-ului: nu ești mai expus decât concurența directă, dar
Trainerize arată unde ajungi dacă merge — iar prima sală care te întreabă îți cere
contractul ăla.

## Datele de sănătate — ÎNCHIS

Cele 4 rânduri rămase (2 note de sănătate, 2 note private) erau **toate demo sau
conturi de test ale owner-ului** — verificat înainte de ștergere. Șterse din
producție. §3 spune acum lucrul simplu: *nu prelucrăm date privind sănătatea*.

⚠️ Dacă `SHOW_CLIENT_NOTES` revine pe `true`, §3 trebuie rescrisă ÎNAINTE, iar
dovada consimțământului redevine blocantă. Vezi `docs/plans/privacy-health-notes-review.md`.

## Ce a rămas deschis

- **Operator vs împuternicit** — acoperit prin §13, dar alegerea rolului rămâne
  singurul punct unde un jurist chiar aduce ceva.
- **§8 promite 24 de luni de inactivitate** ca termen de retenție. **Nu există
  niciun mecanism care să-l aplice.** E o promisiune fără implementare.
- **Portalul Stripe în live** — termenii presupun că e configurat (anulare din
  aplicație, downgrade la finalul perioadei, trialul netaxat la schimbarea
  planului). Vezi [[stripe-integration-state]].
