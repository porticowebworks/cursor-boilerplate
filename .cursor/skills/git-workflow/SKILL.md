---
name: git-workflow
description: >
  Enforce Git branching strategy, commit discipline, PR standards, and deployment workflow
  for small teams (2–4 developers) on Nuxt 4 marketing site projects. Covers branch naming,
  commit message rules, PR process, merge strategy, environment mapping, and Coolify/VPS vs
  Netlify/Vercel deployment triggers. Use this skill whenever a developer is setting up a
  repository, creating a branch, writing a commit, opening a PR, or deploying a project.
  Trigger when the user mentions Git, branching, commits, pull requests, merging, deployment,
  staging, production, Coolify, Netlify, or Vercel in the context of project workflow.
---

# Git Workflow

## Core Principle

Small teams fail at Git in two ways: everyone commits to `main` directly (chaotic), or the
process is so heavy it slows a 2-person team to a crawl (bureaucratic). This workflow sits
between both — enough structure to prevent accidents, light enough that one developer working
alone on a feature doesn't feel friction.

> **All examples are samples for reference only.** Adapt branch names and environment URLs
> to your actual project.

---

## Branch Structure

```
main                  ← production-ready code only. Always deployable.
  └── develop         ← integration branch. staging environment tracks this.
        ├── feature/room-page-redesign
        ├── feature/booking-form
        ├── fix/hero-image-cls
        └── chore/update-nuxt-deps
```

### Branch Definitions

| Branch | Purpose | Who merges into it | Protected |
|---|---|---|---|
| `main` | Live production code | PR from `develop` only | ✅ Yes |
| `develop` | Latest integrated work; maps to staging | PR from feature branches | ✅ Yes |
| `feature/*` | New features or pages | Creator, merged to `develop` | No |
| `fix/*` | Bug fixes | Creator, merged to `develop` | No |
| `hotfix/*` | Critical production fixes only | Creator, merged to `main` AND `develop` | No |
| `chore/*` | Deps, config, non-functional changes | Creator, merged to `develop` | No |

### Rules

- **No one commits directly to `main` or `develop`** — all changes go through PRs
- `main` is always in a deployable state — if it isn't, that's an incident
- `develop` may have WIP but must pass CI before any PR merges into it
- `feature/` branches are short-lived — merge and delete within the same sprint/task cycle
- Never let a feature branch fall more than 2 days behind `develop` without rebasing

---

## Branch Naming

```
type/short-description-in-kebab-case
```

| Type | When to use | Example |
|---|---|---|
| `feature/` | New page, section, or functionality | `feature/rooms-page` |
| `fix/` | Bug fix that isn't production-critical | `fix/mobile-nav-overlap` |
| `hotfix/` | Production is broken — fix now | `hotfix/contact-form-500` |
| `chore/` | Dependencies, config, refactor | `chore/upgrade-nuxt-4-2` |

**Rules:**
- Lowercase and hyphens only — no spaces, underscores, or slashes beyond the prefix
- Short and descriptive — `feature/rooms-page` not `feature/work-on-the-rooms-page-for-client`
- No personal names — `feature/rooms-page` not `feature/rahul-rooms`
- No ticket numbers as the entire name — `feature/rooms-page` not `feature/JIRA-421`

---

## Commit Messages

No formal convention enforced. Three rules only:

1. **Say what changed, not what you did**
   - ✅ `Add hero image to rooms page`
   - ❌ `Worked on rooms page`

2. **Be specific enough that anyone can understand it without context**
   - ✅ `Fix CLS on mobile hero caused by missing NuxtImg dimensions`
   - ❌ `Fix image bug`

3. **One logical change per commit** — don't bundle unrelated changes into one commit

### Commit Message Format

```
Short summary (max 72 chars) — present tense, no period

Optional body if the why isn't obvious:
- Why this change was needed
- Any tradeoffs or decisions made
- Reference to issue/ticket if applicable
```

