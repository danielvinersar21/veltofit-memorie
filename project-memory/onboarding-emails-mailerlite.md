---
name: onboarding-emails-mailerlite
description: Onboarding emails for new trainers via MailerLite — code + privacy PUSHED 2026-09-22 but INERT until the owner sets MAILERLITE_API_KEY + MAILERLITE_TRAINERS_GROUP_ID (Production). Switch-on checklist in the body.
metadata: 
  node_type: memory
  type: project
  originSessionId: 93a5f689-8aef-49ab-bdcb-53f72a7d8d01
  modified: 2026-09-23T13:13:01.896Z
---

Owner's words: emails „prietenoase ca trainerul să se simtă bine, inclus, ca într-o comunitate" —
not newsletters. Decided 2026-09-22 (AskUserQuestion):
- **Where:** MailerLite automations (already linked to the domain; owner edits emails visually).
- **Sender:** „Daniel de la Veltofit", replies go to the owner.
- **Data to MailerLite:** name, email, package only (no city, provenance, client counts).
- **Who:** only trainers who become trainers from now on; not exempt friends, not demo; the three
  organic ones get a personal email from the owner instead.
- Copy: first person only about building, never about coaching ([[owner-never-a-personal-trainer]]);
  content-reviewer before activation. Privacy policy must add MailerLite (it wasn't listed; the old
  `/api/subscribe` MailerLite route was dead and a spam vector — removal proposed).

Also 2026-09-22: the owner's test account's DB subscription + 99 RON payment are Stripe TEST-mode
(written by local testing into the prod DB) — nothing real to invoice; the live one is a trial he
cancels in the Stripe Dashboard, not via the app. See [[billing-payments-ledger]].

## Built and PUSHED 2026-09-22 (d12ccc9 code, 331818e privacy/terms, 1265e7f docs) — inert until the env vars are set
`src/lib/api/mailerlite.ts` + `src/features/onboarding/onboarding.server.ts`, hooked into the
trainer-created route and the webhook's after(). Dead `/api/subscribe` deleted. API behaviour
checked against developers.mailerlite.com the same day (unsubscribed can't be reactivated via API
without `resubscribe:true`, never sent; PUT with `groups` removes unlisted groups — fields-only).
Privacy policy names MailerLite (§6, §7, §8, §9) + §1 / Terms §13 operator lists.

**Before the owner switches it on (env vars MAILERLITE_API_KEY + MAILERLITE_TRAINERS_GROUP_ID,
Production only, AFTER the privacy page is live):** in MailerLite check (1) open/click tracking —
if on, turn it off or add two sentences to §7/§8 (content-reviewer gave the wording); (2) every
automation email has the unsubscribe link; (3) emails are welcome + app tips, not upsell —
through content-reviewer before activation; (4) double opt-in for API subscribers OFF; (5) old
waitlist subscribers from `/api/subscribe` in the account — decide (backlog); (6) MailerLite DPA
in force for the PFA account, hosting region, non-EEA sub-processors. Create the group, the text
field key `pachet`, automation trigger "joins group" without re-entry.
Runbook: to keep someone out, UNSUBSCRIBE them in MailerLite — never delete the subscriber.

## Contul MailerLite, citit pe 22 sep (Chrome, fără să schimb nimic)
- Plan Free. **Contul NU e confirmat**: pasul 3 „Confirm account” din Dashboard e nefăcut, iar
  Subscribe settings dă 403 „Account is not active yet”. Fără confirmare nu pleacă niciun email.
  MailerLite verifică manual formularul, deci durează. E PRIMUL pas și îl completează owner-ul.
- Curat: 1 abonat (owner-ul, iul 2025), 0 grupuri, 0 automatizări, 0 dezabonați/neconfirmați/
  respinși. Punctul (5) de mai sus (abonați vechi de pe waitlist) → nu există, închis.
