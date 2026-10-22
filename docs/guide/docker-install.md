# Getting Started with Docker

This guide is for people who have never used Docker before. It walks you through installing Docker and running Grimoire, no technical background required.

::: info Local access only
This guide sets up Grimoire for local access. You can use it from your own computer and other devices on your home network. Making it accessible outside your network requires additional steps (port forwarding, VPN, reverse proxy) that are outside this guide's scope.
:::

## What is Docker?

Docker lets you run applications in a self-contained package called a **container**. You don't need to install Python, configure databases, or set up servers. Docker handles all of that. Installing Docker is the only technical step required.

## Step 1: Install Docker

### Windows

1. Make sure you're running **Windows 10 (version 2004+)** or **Windows 11**.
2. Go to [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/) and click **Download for Windows**.
3. Run the installer and follow the prompts. Leave **Use WSL 2 instead of Hyper-V** checked.
4. Restart your computer if prompted.
5. Open **Docker Desktop** from the Start menu. You'll see a whale icon in your system tray when it's running.

::: tip
If you see a message about enabling virtualization, search for your computer model + "enable virtualization" for BIOS instructions specific to your hardware.
:::

### macOS

1. Go to [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/) and click **Download for Mac**.
   - **Apple Silicon (M1/M2/M3/M4):** choose Mac with Apple Silicon
   - **Intel:** choose Mac with Intel Chip
   - Not sure? Click the Apple menu → **About This Mac** and look at the chip line.
2. Open the downloaded `.dmg` and drag Docker to Applications.
3. Open Docker from Applications. Enter your password when prompted.
4. Docker is ready when you see the whale icon in the menu bar.

### Linux

1. Visit [docs.docker.com/engine/install](https://docs.docker.com/engine/install/) and follow the steps for your distribution.
2. After installing, run:
   ```bash
   sudo usermod -aG docker $USER
   ```
   Then log out and back in.
3. Verify it works:
   ```bash
   docker run hello-world
   ```

## Step 2: Create your folders

Create two folders somewhere on your computer:

- **Library**: your PDFs, maps, and tokens (e.g. `Documents/grimoire/library`)
- **Data**: app database and thumbnails (e.g. `Documents/grimoire/data`)

Inside `library/`, create:

```
library/
├── books/
├── maps/
└── tokens/
```

Inside `books/`, create a subfolder for each game system, then place your PDFs:

```
library/books/
└── Dungeons and Dragons 5e/
    └── core/
        ├── Players Handbook.pdf
        └── Dungeon Masters Guide.pdf
```

::: warning Already have a folder full of PDFs?
Grimoire only scans inside `books/`, `maps/`, and `tokens/`. If your PDFs live
directly under the mounted folder without a `books/` subfolder, the scanner finds
nothing.

Rather than reshuffling your files, mount your existing folder straight onto
`books/` in [Step 3](#step-3-create-the-compose-file):

```yaml
volumes:
  # Mount an existing PDF folder directly as the books/ directory
  - /path/to/your/rpgs:/library/books:ro
  - /YOUR/DATA/PATH:/data
```

This keeps your host structure untouched. See the
[FAQ](/faq#the-scanner-finds-no-books-after-i-reorganized-my-library) for more.
:::

## Step 3: Create the compose file

Use the **[Compose Generator](/compose-generator)** to build your `docker-compose.yml` with your paths and a generated secret key already filled in, then just download and run it.

Or create a file called `docker-compose.yml` manually and paste in:

```yaml
services:
  grimoire:
    image: hunterreadca/grimoire:latest
    container_name: grimoire
    restart: unless-stopped
    ports:
      - "9481:9481"
    environment:
      - LIBRARY_PATH=/library
      - DATA_PATH=/data
      - WORKERS=1
      - SECRET_KEY=<paste the generated value here>
    volumes:
      - /YOUR/LIBRARY/PATH:/library:ro
      # Or, if your PDFs already sit in one folder, mount it as books/ instead:
      # - /path/to/your/rpgs:/library/books:ro
      - /YOUR/DATA/PATH:/data
```

### Volume path format by OS

**Windows**
```yaml
volumes:
  - C:/Users/YourName/Documents/grimoire/library:/library:ro
  - C:/Users/YourName/Documents/grimoire/data:/data
```

**macOS / Linux**
```yaml
volumes:
  - /Users/YourName/Documents/grimoire/library:/library:ro
  - /Users/YourName/Documents/grimoire/data:/data
```

Generate a random value of at least 32 characters, for example with `openssl rand -hex 32` or a password generator, and paste it in as `SECRET_KEY`. You can also delete the `SECRET_KEY` line: Grimoire then generates a key on first start and saves it in your data folder, so you stay signed in across restarts.

::: warning Don't copy a key
Never use a value from documentation or a forum post, or a placeholder like `change-me`. A known signing key would let anyone sign in as any user on your instance, so Grimoire refuses to start on values that have appeared in its own docs, including the `<paste the generated value here>` line above left as it is. See [the FAQ](/faq#grimoire-won-t-start-and-the-log-mentions-secret-key) if you hit that error.
:::

## Step 4: Run Grimoire

Open a terminal in your Grimoire folder and run:

```bash
docker compose up -d
```

Then open [http://localhost:9481](http://localhost:9481) in your browser.

::: tip Checking container health
The image ships a `HEALTHCHECK` that probes `GET /api/health`, so `docker ps` shows `(healthy)` or `(unhealthy)` rather than just "running" once the app is serving and can reach its database. Orchestrators can gate startup on it with `depends_on: { grimoire: { condition: service_healthy } }`.
:::

::: info Image variants
The default `latest` tag bundles the Tesseract OCR engine so scanned, image-only PDFs are searchable. For a smaller image without OCR, use the matching `-slim` tag (e.g. `hunterreadca/grimoire:slim`). See [OCR](/configuration/performance#ocr).
:::

::: info Docker Hub or GitHub
Grimoire is published to both [Docker Hub](https://hub.docker.com/r/hunterreadca/grimoire) and the [GitHub Container Registry](https://github.com/hunter-read/grimoire/pkgs/container/grimoire), with the same images under the same tags. Either works - if you'd rather pull from GitHub, use `ghcr.io/hunter-read/grimoire:latest` as the `image:` instead. The [Compose Generator](/compose-generator) has a registry option for this.
:::

## Step 5: Create your admin account

On first visit you'll be prompted to create an admin account. Pick a username and password.

Grimoire will start indexing your library in the background. You can already browse the app while it works.

## Troubleshooting

**The page won't load at `localhost:9481`**
- Make sure Docker Desktop is running (check for the whale icon).
- Run `docker compose up -d` again and wait a few seconds before refreshing.

**Port is already in use**
- Change `"9481:9481"` to `"9482:9481"` in the compose file and access the app at `localhost:9482`.

**My PDFs are not showing up**
- Double-check the volume path in `docker-compose.yml`: make sure it points to the right folder.
- After adding new files, use **Rescan** in the Grimoire sidebar.

**Windows: Docker says WSL 2 is not installed**
- Open PowerShell as Administrator and run `wsl --install`, then restart.

**The log says Grimoire is "staying root"**
- Grimoire runs as an ordinary user, not root, and it only stays root when that user couldn't reach something. The log line names the folder. Set `PUID`/`PGID` to the folder's owner, or see [File ownership](/configuration/volumes#file-ownership).
