# Skill Git Push Pattern

When updating a skill on GitHub, work directly from the skill directory (e.g. `~/.hermes/skills/<name>/`), not from a separate `/tmp` clone. The `/tmp` clone has no push credentials.

## Push flow (skill already has a remote/git history)

```bash
cd ~/.hermes/skills/<name>

# Ensure remote uses token auth
source ~/.hermes/.env
git remote set-url origin "https://$GITHUB_TOKEN@github.com/<owner>/<repo>.git"

# Add, commit, push
git add <changed-files>
git commit -m "<message>"
git push
```

## Push flow (skill dir is a fresh git init)

```bash
cd ~/.hermes/skills/<name>

git init
git remote add origin "https://github.com/<owner>/<repo>.git"

source ~/.hermes/.env
git remote set-url origin "https://$GITHUB_TOKEN@github.com/<owner>/<repo>.git"

git add .
git commit -m "initial commit"
git branch -M main              # note: default branch may be 'main', not 'master'
git push -u origin main
```

## Branch name gotcha

Local `git init` often creates branch `master`, but GitHub repos created via web UI default to `main`. If you see `error: src refspec master does not match any`, check `git branch` and push the correct branch name.

## Token sourcing

The GitHub token lives at `~/.hermes/.env` as `GITHUB_TOKEN=...`. Always `source ~/.hermes/.env` before using `$GITHUB_TOKEN` in a URL.
