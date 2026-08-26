<template>
  <div class="compose-generator">
    <!-- Runtime -->
    <section class="cg-section">
      <h2>Container runtime</h2>
      <div class="cg-radio-group">
        <label v-for="rt in runtimes" :key="rt.value" :class="['cg-radio', { active: form.runtime === rt.value }]">
          <input type="radio" :value="rt.value" v-model="form.runtime" />
          <span class="cg-radio-label">{{ rt.label }}</span>
          <span class="cg-radio-desc">{{ rt.desc }}</span>
        </label>
      </div>
      <p v-if="form.runtime === 'quadlet'" class="cg-note">
        Quadlets generate systemd <code>.container</code> unit files: the recommended rootless Podman deployment.
        One file per service, placed in <code>~/.config/containers/systemd/</code>.
      </p>
    </section>

    <!-- Image -->
    <section class="cg-section">
      <h2>Grimoire image</h2>
      <div class="cg-row">
        <div class="cg-field">
          <label>Image tag</label>
          <div class="cg-input-row">
            <select v-model="form.imageTag">
              <option value="latest">latest</option>
              <option value="custom">pin to version…</option>
            </select>
            <input v-if="form.imageTag === 'custom'" v-model="form.customTag" placeholder="e.g. 1.0.0" class="cg-inline-input" />
          </div>
        </div>
        <div class="cg-field">
          <label>Host port</label>
          <input type="number" v-model.number="form.hostPort" min="1" max="65535" />
        </div>
        <div class="cg-field">
          <label>Workers</label>
          <input type="number" v-model.number="form.workers" min="1" max="16" />
        </div>
      </div>
      <label class="cg-toggle" style="margin-top: 0.75rem;">
        <input type="checkbox" v-model="form.slimImage" />
        <span>Slim image <span class="cg-hint">(smaller, no Tesseract / OCR — uses the <code>-slim</code> tag)</span></span>
      </label>
      <p class="cg-note">Resolved image: <code>{{ imageRef }}</code></p>
    </section>

    <!-- Paths -->
    <section class="cg-section">
      <h2>Volume paths</h2>
      <div class="cg-field">
        <label>Library path <span class="cg-hint">(host path to your TTRPG library folder)</span></label>
        <input v-model="form.libraryPath" placeholder="/path/to/your/library" />
      </div>
      <div class="cg-field">
        <label>Data path <span class="cg-hint">(host path for database, thumbnails, cache)</span></label>
        <input v-model="form.dataPath" placeholder="/path/to/grimoire/data" />
      </div>
    </section>

    <!-- Secret key -->
    <section class="cg-section">
      <h2>Secret key</h2>
      <p class="cg-desc">Used to sign JWT tokens. Must be a long random string; keep it private.</p>
      <div class="cg-input-row">
        <input
          :type="showSecret ? 'text' : 'password'"
          v-model="form.secretKey"
          class="cg-secret-input"
          placeholder="Paste your own or generate one below"
        />
        <button class="cg-btn cg-btn-secondary" @click="toggleShowSecret">
          {{ showSecret ? 'Hide' : 'Show' }}
        </button>
        <button class="cg-btn cg-btn-secondary" @click="generateSecret">Regenerate</button>
        <button class="cg-btn cg-btn-secondary" @click="copySecret">Copy</button>
      </div>
      <p v-if="copiedSecret" class="cg-copied">Copied!</p>
    </section>

    <!-- Optional: base URL -->
    <section class="cg-section">
      <h2>Optional settings</h2>
      <div class="cg-row">
        <div class="cg-field">
          <label>
            Base URL
            <span class="cg-hint">(set this when running behind a reverse proxy; used for OPDS and OIDC redirect URIs)</span>
          </label>
          <input v-model="form.baseUrl" placeholder="https://grimoire.example.com" />
        </div>
        <div class="cg-field">
          <label>Log level</label>
          <select v-model="form.logLevel">
            <option value="debug">debug</option>
            <option value="info">info</option>
            <option value="warning">warning</option>
            <option value="error">error</option>
            <option value="critical">critical</option>
          </select>
        </div>
      </div>
      <div class="cg-toggles">
        <label class="cg-toggle">
          <input type="checkbox" v-model="form.opdsEnabled" />
          <span>Enable OPDS catalog</span>
        </label>
        <label class="cg-toggle">
          <input type="checkbox" v-model="form.valkeyEnabled" />
          <span>Valkey page cache <span class="cg-hint">(recommended for large libraries)</span></span>
        </label>
      </div>
    </section>

    <!-- Authentication -->
    <section class="cg-section">
      <h2>Authentication <span class="cg-hint optional">(optional)</span></h2>
      <p class="cg-desc">
        These pin auth behaviour via environment variables, overriding the in-app toggles (which then show read-only).
        Leave them alone to manage everything from <strong>Settings → Authentication</strong> in the app.
      </p>
      <div class="cg-toggles">
        <label class="cg-toggle">
          <input type="checkbox" v-model="form.guestAccessEnabled" />
          <span>Enable guest access <span class="cg-hint">(per-campaign invite codes for players without accounts)</span></span>
        </label>
      </div>
      <div class="cg-field" style="margin-top: 0.75rem; max-width: 320px;">
        <label>Password authentication</label>
        <select v-model="form.passwordAuth">
          <option value="default">App default (manage in-app)</option>
          <option value="on">Force on</option>
          <option value="off">Force off (OIDC only)</option>
        </select>
        <span v-if="form.passwordAuth === 'off'" class="cg-hint">
          First-run admin setup still requires a password. Configure OIDC in-app or via env vars before disabling.
        </span>
      </div>
    </section>

    <!-- OCR -->
    <section v-if="!form.slimImage" class="cg-section">
      <h2>OCR <span class="cg-hint optional">(scanned / image-only PDFs)</span></h2>
      <p class="cg-desc">
        Scanned PDFs with no text layer are run through OCR in the background so they become searchable.
        Bundled with the default image; omitted from <code>-slim</code> tags.
      </p>
      <div class="cg-radio-group">
        <label v-for="m in ocrModes" :key="m.value" :class="['cg-radio', { active: form.ocrMode === m.value }]">
          <input type="radio" :value="m.value" v-model="form.ocrMode" />
          <span class="cg-radio-label">{{ m.label }}</span>
          <span class="cg-radio-desc">{{ m.desc }}</span>
        </label>
      </div>
      <div v-if="form.ocrMode === 'custom'" class="cg-sub-fields">
        <div class="cg-row">
          <div class="cg-field">
            <label>Languages <span class="cg-hint">(+-joined Tesseract codes)</span></label>
            <input v-model="form.ocrLanguages" placeholder="eng or eng+deu+fra" />
          </div>
          <div class="cg-field">
            <label>Concurrency <span class="cg-hint">(books at once; ~250–400 MB RAM each)</span></label>
            <input type="number" v-model.number="form.ocrConcurrency" min="1" max="32" />
          </div>
          <div class="cg-field">
            <label>DPI <span class="cg-hint">(render resolution, 72–600)</span></label>
            <input type="number" v-model.number="form.ocrDpi" min="72" max="600" step="10" />
          </div>
        </div>
        <p class="cg-note">
          Raise concurrency on multi-core hosts with spare RAM; keep it at 1 on small devices.
          Higher DPI improves faint scans but is slower and heavier.
        </p>
      </div>
    </section>

    <!-- Library access -->
    <section class="cg-section">
      <h2>Library access</h2>
      <p class="cg-desc">
        A writable library mount lets admins upload, move, rename, and delete files from inside
        Grimoire, and lets it export metadata sidecars. Uncheck it to mount the library
        <code>:ro</code> — everything else works the same, and those two features are hidden.
      </p>
      <div class="cg-toggles">
        <label class="cg-toggle">
          <input type="checkbox" v-model="form.libraryWritable" />
          <span>Writable library <span class="cg-hint">(in-app file management and sidecar export)</span></span>
        </label>
      </div>
    </section>

    <!-- Companion tools -->
    <section class="cg-section">
      <h2>Companion tools <span class="cg-hint optional">(optional)</span></h2>
      <p class="cg-desc">
        Grimoire manages library files itself. Add Calibre alongside it for ebook conversion and
        bulk metadata editing.
      </p>
      <div class="cg-radio-group">
        <label v-for="fm in fileManagers" :key="fm.value" :class="['cg-radio', { active: form.fileManager === fm.value }]">
          <input type="radio" :value="fm.value" v-model="form.fileManager" />
          <span class="cg-radio-label">{{ fm.label }}</span>
          <span class="cg-radio-desc">{{ fm.desc }}</span>
        </label>
      </div>
      <div v-if="form.fileManager === 'calibre' || form.fileManager === 'calibre-web'" class="cg-sub-fields">
        <div class="cg-row">
          <div class="cg-field">
            <label>PUID</label>
            <input type="number" v-model.number="form.puid" min="0" />
          </div>
          <div class="cg-field">
            <label>PGID</label>
            <input type="number" v-model.number="form.pgid" min="0" />
          </div>
          <div class="cg-field">
            <label>Timezone</label>
            <input v-model="form.timezone" placeholder="America/New_York" list="tz-suggestions" />
            <datalist id="tz-suggestions">
              <option v-for="tz in commonTimezones" :key="tz" :value="tz" />
            </datalist>
          </div>
        </div>
      </div>
    </section>

    <!-- Output -->
    <section class="cg-section cg-output-section">
      <div class="cg-output-header">
        <h2>{{ outputFilename }}</h2>
        <div class="cg-output-actions">
          <button class="cg-btn cg-btn-secondary" @click="copyOutput">
            {{ copiedOutput ? 'Copied!' : 'Copy' }}
          </button>
          <button class="cg-btn cg-btn-primary" @click="downloadOutput">
            Download {{ outputFilename }}
          </button>
        </div>
      </div>
      <div v-if="form.runtime === 'quadlet'" class="cg-quadlet-tabs">
        <button
          v-for="(file, name) in quadletFiles"
          :key="name"
          :class="['cg-tab', { active: activeQuadletTab === name }]"
          @click="activeQuadletTab = name"
        >
          {{ name }}
        </button>
      </div>
      <pre class="cg-preview"><code>{{ currentOutput }}</code></pre>
    </section>
  </div>
