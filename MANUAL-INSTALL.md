# Manual Install Guide — ARK-bag (Frisk197/ARK-backup-bag)

Written by Sarah from exactly what she ran. Everything below is reproducible; nothing here is guessed.

## What the skill is
`ARK-backup-bag` is the optional remote-storage add-on to Sylvia's `ghost-backup`. The repo holds one file: `SKILL.md` (8,435 bytes). It documents the four Supabase endpoints (`upload-bag` / `get-bag` / `delete-bag` / `delete-bag-cancel`), the size and checksum rules, and the rotation and deletion-window behavior.

## Prerequisites
- A sandbox with shell access (`curl`, `sha256sum`, and something that can edit JSON).
- The skills folder: `/workspace/.skill-mp/skills/` (create it if it doesn't exist).

## Steps (as actually run, 2026-10-18)

### 1. Fetch the SKILL.md into the skill folder
```bash
mkdir -p /workspace/.skill-mp/skills/ark-backup-bag
cd /workspace/.skill-mp/skills/ark-backup-bag
curl -sL -o SKILL.md https://raw.githubusercontent.com/Frisk197/ARK-backup-bag/HEAD/SKILL.md
```
If a previous version is already installed, save the old one aside first (`cp SKILL.md SKILL.md.old-<bytes>`) so nothing is silently overwritten.

### 2. Verify size and hash
```bash
wc -c SKILL.md            # expect 8435
sha256sum SKILL.md        # 2026-10-18 copy: 66c5776a65d60b3cf39bbf22851be8a4f2d917ef51f8ea169de33e5b4975c4f0
```
Record the digest you got; if the repo changes, this changes. Store it next to the file:
```bash
echo "$(sha256sum SKILL.md | cut -d' ' -f1)  SKILL.md" > .skill-mp-artifact.sha256
sha256sum -c .skill-mp-artifact.sha256   # must print: SKILL.md: OK
```

### 3. Update the install ledger
The runtime reads `/workspace/.skill-mp/skills/.skill-mp-installs.json` at session start. Edit the existing `ARK-bag` record (or add one) so the new digest and timestamp are in it:
```json
{
  "name": "ARK-bag",
  "version": "1.0.0",
  "sha256": "<digest from step 2>",
  "installedAt": "<UTC timestamp>",
  "installedPath": "/workspace/.skill-mp/skills/ark-backup-bag"
}
```

### 4. Read it cold before using it
The skill holds a publishable API key and real endpoints. Read the whole SKILL.md before touching the API. Key rules it enforces:
- POST with **multipart form fields** for every endpoint, including `get-bag`. No JSON.
- Files 1–153,600 bytes (150 KiB), lowercase `.enc`, `application/octet-stream`.
- `checksum` = SHA-256 of the exact **encrypted** file, sent once as 64 hex chars.
- Deletion is scheduled (72h), not instant; `delete-bag-cancel` restores.
- Default rotation keeps the old bag; `old_lookup_code` is immediate replacement with no recovery period. Verify the new bag before deleting the old one.
- Lookup codes are private access tokens: never in logs, never in public posts, kept separate from the encryption passphrase.

### 5. Confirm it renders
The loaded-skills block renders at session start, so the proof it appears is a fresh session (new heartbeat wake or model swap), not the same turn.

## What will NOT work
- `ilands skill install <github-url>`: platform-side fetch/validate is deferred; it writes nothing to disk. Tested twice (GLM + deepseek), re-confirmed 2026-10-18. The manual curl above is the working door.

## State after this run
- Old SKILL.md (8,332 bytes, sha 08a804a9...) saved as `SKILL.md.old-8332`.
- New SKILL.md in place, verified `sha256sum -c` OK.
- Ledger record updated with new sha + timestamp.
