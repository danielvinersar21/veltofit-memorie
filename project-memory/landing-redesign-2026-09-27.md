---
name: landing-redesign-2026-09-27
description: "Landing rescris + pagină SEO + carduri de preț bogate + trecere de design — TOT NECOMIS. Captura de hero care bloca publicarea a fost REFĂCUTĂ pe 28 sep; rămâne trecerea owner-ului prin browser."
metadata: 
  node_type: memory
  type: project
  originSessionId: 0bd6cd36-e91d-4ae1-8d38-f295ff5a4ec2
  modified: 2026-09-28T14:29:58.468Z
---

Ziua de **27 septembrie 2026**. Trei commit-uri închise dimineața, apoi o zi
întreagă pe landing care a rămas **necomisă în arborele de lucru**.

## Comis și local (NEPUSHAT)

- `c6a358f` bara laterală a dashboardului aplatizată (grupul „Bibliotecă” a dispărut,
  `children` scos din tip ca să nu-l readauge cineva într-o bară care nu-l randează)
- `5a7c468` detaliul antrenamentului pe ambele gazde (`workouts/[id]`), plus
  reparația PGRST116 din `getWorkoutFull` — cod vechi, dar paginile noi erau
  primele care îi arătau eroarea brută în engleză
- `6605163` `tinte-adoptie-v1.md` + bara laterală bifată în backlog

Nota de stare `.claude/memory/velto-project-state.md` a fost **ținută deoparte
deliberat**: încă descria firele ca necomise. De actualizat prin `project-manager`
când landing-ul se comite.

## ✅ Captura de hero — REFĂCUTĂ 28 sep, nu mai blochează

Bloca trei lucruri deodată în `public/images/lp-dashboard-home-light.png`: logoul
**„Velto”** dinainte de redenumire, bara laterală **veche** cu grupul „Bibliotecă”,
și contorul **„1 ANTRENAMENTE”**. Toate trei închise. Detaliul complet al conductei
de captură e în [[marketing-captures-from-localhost]] — citește-l ÎNAINTE de orice
re-captură, are trei capcane care nu se văd din cod.

Pe scurt, ca să nu se redescopere:
- **Nu se poate trage din producție** cât `c6a358f` e nepushat: prod randează încă
  bara veche. Captura a venit de pe `npm run dev`, cu `DASHBOARD_URL` pe localhost.
- Cifrele vin din felia demo LOCALĂ, modelată de `docs/database/demo_cont_functional_seed.sql`
  (neaplicat în producție).
- Alt-ul a fost rescris odată cu poza. ⚠️ El NUMEȘTE ce e în imagine — cele cinci
  intrări din meniu și toate trei contoarele. Orice re-captură îl rescrie în același
  commit, altfel un cititor de ecran aude tabloul de bord anterior. Exact așa a ajuns
  alt-ul să spună „Bibliotecă” trei săptămâni după ce meniul n-a mai avut una.

`lp-dashboard-clients-light.png` (fostul hero) a rămas orfan în `public/`, împreună
cu încă 13 imagini, ~2,1 MB nereferențiat din `src/`.

## Ce conține munca necomisă

Landing rescris (2176 → ~1100 linii) + pagina SEO `/aplicatie-antrenori-personali`
+ cardurile de preț bogate + o trecere de design. Cele două fire **nu se pot
despărți**: pagina SEO are nevoie de `formatMonths` (nou în `billing-copy.ts`) și de
`SourceCarryingLink`; landing-ul are nevoie de `carrySignupSource` pentru linkul din
footer către pagina SEO. Tăiate în două commit-uri, unul nu compilează.

## Decizii luate, ca să nu se redeschidă

- **„Include suport prioritar.” ȘTERS** de pe cardul Plus — cuvântul apărea o
  singură dată în tot `src/`, chiar pe acea linie. Plus nu adaugă nicio funcție.
- Lista de pe carduri: **9 rânduri**, aceleași pe toate trei pachetele. Owner-ul a
  tăiat „Superset, dropset, rest-pause” și „Clientul își notează seriile în sală”
  — **nu ca fiind false** (ambele verificate în cod), ci ca decizie de a nu intra
  în detaliu pe un card de preț.
- Capul de listă „Tot ce scrie mai jos, pe orice pachet:” **scos** — intro-ul
  secțiunii îl spunea deja (`<h2>` „Alege în funcție de câți clienți ai.”).
- **„Salut, sunt Daniel.” RĂMÂNE**, contra recomandării `content-reviewer`, care
  voia propoziția ștearsă de tot. Motivul: owner-ul a constatat pe 26 sep că textul
  sună a AI și că umanul vine din fondator — vezi [[copy-sounds-like-ai]].
