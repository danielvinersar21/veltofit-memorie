---
name: competitor-teardown-2026-09
description: "Third RO competitor found 2026-09-02 (i-am-pt.com, Alba Iulia, a solo PFA with ZERO users, declared on his own site) plus a Trainerize teardown. Full steal-list in docs/plans/competitor-teardown-v1.md."
metadata: 
  node_type: memory
  type: project
  originSessionId: 0ef13842-cdd0-40af-8f7a-58db72902ca0
  modified: 2026-09-23T07:19:46.828Z
---

Two competitor looks on 2026-09-02, written up in
`docs/plans/competitor-teardown-v1.md`. **Read the doc for the steal-list; this
note holds only what the doc can't say about itself.**

**i-am-pt.com — the third RO competitor, and the closest one physically.**
Scrob Radu Iosif PFA, **Alba Iulia — the owner's own city**. Domain registered
2026-03-12, Laravel/Tailwind/Alpine on Hetzner. Free, 60-day trial no card,
Stripe named in the terms but not wired.

The finding that changes how to read him: his landing's own stats section shows
**0+ active trainers, 0+ appointments, 0+ partner gyms**, labelled "the numbers
above represent the goals we've set for ourselves". He is at the same stage as
Velto — built, unlaunched, no users. Not ahead. Don't react to his feature list.

He built the **admin** of a PT business (bookings, credits, QR check-in,
nutrition). His training plan is a table the client reads — **no set logging
anywhere in his full public guide**. Velto's core is the part he doesn't have.

**The colour "coincidence" is closed — no action.** Measured: his `#3A7AFE` /
`#3DDC97` vs Velto's `#437AB1` / `#40B58A`. Greens 4° apart, blues 10°. But his
values are essentially default Tailwind `blue-500` + `emerald-400` — where anyone
who doesn't choose deliberately lands. His read electric, ours muted, and our
landing is light by default now. They don't confuse. Don't re-litigate this.

**Trainerize:** trial account `danielvin.trainerize.com`, explored 2026-09-02.
Caveat that limits every claim: the test client had **zero logged workouts**, so
what a completed workout looks like to their trainer was never seen. The top
three steals (client-list lenses, three last-touch timestamps, compliance dot
grid) are in the doc §4.1 with a proposed order in §5. None is launch-blocking.

Related: [[launch-gtm-decision]] (the other two RO competitors),
[[notifications-never-worked]] (why the passive last-seen signal matters more
than another notification channel), [[roadmap-features-v1]] via
`docs/plans/roadmap-features-v1.md` (calendar was already vision; i-am-pt shipped it).

**2026-09-22: al patrulea concurent, My PT Hub** (UK, grupul EverCommerce, clienți nelimitați, funcții pe care Veltofit nu le are). Îl folosește un antrenor abordat în DM, deci unii antrenori români plătesc deja pentru așa ceva. Nu ne comparăm pe funcții. Detalii în `docs/plans/competitor-teardown-v1.md` §7, **VERIFICATE pe site-ul lor tot pe 22 sep**. **Prețul real, afișat pentru România:** Starter 25 € (3 clienți), Premium **59 €/lună** fără angajament sau 52 € pe 12 luni, Ultimate 215 €. Cifra de „~105 $” scrisă inițial venea de pe blogul unui concurent de-al lor (quickcoach.fit), deci nu o mai folosi. **La 50 de clienți costăm la fel** (Max 299 lei ≈ 59 €). Sub 50 suntem mai ieftini, peste 50 mai scumpi. Plângerea lor recurentă, „taxat după anulare”, vine din contract: 12 luni de angajament, plătești restul lunilor dacă ieși mai devreme, iar reînnoirea e automată. **23 sep: partea clientului, văzută.** My PT Hub are un **comutator de identitate în meniul de cont** („Other Accounts → Account Type: Customer”): același om, un clic, fără deconectare, vezi aplicația clientului. Owner-ul l-a găsit, eu îl ratasem. La noi nu există și e mai greu de făcut (sesiune per-origine + rol din `profiles.role`), dar e cea mai bună idee luată de la ei — pus pe locul 0 în lista din §7. Pe web, clientul lor are **mai puțin** decât al nostru: antrenamentul e un tabel de citit, fără start/bifă/cronometru — logarea se face în aplicația de telefon, **pe care tot n-am văzut-o**. Detaliu de reținut: „Marketplace”-ul din aplicația clientului e al LOR (Fitbit, ebook-uri proprii), nu al antrenorului.

**22 sep, seara: owner-ul și-a făcut cont de probă** (`vinersarfitness.mypthub.net`) și am umblat prin partea antrenorului, pe desktop — ecran cu ecran în `docs/plans/competitor-teardown-v1.md` §7, „În aplicația lor”. **Aplicația clientului tot nu a fost văzută.** Cele mai ieftine idei de luat: „recent/des folosite” în selectorul de exerciții, exercițiu propriu pornit dintr-unul existent (rezolvă cele 13 nume în engleză fără să atingem globalele), antrenamentele din bibliotecă puse într-o zi de plan (codul există, `addWorkoutToDay`), „last seen” în lista de clienți, PAR-Q la înscriere. **Owner-ul a trecut trei pe backlog pe 22 sep** (`docs/plans/backlog.md`, „Din discuțiile cu antrenori"): grila de 7 zile la pasul 2, doar pe desktop; antrenamentele din bibliotecă puse într-o zi de plan; ziua de odihnă în interiorul săptămânii. **Pe backlog, nu pornite — în afară de al doilea, marcat 🔴 „DE REPARAT" la cererea owner-ului**, după ce am umblat și prin aplicația noastră: un antrenament din biblioteca noastră **n-are nicio ieșire** (acțiunile sunt doar creion și coș, „Copiază din…" oferă doar alte săptămâni ale aceluiași plan, iar „Folosește template" de la pasul 1 clonează un plan întreg). Textul de deasupra bibliotecii promite „reutilizează pentru clienții tăi", ceea ce e fals azi. Restul ideilor — recent/des folosite, exercițiu pornit dintr-unul existent, PAR-Q, check-in săptămânal, „last seen" — nu i-au fost cerute încă, deci nu sunt decise. **Același antrenor, pe vocal:** i-ar fi plăcut exercițiile și meniul în română, iar în rest My PT Hub i se pare „foarte complexă și foarte bine structurată”. E un compliment, deci simplitatea nu e argument. E exact golul pe care îl acoperă Veltofit, deci e primul argument de poziționare venit de la un antrenor real. Pe social se folosește doar atribuit. **Decizii ale owner-ului pe 22 sep:** numele săptămânilor (etapele) intră pe backlog. Cele 13 exerciții în engleză rămân cum sunt, deliberat, deci nu le repropune.
