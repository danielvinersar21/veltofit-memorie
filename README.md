# veltofit-memorie

Backup-ul notelor de memorie ale lui Claude pentru proiectul Veltofit. **Privat.**
Conține adrese de email ale unor oameni reali și raționamente de business — nu-l
face public și nu-l partaja.

- `project-memory/` — notele din `~/.claude/projects/-Users-danielvinersar-repos-velto-webapp/memory/`
  (indexul `MEMORY.md` + câte o notă pe subiect: decizii și de ce, ce e aplicat în
  producție, preferințele de lucru).
- `agent-memory/` — memoria agenților de proiect, din `~/.claude/agent-memory/`.

Se actualizează automat la fiecare `git push` din `velto-webapp` (hook-ul
`.githooks/pre-push` rulează `scripts/memory-backup.sh` în fundal) și de mână cu
`npm run memory:backup`. Fiecare rulare e un commit — istoricul E backup-ul.

## Restaurare pe alt laptop

```sh
git clone git@github.com:danielvinersar21/veltofit-memorie.git ~/.claude/memory-backup
mkdir -p ~/.claude/projects/-Users-danielvinersar-repos-velto-webapp
rsync -a ~/.claude/memory-backup/project-memory/ ~/.claude/projects/-Users-danielvinersar-repos-velto-webapp/memory/
rsync -a ~/.claude/memory-backup/agent-memory/ ~/.claude/agent-memory/
```

(Numele folderului din `projects/` vine din calea repo-ului pe disc; dacă repo-ul
stă altundeva pe laptopul nou, numele diferă — vezi ce creează Claude Code acolo.)
