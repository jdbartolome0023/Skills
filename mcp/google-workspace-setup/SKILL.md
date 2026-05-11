---
description: "Install and configure the Google Workspace CLI (gws) for accessing Drive, Sheets, Docs, Slides, Gmail, Calendar. Run this once per machine."
---

# Google Workspace CLI Setup

One-time setup for the `gws` CLI — gives Claude Code programmatic access to Google Drive, Sheets, Docs, Slides, and more.

## Prerequisites

- macOS (arm64 or x86_64) or Linux
- A `@webprofits.com.au` Google Workspace account

## Tell Claude Code

> Install and set up the Google Workspace CLI for me. Follow the google-workspace-setup skill.

Claude Code will run through the steps below.

## Step 1 — Install the binary

### macOS (Apple Silicon)

```bash
curl -L -o /tmp/gws.tar.gz "https://github.com/googleworkspace/google-workspace-cli/releases/latest/download/gws-aarch64-apple-darwin.tar.gz"
tar -xzf /tmp/gws.tar.gz -C /tmp/
mkdir -p ~/.local/bin
mv /tmp/gws ~/.local/bin/gws
chmod +x ~/.local/bin/gws
```

### macOS (Intel)

```bash
curl -L -o /tmp/gws.tar.gz "https://github.com/googleworkspace/google-workspace-cli/releases/latest/download/gws-x86_64-apple-darwin.tar.gz"
tar -xzf /tmp/gws.tar.gz -C /tmp/
mkdir -p ~/.local/bin
mv /tmp/gws ~/.local/bin/gws
chmod +x ~/.local/bin/gws
```

### Verify PATH

`~/.local/bin` must be in your PATH. Check with:

```bash
echo $PATH | tr ':' '\n' | grep local/bin
```

If missing, add to your shell profile:

```bash
# For zsh (~/.zshrc)
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# For bash (~/.bashrc)
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Verify install

```bash
gws --version
```

Should show `gws 0.20.0` or later.

## Step 2 — Configure OAuth credentials

Copy the bundled `client_secret.json` into the gws config directory:

```bash
mkdir -p ~/.config/gws
cp ~/.claude/skills/google-workspace-setup/scripts/client_secret.json ~/.config/gws/client_secret.json
```

This file is bundled with the skill — it identifies the Webprofits OAuth app, not your personal credentials. It's an OAuth installed-app client ID (not confidential per Google's own documentation).

## Step 3 — Authenticate

Run login interactively (opens your browser):

```bash
gws auth login --scopes "https://www.googleapis.com/auth/drive,https://www.googleapis.com/auth/gmail.modify,https://www.googleapis.com/auth/calendar,https://www.googleapis.com/auth/spreadsheets,https://www.googleapis.com/auth/documents,https://www.googleapis.com/auth/presentations,https://www.googleapis.com/auth/tasks"
```

**Important:** This must be run by the user, not Claude Code — it requires browser interaction.

1. A browser window opens → sign in with your `@webprofits.com.au` account
2. Grant the requested permissions
3. The CLI stores encrypted credentials in `~/.config/gws/credentials.enc` with the encryption key in your system keychain

### Google may show "unverified app" warning

Click **Advanced** → **Go to Report Notes Aggregator (unsafe)**. This is our internal GCP project — the warning appears because it hasn't gone through Google's app verification (only needed for public apps).

## Step 4 — Verify

```bash
gws auth status
gws drive files list --params '{"pageSize": 3, "fields": "files(id,name,mimeType)", "orderBy": "modifiedTime desc"}'
```

You should see your 3 most recent Drive files.

## Troubleshooting

### "Insufficient authentication scopes"
The token cache has old scopes. Delete it and re-login:
```bash
rm ~/.config/gws/token_cache.json
gws auth login --scopes "https://www.googleapis.com/auth/drive,https://www.googleapis.com/auth/gmail.modify,https://www.googleapis.com/auth/calendar,https://www.googleapis.com/auth/spreadsheets,https://www.googleapis.com/auth/documents,https://www.googleapis.com/auth/presentations,https://www.googleapis.com/auth/tasks"
```

### "Access blocked: Authorization Error"
The `--scopes` flag needs full URLs, not shorthand. Use the exact command above — don't use `drive` alone, use `https://www.googleapis.com/auth/drive`.

### Windows (Git Bash)
Same install steps but use the Windows binary:
```bash
curl -L -o /tmp/gws.zip "https://github.com/googleworkspace/google-workspace-cli/releases/latest/download/gws-x86_64-pc-windows-msvc.zip"
unzip /tmp/gws.zip -d /tmp/
mkdir -p ~/.local/bin
mv /tmp/gws.exe ~/.local/bin/gws.exe
```

### Scopes reference

| Scope | What it unlocks |
|---|---|
| `drive` | Search, download, export files from Google Drive |
| `spreadsheets` | Read and write Google Sheets data |
| `documents` | Read and write Google Docs |
| `presentations` | Read and export Google Slides |
| `gmail.modify` | Read, send, and manage email |
| `calendar` | Read and manage calendar events |
| `tasks` | Read and manage Google Tasks |

Add only the scopes you need. More scopes = broader permissions on your account.

## What's next

Once installed, the **google-workspace** skill covers day-to-day usage — searching Drive, reading Sheets, exporting Docs, etc.
