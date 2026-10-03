---
name: next-upgrade-and-dead-deps
description: "Next 16.0.7 → 16.3.8 pe 2 oct: de la 23 vulnerabilități în producție (1 critică) la ZERO. next-pwa șters — nu producea nimic. Comis fc3993f, NEPUSHAT. Plus ce a mai rămas mort în proiect."
metadata: 
  node_type: memory
  type: project
  originSessionId: fdf31a5b-0e5d-471f-92a5-d45ad32eda3a
  modified: 2026-10-03T07:58:48.404Z
---

Găsit **din întâmplare**, verificând biblioteca de xlsx pentru etapa 7: proiectul
era pe `next@16.0.7` cu **23 de vulnerabilități în dependențele de producție**,
dintre care 1 critică și 18 „high". După upgrade: **zero**.

Comis ca `fc3993f` pe ramura `chore/next-16.3.8`. ⚠️ **NEPUSHAT** la 3 oct 2026 —
owner-ul voia să pornească întâi `npm run dev` și să se plimbe prin aplicație.

## De ce conta la Veltofit anume

Nu era zgomot de `npm audit`. Două lucruri atingeau direct produsul ăsta:

- **„Middleware / Proxy bypass in App Router applications", în PATRU forme.** La
  noi middleware-ul face TOATĂ rutarea pe subdomenii **și** e singurul lucru care
  ține `/design-system` pe 404 în producție.
- **RCE neautentificat în Image Optimization API pe fișiere AVIF.** Folosim
  `next/image` cu `remotePatterns` deschise către Unsplash și bucket-ul Supabase.

A doua critică era RCE pe servere **Windows** — pe Vercel nu ne atinge. De spus
așa, nu „două critice", ca să nu sune mai rău decât e.

## `next-pwa` — tiparul dependenței moarte

Scos în același comit. Verificat de două ori că nu producea nimic: **nu există
`sw.js`, nu există `workbox-*.js`, nimic din cod nu înregistrează vreun service
worker.** Nu apărea niciodată în `next.config.ts`.

🔑 **Întrebarea owner-ului a fost „nu o să mai funcționeze pus pe home screen?" —
și răspunsul e nu, pentru că `next-pwa` NU face instalarea.** El generează un
service worker pentru offline. Instalarea vine din `public/manifest.json`
(`display: standalone` + iconițe) plus HTTPS, și alea rămân neatinse.

Ce dispare odată cu el: lanțul `workbox-build` → `workbox-webpack-plugin` →
`rollup-plugin-terser` → `serialize-javascript`, adică patru „high" plătite
pentru o funcție inexistentă.

## 🔴 Capcana de reținut la ORICE upgrade de Next

Prima rulare a porții a picat cu erori de tipuri în **`.next/dev/types/`** —
fișiere generate de versiunea VECHE, rămase în cache, care cereau un export
intern redenumit în 16.3.8. **Nu era codul nostru.** `rm -rf .next` înainte de
`type-check`, după orice upgrade.

Versiunea e **fixată exact** (`"next": "16.3.8"`), nu cu `^`, ca `react` și
`react-dom`. `npm install` o pusese cu caret.

## Ce a rămas mort și nu s-a atins

Mai sunt **5 „critice" doar în uneltele de dezvoltare**, tot lanțul `vitest`,
tras de **`@storybook/addon-vitest`** — adică de Storybook, care după CLAUDE.md
§6 nici nu poate porni (nu există `.storybook/`). Opt pachete de balast. Nu ajung
la niciun utilizator, deci n-au fost amestecate cu un comit de securitate —
dar e aceeași poveste ca `next-pwa`, de făcut separat.

Avertismentul `middleware` → `proxy` e **NESCHIMBAT** față de înainte (pe backlog
din 29 aug), iar build-ul raportează în continuare `ƒ Proxy (Middleware)`.
Steagul `--webpack` din `npm run dev` e încă acceptat de 16.3.8 — verificat din
`--help`, fără a porni serverul.

Legat: [[import-clienti-etapa-0]], [[backlog-middleware-proxy-deprecation]],
[[dev-server-runaway-cpu]].
