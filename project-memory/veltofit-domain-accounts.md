---
name: veltofit-domain-accounts
description: "Ce e legat la domeniul veltofit.app — Google Workspace (MX!), două conturi Google verificate, MailerLite, și DMARC — PUS de owner pe 2026-09-07 (p=none). Descoperit din DNS 2026-09-07. ⚠️ Descrierea veche zicea „DMARC lipsește" și a indus în eroare o sesiune pe 22 sep."
metadata: 
  node_type: memory
  type: reference
  originSessionId: a7ae8f01-3d56-484c-8f25-f0674d4650fb
  modified: 2026-09-07T13:39:11.804Z
---

Owner a întrebat pe 2026-09-07 dacă există un cont Gmail „Velto", fiindcă îl
uitase o dată. Nu era în nicio notă și nu e în repo. **Există.** Citit direct din
DNS-ul live, nu din memorie:

| Înregistrare | Valoare | Ce înseamnă |
|---|---|---|
| MX | `1 smtp.google.com` | **Domeniul are Google Workspace.** Orice adresă `@veltofit.app` ajunge într-o cutie Google. |
| TXT | `google-site-verification=wbGxovVyUkxuzdLQM2G7KjlIcqgNGstJTNjKwoNqRgE` | ✅ **AL LUI.** Vercel îi dă vârsta „Aug 28" — e cel pus când s-a apucat de Search Console. Misterul NU e ăsta. |
| TXT | `google-site-verification=BK3hSmPA6g3V31pn-x9-tzQ2-i1yTI7o78JH3kmWPV0` | 🔴 **AL WORKSPACE-ULUI. NU SE ȘTERGE NICIODATĂ.** Lămurit 2026-09-08: owner-ul a intrat în `admin.google.com` → Manage domains, unde `veltofit.app` apare ca **Primary Domain / Verified / Gmail activated**. Configurarea unui Workspace cere exact o astfel de verificare, iar jetonul ăsta precedă cu mult Search Console-ul din 28 august. Ipoteza „e al lui Robert" a picat. **Ștergerea lui poate invalida verificarea domeniului în Workspace, deci Gmail-ul pe domeniu, deci cutia `support@veltofit.app`.** |
| TXT | `mailerlite-domain-verification=3232643687…` + `include:_spf.mlsend.com` în SPF | MailerLite e legat la domeniu. Nimeni n-a pomenit vreodată serviciul ăsta. |
| SPF apex | ✅ `v=spf1 a mx include:_spf.google.com include:_spf.mlsend.com ~all` | **REPARAT 2026-09-08.** Înainte era `a mx include:_spf.mlsend.com ?all` — fără Google, deși MX-ul arată spre Workspace, deci emailul scris de owner de pe `support@` nu era autorizat. Costă **4 interogări DNS din 10**: ambele `include` sunt „plate", conțin doar IP-uri, fără includeri imbricate (verificat). |
| `_dmarc.veltofit.app` | ✅ `v=DMARC1; p=none; rua=mailto:daniel.vinersar.ux@gmail.com` | **PUS de owner 2026-09-07**, verificat în DNS. `p=none` doar raportează. Rapoartele XML încep să vină în câteva zile la adresa aia. |

**Pentru Resend e totul corect** — `send.veltofit.app` are `v=spf1
include:amazonses.com ~all`, iar `resend._domainkey` există. Antetele confirmă:
`mailed-by: send.veltofit.app`, `signed-by: veltofit.app`. Nu acolo e problema.

✅ **CONDIȚIA E ÎNDEPLINITĂ, 2026-09-08.** SPF-ul apexului include acum Google și
s-a strâns la `~all`, deci **`p=quarantine` nu mai e blocat**. Se pune după câteva
săptămâni de rapoarte `rua`, când se vede că tot ce pleacă legitim trece.
⚠️ Ce s-a schimbat efectiv la trecerea de la `?all` la `~all`: un expeditor care
nu e pe listă era înainte ignorat, acum e marcat suspect. Din DNS reiese că
singurii care trimit sunt Google și Resend, iar Resend are subdomeniul lui.

## Contul de Workspace (spus de owner 2026-09-07)

