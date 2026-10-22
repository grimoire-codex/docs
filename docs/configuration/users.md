# Pre-seeding Users

Drop a `users.json` file into your data directory before first start and Grimoire will create those accounts automatically. Afterwards the file is replaced by `users.json.imported`, a record of what was imported with every password replaced by its hash, so no plaintext password is left on disk. It is never processed again.

## Format

```json
[
  {
    "username": "admin",
    "password": "a-long-passphrase-you-chose",
    "role": "admin"
  },
  {
    "username": "gm",
    "password": "$bcrypt-sha256$v=2,t=2b,r=12$...",
    "role": "gm"
  },
  {
    "username": "alice",
    "password": "another-long-passphrase",
    "role": "player",
    "denyExplicit": true
  }
]
```

| Field | Required | Description |
|---|---|---|
| `username` | Yes | Login username |
| `password` | Yes | Plaintext password **or** a pre-hashed `$bcrypt-sha256$` string |
| `role` | No | `admin`, `gm`, or `player`: defaults to `player` |
| `denyExplicit` | No | `true` to turn explicit content off for this user, locked so they can't turn it back on themselves; defaults to `false` |

## Rules

- At least one entry must have `"role": "admin"`: the file is rejected otherwise.
- Plaintext passwords follow the same rules as any new password: at least `PASSWORD_MIN_LENGTH` characters (12 by default) and not a commonly used one. An entry that fails is skipped with a line in the log. If no admin entry passes, nothing is imported and the file is left in place to fix. Pre-hashed passwords are taken as they are.
- Entries whose username already exists are silently skipped.
- On parse or validation errors the file is left untouched so you can fix and restart.

## Pre-hashing passwords

Pre-hashing avoids storing plaintext passwords in the JSON file. Grimoire uses the `bcrypt_sha256` scheme (the same format passlib writes, so a hash from passlib works too). To make one with a running Grimoire container:

```bash
docker exec <container_name> python3 -c "from backend.password_hash import hash_password; print(hash_password('yourpassword'))"
```

Copy the output (starts with `$bcrypt-sha256$`) into the `password` field.

## Docker example

```bash
cp users.json /path/to/grimoire/data/users.json
docker compose up -d
```

If you start from the repository's `users.json.example`, replace every `changeme` first: those placeholder passwords are refused on purpose, so an unedited copy imports nothing.