</template>

<script setup>
import { ref, reactive, computed, watch, onMounted } from 'vue'

// ── State ────────────────────────────────────────────────────────────────────

const form = reactive({
  runtime: 'docker',
  imageTag: 'latest',
  customTag: '',
  slimImage: false,
  hostPort: 9481,
  workers: 2,
  libraryPath: '/path/to/your/library',
  dataPath: '/path/to/grimoire/data',
  secretKey: '',
  showSecret: false,
  baseUrl: '',
  logLevel: 'info',
  opdsEnabled: false,
  valkeyEnabled: false,
  guestAccessEnabled: false,
  passwordAuth: 'default',
  ocrMode: 'default',
  ocrLanguages: 'eng',
  ocrConcurrency: 1,
  ocrDpi: 150,
  libraryWritable: true,
  fileManager: 'none',
  puid: 1000,
  pgid: 1000,
  timezone: 'America/New_York',
})

const showSecret = ref(false)
const copiedSecret = ref(false)
const copiedOutput = ref(false)
const activeQuadletTab = ref('grimoire.container')

// ── Constants ────────────────────────────────────────────────────────────────

const runtimes = [
  { value: 'docker', label: 'Docker Compose', desc: 'docker-compose.yml' },
  { value: 'podman', label: 'Podman Compose', desc: 'podman-compose.yml (compatible with docker compose)' },
  { value: 'quadlet', label: 'Podman Quadlets', desc: 'Systemd .container unit files (recommended for rootless Podman)' },
]