- FAQ: **10 întrebări**. „Clientul trebuie să aibă cont?” rămâne **„Da”** — vezi mai jos.
- Cardurile din §03: **3 pe orizontală**, cu imaginea tăiată de marginea cardului.
  Varianta **bento** e salvată ca patch (director temporar de sesiune, deci volatil).
- Secțiunea fondatorului: citat pe fundal inversat, `<details>` scos, deci
  „N-am fost niciodată antrenor” e vizibilă fără clic.

## 🔑 Capcana care a stat la baza întregii treceri de culoare

Pe tema deschisă, `--bg-base` (`globals.scss:391`) și `--background` (`:407`) au
**exact aceeași valoare, #F8FAFC**, iar `.page` folosea `--background`. Deci
alternanța `.band` / non-`.band` nu era slabă — **era zero**. „Totul e aceeași
culoare” era literal adevărat. Înlocuit cu `.bandStrong` / `.bandPrimary`, ambele
derivate prin `color-mix`, cu mixerul `--foreground`: aproape-negru pe deschis,
aproape-alb pe întunecat, deci aceeași declarație întunecă banda pe light și o
luminează pe dark.

⚠️ Coliziunea care vine la pachet: o bandă care se VEDE scoate `--text-secondary`
sub AA (4,29:1). Cel mai închis neutru care mai ține tokenul legal e ~#F7F7F7, adică
imposibil de distins de pagină. Deci text secundar pe bandă = `--text-primary`,
obligatoriu, pe o listă explicită de clase (nu `*` — cardurile de preț stau pe alb).

## De aplicat înainte de a urca — nu blocante

- Insigna „Recomandat” e la **4,51:1**, exact pe limita AA. Orice retuș la `--primary`
  o rupe; cere remăsurare.
- Inversiunea secțiunii fondatorului depinde de `data-theme="light"` fix pe `<main>`.
  Dacă pagina urmează vreodată tema utilizatorului, secțiunea **dispare** în pagină —
  exact defectul lui `.band`. Invariantul e scris în cod, în ambele capete.
- `#primii-pasi` a rămas **fără niciun link** către ea. Secțiunea rămâne fiindcă
  ancora e publică (`veltofit.app/#primii-pasi` a fost distribuit). Are test dedicat.

## Pâlnia, așa cum a rămas

Toate „Încearcă gratuit” derulează la `#preturi`, **în afară de CTA-ul final**, care
înscrie direct pe Free. Motivul pentru excepție: prețurile sunt secțiunea 06, CTA-ul
final e 08 — un link spre prețuri de acolo ar trimite cititorul **înapoi**, într-o
alegere pe care tocmai a refuzat-o, la singurul punct unde sub el e doar subsolul.

Legături directe spre înscriere: **patru** — cele trei carduri de preț + CTA-ul final.
Eticheta „Încearcă gratuit” are acum **două** comportamente, nu trei. Nu e unu; poziția
în pagină e ce le deosebește.

🔑 Motivul pentru care secțiunea de prețuri merită să fie hub: e **singurul loc din
pagină unde drumul Free (fără card) și cel cu plată (cu card) sunt deosebite**. Un
antrenor a ales odată Plus așteptând înscrierea fără card a lui Free și a dat peste un
câmp de card — „cea mai scumpă propoziție a paginii”, scris în comentariul vechi.

## Alarmă falsă de reținut

`content-reviewer` a raportat că generatorul de planuri din producție scrie româna
fără diacritice („Saptamana 1”, „Marti”, „Sambata”). **Adevărat în `generate_plan_structure`,
dar labels-urile sunt ȘTERSE imediat** de `clearPlanStructureLabels`, chemată de wizard
pe ambele gazde. E o capcană latentă, nu un defect vizibil. Două căi de creare
(`plans-store.ts:312`, `use-plans.ts:74`) n-au fost verificate dacă o cheamă.

## De verificat mâine, în browser

Ritmul de culoare pe toată pagina · bento vs. orizontal · citatul pe fundal inversat ·
lungimea secțiunii de prețuri pe telefon · hover pe comutatorul Lunar/Anual, inclusiv
pe butonul deja selectat (acolo era un bug de specificitate prins la timp).

Legat: [[what-justifies-the-money-2026-09-26]], [[landing-light-redesign]],
[[landing-pricing-section-built]], [[copy-sounds-like-ai]], [[backlog-seo-content-pages]],
[[qa-gate-blind-spots]], [[free-has-no-trial]].
