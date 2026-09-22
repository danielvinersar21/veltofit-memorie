# Memory Index

- [Owner profile](user-owner-profile.md) — solo developer/owner, Romanian-speaking, decides everything himself; my job is "where are we", not "how does this work".
- [Plain-language reports](feedback-plain-language-reports.md) — lead with consequences, not mechanisms; every backlog item needs a why and an S/M/L.
- [Never commit or push unasked](feedback-no-commit-or-push-without-asking.md) — unpushed work is usually a decision; report it as one, never as a chore.
- [Verify status from git refs](verify-project-status-from-git-refs.md) — no Bash needed; `.git/packed-refs` lies, loose refs win; written ≠ applied, committed ≠ deployed.
- [State files drift silently](state-file-drifts-when-not-updated.md) — a stale state file looks identical to a correct one; date it and rebuild from files.
- [Legal copy drifts with feature flags](legal-copy-drifts-with-feature-flags.md) — terms/privacy go false when a flag flips, a Stripe setting is test-mode-only, or a promised right has nothing to run.
- [Vendor account ≠ deployment wired](vendor-account-configured-is-not-deployment-wired.md) — "live mode configured" covered the Stripe account, not the env keys; prod ran on test keys for 3 days.
- [First-charge settings are irreversible](first-charge-settings-are-irreversible.md) — invoice numbering/template/descriptor freeze at the first real payment; dashboard-only, so no gate catches them.
- [The repo is blind to real-world events](repo-is-blind-to-real-world-events.md) — refs can't tell you someone signed up; derive unwritten dates, but check the rule first (I once invented a trial deadline).
