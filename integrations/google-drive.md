# Google Drive Integration

## What it does

Upload, download, search, and manage files on Google Drive. Used for storing documents, attachments, and sharing files across workflows.

## Auth

- **Method**: OAuth2 via Google Cloud project (Desktop app credentials)
- **Credentials**: `google-drive/credentials.json` (OAuth client secret, gitignored)
- **Token**: `google-drive/token.json` (auto-refreshed, gitignored)
- **Scopes**: `https://www.googleapis.com/auth/drive` (full read/write)

The same Google Cloud project and `credentials.json` can be shared with other Google integrations (Gmail, Calendar).

To re-authenticate:

```bash
cd google-drive && uv run python auth.py
```

## Common Operations

All commands run from `google-drive/`.

### List recent files

```bash
uv run python client.py list 10
```

### Search files

```bash
uv run python client.py search "name contains 'invoice'"
```

### Download a file

```bash
uv run python client.py download <file_id> <local_path>
```

### Upload a file

```bash
uv run python client.py upload <local_path> <parent_folder_id>
```

### Create a folder

```bash
uv run python client.py mkdir "Folder Name" <parent_folder_id>
```

## Scripts

| Script | Purpose |
|--------|---------|
| `google-drive/auth.py` | OAuth2 flow — handles token refresh and first-time browser auth |
| `google-drive/client.py` | CLI for Drive operations (list, search, download, upload, mkdir) |

## Gotchas

- Token auto-refreshes but will eventually expire if unused for months — re-run `auth.py`.
- File IDs are opaque strings. Use search or list to find them.
- Folder creation requires a parent folder ID. Use `root` for the top level.
- Google Docs/Sheets/Slides export as PDF by default when downloading.
