---
name: import-clienti-etapa-0
description: "Importul de clienți: 0,1,2,2b,3,4,6,7 LIVRATE — comise 2–3 oct, NEPUSHATE. Antrenorul lipește SAU trage fișierul Excel, alege cine ocupă locurile, trimite invitația. Rămâne DOAR etapa 5, blocată pe limita de trimitere."
metadata: 
  node_type: memory
  type: project
  originSessionId: fdf31a5b-0e5d-471f-92a5-d45ad32eda3a
  modified: 2026-10-01T12:53:08.884Z
---

Owner-ul a ales pe **29 septembrie 2026** să construiască importul din Excel,
contra ordinii pe care o propusesem eu (raportul întâi). **Avea dreptate**, și
motivul pe care l-a dat („am discutat mult despre el") era mai slab decât cifra
care îl susține:

🔑 **Din 4 antrenori înscriși, 1 singur a adăugat vreodată un client.** E cea mai
îngustă strâmtoare din toată pâlnia. Raportul contează doar pentru cei care au
trecut de intrare; trei din patru nu trec. **Importul atacă exact numărul care
blochează totul.** Eu prioritizasem ce se vede în cod în loc de ce se vede în
admin — de reținut ca tipar.

## Ce e LIVRAT (29 sep – 1 oct)

| etapă | ce | unde |
|---|---|---|
| 0 | „Arhivat" → „Inactiv", trei straturi | prod |
| 1 | tabela `client_contacts` + ramura 2 în numărătoare | prod |
| 2 | contactele în listă, adăugare manuală, ștergere | main |
| 2b | `set_contact_active` / `set_contact_inactive` — **DOAR în bază** | prod, ⚠️ fără interfață |
| 3 | parser de lipire, endpoint de verificare, modal, `import_client_contacts` | prod + main |
| 6 | textele legale | main |
| 2b (interfața) + 4 | comutatorul Activ⇄Inactiv pe rând + „Trimite invitația" | `37c7cbc`, pe main, **NEPUSHAT** |
| 7 | tragi fișierul `.xlsx`/`.csv` | `cf14c31`, ramura `feat/import-xlsx`, **NEPUSHAT** |

**Rămâne DOAR etapa 5** (invitații în masă) — blocată pe tema owner-ului: cifrele
din **Supabase → Authentication → Rate Limits** și **Resend → Usage**. Numărul
decide 3/minut vs 3/10 minute, deci n-are rost cod înainte. Nu există planificator
în proiect (fără Vercel Cron, fără `pg_cron`), deci coada se golește din browser.

## Etapele 4 și 7 — ce s-a învățat (2–3 oct)

🔴 **Ruta de invitație trebuie să verifice că adresa e a contactului.** Fără asta,
o cerere care împerechea un `contactId` propriu cu adresa altcuiva crea contul
pentru acea adresă și ștampila `converted_client_id` pe contactul NEPOTRIVIT —
care intra în regula de dedublare și **dispărea definitiv din listă**, cu tot cu
butoanele lui. Nu era scurgere între antrenori (proprietatea se verifica), dar
betonea un rând. Din interfață nu se putea ajunge acolo; **ruta e autoritatea, nu
dialogul**.

🔑 **Invitația SARE verificarea de plafon când are `contactId`** — operația
eliberează un loc, nu consumă unul. Fără asta, un antrenor cu pachetul plin de
contacte era refuzat la fiecare apăsare, inclusiv pentru contactele care țineau
chiar ele locurile. Spec-ul numea asta „inutilizabilă din prima zi".

🔴 **Emailul e opțional la un contact, deci „fără adresă" e cazul FRECVENT**, nu
marginal — multe liste au doar nume și telefon. Și nu exista NICIO cale de a edita
un contact. Un buton gri cu „adaugă-i adresa" ar fi arătat spre un ecran
inexistent, fix ce interzice antetul lui `contact-copy.ts`. Soluția: dialogul cere
adresa pe loc și o scrie pe rând înainte de trimitere.

**La etapa 7, partea grea n-a fost citirea fișierului, ci UNDE ÎNCEPE TABELUL.**
La lipire întrebarea nu există — omul selectează cu mouse-ul. Un fișier are titlu
în A1, rând gol sub el, uneori trei foi. Euristica: tabelul începe la primul rând
a cărui lățime e cea mai frecventă din foaie (aceeași „agreement" ca la alegerea
separatorului). Poate fi păcălită, deci ecranul SPUNE ce a sărit.

⚠️ **Zeroul telefoanelor e pierdut de Excel ÎN FIȘIER**, nu de noi: `0721…` într-o
celulă neformatată ca text ajunge `721234567`, și ecranul antrenorului arată la
fel. **Nu-l punem la loc** — ar fi un zero necitit de nicăieri. Îl numărăm și
spunem ambele ieșiri.

**Trei capcane pe care build-ul nu le-ar fi prins niciodată:** `read-excel-file`
nu publică export `"."` (trebuie `/browser`, iar importul fiind dinamic ar fi
crăpat abia la primul fișier tras); în v9 `readXlsxFile` întoarce MEREU toate
foile, `{ sheet }` e ignorat; `Blob.text()` lipsește din Safari dinainte de 2020,
deci un `.csv` pe un telefon vechi ar fi eșuat tăcut (cale de rezervă cu
`FileReader`).

🔴 **„Migrare aplicată" ≠ „funcție existentă", și rândul 2b a mințit exact așa.**
Pe 1 octombrie, căutând în cod, cele două RPC-uri erau în producție și **nimic din
browser nu le chema**: singurul buton de pe un rând de contact era „Șterge", iar
`contact-copy.ts`, ambele pagini de clienți și `seat-wall-archive` purtau încă
`TODO(stage 2b)`. Nota asta scria „prod" fără să spună despre ce jumătate, și se
citea ca „gata". Golul nu era nici pe backlog. **Consecința reală:** importul
alege cine ocupă locurile **după ordinea din lista lipită**, deci un antrenor pe
Free care lipește 20 de nume primește activi rândurile 1 și 2 — iar dacă clientul
lui important e rândul 37, singura cale să-l aducă în față era să ȘTEARGĂ
contacte. Asta e chiar întrebarea „care 2?" din decizia 10: importul răspunde
prin ordine, comutatorul e locul unde antrenorul poate răspunde altfel. Owner-ul
a ales pe 1 oct să bage interfața lui 2b în aceeași lucrare cu etapa 4.
**Tiparul de reținut: când o etapă are și SQL și interfață, nota trebuie să spună
care jumătate a plecat.**

## 🔴 Două bug-uri de PRODUCȚIE găsite testând cu un fișier dezordonat (3 oct)

Amândouă erau în `main` **de pe 1 octombrie**, de la importul prin lipire. Niciunul
n-ar fi fost găsit cu date curate. Tiparul: **fișierul de test trebuie să arate ca
al unui om, nu al unui programator** — titlu, rânduri goale, dublură, și un email
scris greșit.

**1. O adresă strâmbă îngheța pagina.** `checkImportAccounts` aruncă adresele pe
care `isCheckableAddress` le respinge ÎNAINTE de cerere, deci „cristi@" nu venea
niciodată în răspuns, iar magazinul (corect) nu-i scria niciun verdict. Dar „fără
răspuns" însemna două lucruri — *n-am întrebat* și *am întrebat, n-am primit* —
iar efectul din sertar întreba la nesfârșit, pe un endpoint de 5 cereri/minut.
Reparat cu `asked` în magazin, scris doar la un răspuns REUȘIT (altfel „Încearcă
din nou" n-ar mai încerca — bug-ul invers, și invizibil). `a448ca0`.

**2. Eroarea unei încercări moarte stătea peste tabelul următor.** „Înapoi" era
doar `setStep('paste')`. Curățat în `buildFromMatch`, singura funcție pe care o
cheamă amândouă ușile. `51d6d9e`.

🔑 **Metoda care le-a găsit, de refolosit:** am exclus pe rând — logica pură în
Node (8ms), sertarul în jsdom cu magazinul fals (nu se învârte). Apoi proba care
a decis: **aceleași date lipite ca text înghețau identic**, deci fișierul n-avea
nicio vină. Împarte problema în două înainte de a citi cod.

⚠️ **„Eroare de conexiune" la import nu e bug** — e Docker oprit, adică baza
locală. S-a întâmplat o dată și am pierdut timp căutând în cod.

## Regula de text dată de owner (3 oct)

Tăiase două din trei anunțuri de la import. **Antrenorul e informat ce se întâmplă
cu CLIENȚII lui și dacă are erori — nu cum citim noi fișierul.** Cele două tăiate
descriau mecanica noastră (care foaie, câte rânduri sărite). A rămas doar cel
despre un telefon care nu poate fi sunat, scurtat la o propoziție. Se aplică la
orice text viitor din import.

## Deciziile care ordonează restul

- **Emailul e OPȚIONAL** la un contact. Multe liste au doar nume și telefon.
- **`authenticated` NU are INSERT** pe `client_contacts`. Toate scrierile trec
  prin RPC-uri SECURITY DEFINER care dețin verificarea plafonului. Costul: orice
  adăugare nouă cere un RPC, nu un `.insert()`.
- **Decizia 10 reconfirmată 30 sep**: la import, surplusul intră `Inactiv`, NU se
  taie lista. Owner-ul propusese „importă doar cât încape" — respins fiindcă nu
  poate răspunde la „care 2?" și fiindcă 48 de rânduri gri cu „peste pachet" sunt
  cel mai bun argument de upgrade pe care produsul îl are.
- **Ordinea locurilor = ordinea din lista lipită.** Orice altă ordine e o alegere
  pe care antrenorul n-a făcut-o și n-o poate anticipa.

## 🔴 Capcanele care s-au repetat, și merită citite înainte de etapa 4

**`REVOKE` pe coloană e un no-op.** Specificația îl cerea; nu revocă nimic (vezi
[[column-revoke-is-a-noop]]). Tehnica bună: `REVOKE ALL` pe TABELĂ, apoi `GRANT`
doar pe ce trebuie.

**`btrim(x)` cu un argument taie DOAR spații.** Nu tab, nu linie nouă, nu
`chr(160)` — care e **invizibil**, deci un nume format doar din el arată gol.
Importul e drumul pe care sosesc, vin din Excel. Setul e fixat între browser și
RPC printr-un test care **citește SQL-ul** din `SCHEMA.sql`; dacă se despart,
suita cade. ⚠️ `String.trim()` ar fi fost mai LARG, nu mai îngust.

**`ON CONFLICT` pe un index PARȚIAL cere predicatul.** Specificația îl scria
fără. N-ar fi crăpat la aplicare, ci **la primul import real**.

**Un tipar de căutare îngust întoarce puține linii ȘI PARE COMPLET.** Un
`grep | head -12` pe 16 rezultate m-a făcut să ratez a patra funcție — și era
calea mai des umblată. Numeri rădăcina GOALĂ întâi, clasifici pe urmă.

**Contradicțiile cu vecinii, nu afirmațiile false.** Ambele blocante ale
review-ului legal erau texte adevărate care contraziceau alt paragraf din aceeași
pagină. Când se schimbă o afirmație, **se caută IDEEA, nu propoziția**.

**Baza LOCALĂ crapă (SIGSEGV) la un refuz de privilegiu pe FUNCȚIE sub
`SET ROLE`.** Deci proba „intră ca `anon` și arată că pică" nu se mai poate
folosi; se întreabă `has_function_privilege` + `pg_proc.proacl`. Atenție: un
`proacl` **NULL** înseamnă EXECUTE pentru PUBLIC, adică `REVOKE` nerulat.

**`now()` e ora TRANZACȚIEI.** Într-un `BEGIN…ROLLBACK`, `updated_at` nu se
mișcă — o probă corectă pică, iar vecina ei trece **vacuu**.

## Reguli de lucru care s-au cristalizat

- **Blocul de probe de comportament NU se rulează pe producție.** Conține
  `INSERT`/`UPDATE`/`SET ROLE`, iar SQL Editor poate schimba conexiunea între
  instrucțiuni — un `ROLLBACK` poate ateriza pe altă conexiune. Pe prod: doar
  verificare read-only. Comportamentul se probează pe local.
- **Amprenta md5 pe o copie verbatim.** Când o migrare re-creează o funcție
  existentă, pre-flight-ul cere `md5(prosrc)` ÎNAINTE: dacă prod a divergat,
  copia ar da înapoi tăcut munca altcuiva.
- **Cinci căi consumă un loc** și toate stau acum la cheia de lock partajată
  `'reactivate_client:'||uid`.

Legat: [[what-justifies-the-money-2026-09-26]], [[adoption-targets-2026-2027]],
[[column-revoke-is-a-noop]], [[qa-gate-blind-spots]],
[[db-two-databases-migration-order]], [[legal-pages-hardening]].
