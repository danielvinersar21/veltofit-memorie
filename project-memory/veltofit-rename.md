---
name: veltofit-rename
description: "Velto → Veltofit, decis 2026-09-06. Assets TERMINATE și comise; textul, meniul, capturile și materialele de social sunt deschise, în ordinea asta."
metadata: 
  node_type: memory
  type: project
  originSessionId: b930121e-1ae3-40d3-a970-32bcd03f7623
  modified: 2026-09-06T08:30:45.222Z
---

Owner a decis pe **2026-09-06** redenumirea **Velto → Veltofit**. Nu e o
schimbare de direcție: domeniul (`veltofit.app`), adresa de contact și contul de
Instagram (`@veltofit.app`) ziceau deja veltofit. Doar produsul zicea Velto.

**Motivul care a decis-o:** „Velto" simplu e contestat de cel puțin patru firme
de software — un startup de productivitate cu AI din San Francisco (2025), Velto
Technologies în Emirate, VELTO LTD în UK, Velto Software în India. Două fondate
în 2025, deci exact acum își construiesc prezența în căutări. Vezi și
[[landing-seo-baseline]]: Velto se clasa doar pe propriul nume, iar nici ăla nu
e al lui.

## TERMINAT — assets, comise pe `feat/billing-launch`

Trei commit-uri, 2026-09-06, **nepushate**: `950c01e` (logo), `5238fe3`
(regulile de social din 5 sep), `d7afc41` (șabloanele de social).

Logo nou în `public/logos/`: `veltofit-logo-{black,white}.svg` (800×173) și
`logo-mark-{black,white}.svg` (180×172). Alb = `#F8FAFC`, adică `--bg-base` —
owner a refuzat deliberat să inventeze o culoare nouă; vechiul `#FAFAFB` era o
valoare orfană. Cinci fișiere vechi șterse, cinci locuri de referință mutate.

## DE FĂCUT, în ordinea asta — ordinea nu e opțională

**1. Meniul din sidebar-ul dashboard-ului.** Ideea owner-ului: scoate
**Planuri · Antrenamente · Exerciții** la nivelul principal și renunță la grupul
**Bibliotecă**, ca meniul să aibă mai multe iteme. Configurația e în
`src/config/navigation.ts:110-131` — `dashboard-library` e un grup fără `href`,
cu trei copii care arată spre `/dashboard/library/{plans,workouts,exercises}`.
De decis atunci: rămân rutele pe `/library/` sau se mută și ele.

Aplicația mobilă are un singur tab `Bibliotecă → /trainer/library`
(`navigation.ts:46`). Acolo sunt doar 4 sloturi de tab, deci diferența e
justificată de ecran — **nu e o încălcare a regulii din
[[two-hosts-are-one-product]]**, dar merită spus explicit când se face.

**2. Capturile din landing — DUPĂ meniu.** Au logo-ul vechi în ele. Owner a cerut
explicit să se refacă abia după schimbarea meniului, altfel se refac de două ori.
Patru dintre ele trăiesc și în `public/images/` fiindcă le servește landing-ul —
vezi README-ul din `~/Velto/Social/`.

**3. Numele scris în cod.** Neatins intenționat: `manifest.json` (`name`,
`short_name`), câmpul `name` din JSON-LD-ul Organization
(`src/app/(marketing)/page.tsx:435`), titlurile din `layout.tsx`, paginile
„Despre Velto", badge-urile „Template-uri Velto" din bibliotecă. 212 apariții în
66 de fișiere, majoritatea comentarii.

**4. Materialele de social, re-randate.** Cele trei din 5 septembrie — vezi
[[instagram-reels-animate]]. Șabloanele arată deja spre sigla nouă.

**5. Imaginea OG.** Raster, tot cu logo-ul vechi, stăla deja dinainte —
[[landing-pricing-section-built]].

## Ce s-a aflat și nu se re-derivează

- **Semnul E litera V.** Fișierul are 7 path-uri: 1 semn + 2 așchii + **4
  litere**, iar alea patru scriu „elto". Logotipul n-a scris niciodată „Velto".
  De aia redenumirea **nu atinge deloc** semnul, favicon-ul sau PNG-urile mici.
- **Raportul e 800:173.** Cuvântul e cu 43% mai lat la aceeași înălțime. Trei
  din cinci locuri fixau *lățimea*, nu înălțimea — un swap doar de cale ar fi
  micșorat tăcut logo-ul de la 28px la 20px înălțime. Corectate.
- **„fit" e Regular, nu Light.** Owner a încercat Light pentru contrast și a
  ieșit „pixelat". Cauza: la scara 0,162 din navigație, stem-urile de Light cad
  sub un pixel și se randează gri, nu negru. Cele două reclamații — „prea
  subțire" și „pixelat" — erau același defect.
- **Next NU rasterizează SVG-ul.** `get-img-props.js:257` pune singur
  `unoptimized = true` pe orice `.svg` când `dangerouslyAllowSVG` lipsește.
  Verificat în Next 16.0.7 — nu re-investiga.
- **Scripturile de social citesc din `~/Velto/Social/assets/`**, care avea
  propriile copii ale siglelor. Un logo vechi acolo nu dă eroare — randează
  frumos, cu numele greșit.

Decizia de logo, cu variantele comparate: artifact
`https://claude.ai/code/artifact/d73ece6f-f255-44f8-a59b-ae7a539f47e5`
