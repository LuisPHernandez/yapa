# Workflow: how to make a change

The steps for getting a change into `main`. For what each tool is and why we use it, see [tooling.md](tooling.md).

When a new tool changes how we work, its PR updates this file.

## The short version

```bash
git switch main
git pull
git switch -c feat/drop-pickup-window     # 1. branch
# ...edit, commit, repeat...              # 2. work
git push -u origin feat/drop-pickup-window  # 3. push
# 4. open a PR on GitHub, the other person reviews
# 5. squash and merge, then:
git switch main
git pull
git branch -d feat/drop-pickup-window
```

## 1. Start from an issue and a fresh branch

Pick an issue from the board and move it to Doing (create one first if it doesn't exist). Then branch from an up-to-date `main`.

Branch names are `type/short-description`:

| Prefix | For |
| --- | --- |
| `feat/` | New behaviour |
| `fix/` | Bug fixes |
| `chore/` | Tooling, dependencies, configuration |
| `docs/` | Documentation only |
| `refactor/` | Restructuring without changing behaviour |

## 2. Work in small commits

Commit whenever something works. Commit messages on a branch don't need polish, because the branch is squashed into one commit when it merges.

Keep the branch short-lived: aim to open the PR within a day or two. A branch that lives for a week drifts away from `main` and becomes painful to review and merge.

If `main` moved while you were working, bring it in:

```bash
git fetch origin
git merge origin/main
```

## 3. Open the pull request

Push the branch and open a PR against `main`. Fill in the template:

- **Title:** this becomes the commit message on `main`, so write it as one: `Add pickup window validation to drops`.
- **What and why:** include `Closes #12` so the issue closes on merge.
- **How I tested it.**

Open the PR as a **draft** if you want early feedback on something unfinished.

One PR does one thing. If the description needs the word "also", it is probably two PRs.

## 4. Review

The author:

- Reads their own diff on GitHub before asking for review.
- Answers every comment, by changing the code or by replying.

The reviewer:

- Reviews within one working day. A waiting PR blocks the other person.
- Checks that they understand what the code does and would be comfortable maintaining it.
- Says which comments must be fixed and which are suggestions (prefix optional ones with `nit:`).

Pushing new commits after an approval dismisses it, so the reviewer approves again.

## 5. Merge

Once approved (and, later, once CI is green), the author clicks **Squash and merge**. GitHub deletes the remote branch. Then update your machine:

```bash
git switch main
git pull
git branch -d feat/drop-pickup-window
```

## When something goes wrong

- **You committed to `main` by accident (not pushed).** Move the commits to a branch:
  ```bash
  git switch -c fix/my-change        # new branch keeps the commits
  git branch -f main origin/main     # put local main back where GitHub's is
  ```
- **You pushed a secret.** Tell the other person and rotate the secret immediately. Removing the commit is not enough; assume it was copied.
- **`main` is broken.** Fixing it comes before any other work. Revert the offending commit through a PR, then fix properly on a new branch.

## Recording decisions

When a change settles something important or hard to reverse, add a record to `docs/decisions/` in the same PR. See [the README there](../decisions/README.md).
