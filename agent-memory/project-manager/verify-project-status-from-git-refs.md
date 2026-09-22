---
name: verify-project-status-from-git-refs
description: How to establish real project status with no Bash — read .git/refs and .git/logs/HEAD — and the three status distinctions that are always wrong if assumed.
metadata:
  type: project
---

As PM I have no Bash, but git state is still readable as plain files, and it is the
only trustworthy answer to "what is actually done".

**Why:** status claims decay fastest of anything I track, and every wrong report I
have given came from trusting a document instead of the tree. Docs and memory notes
describe intent at the moment they were written; refs describe reality now.

**How to apply:**
- `.git/refs/heads/<branch>` vs `.git/refs/remotes/origin/<branch>` — different SHAs
  mean unpushed work, i.e. **not deployed**.
- 🔴 **`.git/packed-refs` is a STALE SNAPSHOT and will lie to you.** When a ref is
  updated, git writes a *loose* file under `.git/refs/` and leaves the old line in
  `packed-refs` untouched. So packed-refs can show `main` and `origin/main` far apart
  long after they were reconciled. **Loose ref wins, always.** Glob `.git/refs/**/*`
  first to see which refs are loose, and only fall back to packed-refs for the ones
  that are absent there. This nearly produced a report saying "nothing is in
  production" on a day when ten days of work had been live for over a week.
- Reflog lines carry Unix epochs. To date them with no Bash, anchor on one entry whose
  real date is already known and count from there — then sanity-check the anchor. It is
  worth the arithmetic: prose dates in notes drift by a day or two, reflog epochs do not.
- `.git/logs/HEAD` is the reflog with full commit subjects: grep it for the remote SHA
  and everything after that line is unpushed. Grep it for `merge` to learn whether a
  feature branch actually landed on main — a branch that still exists tells you nothing.
- `.git/logs/refs/heads/<branch>` gives a branch's own history and last activity.

**The three distinctions that are always wrong if assumed:**
1. **Written ≠ applied.** SQL files in the repo prove someone wrote a migration, never
   that it ran against the database. Only a note that says "applied", with a date, counts.
2. **Committed ≠ deployed.** Local commits ship nothing.
3. **Built ≠ reviewed ≠ working in production.** A feature can be demonstrably working
   in a test environment and still be judged not ready to take real money.

When a status is load-bearing, open the file and check the specific line rather than
citing a document that claims it — and say which file I read.

**A file's mtime is not its edit date.** A merge (or a branch checkout) rewrites files
on disk, so `ls`/mtime stamps every merged file with the merge time. Anything dated
from mtime after a big merge is wrong — date a file from `git log -- <path>`, or from
the reflog entry that touched it. This cost a real misjudgement about whether some
assets had been regenerated after a rename.

**Branch reflogs also tell you a merge happened without any Bash:** grep
`.git/logs/HEAD` for `merge <branch>` — a fast-forward line there means the branch's
work is in the target branch and the branch itself is now just a leftover label. A
branch ref still existing proves nothing either way.

Related: [[state-file-drifts-when-not-updated]], [[feedback-no-commit-or-push-without-asking]].
