---
name: dev-server-runaway-cpu
description: FIXED 2 sep by switching to `next dev --webpack` (now the default in package.json) — 1.1GB vs 4.5GB. The 'agents must not write while dev runs' rule was WRONG, it was Turbopack. Two Turbopack failure modes documented; `--webpack` is deprecated, so retest turbo after each Next upgrade. Slow one: left up for days, leaks to ~10GB, self-restarts, worker spins at ~6.5 cores. Fast one (~70 min): running `npm run build` while dev is up on the same .next. How to tell them apart.
metadata:
  type: project
---

Observed 2026-08-28: `next-server` at 668% CPU. Root cause chain, confirmed from evidence:

1. `next dev` (Next 16.0.7, Turbopack) had been up since 2026-08-27 ~16:00. V8 heap climbed to **10.4–10.9 GB** against a 13 GB limit.
2. Next restarts its own dev worker at that point — the trace contains `server-restart-close-to-memory-threshold`. It fired twice, ~6 min apart. 14 `start-dev-server` events in one trace file.
3. The restarted worker then **spun**: ~6.4 cores burned continuously, 89% of `sample` frames running inside `next-swc.darwin-arm64.node` (Turbopack's Rust side), while serving **zero** requests and compiling **zero** pages. Not a cold compile — a hang.

## Second mode, found 2026-08-31: `npm run build` while `next dev` is up

Same symptom, different cause, and **much faster** — ~70 minutes instead of days.
`next-server` at **520% CPU / 5.4 GB RSS**, system load climbing 2.5 → 4.3 → 8.1.

**How to tell it apart from the memory-threshold loop above:** grep the trace.

    server-restart-close-to-memory-threshold : 0     ← ZERO. Not the slow mode.
    start-dev-server                          : 8
    compile-path                              : 50

Zero threshold restarts and 5.4 GB (well under the 13 GB limit) rule out the leak.
Eight dev-server starts and fifty compiles in one short session is the tell: the
server was being invalidated over and over, not hanging.

**Suspected amplifier, NOT the cause — corrected 2026-08-31, same session.**
The first guess was `npm run build` run ~6 times WHILE `next dev` was up on the
same `.next`: a production build rewrites that directory wholesale, the dev
server watches it, and recompiles everything it holds.

**That theory was disproved an hour later.** After `rm -rf .next` and a clean
restart, with **zero builds run against it**, the same server reached
**337% CPU / 4.5 GB in about 15 minutes**. Builds cannot explain it. Whatever
this is, a cold start plus ordinary use is enough — and this repo's known
amplifiers (the landing's ~950 `?_rsc=` prefetches, the dev image optimizer
regenerating 13 `lp-*.png` at several widths on every cold start) all fire
hardest on exactly a cold start, which is what `rm -rf .next` guarantees.

Still worth avoiding builds over a live dev server, but do not record it as the
root cause. **The honest summary: `next dev` on this repo becomes unusable
within tens of minutes and nobody has found out why.**

⚠️ Note for whoever automates the gate: `distDir` is a `next.config.ts` option,
not a CLI flag, so there is no quick "build somewhere else" escape. Stopping the
dev server is the fix.

**Where the evidence lives** (non-obvious):
- `.next/dev/trace` — JSONL, one array per line. Grep for `server-restart-close-to-memory-threshold`, `start-dev-server` (has `memory.freeMem`/`heapSizeLimit` tags), `compile-path`, `handle-request`.
- `ps -o %cpu` is a lifetime average — useless here. Measure `ps -o time=` delta over 15s for the real number.
- `sample <pid>` then look for `tokio-runtime-worker` threads: all Turbopack.

**Fix:** kill the `next dev` parent + worker, `rm -rf .next`, restart. Don't leave `next dev` up for days.

**Amplifiers on this repo:** landing `/` fires ~950 `?_rsc=` prefetches; the dev image optimizer regenerates 13 `lp-*.png` at several widths on every cold start; several browser tabs on :3000 reconnect at once after a restart. See [[client-home-perf-findings]] for the prefetch-churn measurement.

**Also found:** a stale `npm exec next start -p 3100` (from `capture:marketing`) running since 2026-08-18 on the same `.next` folder. Idle, but kill it.

## Third occurrence, 2026-09-02 — the first time the trigger is visible in the log

`next-server` pid 30422, up 1h40, **5.95 real cores** (ps time-delta over 15s) and
4.1–5.0 GB RSS. Fast mode again, not the leak: `server-restart-close-to-memory-threshold`
= **0**, 3 `start-dev-server`, 20 `compile-path`, 4.1 GB against the same 13 GB limit.
`sample`: 19.6k of ~23.6k frames in `next-swc.darwin-arm64.node`, tokio workers busy,
while `.next/dev/trace` had not been written for **19 minutes**.

**New: the dev log puts the landing page immediately before the blowup.**
Uptime clock, from `.next/dev/logs/next-development.log`:

    00:50–00:51  ✓ Compiled in 30ms / 32ms / 9ms / 18ms / 10ms   ← normal HMR, dashboard work
    01:22:12     browser opens http://localhost:3000/            ← the LANDING, first time this session
    01:38:01     ✓ Compiled in 725.5s                            ← 12 minutes for one compile

Nothing else was logged between those last two lines. Work up to that point had been on
`(dashboard)/dashboard/profile/billing` with millisecond compiles; the only new input was
the landing tab. Chunks kept being written and ~6 cores kept burning for minutes *after*
that compile reported done.

**Correlation, not proof** — two dashboard tabs were also open, and one sample cannot
separate the landing's ~950 `?_rsc=` prefetches from the dev image optimizer (the
`.next/dev/cache/images` entries were re-written in the same window). But it is the first
time the record has the trigger and the symptom in one timeline, and it points at the same
two amplifiers already listed above. **Next time: note whether `/` was opened, before
anything else.**

