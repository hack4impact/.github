# Security Requirements for the Hack4Impact GitHub Org

*For chapter members and project leads. Last verified against live org settings: 2026-09-15.*

We work on projects that handle real information from real nonprofit partners. The org has been hardened so that a single compromised account — or a single honest mistake — can't leak partner data or break a production branch. Here's what that means for you.

## TL;DR — what changed

- **Two-factor authentication is now mandatory** for every member. No 2FA, no membership.
- **Every change to `main` needs a second person.** Direct pushes to the default branch are blocked everywhere; you open a pull request and someone else approves it.
- **Being an org member no longer grants access to anything private by itself.** You see private repos only through the teams you're on.
- **Forking private repos is disabled**, so partner-sensitive code can't drift into personal accounts.
- Repos can no longer be deleted or transferred out of the org except by org admins.

## 2FA and account requirements

- Your GitHub account must have **two-factor authentication enabled**. GitHub enforces this at the org level: you can't join, and can't remain, without it. Use an authenticator app or passkey rather than SMS if you can.
- Edits made through the GitHub web editor require a **sign-off** (GitHub adds it when you tick the box on the commit form). If a web commit is rejected, that's why.
- Use your own account only. Never share credentials or tokens, and never commit secrets (API keys, passwords, partner data) to any repo, public or private.

## The access model

Access is **deny-by-default**: org membership alone gives you no access to private repos. Everything flows through teams.

- **The National Projects team.** Joining National Projects (or one of its project sub-teams) generally gets you **write access on the shared public repos** and **read access on most private ones** — enough to see and contribute to the cross-chapter codebase. Special and NDA-bound projects are the exception: those stay behind their own restricted teams regardless of Nationals membership. Chapter-specific repos remain read-only through Nationals.
- **Write is scoped to the owning team.** Each chapter and each project has its own team (for example, a chapter's founding team), and that team holds write access to its own repos — nobody else's. Chapter umbrella teams exist for organization; the working sub-teams hold the actual repo access.
- **Some work is invisible until you're granted it.** A few pre-release projects sit behind restricted teams that don't appear in the team list unless you're a member. If you can't see a repo you've heard about, that's the system working — ask your lead for access rather than assuming it's missing.

### Getting write access: public vs. private repos

- **Public repos:** anyone can contribute the open-source way — fork the repo, push to your fork, open a pull request. You don't need write access to contribute; you need it only to review, merge, and manage branches, and that comes from joining the owning team.
- **Private repos:** there is no fork path. The only way in — read or write — is membership on the team that owns the repo.

Either way, **merging to `main` always requires a pull request with at least one approval**, review conversations must be resolved, the approver can't be the same person who pushed the final commits, and force-pushes and branch deletion on `main` are blocked. This applies to every repo in the org, project leads included.

## How to request access

1. **Working on a chapter project?** Ask your chapter/project lead to add you to that project's team. That grants read and write on the chapter's repos.
2. **Working on a national/shared project?** Ask to be added to the National Projects team (or the specific project sub-team under it).
3. **Need to see a pre-release project?** Ask the project's lead directly; if it's behind a restricted team, they'll request you be added.
4. **Leads:** members can't create teams themselves — ask an org owner to create a new team when a new project spins up, and keep team membership current (remove people when they roll off).

## Projects and scrum boards

The same model covers GitHub Projects (the boards). Boards are private by default and scoped to the team that owns the project — you see your project's board through your team membership, not through org membership. Only the org-wide public board is visible to everyone.

## Forking policy

**Forking private repositories is disabled org-wide.** This protects customer- and partner-sensitive information: a fork lives in a personal account outside the org's controls, so private code never gets one. Work on branches inside the repo instead. Public repos can be forked normally.

## Repo creation policy

- Members **can create repos** (public or private). If you create one for org work, put it under the owning team right away — otherwise nobody else can see or maintain it, and it becomes an orphan when you graduate.
- New repos automatically inherit the org-wide rules above (protected `main`, required reviews).
- **Nobody except org admins can delete a repo or transfer it out of the org.** If a repo is truly done, ask an admin to archive it.

---

Questions about any of this? Ask your chapter lead first, or open a thread in the org Slack.
