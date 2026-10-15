# Frequently Asked Questions

## Do you collect telemetry data?

No.

Grimoire is self-hosted, you own your data, and it stays on your server. Sending any of it to an external service without a user directly doing so goes against the ethos of the project. Grimoire can pull data from external sources, but server owners can disable that behavior entirely if they'd prefer a fully isolated instance.

This is a deliberate tradeoff. Without telemetry I don't receive crash and error reports, feature usage statistics, performance metrics, or version adoption data — the kinds of signals most developers rely on to catch bugs early and decide what to work on next. That means I depend on you: if something breaks or feels rough, please open an issue on GitHub or drop a note in the Discord. A good bug report is worth more to me than any analytics dashboard.

---

## I forgot my admin password. How do I reset it?

Reset the password directly in the database by running a one-liner inside the running container:

```bash
docker exec <container_name> python3 -c "
from passlib.context import CryptContext
import sqlite3
pwd = CryptContext(schemes=['bcrypt_sha256']).hash('<your_new_password>')
db = sqlite3.connect('/app/data/grimoire.db')
db.execute(\"UPDATE users SET hashed_password = ? WHERE username = ?\", (pwd, '<your_username>'))
db.commit()
db.close()
print('Done')
"
```

Replace `<container_name>`, `<your_new_password>`, and `<your_username>` with your values. The container name is typically `grimoire`.

Alternatively, create a new admin account by [pre-seeding a users file](/configuration/users) and restarting the stack.

---

## The scanner finds no books after I reorganized my library.

Grimoire expects a `books/` subfolder at the root of the library mount:

```
/app/library/
  books/
    D&D 5e/
      Core Rules/
        Players Handbook.pdf
  maps/
  tokens/
```

If your PDFs live directly under the mounted folder without a `books/` subfolder, the scanner finds nothing.

**Fix**: mount your folder directly as the `books/` directory:

```yaml
volumes:
  - /path/to/your/rpgs:/app/library/books:ro
  - /path/to/grimoire/data:/app/data
```

This keeps your existing host structure without adding an extra `books/` folder. Restart the stack and trigger a rescan.

::: warning Match the path you actually mount
`/app/library` is only the default. If your compose file sets `LIBRARY_PATH`, mount
`books/` under *that* path instead — for example, the
[Docker install guide](/guide/docker-install#step-3-create-the-compose-file) uses
`LIBRARY_PATH=/library`, so the mount there is `/path/to/your/rpgs:/library/books:ro`.
A mount that doesn't match `LIBRARY_PATH` scans nothing.
:::

::: info
**Cleanup missing** removes database records for files that can't be found at their expected paths. If you moved files before fixing the mount, those records were removed; re-mounting correctly and rescanning will re-add everything.
:::

---

## A folder is skipped by the scan, and the file manager shows `�` in its name

The folder's (or file's) name on disk is not valid UTF-8. This usually happens when the name was created on Windows in a legacy code page and copied to a share that stores names as raw bytes. For example, Windows-1252 writes the `Ö` in "MÖRK BORG" as the single byte `0xD6`, which on its own is not UTF-8. Copying through a Windows NFS mount is a common source.

Grimoire can't reliably guess which character was meant (the same byte is `Ö` in one code page and `Ц` in another), so it never renames anything for you. It does this instead:

- **The scan skips that entry** and carries on with everything else. The log shows a warning naming the path, with `�` in place of each bad character and a guess at the intended spelling:

  ```
  Skipping '/app/library/books/M�RK BORG': its name is not valid UTF-8 (read as Windows-1252 it would be '/app/library/books/MÖRK BORG'). Rename it on the share to include it in the library.
  ```

- **The file manager lists it greyed out**, with `�` for each bad character and a `?` icon that explains the problem. It can't be opened, selected, moved, renamed, or deleted from Grimoire, because its real name can't be sent to the browser.

::: warning Older versions
In older versions, one such name stopped the whole scan, so no new systems were added anywhere. The file manager showed "Internal Server Error", and the container log showed `UnicodeEncodeError ... surrogates not allowed`.
:::

**Fix**: rename the entry on the share itself, then rescan. To list every affected path, run this against the container. It reads your `LIBRARY_PATH`, so it works whatever the mount is:

```bash
docker exec -i grimoire python3 - <<'EOF'
import os
for root, dirs, files in os.walk(os.environ.get("LIBRARY_PATH", "./library")):
    for name in dirs + files:
        raw = os.fsencode(os.path.join(root, name))
        try:
            raw.decode("utf-8")
        except UnicodeDecodeError:
            print(raw, "->", raw.decode("cp1252", "replace"))
EOF
```

On the Linux host or NAS that holds the files, [`convmv`](https://www.j3e.de/linux/convmv/) can fix them all at once. Run it without `--notest` first to preview the changes:

```bash
convmv -f cp1252 -t utf-8 -r /path/to/your/library            # preview
convmv -f cp1252 -t utf-8 -r --notest /path/to/your/library   # rename
```

::: tip Stop it happening again
Copy files onto the share over SMB, or from a Linux or macOS machine, rather than through a Windows NFS mount.
:::

---

## How do I configure OIDC with Authentik?

See the dedicated [Authentik Setup](/guide/oidc-authentik) guide for a complete walkthrough including group-to-role mapping and explicit content access control.