### Examples

```bash
# ✅ Good
git commit -m "Add deluxe room page with NuxtImg hero and booking CTA"
git commit -m "Fix mobile nav z-index overlapping sticky header"
git commit -m "Update @nuxt/image to 1.8.0 for AVIF support"
git commit -m "Remove unused TestimonialsCarousel component from homepage"

# ❌ Bad
git commit -m "updates"
git commit -m "WIP"
git commit -m "fixed stuff"
git commit -m "Worked on rooms page and also fixed some CSS and updated a dependency"
```

### When to Add a Commit Body

Add a body when the commit changes something non-obvious — a decision, a tradeoff, a workaround:

```bash
git commit -m "Disable payload extraction in nuxt.config

SSG builds were embedding large JSON payloads in HTML, inflating
page size by ~40KB on content-heavy pages. Disabling payloadExtraction
reduces HTML size. No functional impact — data still available via
hydration from component fetch calls."
```

---

## Pull Request Process

### When to Open a PR

- Feature or fix is complete and tested locally
- Branch is up to date with `develop` (rebased or merged)
- No console errors or TypeScript errors
- Lighthouse score checked on key pages if the change affects rendering

### PR Title

Same rules as commit messages — describe what changed:

```
✅ Add rooms page with three room type sections
✅ Fix hero image LCP regression on homepage
✅ Upgrade Nuxt to 4.2 and resolve breaking changes
❌ Rooms page
❌ Fixes
❌ WIP - do not merge
```

### PR Description Template

```markdown
## What changed
Brief description of what this PR does.

## Why
Why this change was needed. Link to issue/brief/client request if applicable.

## Pages / routes affected
- /rooms
- /rooms/deluxe

## How to test
1. Run `npm run generate`
2. Open /rooms in browser
3. Verify hero image loads without CLS
4. Check mobile nav still works

## Checklist
- [ ] Tested locally
- [ ] No TypeScript errors
- [ ] No console errors
- [ ] NuxtImg used for all images (no raw img tags)
- [ ] CSS tokens used (no hard-coded values)
- [ ] Lighthouse checked if rendering changed
```

### PR Size

- **Small PRs are better than large PRs** — aim for single-feature or single-fix scope
- A PR that changes 20 files across unrelated areas will be reviewed poorly or not at all
- If a PR is growing large, split it: infrastructure changes in one PR, feature in another

### Review Rules

| Team size | Review requirement |
|---|---|
| Solo developer on project | Self-review checklist before merge — no peer required |
| 2 developers | 1 approval required before merge |
| 3–4 developers | 1 approval required; author cannot approve own PR |

- Reviewer approves the logic and approach — not just that "it looks fine"
- Reviewer runs the branch locally if the change is non-trivial
- Author resolves all comments before merging — no merging with open unresolved threads

---

## Merge Strategy

**Always use Squash and Merge when merging feature branches into `develop`.**

Reasons:
- Keeps `develop` history clean — one commit per feature, not 14 "WIP" commits
- Makes `git log` on `develop` readable as a changelog
- Individual commit history preserved on the feature branch until deletion

```
Before squash merge:
feature/rooms-page:  "WIP" → "add structure" → "fix CSS" → "final" → "actually final"

After squash merge into develop:
develop: "Add rooms page with three room type sections" ← clean, single commit
```

**Exception — hotfixes:**
Use a regular merge (not squash) when merging `hotfix/*` into both `main` and `develop` so the fix commit is traceable across both branches.

### Merge Commands

```bash
# Feature/fix/chore → develop (via PR squash merge on GitHub/GitLab)
# Use the UI — don't merge locally for protected branches

# After PR merges — clean up local and remote branch
git checkout develop
git pull origin develop
git branch -d feature/rooms-page
git push origin --delete feature/rooms-page
```

---

## Environment Mapping