const ocrModes = [
  { value: 'default', label: 'Default', desc: 'OCR on with built-in defaults (English, 1 book at a time, 150 DPI)' },
  { value: 'custom', label: 'Customize…', desc: 'Set languages, parallelism, and render resolution' },
  { value: 'disabled', label: 'Disabled', desc: 'Turn OCR off (scanned PDFs stay unindexed)' },
]

const fileManagers = [
  { value: 'none', label: 'None', desc: 'Grimoire only — manage files in-app or from your OS' },
  { value: 'calibre', label: 'Calibre', desc: 'Full Calibre desktop via noVNC: metadata editing + OPF export (port 8080)' },
  { value: 'calibre-web', label: 'Calibre-Web', desc: 'Lightweight browser UI for an existing Calibre library (port 8083)' },
]

const commonTimezones = [
  'America/New_York', 'America/Chicago', 'America/Denver', 'America/Los_Angeles',
  'America/Toronto', 'America/Vancouver', 'America/Sao_Paulo',
  'Europe/London', 'Europe/Paris', 'Europe/Berlin', 'Europe/Amsterdam',
  'Europe/Stockholm', 'Europe/Helsinki', 'Europe/Warsaw',
  'Asia/Tokyo', 'Asia/Seoul', 'Asia/Shanghai', 'Asia/Singapore',
  'Asia/Kolkata', 'Asia/Dubai',
  'Australia/Sydney', 'Australia/Melbourne', 'Pacific/Auckland',
]

