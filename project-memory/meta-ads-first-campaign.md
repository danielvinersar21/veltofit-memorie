---
name: meta-ads-first-campaign
description: Prima reclamă plătită Veltofit (Meta Ads Manager), configurată cu owner-ul pe 2026-09-21 — contul, setările alese și de ce, și ce se verifică după 3 zile.
metadata:
  type: project
---

Pe 2026-09-21, owner-ul a configurat împreună cu mine prima reclamă Meta. Pachetul
complet e în `~/Veltofit/Social/ads/2026-09-21-prezentare/pachet.md`.

**Conturile, cum au rămas:**
- Pagina de Facebook „Veltofit.app”, creată din contul personal real al owner-ului și legată de @veltofit.app.
- Portofoliul redenumit „Velto App” → „Veltofit”. Admini: owner-ul și contul de Instagram.
- Contul de reclame „Veltofit”, în RON, Europe/Bucharest. Facturare pe VINERSAR ILIE-DANIEL PFA, cu codul de TVA, și cardul adăugat.
- ⚠ Owner-ul a creat din greșeală și un **cont personal de Facebook „Veltofit App”**, prin Accounts Centre → Add accounts. Nu se folosește la nimic. Rămâne de șters sau de scos din Accounts Centre al Instagramului, iar decizia e a lui.

**Setările campaniei:**
- Obiectiv Trafic, landing page views, pentru că Meta nu mai cere pixel pentru ele.
- 40 lei pe zi. **Publicată pe 21 sep, seara, cu pornire pe 22 sep la 08:00**, până pe 28 sep.
- Advantage+ audience, cu trei titluri de job de antrenor ca sugestie. Strict, publicul avea sub 1.000 de oameni.
- Doar Instagram Reels și Stories, pentru că videoul e 9:16.
- Toate îmbunătățirile AI oprite, multi-advertiser oprit. Ultimele trei le-am găsit abia la „Review defaults” („relevant comments”, „show spotlights”, „video effects”), deci verifică-l mereu înainte de Publish.
- Titlul „30 de zile gratuite sau rămâi pe Free”. Varianta „cu până la 2 clienți” i s-a părut owner-ului „mică, respingătoare”.
- Linkul cu `utm_source=meta&utm_medium=paid&utm_campaign=prezentare-sep`. Meta a mutat etichetele în Tracking → URL parameters.

**✅ VERIFICAT pe 24 sep, dimineața, cu owner-ul, din capturi.** Reclama se termină
pe **29 sep** (nu 28, cum scria aici). Rezultatele, 22–24 sep:

| | Reclama (prezentare-sep) | Boost (cine-sunt) |
|---|---|---|
| Cheltuit | 77,21 lei | 87,89 lei din 150 |
| Vizite pe site | 38 | 195 |
| **Cost pe vizită** | **2,03 lei** | **0,45 lei** |
| Reach / afișări | 3.621 / 4.635 | 6.370 / 8.523 |
| Altceva | — | 100 vizite profil, 18 atingeri bio, 2 urmăritori, 5 distribuiri |

Ambele numără la fel: Instagram a schimbat „website visits" să însemne click care a
dus la încărcarea efectivă a paginii, adică exact „landing page views" din Ads Manager.
Meta LIVREAZĂ bugetul (96%), deci ipoteza „publicul e prea strict, lărgim" **NU se aplică** —
nu o re-propune. Frecvența 1,28, deci materialul nu e obosit.

**Cele 4 înscrieri din 22–24 sep, pe surse (din cardul Proveniență, pagina fiecăruia):**
- **Reclama** (`meta/paid/prezentare-sep`): 1 — Andrei Diaconu, Constanța. Inert.
- **Boost** (`instagram/boost/cine-sunt`): 1 — Dumitru Amariei, București. S-a întors a doua zi.
- **Linkul din bio** (`ig/social`, fără campanie): 2 — Maxim (Pitești, inert) și
  Ghimpeteanu Dragos (București), **singurul cu un client adăugat**.

**Capcana de atribuire, REZOLVATĂ — owner-ul a avut dreptate, eu hedge-uisem degeaba:**
boost-ul produce vizite pe PROFIL (100) și atingeri pe linkul din bio (18). Omul care vine
prin profil → bio ajunge etichetat `ig/social`, identic cu organicul, deci **boost-ul își
cedează singur meritul**.

Că cei 2 `ig/social` vin tot din boost **se dovedește**, nu se presupune: graficul Vercel
arată 16–20 sep cu **3–4 vizitatori pe zi**, deși linkul din bio exista, caruselul era
publicat din 15 sep și cele 3 reels-uri erau sus. Saltul începe fix în ziua boost-ului.
**16–20 sep e grupul de control.**

Deci atribuirea corectă: **reclama 1 înscriere la 77 lei, boost-ul 3 înscrieri la 29 lei.**
Boost-ul câștigă și la vizite (4,5×) și la înscrieri (2,6×). Total: **165 lei / 4 antrenori
= 41 lei bucata.**

**Concluzia strategică, mai importantă decât clasamentul:** conținutul organic nu are
distribuție deloc — 3 reels + 1 carusel publicate au produs 3 vizitatori pe zi. Nu sunt
slabe (cine le vede se înscrie), dar fără urmăritori Instagram nu le arată nimănui.
**Plata nu e un test de trecut/picat, e deocamdată SINGURUL canal de distribuție.**
Întrebarea nu e „merită", ci „cât pe lună, constant". La 41 lei/antrenor, 300 lei/lună ≈ 7
antrenori. De-asta cei 2 urmăritori noi contează mai mult decât par: sunt singurul lucru
care, adunat, scade dependența de plată.

**Recomandarea dată owner-ului pe 24 sep:** oprește reclama, mută banii pe boost. Nu pe
baza înscrierilor (1 vs 1 nu decide nimic), ci pe 38 vs 195 de vizite la aceiași bani —
și pentru că boost-ul lasă ceva în urmă (urmăritori, profil, bio care aduce și după ce se
termină), iar reclama nu lasă nimic. Rezerva: cele două diferă prin material, loc ȘI
public simultan; pariul e pe material (caruselul „Cine sunt" bate demonstrația de produs),
dar nu se poate dovedi din datele astea.

Toți 4 sunt pe **Free**, niciunul n-a ajuns la Stripe. Cei doi care s-au întors s-au
înscris DUPĂ ce au pornit emailurile MailerLite (23 sep seara); cei doi inerți, înainte.
La 4 oameni e coincidență la fel de probabilă ca o cauză — **de reverificat peste o
săptămână, cu mai mulți**. Vezi [[onboarding-emails-mailerlite]].

**Încă de făcut:** verificarea identității la „Ad transparency", ca reclamele să nu fie
oprite când devine obligatorie.

Owner-ul a acceptat conștient ca sigla din colț să se suprapună cu numele contului în varianta de reclamă. Vezi [[instagram-reels-animate]].

**În paralel, un boost din Instagram:** owner-ul a promovat pe 21 sep postarea „cu
prezentarea mea”, adică, cel mai probabil, caruselul „Cine sunt”, până vineri,
25 sep. În prima zi a adus **61 de vizitatori** pe site, față de aproape zero
înainte (Vercel Analytics: instagram.com 44, l.instagram.com 21). Boost-ul și
reclama se suprapun pe 22–25 sep. **Cum le deosebești:** boost-ul vine fără
etichete (referrer instagram.com), iar reclama vine cu `utm_source=meta`. După
vineri rămâne doar reclama.