Practical rule while it stays unexplained: don't leave a tab on the landing (`localhost:3000/`)
open while working on the app or dashboard subdomains.

The server disappeared on its own at ~11:36 (log stops 11:32, no crash line, no jetsam,
port released) — stopped, not OOM-killed.

## A patra, 2026-09-02 dupa-amiaza — cea mai curata secventa de pana acum

Owner-ul: aplicatia a mers foarte greu, s-a blocat, memoria s-a umplut, Mac-ul
s-a incalzit, au pornit coolerele. Log-ul lui, in ordine:

    GET /login                      398ms
    GET /                           293ms    <- LANDING-UL, rapid
    GET /login?mode=signup&plan=..   33.5s   <- cererea IMEDIAT urmatoare
    GET /dashboard                   34.9s
    GET /dashboard/complete-profile  22.1s
    server approaching memory threshold, restarting

**Landing-ul s-a servit in 293ms si cererea urmatoare a durat 33.5s.** Din alea,
compile = 3.0s, render = 30.5s. Deci NU compila — serverul era sufocat de ce
porneste landing-ul DUPA ce se serveste: ~950 de precarcari `?_rsc=` plus
optimizatorul de imagini. De aia pagina e rapida si tot ce vine dupa nu e.

Asta confirma corelatia din a treia observatie, dar mai strans: acolo erau 16
minute intre deschiderea landing-ului si blowup; aici e URMATOAREA cerere.

**O forma noua, care nu se potriveste cu niciunul din cele doua moduri:**

    server-restart-close-to-memory-threshold : 1     <- modul lent are >0, dar dupa ZILE
    start-dev-server                          : 3
    compile-path                              : 18

Modul lent ajunge la prag dupa ~24h si 10GB. Aici pragul a fost atins intr-o
sesiune scurta, cu 18 compilari. Deci nu e nici leak-ul lent, nici invalidarea
repetata — e **saturare rapida pornita de o singura vizita pe landing**.

⚠️ Amplificator adaugat de mine: am rulat `npm run build` peste serverul viu
(cu acordul owner-ului, dar fara sa cantaresc ca teoria a fost doar *disproved
ca fiind cauza*, nu ca fiind inofensiva).

🔴 **Lectie de proces:** procedura de test pe care i-am scris-o incepea cu
"deschide localhost:3000" si NU spunea sa inchida tabul dupa. Regula era deja
scrisa mai sus in fisierul asta. Cand scrii o procedura manuala pentru repo-ul
asta, verifica-l intai.

## 2026-09-02, dupa a patra: TEORIA PRECARCARILOR E GRESITA

Verificat in cod, nu presupus. `src/app/(marketing)/page.tsx`:

    <Link>            : 3      (toate in subsol: /privacy, /terms, /cookies)
    <a>               : 23     (ancore simple — Next NU le precarca)
    useEffect         : 5      (toate cu lista de dependente)
    scroll/mousemove/rAF/setInterval : 0

**Trei linkuri nu pot produce ~950 de cereri `?_rsc=`.** Ar insemna ~317
re-randari, adica o bucla — si nu exista una: singurul efect care ar putea
naviga (`isAuthenticated && role==='client'` -> `router.replace`) e explicat in
cod ca practic nedeclansabil pe originea aia, fiindca sesiunea Supabase e in
localStorage per-origine si nimic de pe ruta asta nu cheama `fetchUser`.

