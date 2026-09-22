---
name: instagram-reels-animate
description: "Reels animate pentru Velto: doctrina și uneltele trăiesc acum în skill-ul velto-social din repo — aici e doar starea și ce nu intră în skill."
metadata: 
  node_type: memory
  type: project
  originSessionId: a05eeee1-5093-4d09-a5aa-dc84305455dd
  modified: 2026-09-03T20:39:54.038Z
---

**Totul practic e în repo, nu aici: `.claude/skills/velto-social/`** (skill
înregistrat, creat 2026-09-03 la cererea owner-ului). Conține regulile de
conținut, vocea, inventarul de capturi, template-ul de reel și scripturile de
export. Înregistrat și în `CLAUDE.md` §2 și §4. **Citește skill-ul, nu re-deriva
nimic din nota asta** — două copii ale acelorași reguli se despart.

Owner-ul a cerut explicit **skill, nu agent**, după recomandarea că munca asta e
conversație iterativă cu el, nu delegare într-un context izolat.

## Starea, 2026-09-03

- **Primul reel: TERMINAT ȘI APROBAT.** „Velto în 26 de secunde", 6 scene,
  26,4 s. Owner: „sunt ok asa cum a iesit, nu mai schimbam nimic."
  - Pagina: https://claude.ai/code/artifact/61b4e6c3-e179-41d4-9d0d-b7d2a09c9bd7
  - Videoul livrat: `~/Desktop/velto-reel.mp4` — 1080×1920, 30 fps, H.264, 25 MB.
  - ⚠️ **2026-09-06: fișierele de pe Desktop sunt moarte.** Owner-ul le-a refăcut
    ca materiale proprii în `~/Veltofit/Social/` — reelul a devenit
    `reels/2026-09-05-prezentare` (20 s, 5 ecrane), caruselul a devenit
    `posts/2026-09-05-un-loc` (4 slide-uri). Nu raporta Desktop-ul ca restanță.
- **Skill-ul e COMIS** pe `feat/billing-launch`: `6e4258a`, 10 fișiere, împreună
  cu modificările din `CLAUDE.md`. Nepushat. Owner-ul a cerut explicit commit-ul.
- `.env.local.bak-1126` (backup de chei, neignorat de git) a fost ȘTERS la
  cererea lui, după ce am verificat că toate cheile secrete erau identice cu
  `.env.local` și difereau doar cele două URL-uri de host.

## Caruselul — DRAFT, se reia 2026-09-04

Primul carusel: **„Ce se întâmplă între două antrenamente"**, 7 slide-uri
1080×1350, exportate la 1440×1800. Owner: *„e ok pt un first draft, mai e de
lucrat la el dar o facem maine."* Deci NU e aprobat — așteaptă observațiile lui.

- Previzualizare: https://claude.ai/code/artifact/fa7a9219-839d-4af7-beb7-ac38e0666d34
- Fișierele: `~/Desktop/velto-carusel/slide-1..7.png` — **abandonate**, vezi
  avertismentul de mai sus; a fost refăcut ca `posts/2026-09-05-un-loc`.
- **Sursa: `.claude/skills/velto-social/assets/carusel-bucla.template.html`**,
  NECOMISĂ. Am salvat-o în skill ca să nu se piardă odată cu scratchpad-ul.

Subiectul l-a ales el dintre trei propuneri: bucla client→antrenor, pentru că
duce afirmația „întărești relația cu clientul" din [[velto-positioning-statement]],
singura care nu apare nici pe landing. Slide-ul 6 o poartă.

**Rămas de făcut:** observațiile lui pe slide-uri, apoi descrierea postării.
Slide-ul 3 e singurul fără captură — ecranul de antrenament activ tot nu există
în repo. Când apare, slide-ul ăla devine cel mai bun din set.

**De adăugat în skill după aprobare:** o secțiune `carousel-recipe` sau un
paragraf în SKILL.md; acum skill-ul descrie doar reel-uri. Convențiile noi care
merită scrise: șina colorată din stânga care schimbă culoarea la trecerea
client↔antrenor, dispozitivele care ies din cadru pe jos, `line-height: 1.14` la
titluri (1.07 lipea diacriticele românești), numărătoarea sus-dreapta ca să nu
stea peste capturi.

## Ce NU e în skill și merită ținut minte