Adresa este **`daniel.vinersar@veltofit.app`** — creată la înființarea
Workspace-ului, deci aproape sigur **super-admin**. ✅ **Owner a reintrat în cont
pe 2026-09-07**; nu mai trimite pe nimeni pe drumul de recuperare.

Confuzia de bază, care l-a blocat: `daniel.vinersar.ux@gmail.com` și
`daniel.vinersar@veltofit.app` sunt **două conturi Google complet separate**.
Workspace-ul aparține celui de-al doilea; de aceea `admin.google.com` nu arată
nimic când e logat pe Gmail. Nu se „leagă" unul de altul — se intră pe celălalt.

Calea de recuperare garantată, dacă emailul/telefonul de recuperare nu merg:
formularul de recuperare admin cere **dovada că deții domeniul**, printr-un TXT
pus în DNS. DNS-ul e la Vercel și e al lui → drumul ăsta nu poate eșua.

## ✅ Aliasurile EXISTĂ — confirmat vizual 2026-09-07

Pe contul de Workspace sunt trei alias-uri: `contact@`, `support@`, `social@`.
**Asta închide elementul de backlog „`support@veltofit.app` nu e confirmat că
primește"** (era în `docs/plans/backlog.md` și listat ca blocant de lansare în
`docs/plans/launch-readiness-v1.md`). Cutia e reală, ajunge în Workspace.

Împărțirea din cod, numărată nu ghicită:

| Adresă | Unde apare | Aparții |
|---|---|---|
| `contact@` | privacy (12), terms (10), landing FAQ+footer (4), cookies (2) | fața juridică și publică |
| `support@` | cele 3 pagini profile/email, `manual-add-refusals.ts`, `billing-copy.ts` (2) | probleme în aplicație |
| `social@` | nicăieri în cod | Instagram/marketing |

**Nu șterge niciunul.** `contact@` și `support@` sunt deja tipărite în producție,
inclusiv pe facturile Stripe. Un alias șters face mailul să dea bounce.

Ce lipsește: filtre în Gmail per alias. `contact@` e adresa din politica de
confidențialitate, deci o cerere GDPR primită acolo are **termen legal de 30 de
zile**; una la `support@` n-are. Fără filtru, arată identic în inbox.

## ✅ `support@veltofit.app` PRIMEȘTE — confirmat 2026-09-08

Owner-ul a intrat în cont și și-a trimis un email de pe adresa lui de Yahoo. A
ajuns. Rândul de pe backlog care spunea „nu e confirmat că primește" e închis.

**Cutia reală e `daniel.vinersar@veltofit.app`**, iar `support@`, `contact@` și
`social@` sunt ALIASURI pe ea. ⚠️ **Niciun alias nu se șterge**: `support@` și
`contact@` sunt deja tipărite în producție, inclusiv pe facturile Stripe, iar un
alias șters dă bounce — nu redirecționează nicăieri.

Contează mai mult decât părea când a fost scris: adresa nu e doar în cele trei
pagini legale și în mesajele de eroare — e **tipărită pe fiecare factură**, ca
dată de contact a emitentului. E ce vede contabilul clientului.

Workspace-ul: `veltofit.app` e Primary Domain, verificat, cu Gmail activat.
Alături apar și `veltofit.app.guest.google` (Guest Domain) și
`veltofit.app.test-google-a.com` (alias de test) — ambele create automat de
Google, nu de configurat și nu de atins.

DNS-ul se editează în **Vercel** (nameserverele sunt `ns1/ns2.vercel-dns.com`),
Domains → veltofit.app → DNS. Mai multe TXT de verificare pot coexista legal —
**nu se șterge niciunul** fără să se știe al cui e.

Legat: [[landing-seo-baseline]] (Search Console, jetonul neexplicat),
[[add-client-production-config]] (SMTP-ul Resend, configurat 2026-07-22).

## De unde e cumpărat domeniul (lămurit 2026-09-25)

Din **Vercel**, echipa „Daniel's projects" (Hobby), NU din contul personal —
de-asta `vercel.com/account/domains` apare gol. Lista reală:
`vercel.com/daniels-projects-6e492e56/~/domains`. Registrar Name.com (prin el
vinde Vercel). Factura: **9,99 USD, 11 dec 2025**. **Auto-renew pe 11 dec 2026.**