Cifra de ~950 venea din [[client-home-perf-findings]], unde fusese masurata pe
**client-home**, si a fost atribuita landing-ului fara verificare. S-a repetat
apoi din sesiune in sesiune ca fapt.

**Balast gasit pe drum, dar NU cauza:** `public/images/hero-image.png` are
**5,4 MB, 2832x1956** — de 15-50x mai mare decat orice alta imagine din repo.
Nu e folosita de landing: apare doar ca imagine OG in `src/app/layout.tsx:56,69`,
iar `opengraph-image.tsx:12` spune ca a inlocuit-o. Imaginile din metadate nu
trec prin optimizator. Merita stearsa oricum.

**Suspectul care ramane, netestat:** optimizatorul de imagini din dev. 9
`<Image>` pe landing, `next.config.ts` nu restrange formatele, deci avif+webp la
mai multe latimi, la cerere. Se potriveste cu cronologia (porneste DUPA ce
pagina s-a servit) si cu profilul CPU. **Dar e o ipoteza.**

🔬 **Masuratoarea care ar lamuri, cand se mai intampla:** in log-ul de dev,
numara cererile `/_next/image` in fereastra de blocaj, si uita-te daca
`.next/dev/cache/images` creste in acelasi timp. Daca da, e optimizatorul si
fixul e trivial (restrange `formats` la webp in dev, sau pre-genereaza).
Daca nu — suspectul cade si el, si ramanem fara ipoteza, ceea ce e o pozitie
mai onesta decat cea de acum o saptamana.

## 2026-09-02 seara: NU se reproduce — si asta reabiliteaza teoria cu build-ul

Owner-ul a repornit serverul curat (dupa `rm -rf .next`), a pornit scriptul de
masurare, a deschis landing-ul — **si nu s-a mai blocat.**

Deci "o vizita pe landing" NU e suficienta. Ipoteza care era in frunte dupa a
patra observatie slabeste.

**Ce a fost diferit la blocajul de dimineata:** rulasem `npm run build` peste
serverul viu, la cererea owner-ului (verificare de `<Suspense>`). Blocajul a
venit dupa. Acum, fara niciun build de la repornire, serverul e sanatos.

⚠️ Teoria build-ului fusese marcata "disproved" pe 31 august fiindca problema
s-a reprodus si FARA build. Dar **"se poate reproduce si fara" nu inseamna "nu
contribuie".** Poate fi un amplificator care singur nu explica modul lent, dar
care e suficient sa declanseze modul rapid. Cele doua observatii nu se exclud:

  31 aug — fara build, dupa `rm -rf .next`  -> s-a blocat in ~15 min
  02 sep dimineata — CU build peste serverul viu -> s-a blocat
  02 sep seara — fara build, landing deschis    -> NU s-a blocat

Al treilea rand e nou si e cel care conteaza: e prima data cand landing-ul e
deschis pe un server pe care nu s-a rulat build, si nu se intampla nimic.

**Experiment in curs, cost zero:** nu mai rulez `npm run build` peste serverul
viu. Deloc. Daca in urmatoarele zile nu se mai blocheaza, "disproved" a fost o
concluzie trasa prea repede dintr-o singura contra-observatie.

🔑 Regula practica pana atunci, pentru oricine automatizeaza poarta: **build-ul
si `next dev` nu coexista.** Nu ca amplificator suspectat — ca regula.

## 2026-09-02, 21:47 — PRINS LIVE, cu unelte. Cea mai buna ipoteza de pana acum.

Prima data cand blocajul e masurat in timp ce se intampla, nu reconstituit dupa.

**Procesul real e COPILUL.** `pgrep -f 'next-server|next dev' | head -1` prinde
invelisul (pid parinte), care consuma 0,11s CPU in 30 de minute. Muncitorul e
copilul lui, `next-server (v16.0.7)`. Scriptul meu de masurare a urmarit 28 de
minute procesul gresit si a scos zerouri. **Foloseste `ps -Ao pid=,ppid=,rss=,time=`
si sorteaza dupa CPU total, nu pgrep+head.**

Masurat pe procesul corect (pid 42065, uptime 30 min):

    4,49 nuclee reale (delta ps time pe 15s)
    4,46 GB RSS
    sample: 21.540 din ~25.000 de cadre in next-swc.darwin-arm64.node

Trace:

    server-restart-close-to-memory-threshold : 0
    start-dev-server                          : 1
    compile-path                              : 3     <- DOAR TREI
    ultima scriere                            : cu 5 min inainte, apoi inghetat

**Nu compileaza in bucla. O SINGURA recompilare s-a agatat.** Consola browserului
confirma: ultima linie e `[Fast Refresh] rebuilding`, si nu se mai termina.