// ── Helpers ──────────────────────────────────────────────────────────────────

function generateRandomHex(bytes = 32) {
  const arr = new Uint8Array(bytes)
  crypto.getRandomValues(arr)
  return Array.from(arr).map(b => b.toString(16).padStart(2, '0')).join('')
}

function generateSecret() {
  form.secretKey = generateRandomHex(32)
}

function toggleShowSecret() {
  showSecret.value = !showSecret.value
}

async function copySecret() {
  if (!form.secretKey) return
  await navigator.clipboard.writeText(form.secretKey)
  copiedSecret.value = true
  setTimeout(() => { copiedSecret.value = false }, 2000)
}

onMounted(() => {
  if (!form.secretKey) generateSecret()
})

// ── Computed image ref ───────────────────────────────────────────────────────

const imageRef = computed(() => {
  const base = form.imageTag === 'custom' ? (form.customTag || 'latest') : form.imageTag
  const tag = form.slimImage ? `${base}-slim` : base
  return `hunterreadca/grimoire:${tag}`
})

// ── Compose generation ───────────────────────────────────────────────────────

// Env vars beyond the always-present core, as [KEY, value] pairs. Shared by the
// compose and quadlet generators so both stay in sync.
function extraEnvPairs() {
  const pairs = []
  if (form.baseUrl) pairs.push(['BASE_URL', form.baseUrl])
  if (form.logLevel && form.logLevel !== 'info') pairs.push(['LOG_LEVEL', form.logLevel])
  if (form.opdsEnabled) pairs.push(['OPDS_ENABLED', 'true'])
  if (form.valkeyEnabled) pairs.push(['VALKEY_URL', 'redis://valkey:6379/0'])
  if (form.guestAccessEnabled) pairs.push(['GUEST_ACCESS_ENABLED', 'true'])
  if (form.passwordAuth === 'on') pairs.push(['ALLOW_PASSWORD_AUTHENTICATION', 'true'])
  else if (form.passwordAuth === 'off') pairs.push(['ALLOW_PASSWORD_AUTHENTICATION', 'false'])
  // OCR settings only apply to the OCR-capable (non-slim) image.
  if (!form.slimImage) {
    if (form.ocrMode === 'disabled') {
      pairs.push(['OCR_ENABLED', 'false'])
    } else if (form.ocrMode === 'custom') {
      if (form.ocrLanguages && form.ocrLanguages !== 'eng') pairs.push(['OCR_LANGUAGES', form.ocrLanguages])
      if (Number(form.ocrConcurrency) !== 1) pairs.push(['OCR_CONCURRENCY', String(form.ocrConcurrency)])
      if (Number(form.ocrDpi) !== 150) pairs.push(['OCR_DPI', String(form.ocrDpi)])
    }
  }
  return pairs
}

function envBlock(indent = '      ') {
  const lines = [
    `${indent}- LIBRARY_PATH=/library`,
    `${indent}- DATA_PATH=/data`,
    `${indent}- WORKERS=${form.workers}`,
    `${indent}- SECRET_KEY=${form.secretKey || 'change-me'}`,
  ]
  for (const [k, v] of extraEnvPairs()) lines.push(`${indent}- ${k}=${v}`)
  return lines.join('\n')
}

function volumeBlock(indent = '      ') {
  const ro = form.libraryWritable ? '' : ':ro'
  return [
    `${indent}- ${form.libraryPath}:/library${ro}`,
    `${indent}- ${form.dataPath}:/data`,
  ].join('\n')
}

function grimoireService(useNetworks = false) {
  const net = useNetworks ? '\n    networks:\n      - grimoire' : ''
  const dep = form.valkeyEnabled ? '\n    depends_on:\n      - valkey' : ''
  return `  grimoire:
    image: ${imageRef.value}
    container_name: grimoire
    restart: unless-stopped
    ports:
      - "${form.hostPort}:9481"
    environment:
${envBlock()}
    volumes:
${volumeBlock()}${dep}${net}`
}

function valkeyService() {
  return `
  valkey:
    image: valkey/valkey:8-alpine
    container_name: grimoire-valkey
    restart: unless-stopped
    volumes:
      - valkey_data:/data
    networks:
      - grimoire`
}

