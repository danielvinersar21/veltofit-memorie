---
name: what-justifies-the-money-2026-09-26
description: "Analiza din 26 sep 2026 — ce mai merită construit ca un antrenor să simtă că 99 lei sunt justificați, plus linia de „gata\"; trei agenți, totul verificat în cod."
metadata: 
  node_type: memory
  type: project
  originSessionId: 7ecbafbe-e9f2-4d56-b149-7f522d189412
  modified: 2026-09-26T07:46:44.655Z
---

Cerut de owner pe **2026-09-26** (vezi [[owner-priorities-2026-09-job-consulting-velto]]): o analiză a ce merită încă construit, apoi pauză ca să se vadă adopția. Făcută cu trei agenți, fiecare afirmație verificată în cod sau în git — nu din note.

## Constatarea de plecare
**Plus (99 lei) nu adaugă nicio funcție peste Free — doar 15 locuri în loc de 2.** Singura diferență de funcție din tabelul de pe landing e „Suport prioritar" (`(marketing)/page.tsx:799`), iar `grep -rn prioritar src docs` întoarce **exact acel rând în tot repo-ul**. Nu există mecanism. Modelul pe locuri e onest, dar tabelul sugerează altceva.

## Nu sunt 25 de probleme, sunt trei grămezi
1. **Raportul = produsul antrenorului, și nu e de încredere.** Se construiește exclusiv din seriile logate (`sessions-api.ts:549,564-569`), deci un exercițiu la care clientul n-a scris nimic **nu apare** — antrenorul nu poate deosebi „a sărit" de „nu i-am dat". Plus: o zi nu se poate marca „sărit" deși UI-ul afișează deja starea (țeavă pusă, robinet lipsă); seturile „doar greutate" se pierd; raportul nu poate ieși din aplicație.
2. **Sala = produsul clientului, cu două găuri în mijlocul poveștii de vânzare.** Nu merge fără semnal, și antrenorul nu poate bifa în locul clientului (`sessions_trainer` și `set_perf_trainer` sunt `FOR SELECT`; ruta activă e doar sub `(app)/client/`).
3. **Intrarea.** Fără import din Excel, un antrenor cu 40 de clienți nu poate începe.

## Unde produsul minte (mic, de făcut primul)
Grafic de câștiguri cu cifre scrise în cod pe prima pagină a antrenorului, pe ambele gazde (`(app)/trainer/home/page.tsx:59-62`, `(dashboard)/dashboard/page.tsx:275-277`) — 1800/2200/2600 lei, în spatele unui blur de 4px, **fără niciun tabel de câștiguri în cele 31 din prod și fără loc în roadmap** · câmpul „Link video" aruncă tăcut valoarea în 4 pagini (`if (field === 'video') return;`) · „Suport prioritar" · „Mută-ți clienții din Excel" promis pe landing fără cod · biblioteca zice „pentru clienții tăi" dar un antrenament nu se poate atribui unui client · **ștergerea contului unui CLIENT nu există în prod** deși paginile legale o promit (SQL scris + testat local, `stergere_cont_client.sql`, netrackuit).

## ⚠️ Decizia de produs care ordonează restul (confirmată de owner, 26 sep)
**„Clientul își notează singur antrenamentul, pe telefonul lui, logat pe contul lui."** Două consecințe:
- **„Antrenorul loghează în locul clientului" NU e o gaură, e o decizie.** Codul o reflectă corect (politici `FOR SELECT`, ruta activă doar sub `(app)/client/`). **Scos din listă — nu-l re-propune.**
- **Raportul e singura fereastră a antrenorului** — el nu e acolo când se lucrează. Deci calitatea raportului *este* produsul pentru care plătește. De-asta grămada 1 e prima.
- Și **offline urcă**: clientul e singur în sală, iar logarea lui e conducta care alimentează raportul.

## Ordinea propusă, și linia
0 cele șase unde produsul minte (mic) · 1 raportul arată ce s-a cerut (mare) · 2 zi „sărit" (mediu) · 3 seturi doar-greutate (mic) · 4 merge PDF + pe app (mic) · 5 import Excel (mare) — **LINIA e după 5** · 6 offline (mare), imediat după linie.

⚠️ **Offline e singurul punct bazat pe o presupunere nemăsurată** (că sălile n-au semnal); tot restul e citit în cod. Înainte de a investi „mare", întrebarea de 5 minute către cei 4 antrenori: s-a plâns vreun client că aplicația nu se încarcă în sală?

**📄 Documentul complet e în repo: `docs/plans/ce-mai-construim-v1.md`** (scris 26 sep, necomis la acel moment). Ăsta e locul de întors; nota de aici e doar indexul.

**Why:** owner-ul are nevoie de un capăt definit, nu de o listă. Riscul nu e lipsa de ore, e că se construiește fără capăt și adopția nu se vede niciodată. Criteriul lui: *„cred că cu asta se descurcă antrenorii și avem destule funcții care să justifice banii."*

**How to apply:**
- **Singura cerere reală găsită: Radu a cerut importul din Excel, are 40–50 de clienți într-un fișier.** Bate orice deducție din cod. Dacă apar altele de la antrenori, rearanjează clasamentul după ele, nu după analiza asta.
- Ordinea 6 vs. 7 depinde de un lucru care NU e în repo: antrenorii lui lucrează față în față în sală, sau scriu planul și clientul execută singur? Întreabă, nu presupune.
- De tăiat explicit, chiar dacă le are concurența: notificări/mesagerie (șterse deliberat — [[notifications-never-worked]]), calendar, nutriție, marketplace, bibliotecă de video-uri.

## Lucru terminat care nu ajunge la nimeni
`origin/report-pdf-export` — un commit al lui Robert din **18 aug**, print nativ cu `@media print`, zero dependențe noi, nemerged. Merge-ul aduce și `tsconfig.json` + un fișier de datorie tehnică, nelegate. · `origin/feat/exercise-media` — un commit din **16 aug**, „UNVERIFIED", migrarea neaplicată, și e **doar link extern, nu upload**. · Plus 11 fișiere modificate + 6 netrackuite în arbore, cod nu documentație (pagina SEO întreagă, `og-card.tsx`, componenta de link cu proveniență).

## Afirmații din backlog dovedite stătute (26 sep)
`admin_delete_trainer_account` **E aplicat** (4 apariții în `SCHEMA.sql`), nu „scris, neaplicat" (`backlog.md:158`) · `addWorkoutToDay` **are apelant real** (`use-plan-builder-v3.ts:539`) — jumătatea dinspre constructorul de planuri e livrată · defectul „raportează salvat peste o re-citire picată" e reparat la `addExercise`/`addWorkout`, rămâne la `removeExercise`, `copyWeekTo`, `clearWeek`. Șase rânduri din `backlog.md` zic „deschis" pentru lucruri închise; textul de corectură e în `.claude/memory/velto-project-state.md`, blocul din 26 sep. Vezi [[memory-status-claims-need-a-git-sweep]].