| Branch | Environment | Deployment trigger |
|---|---|---|
| `develop` | Staging | Auto-deploy on push |
| `main` | Production | Auto-deploy on push (or manual confirm) |
| `feature/*` | None (local only) | `npm run dev` locally |

### Coolify / VPS Setup

```
Repository: github.com/org/project
Branch: develop → Staging environment (auto-deploy)
Branch: main    → Production environment (auto-deploy or manual trigger)
```

In Coolify:
- Set **Watch Paths** to avoid deploying on README or docs-only changes
- Enable **Health Check** URL — Coolify verifies the deploy succeeded before marking live
- Set **Build Command:** `npm run generate`
- Set **Publish Directory:** `.output/public` (Nuxt 4 SSG output)

### Netlify / Vercel Setup

```toml
# netlify.toml
[build]
  command   = "npm run generate"
  publish   = ".output/public"

[context.production]
  branch = "main"

[context.staging]
  branch = "develop"
```

```json
// vercel.json
{
  "buildCommand": "npm run generate",
  "outputDirectory": ".output/public",
  "git": {
    "deploymentEnabled": {
      "main": true,
      "develop": true
    }
  }
}
```

---

## Hotfix Process

For production bugs that cannot wait for the normal `develop` → `main` cycle:

```bash
# 1. Branch from main (not develop)
git checkout main
git pull origin main
git checkout -b hotfix/contact-form-500

# 2. Fix the bug
# 3. Test locally
# 4. Open PR → main
# 5. After merge to main, immediately merge to develop as well

git checkout develop
git pull origin develop
git merge main   # or cherry-pick the hotfix commit
git push origin develop
```

> Never skip the `develop` merge after a hotfix. If `develop` doesn't get the fix, it will be overwritten on the next production deploy.

---

## Daily Workflow (Per Developer)

```bash
# Start of day — sync with develop
git checkout develop
git pull origin develop

# Start new work
git checkout -b feature/new-feature-name

# Work, commit regularly
git add .
git commit -m "Descriptive message about what changed"

# Before opening PR — update from develop
git checkout develop
git pull origin develop
git checkout feature/new-feature-name
git rebase develop   # or git merge develop

# Push and open PR
git push origin feature/new-feature-name
# Open PR on GitHub/GitLab → target: develop
```

---

## What Claude Flags

When reviewing code or workflow in Cursor:

| Situation | Flag |
|---|---|
| Commit message is "WIP", "updates", "fix", or fewer than 5 words | 🚩 **Commit message:** Too vague. Describe what specifically changed. |
| Direct push to `main` or `develop` mentioned | 🚩 **Branch protection:** Never push directly to `main` or `develop`. Open a PR. |
| Feature branch more than 3 days old without rebase | 🚩 **Stale branch:** Rebase against `develop` to avoid large merge conflicts. |
| PR with no description or checklist | 🚩 **PR hygiene:** Add what changed, why, and how to test. |
| Multiple unrelated changes in one commit | 🚩 **Commit scope:** Split into separate commits — one logical change per commit. |
| `hotfix/` merged only to `main`, not `develop` | 🚩 **Hotfix sync:** Hotfixes must merge into both `main` and `develop`. |
| `feature/` branch not deleted after merge | ⚠️ **Cleanup:** Delete merged branches — `git push origin --delete branch-name`. |

---

## Quick Reference

```
BRANCH TYPES:
  feature/*  → new work       → PR to develop
  fix/*      → bug fix        → PR to develop
  hotfix/*   → prod emergency → PR to main + sync to develop
  chore/*    → maintenance    → PR to develop

COMMIT RULES:
  Present tense. Say what changed. Be specific. One change per commit.

PR RULES:
  Title = what changed. Description = what + why + how to test.
  Squash merge into develop. Regular merge for hotfixes.

ENVIRONMENTS:
  develop → staging (auto)
  main    → production (auto or manual)
  feature → local only
```