**Sase procese `webpack-loaders.js` pornite TOATE in aceeasi clipa**, cu 5:26
inainte de masuratoare — exact cand s-a inghetat trace-ul. Sunt inactive
(0,3-0,9s CPU fiecare); parintele arde nucleele.

### DOUA IPOTEZE ELIMINATE

**Optimizatorul de imagini: MORT.** `.next/dev/cache/images` avea **140 KB, 3
fisiere**, si `grep -c '_next/image'` in log dadea **0**. Zero cereri de imagini
in toata sesiunea. Nu e el.

**Landing-ul singur: NU E SUFICIENT.** Owner-ul l-a deschis pe un server curat si
nu s-a intamplat nimic timp de ~25 de minute.

### IPOTEZA NOUA, care se potriveste cu TOATE observatiile

**Fisiere scrise in `src/` in timp ce serverul ruleaza** — adica agenti care
lucreaza sub un `next dev` pornit. Fiecare scriere declanseaza Fast Refresh;
una dintre recompilari se agata si serverul se pineaza.

| | server pornit | agenti scriau fisiere | rezultat |
|---|:--:|:--:|---|
| 31 aug | da | da | blocat |
| 2 sep dimineata | da | da | blocat |
| 2 sep 21:20-21:45 | da | **nu** | **liniste** |
| 2 sep 21:47 | da | **da** (qa scria teste) | blocat |

Randul al treilea e cel care lipsea pana azi: server pornit, landing deschis,
**nimeni nu scria fisiere** -> nimic. Al patrulea inchide argumentul: qa-agent a
inceput sa scrie teste, si in aceeasi clipa au aparut cei sase loaderi.

Explica si de ce "teoria build-ului" parea infirmata pe 31 august: acolo NU s-a
rulat build, dar agentii scriau. Variabila comuna nu era build-ul — era scrisul.

🔑 **REGULA PRACTICA, pana la o dovada mai buna: `next dev` si agentii care scriu
fisiere nu coexista.** Ori serverul e oprit cat lucreaza agentii, ori se accepta
ca se va bloca. Nu e o precautie — e singura variabila care se potriveste cu
toate cele patru observatii.

⚠️ Inca e corelatie, nu cauza. Ce ar dovedi-o: server curat, deschizi ce vrei,
NU scrie nimeni in `src/` timp de o ora. Daca ramane sanatos, e confirmat.

---

## 2 sept, 22:00 — REPARAT, si regula de mai sus era gresita

Trecut pe `next dev --webpack`. Dupa **45 de minute**, cu agenti scriind activ
fisiere in `src/` tot timpul:

    1.14 GB RSS  ·  1:38 timp de procesor  ·  ~3.6% dintr-un nucleu

Fata de Turbopack, pe aceeasi masina si aceeasi zi: **4.49 nuclee, 4.46 GB**, si urca.

### Randul care rastoarna tabelul de mai sus

| | server | agenti scriau | motor | rezultat |
|---|:--:|:--:|:--:|---|
| 2 sep 21:47 | da | da | turbopack | blocat |
| **2 sep 21:15-22:00** | **da** | **da** | **webpack** | **liniste** |

🔑 **Regula „`next dev` si agentii care scriu fisiere nu coexista" era o corelatie,
nu cauza.** Scrisul doar declansa recompilarea; ce se agata era Turbopack. Pe
webpack, aceleasi scrieri, acelasi landing, zero probleme. **Nu mai opri serverul
cat lucreaza agentii** — asta era tratarea simptomului.

Ce ramane valabil din regula veche: **nu rula `npm run build` peste un `next dev`
pornit.** Aia e o coliziune pe acelasi director `.next`, independenta de motor.

### Unde e reparatia

`package.json` — `dev` poarta acum `--webpack`; Turbopack a ramas ca `dev:turbo`.
Owner-ul o scrisese de mana in terminal, deci nu supravietuia unui `npm run dev`
scris din obisnuinta.

⚠️ **`--webpack` are termen de expirare.** Next 16 il marcheaza depasit si il va
scoate. Cand dispare, singura iesire e Turbopack reparat — de aceea contorul de
mai jos conteaza.

### De verificat la fiecare actualizare

Instalat **16.0.7**; publicat la 2 sept: **16.3.4**. Adica ~30 de versiuni de
corectii nevazute, printre care aproape sigur si asta. Dupa orice actualizare:
porneste `npm run dev:turbo`, lasa agentii sa scrie o ora, si masoara. Daca e
curat, `dev` se muta inapoi pe Turbopack si `--webpack` dispare.

Nu am actualizat acum: 30 de versiuni peste un proiect care merge, in seara
dinaintea lansarii, se face cand exista timp de reparat ce se strica.