- Owner-ul a respins zoom-ul pe o regiune **de două ori** („se vede tăiat",
  „avem un zoom ciudat"). Preferă ecranele întregi chiar cu textul mai mic.
  Skill-ul spune regula, dar nu cât de ferm a fost.
- Scena de progres a fost **scoasă de el**, nu uitată. Nu o repropune.
- A ales dintre variantele de titlu pe cea care repetă textul de pe ecran
  („Configurează planul" e chiar titlul din captură). Pattern-ul îi place:
  textul și imaginea să se confirme reciproc.

Vezi [[instagram-content-brief]] pentru trierea celor 50 de postări (rescrierea
celor 19 e următorul pas neînceput) și [[owner-never-a-personal-trainer]].

## 2026-09-06 — starea reală a materialelor

Cinci materiale pe disc, **zero publicate**. Trei trăiesc corect în
`~/Veltofit/Social/`: `reels/2026-09-05-prezentare` (ziua 1),
`reels/2026-09-05-partea-clientului` (ziua 8), `posts/2026-09-05-un-loc`
(ziua 22). **Niciunul n-are stare de aprobare scrisă** — `descriere.md` spune ce
conține materialul, nu dacă owner-ul l-a acceptat.

**„partea clientului" a fost re-randat pe 6 septembrie** cu sigla Veltofit — a
fost singurul care s-a putut, fiindcă folosește doar ecrane de client. Celelalte
două poartă sigla veche în capturi de dashboard și așteaptă refacerea capturilor,
care la rândul ei așteaptă schimbarea meniului din sidebar
(vezi [[veltofit-rename]]).

**Cum se recunoaște un material cu sigla veche, fără să deschizi videoul:**
`grep -o 'viewBox="0 0 [0-9]* 173"' reel.html` — `558` e vechi, `800` e nou.

**„Planul de 30 de zile" nu există ca document.** Cele trei materiale se declară
„ziua 1 / 8 / 22" dintr-un calendar scris nicăieri; `caption-recipe.md` are doar
două exemple din el.

**Închiderea „Începe gratuit" e o întrebare ÎNCHISĂ.** Owner-ul a răspuns de
două ori, ultima dată pe 2026-09-06 cu „mă repet": **nimic nu se publică înainte
de lansare**, deci îndemnul e corect prin construcție în orice material. Nu-l mai
semnala ca „de verificat înainte de publicare". Regula stă acum în
`references/caption-recipe.md`, punctul 3, iar cele două `descriere.md` care o
ridicau au fost corectate.

## 2026-09-07 — reelul de prezentare re-randat, și de ce blocajul era prost înțeles

**„prezentare" e re-randat** cu sigla nouă și cu îndemn pe cartonașul final:
21,0 s (era 20,0), coada finalului 2,0 s → 3,0 s ca îndemnul să apuce să fie
citit. Regula 9 din `visual-system.md` fusese actualizată pe 6 sep doar în
`carusel-excel`; am dus-o în toate patru șabloanele.

**Owner-ul a decis să publice cu sigla veche în bara laterală a capturilor de
dashboard** („nu cred că o să observe cineva diferența într-un reel de câteva
secunde"). Decizia e scrisă și în `descriere.md` a materialului. Se vede în
scenele 0 și 3; scena 3 o pune la ~300 px sub marca nouă, în același colț.

**Blocajul real NU e meniul din sidebar — e deploy-ul.** `npm run capture:marketing`
capturează din **producție** când `APP_URL`/`DASHBOARD_URL` nu sunt setate, iar
producția e `origin/main`, care n-are deloc `public/logos/` — doar
`public/images/velto-logo-*.svg`. Sigla nouă e pe `feat/billing-launch`
(`(dashboard)/layout.tsx:253`), nemerged. Deci capturile arată sigla nouă abia
după ce se lansează facturarea — SAU dacă owner-ul pornește dev server-ul și se
setează cele două variabile. Nota veche zicea „așteaptă schimbarea meniului";
ăla era doar motivul lui de a nu captura de două ori.

**Corecție la ce credea sesiunea dinainte: `un-loc` n-a fost niciodată blocat.**
Capturile lui sunt decupate la `--x:-318px`, deci bara laterală nu se vede deloc;
sigla veche era doar marca proprie a materialului. O re-randare îl rezolvă
complet. La fel `partea-clientului`, care are nevoie doar de cartonașul nou.

**O afirmație falsă scoasă din două locuri:** „30 de zile gratuit, fără card"
stătea în descrierea reelului ȘI ca exemplu ✅ în
`references/caption-recipe.md`. Pe Plus și Max cardul se cere la înscriere din
29 august. Landing-ul se corectase pe 2 septembrie; fișierul de referință
rămăsese în urmă și o preda mai departe. Formularea verificată:
**„30 de zile gratuite, anulezi oricând."**

**Rămâne de făcut, tot printr-o singură pornire de dev server** (aceeași de care
are nevoie și slide-ul 3 din caruselul Excel): `DETAIL_CLIENT_ID` din
`.env.capture.local` arată spre un client mort — 0/3 sesiuni, streak 0, ultima
activitate acum 83 de zile, iar cel mai colorat lucru din cadru e butonul roșu
„Anulează plan". E scena 3 din reel.

### Toate cele patru materiale sunt acum la zi (7 sep, seara)

`prezentare`, `partea-clientului`, `un-loc`, `din-excel` — toate cu sigla nouă
(800×173) și cu îndemn pe cartonașul final. Verificarea rapidă, fără să deschizi
nimic:

```bash
for f in ~/Veltofit/Social/reels/*/reel.html ~/Veltofit/Social/posts/*/carusel.html; do
  echo "$(basename $(dirname $f)) $(grep -o 'viewBox="0 0 [0-9]* 173"' $f | sort -u) $(grep -c 'class="cta"' $f)"
done
```

Durate noi: `prezentare` 21,0 s (era 20,0), `partea-clientului` 14,6 s (era 13,6)
— coada finalului a crescut cu o secundă în ambele, ca îndemnul să apuce să fie
citit.

**Singura restanță care rămâne pe toate trei materialele de dashboard e o
pornire de dev server de la owner**, pentru o rundă de capturi cu
`DETAIL_CLIENT_ID` schimbat: rezolvă simultan clientul mort din scena 3 a
reelului de prezentare, al cincilea slide lipsă din `un-loc`, și formularul gol
de la slide-ul 3 din caruselul Excel. Fără dev server, `capture:marketing`
merge pe producție și aduce înapoi exact aceleași capturi vechi.

## 2026-09-21 — TREI SUNT PUBLICATE

Owner-ul, 2026-09-21: **„am postat din ele. prezentare, partea clientului, și
cine sunt."** Nota din 6 sep („zero publicate") nu mai e adevărată.

| Material | Stare |
|---|---|
| `reels/2026-09-05-prezentare` | ✅ publicat |
| `reels/2026-09-05-partea-clientului` | ✅ publicat |
| `posts/2026-09-15-cine-sunt` | ✅ publicat (pinuit pe ambele conturi, după plan) |
| `posts/2026-09-05-un-loc` | nepublicat — așteaptă slide-ul 5 (capturi noi) |
| `posts/2026-09-06-din-excel` | nepublicat — slide 3 formular gol, text nerecenzat |

**Datele exacte de publicare nu se știu.** Contează pentru prezentare:
`visual-system.md` regula 9 (scris 15 sep) spune că reelul era „deja publicat"
când purta linia falsă „30 de zile gratuite. Anulezi oricând, nu plătești
nimic.” Re-randarea corectă e din 15 sep 22:56. Deci versiunea de pe Instagram
e aproape sigur cea veche, iar un video nu se poate edita, doar șterge și
republica. Întrebarea i-a fost pusă owner-ului pe 21 sep. **Răspunsul lui: „înainte de 15, cred”, pentru ambele
reels.** Deci și partea clientului e probabil pe varianta cu linia falsă. Reparația
există pe disc (re-randate 15 sep 22:55–56); decizia de a republica sau nu e a lui.

**Cine sunt a ieșit fără recenzie pe textul final.** `content-reviewer` a fost
rulat abia după publicare, pe 21 sep. Descrierea și alt-ul se pot edita pe
Instagram; slide-urile nu.

**Decis de owner pe 2026-09-21, după recenzie: cele două reels rămân publicate așa
cum sunt**, cu linia veche pe final: „nu e așa mare dezinformarea, și are un
sâmbure de adevăr.” Regula din `content-rules.md` se aplică: semnalat o dată,
decis. **Nu re-semnala.** Corecturile propuse pentru descrierea prezentării și
alt-urile 5–6 din „cine sunt” sunt scrise în `descriere.md`. Pe Instagram se
editează doar dacă owner-ul vrea. Iar dacă un reel se republică vreodată, se
folosește `reel.mp4` de pe disc, care e deja corect.

## 2026-09-21 (după-amiaza) — săptămâna de postări + DM-uri

Owner-ul a cerut **3, eventual 5 postări finalizate săptămâna asta**, și a pornit
dev server-ul pentru capturi.

**Datele clienților de test sunt FICTIVE** — owner, 2026-09-21, ca răspuns când
clasificatorul de permisiuni a blocat deschiderea fișelor pe motiv de date
personale. Asta a deblocat fișele și înregistrarea unui antrenament pe
`testclient@veltofit.app` (Alex M.) direct în producție, pentru capturi.
**Clienții de demo au toți ultima activitate la 140+ zile.** Singurul „viu” e
Alex M., și doar pentru că i s-a înregistrat azi un antrenament (4/12, streak 1).

Capturi noi în `~/Veltofit/Social/assets/` (inventarul e în `visual-system.md`):
pașii 1–3 din wizard, completați, pe „Spartan Recomp — Foundation” (contul
PLANS, deschis în modul de editare, nimic salvat), antrenamentul activ, ecranul
de final și fișa lui Alex M. `DETAIL_CLIENT_ID` arată acum spre el.

Materialele săptămânii:
- `posts/2026-09-06-din-excel`: slide-ul 3 are acum planul completat. Trimis la recenzie.
- `posts/2026-09-05-un-loc`: 5 slide-uri (slide nou cu fișa clientului), descriere rescrisă. Trimis la recenzie.
- `posts/2026-09-21-trei-pasi`: NOU, 6 slide-uri, template `carusel-trei-pasi`. Trimis la recenzie.
- Mesajele de DM pentru antrenori: `~/Veltofit/Social/mesaje/2026-09-21-abordare-dm.md`,
  recenzate și corectate. Lecția din recenzie: **„ce aud de la antrenori contează
  în ce construiesc”, lipit de o funcție cerută, se citește ca promisiune.**

Două defecte ale aplicației, văzute în capturi, pe care owner-ul trebuie să le
afle: fișa clientului afișează greutatea neformatată (`64.6627333745685 kg`) la
mai mulți clienți de demo, iar la 1150 px lățime panoul principal scapă a treia
coloană din „Activitate recentă” peste marginea dreaptă.

**Formatul s-a schimbat pe 2026-09-21: postările sunt 1080×1440 (3:4), nu 4:5.**
Owner-ul a arătat o captură din propriul profil: grila Instagram e 3:4, iar un
4:5 ieșea cu benzi albe sau cu textul tăiat. Detaliile și prețul (o postare 3:4
nu se poate promova ca reclamă) sunt în `visual-system.md`. Tot atunci s-a găsit
că renderul scotea 1351 px, nu 1350, la toate slide-urile de până atunci,
inclusiv la „cine sunt”, care e publicat. E reparat în `render-carousel.mjs`.

În aceeași captură, grila owner-ului mai arată **postări vechi publicate** din
setul celor 50, de exemplu „NU MAI SCRIE ACELAȘI PLAN DE 10 ORI” și una cu un
ecran de venituri „RON 126,600”. Nu s-a verificat care dintre ele erau printre
cele 9 păstrate. „Din Excel” apare deja în grilă, cu 0 vizualizări.

**🔴 Suita e2e și capturile folosesc ACELAȘI client de test** (`E2E_CLIENT_EMAIL`
= `CLIENT_EMAIL` din `.env.capture.local`). O rulare e2e, pe 2026-09-21 la 12:24,
dintr-o altă sesiune, i-a reasignat lui Alex M. planul (0%) și i-a lăsat un
antrenament „În desfășurare”. Capturile făcute înainte și după nu se mai potriveau
între ele. **Capturile unei postări se fac într-o singură rundă, fără e2e rulat
între ele.** Înainte de rundă, un antrenament terminat pe contul de test, ca fișa
să arate activitate de azi.

Pe 21 sep, „Un singur loc” a primit runda 2 de observații de la owner: subtitlul
copertei, panoul și lista recapturate la 1440 px, titluri noi. Detaliile sunt în
`descriere.md`.
