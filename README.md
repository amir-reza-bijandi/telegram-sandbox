# telegram-sandbox

A GitHub Actions workflow that downloads every file from a Telegram channel when you include a channel ID in your commit message, bundles them into ≤ 90 MB zip archives, and commits them to this repository under `downloads/`.

> [!NOTE]
> This project was built with the assistance of Claude (Anthropic). The code, structure, and documentation were generated through an AI-assisted development session and reviewed by the author.

---

## How it works

1. Push a commit to `main`/`master` with `[tg:<channel_id>]` anywhere in the message.
2. The workflow detects the channel ID, authenticates with Telegram using a stored session, and downloads all documents, photos, audio, video, and voice files.
3. Files are grouped into ≤ 90 MB zip parts and committed to `downloads/<channel_id>/`.

**Example commit messages**

```
fetch assets [tg:-1001234567890]
[tg:-1001234567890] archive channel
```

---

## One-time setup

### 1 — Get Telegram API credentials

1. Go to [https://my.telegram.org](https://my.telegram.org) and log in.
2. Open **API development tools → App configuration**.
3. Note your **App api_id** and **App api_hash**.

### 2 — Generate a session string

Run the session creator locally (requires [Bun](https://bun.sh) installed):

```bash
cd scripts
bun install
bun run create-session.ts
```

Follow the prompts (phone number → Telegram code → optional 2FA password).
Copy the printed session string.

### 3 — Add repository secrets

Go to **Settings → Secrets and variables → Actions** in your repo and create:

| Secret name   | Value                          |
| ------------- | ------------------------------ |
| `TG_API_ID`   | Your App api_id (integer)      |
| `TG_API_HASH` | Your App api_hash              |
| `TG_SESSION`  | The session string from step 2 |

### 4 — Commit with a channel ID

```bash
git commit --allow-empty -m "download files [tg:-1001234567890]"
git push
```

Watch the **Actions** tab — each step streams live logs.

---

## Output structure

```
downloads/
└── -1001234567890/
    ├── channel_-1001234567890_part001.zip
    ├── channel_-1001234567890_part002.zip
    └── …
```

---

## Scripts

| Script                        | Purpose                                          |
| ----------------------------- | ------------------------------------------------ |
| `scripts/create-session.ts`   | Interactive session generator — run locally once |
| `scripts/download-channel.ts` | Downloader — executed by the workflow            |

---

## Notes

- The session string grants **full access** to the Telegram account — treat it like a password.
- The account running the workflow must already be a member of the target channel.
- Channel IDs for supergroups/channels are usually negative and start with `-100`.
- If a zip part is still larger than 90 MB after compression, it means a single source file exceeds 90 MB. The file is placed in its own part; there is no way to split a binary file without corrupting it.
- Re-running with the same channel ID will re-download all files. Existing zip parts in the `downloads/` folder are overwritten.
