---
name: supabase-email-templates-brace-parsing
description: "Cele trei săptămâni de șabloane Supabase „salvate dar neutilizate\" aveau cauza în comentariul NOSTRU: acoladele duble se parsează și în comentarii HTML, iar o expresie invalidă face Supabase să cadă TĂCUT pe implicitul englezesc."
metadata: 
  node_type: memory
  type: project
  originSessionId: 7b06e31a-1032-4cbc-89e4-b666f1982911
  modified: 2026-09-14T13:00:57.971Z
---

**Diagnosticat de suportul Supabase (Akash) pe 2026-09-14**, din logurile lor de
auth, după un tichet depus pe 8 septembrie.

## Ce era

Comentariul de sus din `docs/email-templates/reset-password.html` conținea:

    ⚠️ {{ .Data.* }} NU există aici — la resetare nu se trimite metadata.

Un avertisment pus acolo ca să explice că metadata NU e disponibilă la resetare.
`*` nu e un nume de câmp valid, deci expresia nu se poate parsa.

**Serviciul de auth parsează `{{ }}` peste tot în fișier, inclusiv în
comentariile HTML.** Un comentariu nu e ascuns de parser — e ascuns doar de ochi.
Parsarea pica, și Supabase **cădea tăcut pe șablonul lui implicit, în engleză**.

🔑 **Ironia exactă, de reținut: avertismentul care explica o variabilă
inexistentă e chiar cel care a stricat emailul.**

## De ce nicio verificare n-a găsit-o

Trei săptămâni de eliminări au ieșit toate „corect", fiindcă **nimic nu era
stricat în configurație** — se strica la randare:

- panoul arăta textul salvat, cu „Save changes" gri;
- Management API-ul (`GET /v1/projects/{ref}/config/auth`) raporta
  `mailer_templates_recovery_content` cu 8470 de caractere și **toate** flagurile
  din `mailer_*_custom_contents` pe `true`;
- SMTP-ul Resend era în regulă — ba chiar arăta subiectul „Reset your password",
  ceea ce dovedea doar că substituția se face înainte de SMTP;
- fuseseră eliminate pe rând: hook-uri de auth, proiect greșit, filă greșită,
  config-as-code, branching, propagare (retestat la 24h pe cont nou).

**Lecția de metodă:** când fiecare verificare de CONFIGURARE iese curată dar
rezultatul e greșit, defectul e în CONȚINUT, la randare. Nu mai căuta comutatoare.

## Cum se relipește, ca să nu pierzi o oră

Editorul din panou e **Monaco**. Trei capcane, toate întâlnite:

1. **`cmd+A` fără focus în editor selectează toată PAGINA**, iar `cmd+V` nu face
   nimic — și arată exact ca un paste reușit dacă nu verifici. Dă un **click fizic
   pe o linie de cod** înainte; `focus()` prin API raportează `true` dar tastele
   tot nu ajung.
2. **Verifică după paste, nu după impresie:** citește
   `monaco.editor.getModels()[0].getValue()` și compară SHA-256 cu fișierul local.
   De două ori conținutul a rămas cel vechi cu totul.
3. **Uneori cere DOUĂ salvări.** Prima dă toast de succes dar lasă formularul
   „murdar" — se vede doar prin faptul că browserul blochează navigarea cu
   „Leave site?". A doua salvare curăță starea.

⚠️ Nu chema `navigator.clipboard.readText()` în pagină: cere o permisiune care nu
se rezolvă și **îngheață randerul** (CDP timeout la 45 s).

📌 Rutele: `/auth/templates/reset-password`, `/auth/templates/invite-user`,
`/auth/templates/confirm-sign-up` (cu cratime — `confirm-signup` nu există).

## Regula, de aici înainte

În comentarii, numele variabilei se scrie **fără acolade** — `.ConfirmationURL`,
`.RedirectTo`, `.TokenHash`. Acoladele există doar în corpul emailului, unde
chiar vrei substituția. Verificare:

    grep -n '{{' docs/email-templates/*.html

Fiecare rezultat trebuie să fie în corp. Azi: 8 rânduri, 14 expresii, toate în
corp. Regula e scrisă în capul lui `docs/email-templates/README.md`.

## Starea

✅ Cele trei fișiere reparate în repo pe 2026-09-14 (comentarii curățate în toate
trei, nu doar în cel vinovat — Akash a recomandat explicit verificarea celorlalte
două; `confirm-signup-trainer.html` avea `{{ .Data.full_name }}` în comentariu).

✅ **RELIPITE ȘI DOVEDITE 2026-09-14.** Toate trei au fost puse în panou și
verificate prin reîncărcare: amprenta SHA-256 a conținutului din editor e
identică cu a fișierului din repo (`reset-password` 9283 b / `e8eb74a51ca32741`,
`invite-user` 6473 b / `929c4f9e20bd8a63`, `confirm-sign-up` 6622 b /
`0187e51b96b5b30f`).

🔑 **Dovada finală e emailul, nu panoul:** owner-ul a testat ambele căi pe bune —
**resetarea de parolă** și **invitația de client** — și amândouă sosesc în română,
cu logo și buton. Invitația conta cel mai mult: e primul email pe care îl vede
orice client al oricărui antrenor. Subiect închis.

⚠️ **Supabase NU citește din repo** — orice modificare viitoare a fișierelor cere
relipirea manuală. Nu presupune că un commit ajunge acolo.

✅ **Tot ce depindea de asta e livrat, 2026-09-14/15:** invitația clientului în
română (dovedită), resetarea (dovedită), și **confirmarea pe email la înscriere —
PORNITĂ în Supabase și dovedită pe producție** cu un cont de antrenor nou.

🔴 **Capcana de livrare, plătită pe viu:** setarea a fost pornită înainte ca
reparația de cod să fie în producție. `origin/main` încă făcea `signUp` urmat
imediat de `signInWithPassword`, deci orice înscriere ar fi lăsat un cont creat
și inaccesibil, cu eroare brută pe ecran. Reparat prin push (`7054e24..168f050`).
**Un comutator din panou și codul care îl servește sunt două lucruri separate —
livrează codul întâi.** Aceeași formă ca „configurat în Stripe ≠ configurat în
deployment" din [[stripe-integration-state]]; a doua oară când lucrul ăsta ne
costă, deci merită tratat ca regulă, nu ca accident.

⚠️ Soluția de ocolire cu **Send Email Auth Hook** nu mai e necesară pentru asta.
Rămâne relevantă doar pentru emailurile pe care le trimite aplicația singură —
expirare de trial, card picat. Vezi [[notifications-never-worked]] și
[[launch-readiness-plan]].