- Câmpuri: doar cele implicite. `pachet` nu există încă.
- **Niciun sending domain adăugat în cont**, deși DNS-ul veltofit.app are deja TXT-ul
  `mailerlite-domain-verification`, `include:_spf.mlsend.com` în SPF și DKIM `litesrv._domainkey`
  → mlsend.com. Probabil setate pentru un cont anterior/Classic. La adăugare, verifică dacă
  MailerLite cere alt TXT de verificare.
- Expeditorul implicit: „Daniel Vinersar” <daniel.vinersar.ux@gmail.com>. Trebuie „Daniel de la
  Veltofit” pe o adresă @veltofit.app.
- Urmărirea deschiderilor e PORNITĂ, și la campanii, și la automatizări (punctul 1). UTM: gol.
- ✅ 22 sep, mai târziu: owner-ul a trimis formularul „Confirm account” (No subscribers, site
  veltofit.app, adresa `daniel.vinersar@veltofit.app`, a confirmat emailul). Domeniul veltofit.app
  → **Authenticated** (a mers prin conectarea MailerLite cu Vercel DNS; a pus un al doilea TXT
  `mailerlite-domain-verification=832b…`, primul `3232…` e acum redundant, inofensiv). Contul
  rămâne „not active yet” până la revizia MailerLite: „within 24 hours”, anunț pe email.
- ✅ 22 sep, aprobat de owner și verificat după reîncărcare (Default settings): expeditor
  „Daniel de la Veltofit” <daniel.vinersar@veltofit.app>; Track opens OPRIT la campanii ȘI la
  automatizări (deci §7/§8 din politică rămân adevărate); Company details = „VINERSAR ILIE-DANIEL
  PFA / B-dul Bucureștii Noi 136, Sector 1, București / Romania”. Company profile (nume, adresă,
  oraș) are încă Alba Iulia: formularul are câmpul anti-bot `talon`, deci îl editează owner-ul.
- ✅ 22 sep: owner-ul a completat Company profile (adresa PFA completă, București). Eu am creat
  grupul **„Antrenori noi”, id `199323107069002817`** (→ MAILERLITE_TRAINERS_GROUP_ID) și câmpul
  **„Pachet”, TEXT, tag `{$pachet}`** = cheia `pachet` din cod. Verificate în listă.
- ✅ 23 sep: **contul APROBAT de MailerLite, automatizarea „Bun venit antrenori” e ACTIVĂ**
  (buton „Pause”, Started 0). Double opt-in pentru API: OPRIT (verificat). Owner-ul a citit
  ambele emailuri pe un test real primit pe Gmail, a schimbat butonul pe albastrul aplicației
  și a ajustat textele; eu am pus diacriticele lipsă și am readăugat cererea explicită de
  răspuns în emailul 2. Subsolul: fraza implicită în engleză („you signed up on our website or
  made a purchase from us”) era și falsă — înlocuită cu „Primești acest email pentru că ți-ai
  făcut cont de antrenor în Veltofit.”, link „Dezabonează-te”, numele PFA nu se mai repetă,
  force-update bifat ca să intre și în emailurile automatizării. Adresa PFA rămâne în subsol
  (cerută de MailerLite, semnal anti-spam, identificare expeditor — decis 23 sep).