function calibreService() {
  return `
  calibre:
    image: lscr.io/linuxserver/calibre:latest
    container_name: grimoire-calibre
    restart: unless-stopped
    environment:
      - PUID=${form.puid}
      - PGID=${form.pgid}
      - TZ=${form.timezone}
    ports:
      - "8080:8080"
      - "8081:8081"
    volumes:
      - ${form.libraryPath}:/library
      - calibre_config:/config
    networks:
      - grimoire`
}

function calibreWebService() {
  return `
  calibre-web:
    image: lscr.io/linuxserver/calibre-web:latest
    container_name: grimoire-calibre-web
    restart: unless-stopped
    environment:
      - PUID=${form.puid}
      - PGID=${form.pgid}
      - TZ=${form.timezone}
      - DOCKER_MODS=linuxserver/mods:universal-calibre
    ports:
      - "8083:8083"
    volumes:
      - ${form.libraryPath}:/library
      - calibre_web_config:/config
    networks:
      - grimoire`
}

function volumesBlock() {
  const vols = []
  if (form.valkeyEnabled) vols.push('  valkey_data:')
  if (form.fileManager === 'calibre') vols.push('  calibre_config:')
  if (form.fileManager === 'calibre-web') vols.push('  calibre_web_config:')
  return vols.length ? `\nvolumes:\n${vols.join('\n')}` : ''
}

function networksBlock() {
  return `\nnetworks:\n  grimoire:`
}

const needsNetwork = computed(() =>
  form.valkeyEnabled || form.fileManager !== 'none'
)

function buildCompose() {
  const useNet = needsNetwork.value
  let out = `services:\n${grimoireService(useNet)}`
  if (form.valkeyEnabled) out += valkeyService()
  if (form.fileManager === 'calibre') out += calibreService()
  if (form.fileManager === 'calibre-web') out += calibreWebService()
  out += volumesBlock()
  if (useNet) out += networksBlock()
  return out
}

// ── Quadlet generation ───────────────────────────────────────────────────────

function buildQuadletGrimoire() {
  const envLines = [
    `Environment=LIBRARY_PATH=/library`,
    `Environment=DATA_PATH=/data`,
    `Environment=WORKERS=${form.workers}`,
    `Environment=SECRET_KEY=${form.secretKey || 'change-me'}`,
  ]
  for (const [k, v] of extraEnvPairs()) envLines.push(`Environment=${k}=${v}`)

  const deps = []
  if (form.valkeyEnabled) deps.push('After=valkey.service')
  if (form.fileManager === 'calibre') deps.push('After=grimoire-calibre.service')
  if (form.fileManager === 'calibre-web') deps.push('After=grimoire-calibre-web.service')

  return `[Unit]
Description=Grimoire TTRPG Library Manager
${deps.length ? deps.join('\n') + '\n' : ''}
[Container]
Image=${imageRef.value}
ContainerName=grimoire
PublishPort=${form.hostPort}:9481
Volume=${form.libraryPath}:/library${form.libraryWritable ? '' : ':ro'}
Volume=${form.dataPath}:/data
${envLines.join('\n')}
AutoUpdate=registry

[Service]
Restart=always

[Install]
WantedBy=default.target`
}

function buildQuadletValkey() {
  return `[Unit]
Description=Valkey page cache for Grimoire

[Container]
Image=valkey/valkey:8-alpine
ContainerName=grimoire-valkey
Volume=grimoire-valkey-data:/data
AutoUpdate=registry

[Service]
Restart=always

[Install]
WantedBy=default.target`
}

function buildQuadletCalibre() {
  return `[Unit]
Description=Calibre desktop for Grimoire library management

[Container]
Image=lscr.io/linuxserver/calibre:latest
ContainerName=grimoire-calibre
PublishPort=8080:8080
PublishPort=8081:8081
Volume=${form.libraryPath}:/library
Volume=grimoire-calibre-config:/config
Environment=PUID=${form.puid}
Environment=PGID=${form.pgid}
Environment=TZ=${form.timezone}
AutoUpdate=registry

[Service]
Restart=always

[Install]
WantedBy=default.target`
}

