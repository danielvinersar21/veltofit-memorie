# Memory Index

- [**BACKLOGUL**](../../../repos/velto-webapp/docs/plans/backlog.md) — lista unică de lucruri deschise, în repo la `docs/plans/backlog.md`. Când owner-ul zice „trece pe backlog", acolo se scrie. Notele de aici sunt arhiva; fișierul ăla e indexul.

## Cum vrea owner-ul să lucrez
- [Prioritățile owner-ului, 26 sep](owner-priorities-2026-09-job-consulting-velto.md) — jobul la 8x8 întâi, consultanța a doua, Veltofit pe probă până în 2027 — dar **cu timp disponibil**: bugetul de 3–4h/săpt. și „primul plătitor până la 31 dec" au fost RESPINSE ca reguli. Vrea o definiție de „gata pentru bani", apoi pauză ca să vadă adopția.
- [Folosește agenții din proiect](use-project-agents-by-default.md) — instrucțiune permanentă 2026-08-20: rutarea din CLAUDE.md §4, fără să întreb de fiecare dată.
- [Întreabă înainte de commit](feedback-ask-before-committing.md) — 2026-08-30: niciun commit/push necerut. Niciodată `git add -A`; stage pe cale, cu lista afișată.
- [Vorbește simplu în rapoarte](feedback-plain-language.md) — 2026-08-21: rigoarea în muncă, nu în raport; consecințe înainte de mecanisme.
- [Nu porni serverul de dev](feedback-never-start-dev-server.md) — 2026-08-31: nici ca să verific o pagină; el îl pornește. **Se încalcă și INDIRECT** — heredoc neghilimetat a executat un `npm run dev` scris ca text. Folosește `<<'EOF'`.
- [Ghidare pas cu pas la testele manuale](feedback-guide-manual-tests-step-by-step.md) — 2026-09-07: testele se rulează împreună, un pas o dată, nu i se dă lista.
- [Citește textul legal ÎNAINTE de a oferi opțiuni](feedback-check-legal-text-before-offering-options.md) — 24 sep: o opțiune pe care i-am dat-o contrazicea propria pagină de termeni. Paginile legale sunt promisiuni publicate.
- [Citește regulile de social din fișier](feedback-social-rules-read-the-file.md) — 2026-09-06: skill încărcat ≠ reguli citite; review-ul rulează ÎNAINTE de a-i arăta.
- [DM-uri fără diacritice](feedback-dm-without-diacritics.md) — 2026-09-22: DM-urile către antrenori fără diacritice; tot restul, cu.
- [Două baze de date, și ordinea migrărilor](db-two-databases-migration-order.md) — din 25 sep există și o bază LOCALĂ. Local întâi, apoi prod, apoi `npm run db:schema`. „O să uit sigur" → rulează pașii tu. Plus capcanele (IPv6, tokenul expirat).
- [Două gazde, un produs](two-hosts-are-one-product.md) — 2026-09-03: dashboard=desktop, app=mobile, UN produs. O funcție doar pe o gazdă e o LIPSĂ, nu o decizie. Sesiunea pe origine e singura excepție reală.
- [Robert — al doilea contribuitor](robert-contributor-tasks.md) — fratele owner-ului, a construit dashboardul de admin; briefuri la nivel de obiective+constrângeri. Poza de profil LIVRATĂ (PR #16), exportul PDF al lui din 2026-08-20.

- [Ținte de adopție pe trimestre](adoption-targets-2026-2027.md) — 26 sep: plătitori cumulați Q4 1→3, Q1 4→7, Q2 8→12, Q3 2027 12→18. Recalibrare ~31 oct, decizie 31 mar 2027. Sursa: `docs/plans/tinte-adoptie-v1.md`.
- [Ce justifică banii — analiza din 26 sep](what-justifies-the-money-2026-09-26.md) — **START AICI pentru „ce mai construim".** Plus nu adaugă nicio funcție peste Free, doar locuri. Trei grămezi (raportul / sala / intrarea), ordinea propusă și linia de „gata" după punctul 5. Singura cerere reală: importul din Excel, de la Radu.

## Stare produs — deschise
- [Notificările n-au funcționat niciodată](notifications-never-worked.md) — zero notificări livrate vreodată unui client, confirmat în prod 2026-08-30. **Feature-ul a fost ȘTERS, nu reparat** (`d09fba5`, în main, verificat 2026-09-26) — `src/features/notifications/` nu mai există. Nu e un bug de reparat, e un canal de reconstruit; forma reparației (RPC SECURITY DEFINER, nu politică) e în notă. Patru defecte documentate, plus ce a rămas viu în două RPC-uri.
- [Backlog: trio active-workout](backlog-active-workout-trio.md) — 3 reparații (`dad3d35`) ✅ în main și pe GitHub, verificat 2026-09-24. `exercise_equipment` e tabel mort pentru exercițiile create în app. **Deschis: seturile „doar greutate" se pierd tăcut la finalizare.**
- [Backlog: freeze la antrenamentul activ](backlog-active-workout-freeze.md) — freeze în prod din furtuna de scrieri per-tastă. Reparațiile #1–#3 implementate + review 2026-07-29, NECOMISE, netestate pe telefon. #4 Pro/Micro + #5 realtime deschise.
- [Tehnici de seturi](set-techniques-draft.md) — ambele faze în main (`f67bd40`, `f9d1a17`), migrare aplicată, 20 teste e2e. **Deschis: testul la sală — e în producție netestat.**
- [Backlog: ordinea scrierilor de seturi](backlog-set-write-ordering.md) — bug-ul 25kg→2kg, REPARAT `42dbab9`. Capcane: prefix de cache redenumit omoară invalidarea tăcut; `!inner` din PostgREST conduce join-ul din tabelul greșit. Deschis: ipoteza tastei pierdute.
- [PWA, înălțimea ecranului](pwa-viewport-shortfall.md) — PWA-ul instalat primește 793px din 852, deci bara de jos se oprește cu 59px mai sus. Reparație scrisă + urcată, NEVERIFICATĂ unde contează.
- [Backlog: dovada consimțământului la notele de sănătate](backlog-health-notes-consent-proof.md) — controlul clientului livrat, dar ștampila stă în metadatele userului și nu e dovadă. Amânat 2026-08-22; de făcut înainte de înscrieri reale.
- [Column REVOKE e un no-op](column-revoke-is-a-noop.md) — un REVOKE pe coloană nu poate scădea din grantul pe tabel. Gaura `trainer_notes` ✅ închisă în prod 2026-08-29; constatarea despre integritatea măsurătorilor, din același audit, e deschisă.
- [Backlog: expunere, amânat](backlog-exposure-deferred.md) — email personal într-un comentariu .sql comis, și 37 de pagini design-system fără gardă în build (⚠️ middleware-ul răspunde 404 în prod din 7 sep). De ridicat înainte de înscrieri reale.
- [Backlog: focus a11y](backlog-focus-a11y-sweep.md) — 14 locuri fără indicator de focus la tastatură, cu fișier+linie. AMÂNAT 2026-08-20; nu re-propune nesolicitat.
- [Backlog: middleware → proxy](backlog-middleware-proxy-deprecation.md) — Next a depreciat convenția care duce TOATĂ rutarea pe subdomenii. Doar avertisment; amânat 2026-08-29.
- [Backlog: review 2026-08-20](backlog-review-round-2026-08-20.md) — 5 constatări reale, **amânate deliberat**, fiecare cu motivul. Nu sunt omisiuni — citește înainte de a le re-raporta.
- [Verificarea afirmațiilor din note](memory-status-claims-need-a-git-sweep.md) — TASK nefăcut: o trecere care verifică în git afirmațiile „merged/pushat" din toate notele. Cerut 2026-09-24 după o alarmă falsă.
- [Backlog: pagini SEO](backlog-seo-content-pages.md) — 3 pagini pentru căutări de categorie. DEPARCAT 2026-09-25 — owner-ul le vrea construite.
- [Idee: texte mai calde în aplicație](idea-warm-in-app-copy.md) — PARCAT 26 sep: 6 texte verificate + graficul cu cifre inventate. De rescris în vocea nouă înainte de implementare.
- [Copy-ul sună a AI, ca al concurenței](copy-sounds-like-ai.md) — owner 26 sep: noi și peak-fitness.eu spunem același lucru. Umanul vine din fondator și din momente reale în capturi, nu din titluri.

## Stare produs — închise, păstrate pentru tipar
- [Măsurători bilaterale](measurements-bilateral-status.md) · [Dashboard de admin](backlog-admin-dashboard.md) (`0ea14ea`+`844ce07`, split demo/real prin `is_demo`, invariantul „rolul nu autorizează niciodată") · [Arhivarea clientului](archive-client-feature.md) (a închis o spălare `pending→inactive→active`) · [Găuri RLS la scriere](rls-write-side-holes.md) (2 politici `FOR ALL` fără `WITH CHECK`; capcana testului vacuu) · [Sănătatea bazei](backlog-supabase-db-health.md) (bypass RLS critic + indexuri, aplicate) · [Scrieri de antrenor care nu fac nimic](backlog-trainer-writes-silent-noop.md) (tiparul: RLS taie scrierea, PostgREST răspunde 200) · [Două migrări de aplicat](backlog-two-migrations-to-apply.md) (tiparul pre-flight care a prins două eșecuri) — toate APLICATE și verificate în prod.
- Fluxul de adăugare client, închis: [configurația de producție](add-client-production-config.md) · [cele 3 urmări](add-client-followups.md) · [fluxul CASE A](backlog-case-a-consent-flow.md) (adăugarea manuală refuză un cont existent; promptul in-app a fost ABANDONAT — nu e task următor).
- Fluxul de antrenament al clientului, închis: [privire în timpul sesiunii](backlog-view-workout-mid-session.md) · [fluxul complet](backlog-client-active-workout-flow.md) · [raportare & istoric](backlog-workout-reporting-history.md) · [urmări de performanță](backlog-perf-followups.md).
- De confirmat la sală, nu în cod: [ultima greutate folosită](backlog-verify-last-weight-hint.md) · [pornirea optimistă + paralelizarea](backlog-verify-start-and-perf.md).
- [Performanța pe client-home](client-home-perf-findings.md) — MĂSURAT: agitația de prefetch e routerul Next, NU remount-uri — nu re-deriva. ProtectedRoute reparat (`db7b19e`).
- [Suita e2e + upgrade DB + CI](e2e-suite-and-db-upgrade.md) — 8 spec-uri: 17 rulabile + 1 fixme, plus 3 setup + 1 teardown (2026-09-22). Supabase **Pro/MICRO** din 2026-08-11; rămâne un flake de latență cu semnătura „Conexiune lentă". ✅ GitHub Actions RULEAZĂ din 23 sep; a prins imediat un throw la nivel de modul care rupea `next build` (`172ee88`).
- [Puncte orbe ale porții de QA](qa-gate-blind-spots.md) — poarta NU prinde importurile/localele nefolosite (51 măsurate), și un `vi.fn()` înghite respingerile netratate. Strică sursa și uită-te dacă testul cade; dacă NU cade, fixtura face treaba, nu codul.

## Bani, facturare, lansare
- [Starea integrării Stripe](stripe-integration-state.md) — **START AICI pentru facturare.** Audit 22 sep: prețuri, webhook, portal OK (portalul se verifică DOAR în dashboard, API-ul minte). Live mode complet din 6 sep: 7 produse, 14 prețuri, dunning, dispute, cheile în Vercel. Codul e în producție din 7 sep.
- [Jurnalul de încasări](billing-payments-ledger.md) — viu în prod din 17 sep: migrare + cod + `invoice.paid` pe endpoint-ul LIVE. Bifa din Stripe nu există în repo. Proba finală: prima încasare reală.
- [Schema de facturare, faza 1](billing-phase1-schema.md) — ✅ aplicată în prod 2026-08-21. `trainer_subscriptions` + registrul de trial; 5 antrenori grandfathered. De ce facturarea nu poate sta pe `profiles`.
- [Facturarea pe telefon](billing-on-the-phone.md) — V8 livrat: gazda app are ecran de abonament, Stripe se întoarce la gazda de pe care ai plătit. Bug-urile poll-never-re-armed și `overflow-x`; un număr încă lipsește.
- [Webhook-urile Stripe cer CLI-ul local](stripe-webhooks-need-cli-locally.md) — o plată locală reușește și TOT lasă contul pe Free; webhook-ul nu ajunge la laptop. `stripe listen` e puntea. Spune-o ÎNAINTE de orice test local de plată.
- [Runbook de facturare](../../../repos/velto-webapp/docs/plans/billing-operations-v1.md) — pașii Stripe/factură/Solo, în repo la `docs/plans/billing-operations-v1.md`.
- [Preț și monetizare](pricing-monetization-plan.md) — abonament antrenor→Velto, RON, Free/Plus(99)/Max(299), axa = numărul de clienți. Decis 2026-07-31.
- [Free nu are trial](free-has-no-trial.md) — din 31 aug: Free = 2 locuri, fără card, fără trial, pentru totdeauna; trialul îl dă Stripe la Checkout. Un cont Free cu `trial_ends_at` NULL e normal.
- [Decizia de go-to-market](launch-gtm-decision.md) — 2026-08-20: FĂRĂ beta gratuit. Prețuri reale + trial 30z fără card. Facturarea manuală RĂSTURNATĂ în aceeași zi. Solo nu are integrare Stripe.
- [Plan de pregătire a lansării](launch-readiness-plan.md) — **start aici pentru lansare.** Promovăm da, înscrieri deschise nu; trialul pornește un ceas de 3 săptămâni.
- [Blocaje de lansare 2026-08-22](backlog-launch-blockers.md) — ce e aplicat în prod vs. scris-neaplicat, capcanele de ordonare, și de ce owner-ul nu vedea UI-ul de facturare în propria aplicație.
- [Test 3: peretele de locuri](test3-seat-wall-results.md) — peretele de locuri + fluxul „în așteptare" + refuzurile la adăugare manuală, TRECUTE în prod 2026-08-30/31.
- [Abonamentul în admin](admin-trainer-billing.md) — LIVE 22 sep (`ef6e1c0`): cardul „Abonament" pe pagina antrenorului; modul Stripe live/test doar dedus.

## Creștere, conturi, marketing
- [Cine e fiecare cont de antrenor](trainer-accounts-who-is-who.md) — Bogdan și Radu sunt PRIETENI invitați personal (de-asta scutiți); organici doar Crăciun și Dumitru. Lămurit de 2 ori, pierdut de 2 ori — nu re-deriva din emailuri.
- [Prima reclamă Meta](meta-ads-first-campaign.md) — 21 sep, rezultate la 3 zile (24 sep): boost-ul bate reclama de 4,5 ori la cost pe vizită (0,45 vs 2,03 lei); ~47 lei pe antrenor adus; capcana prin care boost-ul cedează meritul linkului din bio.
- [Proveniența antrenorilor](trainer-signup-source.md) — LIVE 21 sep (`5afff65`), verificat 17/17. Capcana locală cu Redirect URLs e în notă.
- [Alertele pentru owner](owner-alerts.md) — LIVE 22 sep (`65f1433`): emailuri FĂRĂ nume/email/oraș (decizia lui), la contact@.
- [Emailuri de onboarding în MailerLite](onboarding-emails-mailerlite.md) — ✅ LIVE 23 sep, dovedit cap-coadă. Capcana variabilei `MAILERLITE_API_KEY` deja existente din waitlist e în notă.
- [Analiza concurenței 2026-09](competitor-teardown-2026-09.md) — al treilea concurent RO e din Alba Iulia și are zero utilizatori, declarat pe propriul site. + My PT Hub (59 €/lună; la 50 de clienți costăm la fel). Lista în `docs/plans/competitor-teardown-v1.md`.
- [Antrenorii preferă română](trainers-prefer-romanian.md) — mai mulți i-au spus că preferă aplicația în română, pentru ei și clienți; doar atribuit.
- [Poziționarea, în cuvintele lui](velto-positioning-statement.md) — ce ESTE Veltofit, la nivel de produs. „Întărești relația cu clientul" e singura afirmație pe care landing-ul încă nu o face.
- [Funcțiile de pe foaia de drum](roadmap-features-plan.md) — „Ce urmează" de pe landing (Calendar, Clase, Nutriție, Comunicare) e VIZIUNE, NU construit. `docs/plans/roadmap-features-v1.md`.
- [Brief de conținut Instagram](instagram-content-brief.md) — cele 50 de postări triate: 9 păstrez / 19 refac / 22 șterg, aprobat 2026-09-02. Ce nu se mai poate afirma niciodată.
- [Reels animate](instagram-reels-animate.md) — **3 PUBLICATE (21 sep): prezentare, partea clientului, cine sunt.** Doctrina + uneltele sunt în skill-ul `velto-social`.
- [Owner-ul n-a fost niciodată antrenor personal](owner-never-a-personal-trainer.md) — confirmat 2026-09-02. Persoana întâi doar despre ce construiește.
- [Schimbul de atelier cu Ionuț](workshop-exchange-ionut.md) — manualul PrintBox al vărului + grila de 15 întrebări, 2026-08-29. Al nostru: 5 DA / 6 PARȚIAL / 4 NU. Nimic adoptat, deliberat.

## Landing, SEO, legal, infrastructură
- [**Landing rescris, 27 sep — NECOMIS**](landing-redesign-2026-09-27.md) — **START AICI dacă se reia landing-ul.** O zi întreagă în arborele de lucru: landing + pagină SEO + carduri bogate + trecere de design. Captura de hero care bloca publicarea a fost REFĂCUTĂ pe 28 sep; rămâne trecerea prin browser a owner-ului. Plus capcana `--bg-base` == `--background` pe tema deschisă, din care a pornit tot.
- [Capturi de marketing de pe localhost](marketing-captures-from-localhost.md) — cum se trage o poză de UI nepushat, plus 3 capcane: bulina Next arsă în imagine, titlul ales cu `Math.random()`, și pinul care rupe login-ul dacă e pus pe context. Și cele două goluri din datele demo care există și în producție.
- [Landing, redesign light](landing-light-redesign.md) — TERMINAT ȘI ÎN MAIN (verificat 2026-09-02). Convențiile de container/eyebrow/crop + pivotul de preț. Nota veche „se rescrie pe ramură" a indus în eroare doi agenți.
- [Landing dark/light](landing-dark-light-branch.md) — ✅ în main 2026-08-03 (`2924bb2`). Deschis: imaginea OG e tot cea veche.
- [Secțiunea de prețuri pe landing](landing-pricing-section-built.md) — `1dd353c` pe main. Convențiile `.linkButton` + fără grilă de prețuri pe 2 coloane. OG stale + `metadataBase` nesetat.
- [Baza SEO 2026-08](landing-seo-baseline.md) — apăream doar la propriul nume; titlul paginii era literal „Velto". Ce s-a reparat și ce a rămas muncă de text.
- [Paginile legale, întărite](legal-pages-hardening.md) — 6 sep: termeni 9→14 secțiuni + anexă de prelucrare. Platforma SOL e DESFIINȚATĂ. Tiparul care le-a stricat de 4 ori: corectura se face unde a fost observată, vecinii păstrează povestea veche.
- [Redenumirea Veltofit — decizia](veltofit-rename.md) — decisă 2026-09-06, assets terminate. Ordinea: meniul → capturile → textul → materialele. Semnul nu se atinge: el E litera V.
- [Redenumirea — ce NU se atinge](rename-veltofit.md) — prefixul `velto_` din cheile de preț, `velto-theme`, entitatea PFA… + capcana fallback-ului tăcut din scripturile social. Citește ÎNAINTE de orice sed global.
- [Ce e legat la domeniul veltofit.app](veltofit-domain-accounts.md) — cumpărat din Vercel, echipa „Daniel's projects", auto-renew 11 dec 2026. Are Google Workspace (MX = smtp.google.com). DMARC pus pe 7 sep (p=none).
- [Șabloanele de email Supabase](supabase-email-templates-brace-parsing.md) — cele 3 săptămâni de „salvat dar neutilizat" aveau cauza în comentariul NOSTRU: parserul citește `{{ }}` și în comentarii HTML, iar eșecul e TĂCUT. Rămâne relipirea în panou.
- [Serverul de dev mânca procesorul](dev-server-runaway-cpu.md) — ✅ reparat 2026-09-02: era Turbopack. `npm run dev` are `--webpack` (1,1GB vs 4,5GB). Regula „agenții nu pot scrie cât rulează dev" era GREȘITĂ. `--webpack` e depreciat: retestează după fiecare upgrade de Next.
- [Backup-ul memoriei](backlog-memory-has-no-backup.md) — ✅ făcut 22 sep: repo PRIVAT `veltofit-memorie`, automat la fiecare push + `npm run memory:backup`. Nu în repo-ul aplicației: notele au emailuri reale.
- [Prioritățile din august](backlog-owner-priorities-2026-08.md) — top 5 de atunci: PWA offline → media exerciții → editare plan în wizard → poză profil → export raport. #4 livrat, #5 la Robert. Suprascris de nota de priorități din 26 sep.
