---
name: idea-warm-in-app-copy
description: IDEE PARCATĂ 2026-09-26 — 6 texte mai calde în aplicație (bun venit antrenor, invitație, plată, client fără plan, final de antrenament), deja verificate de content-reviewer. Owner: „le lăsăm așa deocamdată".
metadata:
  type: project
---

Pornit de la un bilețel Zola pe care owner-ul l-a plăcut (25 sep): căldura merge când e
SPECIFICĂ și ADEVĂRATĂ; generică = șablon. Owner-ul a parcat implementarea 2026-09-26.
Direcția generală s-a mutat apoi pe „copy mai uman, nu sună a AI" — vezi [[copy-sounds-like-ai]].
Rescrie aceste texte în vocea nouă înainte să le implementezi.

Emailurile MailerLite (bun venit + „Cum ți se pare?") sunt DEJA în tonul bun — nu se ating.

Propunerile, după content-reviewer (fiecare afirmație verificată în cod):
- A (doar app, trainer/home, la 0 clienți): „Bun venit în Veltofit, {prenume}." + „Adaugi primul
  client cu numele și emailul lui sau îi trimiți link-ul tău de invitație. Pe amândouă le găsești
  la «Adaugă client»." Buton deschide drawer-ul (useAddClientDrawerStore().open). Dashboard are deja salut.
- B (dashboard): citatele aleatorii (dashboard/page.tsx:20-30, slogane) → „Săptămâna asta, clienții
  tăi au terminat {N} antrenamente." Cere o interogare NOUĂ de numărare (feed-ul are limit 10);
  de decis luni-00:00 Europe/Bucharest vs ultimele 7 zile; plural românesc; doar clienți activi.
- C (add-client-drawer): „Invitația a plecat către {nume}." + „E deja în lista ta de clienți, marcat
  «Invitat». Din email își alege parola și intră în cont." (clientul apare IMEDIAT ca pending —
  „după parolă" ar fi fals). Formularul se golește înainte de succes: păstrează numele separat.
- D (billing-copy.ts:977): titlul rămâne; body începe „Mulțumim că ai ales Veltofit." (+ test).
- E (client/home, fără plan): „Bun venit, {prenume}!" + „{Nume antrenor} îți atribuie planul de
  antrenament. După ce o face, îl găsești pe pagina asta." Numele e deja încărcat (useTrainerProfile).
- F (workout/active, rating): „Cum a fost? {Nume antrenor} vede evaluarea pe care o dai." Adevărat
  (nota apare în 4 locuri la antrenor); trebuie adăugat useTrainerProfile pe pagină. NU „ajunge la" (sună a notificare).

Găsite pe drum, nerezolvate: la 0 clienți home arată „Toți clienții sunt pe drumul cel bun" (fals);
graficul „Clienți vs. Câștiguri" are cifre inventate hardcodate sub „În curând" (trainer/home:59-63,
dashboard:249-302) — „Câștiguri" atinge funcția interzisă de plăți client-antrenor. Owner nedecis.
Idee fără cod: email scris de mână de owner după fiecare plată (alertele n-au nume → caută în admin).
