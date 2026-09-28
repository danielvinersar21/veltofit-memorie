---
name: marketing-captures-from-localhost
description: "Cum se trage o captură de marketing de pe dev server, și cele trei capcane care nu se văd din cod: bulina Next, titlul aleatoriu, și pinul care rupe autentificarea."
metadata: 
  node_type: memory
  type: project
  originSessionId: fdf31a5b-0e5d-471f-92a5-d45ad32eda3a
  modified: 2026-09-28T14:30:36.834Z
---

`scripts/capture-marketing.ts` a fost scris ca să tragă din **producție**. Pe
2026-09-28 a trebuit pentru prima dată să tragă din **localhost**, fiindcă poza
cerută arăta bara laterală aplatizată (`c6a358f`), iar commit-ul era nepushat —
producția randa încă meniul vechi.

**Regula care rezultă:** dacă poza trebuie să arate UI scris dar nepushat,
producția nu poate fi sursa. Nu e o preferință, e o imposibilitate.

```
CAPTURE_ONLY=dashboard-home DASHBOARD_URL=http://dashboard.localhost:3000 \
  npm run capture:marketing
```

1440×900 la `deviceScaleFactor: 2` → fișier 2880×1800, exact dimensiunea pe care
o așteaptă `SHOT_A_HOME` din landing. Contul e `testtrainer@veltofit.app`, din
`.env.capture.local`; există și local, cu aceeași parolă, fiindcă felia demo se
exportă din prod cu tot cu conturile auth.

## Cele trei capcane, toate găsite în aceeași oră

**1. Bulina de dezvoltare Next e arsă în poză.** Colțul din stânga-jos, cerc
închis cu „N". Nu există într-un build de producție, deci un an întreg de capturi
din prod au fost curate și nimic n-a avertizat vreodată. Reparat în script:
`nextjs-portal { display: none !important; }` — se ascunde GAZDA, fiindcă
conținutul stă în shadow root, unde o foaie de stil de pagină nu ajunge.

**2. Titlul tabloului de bord e ALEATORIU.** `MOTIVATIONAL_QUOTES` în
`(dashboard)/dashboard/page.tsx`, ales cu `Math.random()` din cinci. Deci aceeași
rută trasă de două ori dă două titluri diferite — iar ăla e cel mai mare text din
heroul landing-ului. O re-captură făcută pentru alt motiv rescria tăcut cea mai
mare propoziție a paginii. Fixat: `Math.random` e pinuit pe 0,5 → indicele 2 →
„Disciplina ta de azi e progresul lor de mâine.", cel ales de owner.
⚠️ **Indicele e pozițional**: reordonarea listei sau un al șaselea citat schimbă
tăcut ce primesc toate capturile viitoare. `CAPTURE_RANDOM=off` întoarce zarul.

**3. 🔴 Pinul pe `Math.random` TREBUIE pus pe pagină, nu pe context.** Pus pe
context, prinde și pagina de login, iar supabase-js își trage verificatorul PKCE
tot din `Math.random`; o constantă face ca orice autentificare să atârne până la
timeout-ul de 30s. Simptomul e „TimeoutError la login", care nu seamănă deloc cu
cauza. Pagina de captură moștenește sesiunea deja stabilită, deci n-are nevoie de
aleatoriu propriu.

## Datele demo

Cifrele din poză vin din felia demo, care trăiește în **producție** marcată
`profiles.is_demo = true`, și ajunge local printr-un export read-only
(`npm run db:seed:export -- --load`). Deci ce e subțire în prod e subțire și local.

Modelarea pentru „să pară un cont funcțional" e în
`docs/database/demo_cont_functional_seed.sql` — **aplicat DOAR local**, idempotent
(id-uri din `md5(cheie_naturală)::uuid`, nu `gen_random_uuid()`).

🔑 **Două goluri descoperite acolo există și în producție**, fiindcă localul e o
copie a ei — nereparate, deliberat:
- Două din trei planuri asignate aveau **săptămâni complet goale** (săpt. 2 și 3
  fără niciun exercițiu). Un antrenament deschis din ele se deschidea gol.
- **Niciuna din cele 40 de sesiuni finalizate n-avea serii logate.** Adică funcția
  „un antrenament se poate DESCHIDE" (`5a7c468`) ducea la o pagină goală pe datele
  demo. Contează: suita e2e rulează pe datele demo din prod.

Cache-ul înșală la verificare: `dashboard:counts:` 5 min, `clients:list:` 2 min.
Dacă ecranul arată cifrele vechi după un seed, e cache — hard refresh, nu re-rula.

Legat: [[landing-redesign-2026-09-27]], [[e2e-suite-and-db-upgrade]],
[[db-two-databases-migration-order]], [[feedback-never-start-dev-server]].
