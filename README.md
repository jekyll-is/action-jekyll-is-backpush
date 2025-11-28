# Jekyll IS Backpusher

Composite GitHub Action to commit and push changes back to repository after Jekyll build or other generation steps. Includes optional Telegram notifications with git diff summary. Uses `github-actions[bot]` for traceable commits.[7]

## Features

- Conditionally commits only if changes detected via `git diff --cached`
- Safe git config with GitHub bot credentials
- Telegram notifications with git status, commit message, and truncated file changes (MarkdownV2)
- Strict bash mode (`set -euo pipefail`) for reliability
- Custom commit message with label and timestamp
- Handles large diffs by limiting to 20 lines + "..."

## Prerequisites

Action assumes prior `actions/checkout@v4` with write permissions. Repository must allow pushes from `github-actions[bot]` (default for most repos). For protected branches, enable "Allow GitHub Actions to create and approve pull requests".[23]

## Usage

### Basic (without notifications)

```yaml
name: Build & Backpush
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Generate Jekyll site
        run: |
          # Your build steps here, e.g.:
          bundle exec jekyll build
      - name: Commit & Push
        uses: jekyll-is/action-jekyll-is-backpush
        with:
          label: "Jekyll build"
```

### With Telegram notifications

Store `TELEGRAM_BOT_TOKEN` and `TELEGRAM_USER_ID` as repository secrets.

```yaml
      - name: Commit & Push
        uses: jekyll-is/action-jekyll-is-backpush
        with:
          label: "Jekyll build"
          telegram-bot-token: ${{ secrets.TELEGRAM_BOT_TOKEN }}
          telegram-user-id: ${{ secrets.TELEGRAM_USER_ID }}
```

## Inputs

| Input                  | Description                                      | Required | Default |
|------------------------|--------------------------------------------------|----------|---------|
| `label`                | Text label for commit messages/notifications     | ✅ Yes   | -       |
| `telegram-bot-token`   | Telegram bot token (secrets recommended)         | ❌ No    | -       |
| `telegram-user-id`     | Telegram user/chat ID                            | ❌ No    | -       |

## Outputs

Currently none. Future versions may add `changes-detected: true/false`.[7]

## Example Workflow (Jekyll)

Full example for Jekyll site with generated static files:

```yaml
name: Jekyll CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.2'
          bundler-cache: true
      - run: bundle exec jekyll build
      - name: Backpush _site changes
        uses: jekyll-is/action-jekyll-is-backpush
        with:
          label: "Jekyll ${_site}"
          telegram-bot-token: ${{ secrets.TELEGRAM_BOT_TOKEN }}
          telegram-user-id: ${{ secrets.TELEGRAM_USER_ID }}
```

## Known Limitations & Improvements

See detailed analysis in [CHANGELOG.md](CHANGELOG.md) or issues.

- No automatic `git pull` (add before if needed)
- Telegram message may exceed 4096 chars on large changes
- Selective `git add` recommended (e.g., `git add _site/` instead of `git add .`)

## Development

1. Fork/clone repository
2. Test locally: `act -j test` (requires `nektos/act`)
3. Add `.github/workflows/test.yml` for CI validation
4. Tag releases: `git tag v1.0.0 && git push --tags`

**Testing workflow example:**
```yaml
name: Test Backpusher
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: touch test-file.txt  # Simulate change
      - uses: jekyll-is/action-jekyll-is-backpush
        with:
          label: "Test"
```

## License

GPLv3. See [LICENSE](LICENSE) for details.

