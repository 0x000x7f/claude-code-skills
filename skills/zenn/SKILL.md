---
name: zenn
description: Turn work from the current Claude Code session into a Zenn article, validate it with the repository's own checks, register it as a draft, and publish only after explicit approval.
disable-model-invocation: true
---

Turn the work from the current session into a publishable Zenn article and ship it through a GitHub-connected Zenn repository.

Zenn publishing is repository-driven: articles are Markdown files under `articles/`, and the repository's configured branch is deployed by Zenn. Reuse the repository's own validation and publishing rules instead of reimplementing them in this skill.

## Repository discovery

Resolve `<repo>` before doing any work.

1. If `ZENN_REPO` is set, use that path.
2. Otherwise, prefer `$env:USERPROFILE\Documents\GitHub\zenn-content` on Windows or `~/Documents/GitHub/zenn-content` on Unix-like systems.
3. A valid repository must contain both:
   - `CLAUDE.md`
   - `articles/`
4. If the preferred location is absent, search nearby GitHub working-copy directories for a repository that:
   - contains `CLAUDE.md` and `articles/`, and
   - has a Git remote corresponding to the intended Zenn content repository.
5. If no unambiguous repository is found, stop and ask the user for the path. Do not create or guess a repository.

Never embed machine-specific absolute paths, usernames, tokens, or secrets into an article.

## Step 0 — sync and load the source of truth

1. Read `<repo>/CLAUDE.md`. Treat it as the authoritative publishing guide. If it conflicts with this skill, follow `CLAUDE.md`.
2. Run:

```bash
git -C "<repo>" pull --ff-only
```

If the working tree is dirty or the pull is not fast-forwardable, stop and report the state before changing files.

## Step 1 — topic and deduplication

Before drafting:

- Confirm the article angle with the user in one concise sentence.
- Check existing article titles and slugs under `<repo>/articles/`.
- Optionally compare against already published Zenn articles using public pages or a read-only listing source.
- If the topic materially overlaps an existing article, flag the overlap rather than creating a near-duplicate.

## Step 2 — draft

Write a Japanese technical article appropriate for Zenn.

Prefer:

- a clear target reader and problem statement;
- TL;DR or conclusion-first structure where useful;
- concrete commands, logs, diffs, tables, and diagrams;
- primary sources for technical claims;
- explicit labels such as `[UNVERIFIED]` when a decision-driving claim has not been checked;
- honest discussion of failed attempts and limitations.

Avoid:

- filler and hype;
- claims that outrun the evidence;
- private health data, secrets, tokens, internal hostnames, or personal identifiers;
- local absolute paths that reveal the author's machine or account.

Use the repository's existing style rules rather than inventing parallel conventions.

## Step 3 — frontmatter

Create new articles as drafts:

```yaml
---
title: "..."
emoji: "..."
type: "tech"
topics: ["..."]
published: false
---
```

Follow repository validation rules for title length, topics, slug format, encoding, and line endings.

The slug is the article filename and becomes part of the permanent URL, so choose it carefully before publication.

## Step 4 — validate

Run the repository's validator:

```bash
cd "<repo>"
npm run validate
```

If dependencies are missing:

```bash
cd "<repo>"
npm ci
npm run validate
```

Fix errors until validation passes.

For visual verification, start Zenn Preview:

```bash
cd "<repo>"
npx zenn preview
```

When rendering matters, trust the actual browser preview rather than a text-only fetch.

## Step 4b — register a new draft

After a new article passes validation, register it in Git while it is still `published: false`.

This makes the draft available across machines without publishing it publicly.

```bash
git -C "<repo>" pull --ff-only
git -C "<repo>" add "articles/<slug>.md"
git -C "<repo>" commit -m "draft: <slug> (WIP)"
git -C "<repo>" push
```

Do not include `[ci skip]` or `[skip ci]` in commits used for a Zenn-connected repository unless the repository's publishing guide explicitly requires it.

If the repository requires a co-author trailer or other commit convention, follow `CLAUDE.md`; do not hardcode an individual or model identity in this reusable skill.

## Step 5 — publish only after explicit approval

Never publish based on the model's own judgment.

Before publication:

1. Show the user the final draft or preview URL.
2. State the would-be Zenn URL if the account name is known.
3. Wait for explicit approval.

After approval, change only the intended article:

```diff
-published: false
+published: true
```

Then validate again and push:

```bash
cd "<repo>"
npm run validate
git add "articles/<slug>.md"
git commit -m "publish: <slug>"
git push
```

Confirm that the commit reached the remote. Deployment may take a short time.

## Notes

- Unpublish by changing `published: true` back to `published: false` and pushing.
- Follow Zenn and repository documentation for deletion and scheduled publishing behavior.
- Keep authoring in files and Git when that is the repository's established workflow.
- If the repository uses a persistent remote preview server, provide the direct preview URL for the article.