- ✅ **23 sep: PORNIT ȘI DOVEDIT CAP-COADĂ ÎN PRODUCȚIE.** Owner-ul a făcut tokenul
  („Veltofit App (Vercel Production)", All IPs allowed — corect, Vercel nu are IP fix) și a pus
  variabilele în Vercel, **doar Production**, cheia pe Sensitive, apoi Redeploy.
  ⚠️ Capcană găsită atunci: `MAILERLITE_API_KEY` **exista deja** în Vercel de la waitlist-ul vechi
  (12/15/25), pe *All Environments*, cu un token mort de pe contul vechi — deci se EDITEAZĂ, nu se
  adaugă. Sora ei `MAILERLITE_GROUP_ID` e alt nume decât `MAILERLITE_TRAINERS_GROUP_ID`, n-o citește
  nimic (există și un test pentru asta) — rămasă în Vercel, se poate șterge.
  Test real cu `daniel.vinersar.ux+test1@gmail.com`: 16:08:14 adăugat în grup **via api**, Pachet
  `Free`, 16:08:19 automatizarea a trimis „Bun venit în Veltofit". Deci codul — nemaiîncercat până
  atunci pe contul real — funcționează exact cum e documentat.
  Momentul declanșator, de reținut: **salvarea formularului complete-profile**
  (`features/profiles/components/complete-profile-form/index.tsx:248` → ruta owner-alerts), NU
  crearea contului.
- (starea veche) automatizarea a fost construită întâi NEACTIVATĂ: Trigger
  „Joins group(s) → Antrenori noi”; Email 1 „1. Bun venit” (subiect „Bun venit în Veltofit”,
  buton „Deschide Veltofit” → https://app.veltofit.app); pauză 7 zile; Email 2 „2. Cum merge”
  (subiect „Cum ți se pare?”, fără buton, cere răspuns). Ambele: expeditor „Daniel de la
  Veltofit”, limbă Română, opens tracking OFF, preheader pus.
  🔄 **Direcția textelor s-a schimbat 2026-09-23:** owner-ul a oprit varianta „ghid în 3
  emailuri” („ne ducem într-o zonă în care explicăm și instruim trainerul”). Acum: 1 = bun
  venit, 2 = follow-up la o săptămână. Textele de ghid (planuri, ce vezi după antrenament),
  verificate în cod, sunt parcate în scratchpad-ul sesiunii.
- 🔧 Editorul „Simple editor” din MailerLite, capcanele care au costat ~30 de pași:
  (1) **prima apăsare de Enter după ce scrii într-un bloc e înghițită** — folosește DOUĂ
  Enter-uri ca să faci un paragraf nou; (2) **clicul în interiorul unui bloc de text NU mută
  cursorul** (Enter-ul apoi acționează la finalul documentului) — scrie strict liniar, de sus
  în jos, nu te întoarce; (3) după inserarea unui buton cu `/`, textul tastat intră în
  eticheta butonului: eticheta se schimbă cu triple-click pe ea; (4) `cmd+a` de două ori
  selectează tot documentul (util ca să ștergi și s-o iei de la capăt); (5) primul rând scris
  după ce golești tot ajunge în „Title (optional)” — de-asta „Salut,” e titlu (apare ca titlu
  mare) în ambele emailuri; e ușor de schimbat, dar trebuie știut.
- ⏭️ Ordinea rămasă: textele (content-reviewer) → automatizarea „joins group” ACTIVĂ → abia apoi
  tokenul API + cele două variabile în Vercel (Production). Invers, antrenorii intrați în grup
  înainte să existe automatizarea nu primesc nimic: trigger-ul nu se aplică retroactiv.
- 🔧 Capcană de unealtă, REZOLVATĂ: clicurile din extensie merg, dar coordonatele sunt în cadrul
  ULTIMEI capturi (1568 px lățime), nu în pixeli CSS (2560 pe monitorul owner-ului). Fă o captură,
  ia centrul elementului din `getBoundingClientRect()`, înmulțește cu `1568/innerWidth`, apoi dă
  click. Asta explică „Complete”/„Resolve” care „nu se deschideau”. Nota veche de dedesubt rămâne
  pentru metoda cu fetch.
- 🔧 Capcană de unealtă (veche): în dashboard-ul MailerLite, clicurile și tastarea din extensia Chrome NU
  ajung (butoanele de modal „Complete”/„Resolve” nu se deschid, câmpurile nu primesc focus). Ce
  merge: formularele Laravel (`_token` + `_method=PUT`) trimise din pagină cu `fetch(form.action,
  {method:'POST', body:new FormData(form)})`, apoi verificat după reîncărcare. Formularele cu
  `talon` (anti-bot) NU se trimit așa: le face owner-ul.
- Footer-ul (Company details) era „Str. G Sincai, Alba Iulia, Romania” (schimbat, vezi mai sus). NU e sediul PFA din
  paginile legale (B-dul Bucureștii Noi 136, Sector 1, București) și apare în fiecare email.
  Decizia owner-ului.