function buildQuadletCalibreWeb() {
  return `[Unit]
Description=Calibre-Web for Grimoire library management

[Container]
Image=lscr.io/linuxserver/calibre-web:latest
ContainerName=grimoire-calibre-web
PublishPort=8083:8083
Volume=${form.libraryPath}:/library
Volume=grimoire-calibre-web-config:/config
Environment=PUID=${form.puid}
Environment=PGID=${form.pgid}
Environment=TZ=${form.timezone}
Environment=DOCKER_MODS=linuxserver/mods:universal-calibre
AutoUpdate=registry

[Service]
Restart=always

[Install]
WantedBy=default.target`
}

const quadletFiles = computed(() => {
  const files = { 'grimoire.container': buildQuadletGrimoire() }
  if (form.valkeyEnabled) files['grimoire-valkey.container'] = buildQuadletValkey()
  if (form.fileManager === 'calibre') files['grimoire-calibre.container'] = buildQuadletCalibre()
  if (form.fileManager === 'calibre-web') files['grimoire-calibre-web.container'] = buildQuadletCalibreWeb()
  return files
})

// Keep active tab valid when options change
watch(quadletFiles, (files) => {
  if (!(activeQuadletTab.value in files)) {
    activeQuadletTab.value = Object.keys(files)[0]
  }
})

// ── Output ───────────────────────────────────────────────────────────────────

const outputFilename = computed(() => {
  if (form.runtime === 'quadlet') return activeQuadletTab.value
  return form.runtime === 'docker' ? 'docker-compose.yml' : 'podman-compose.yml'
})

const currentOutput = computed(() => {
  if (form.runtime === 'quadlet') {
    return quadletFiles.value[activeQuadletTab.value] ?? ''
  }
  return buildCompose()
})

async function copyOutput() {
  await navigator.clipboard.writeText(currentOutput.value)
  copiedOutput.value = true
  setTimeout(() => { copiedOutput.value = false }, 2000)
}

function downloadOutput() {
  if (form.runtime === 'quadlet') {
    // Download all quadlet files as individual downloads
    for (const [name, content] of Object.entries(quadletFiles.value)) {
      triggerDownload(name, content)
    }
  } else {
    triggerDownload(outputFilename.value, currentOutput.value)
  }
}

function triggerDownload(filename, content) {
  const blob = new Blob([content], { type: 'text/plain' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = filename
  a.click()
  URL.revokeObjectURL(url)
}
</script>

<style scoped>
.compose-generator {
  max-width: 860px;
}

.cg-section {
  margin-bottom: 2rem;
  padding-bottom: 1.5rem;
  border-bottom: 1px solid var(--vp-c-divider);
}

.cg-section:last-child {
  border-bottom: none;
}

.cg-section h2 {
  font-size: 1.1rem;
  font-weight: 600;
  margin: 0 0 0.75rem;
  border: none;
  padding: 0;
}

.cg-desc {
  color: var(--vp-c-text-2);
  font-size: 0.9rem;
  margin: 0 0 0.75rem;
}

.cg-note {
  background: var(--vp-c-bg-soft);
  border-left: 3px solid var(--vp-c-brand);
  padding: 0.5rem 0.75rem;
  font-size: 0.88rem;
  margin-top: 0.75rem;
  border-radius: 0 4px 4px 0;
}

/* Radio group */
.cg-radio-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.cg-radio {
  display: flex;
  align-items: center;
  gap: 0.6rem;
  padding: 0.55rem 0.75rem;
  border: 1px solid var(--vp-c-divider);
  border-radius: 6px;
  cursor: pointer;
  transition: border-color 0.15s, background 0.15s;
}

.cg-radio:hover {
  border-color: var(--vp-c-brand);
}

.cg-radio.active {
  border-color: var(--vp-c-brand);
  background: var(--vp-c-brand-soft);
}

.cg-radio input {
  margin: 0;
  accent-color: var(--vp-c-brand);
}

.cg-radio-label {
  font-weight: 500;
  min-width: 160px;
}

.cg-radio-desc {
  color: var(--vp-c-text-2);
  font-size: 0.85rem;
}

/* Fields */
.cg-field {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
  flex: 1;
}

.cg-field label {
  font-size: 0.9rem;
  font-weight: 500;
  color: var(--vp-c-text-1);
}

.cg-field input,
.cg-field select {
  padding: 0.4rem 0.6rem;
  border: 1px solid var(--vp-c-divider);
  border-radius: 6px;
  background: var(--vp-c-bg);
  color: var(--vp-c-text-1);
  font-size: 0.9rem;
  font-family: inherit;
  transition: border-color 0.15s;
}

.cg-field input:focus,
.cg-field select:focus {
  outline: none;
  border-color: var(--vp-c-brand);
}

.cg-row {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
}

.cg-input-row {
  display: flex;
  gap: 0.5rem;
  align-items: center;
  flex-wrap: wrap;
}

.cg-inline-input {
  flex: 1;
  min-width: 100px;
  padding: 0.4rem 0.6rem;
  border: 1px solid var(--vp-c-divider);
  border-radius: 6px;
  background: var(--vp-c-bg);
  color: var(--vp-c-text-1);
  font-size: 0.9rem;
}

.cg-secret-input {
  flex: 1;
  min-width: 200px;
  padding: 0.4rem 0.6rem;
  border: 1px solid var(--vp-c-divider);
  border-radius: 6px;
  background: var(--vp-c-bg);
  color: var(--vp-c-text-1);
  font-size: 0.9rem;
  font-family: var(--vp-font-family-mono, monospace);
}

.cg-secret-input:focus {
  outline: none;
  border-color: var(--vp-c-brand);
}

.cg-copied {
  color: var(--vp-c-green-1);
  font-size: 0.85rem;
  margin: 0.25rem 0 0;
}

/* Toggles */
.cg-toggles {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  margin-top: 0.75rem;
}

.cg-toggle {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  cursor: pointer;
  font-size: 0.9rem;
}

.cg-toggle input {
  accent-color: var(--vp-c-brand);
  width: 1rem;
  height: 1rem;
  margin: 0;
}

/* Sub fields (Calibre TZ/PUID) */
.cg-sub-fields {
  margin-top: 0.75rem;
  padding: 0.75rem;
  background: var(--vp-c-bg-soft);
  border-radius: 6px;
}

/* Output */
.cg-output-section {
  position: sticky;
  bottom: 0;
  background: var(--vp-c-bg);
}

.cg-output-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-bottom: 0.5rem;
}

.cg-output-header h2 {
  margin: 0;
  font-family: var(--vp-font-family-mono, monospace);
  font-size: 0.95rem;
}

.cg-output-actions {
  display: flex;
  gap: 0.5rem;
}

.cg-quadlet-tabs {
  display: flex;
  flex-wrap: wrap;
  gap: 0.25rem;
  margin-bottom: 0.5rem;
}

.cg-tab {
  padding: 0.25rem 0.6rem;
  border: 1px solid var(--vp-c-divider);
  border-radius: 4px;
  background: var(--vp-c-bg-soft);
  font-size: 0.8rem;
  font-family: var(--vp-font-family-mono, monospace);
  cursor: pointer;
  color: var(--vp-c-text-2);
  transition: all 0.15s;
}

.cg-tab.active {
  border-color: var(--vp-c-brand);
  background: var(--vp-c-brand-soft);
  color: var(--vp-c-brand);
}

.cg-preview {
  background: var(--vp-c-bg-soft);
  border: 1px solid var(--vp-c-divider);
  border-radius: 8px;
  padding: 1rem;
  overflow-x: auto;
  font-size: 0.82rem;
  line-height: 1.6;
  max-height: 520px;
  overflow-y: auto;
}

.cg-preview code {
  background: none;
  font-family: var(--vp-font-family-mono, monospace);
  color: var(--vp-c-text-1);
}

/* Buttons */
.cg-btn {
  padding: 0.4rem 0.85rem;
  border-radius: 6px;
  font-size: 0.88rem;
  font-weight: 500;
  cursor: pointer;
  border: 1px solid transparent;
  transition: all 0.15s;
  white-space: nowrap;
}

.cg-btn-primary {
  background: var(--vp-c-brand);
  color: #fff;
  border-color: var(--vp-c-brand);
}

.cg-btn-primary:hover {
  background: var(--vp-c-brand-dark);
}

.cg-btn-secondary {
  background: var(--vp-c-bg-soft);
  color: var(--vp-c-text-1);
  border-color: var(--vp-c-divider);
}

.cg-btn-secondary:hover {
  border-color: var(--vp-c-brand);
  color: var(--vp-c-brand);
}

/* Hints */
.cg-hint {
  font-weight: 400;
  font-size: 0.82rem;
  color: var(--vp-c-text-2);
}

.cg-hint.optional {
  font-style: italic;
}
</style>
