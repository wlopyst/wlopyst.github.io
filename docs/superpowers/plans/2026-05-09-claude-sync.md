# Claude Sync Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a "Claude Sync" section to the portfolio site that manages Claude Code memory and skills via GitHub API, with upload, edit, delete, and zip download capabilities.

**Architecture:** Single `index.html` file extended with new CSS, HTML section, and JavaScript. A `ClaudeSync` class wraps GitHub REST API v3 calls. Data stored as individual files in `claude-sync/` directory in the repo. JSZip (CDN) handles zip generation for downloads.

**Tech Stack:** Vanilla HTML/CSS/JS, JSZip 3.x (CDN), GitHub REST API v3, localStorage for token.

---

### Task 1: Create Initial manifest.json

Set up the `claude-sync/` directory structure in the repo with an empty manifest.

**Files:**
- Create: `claude-sync/manifest.json`

- [ ] **Step 1: Create directory and manifest file**

```bash
mkdir -p claude-sync/memory claude-sync/skills
```

Create `claude-sync/manifest.json`:

```json
{
  "version": 1,
  "updatedAt": "2026-05-09T00:00:00Z",
  "memory": [],
  "skills": []
}
```

- [ ] **Step 2: Commit**

```bash
git add claude-sync/manifest.json
git commit -m "feat: add claude-sync directory with empty manifest"
```

---

### Task 2: Add CSS Styles for Claude Sync Section

Add styles for the Claude Sync section to the existing `<style>` block in `index.html`, following the existing design tokens (glass-morphism, gradients, Inter + JetBrains Mono fonts).

**Files:**
- Modify: `index.html` — append CSS before the closing `</style>` tag (before line 463)

- [ ] **Step 1: Add CSS styles**

Append the following CSS before the closing `</style>` tag in `index.html`:

```css
/* ── Claude Sync ── */
#sync { background: transparent; }
.sync-settings {
  background: var(--glass-bg);
  border: 1px solid var(--glass-border);
  border-radius: var(--card-radius);
  padding: 24px;
  margin-bottom: 32px;
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
}
.sync-settings-row {
  display: flex;
  gap: 12px;
  align-items: center;
  flex-wrap: wrap;
}
.sync-settings label {
  font-family: var(--font-mono);
  font-size: 13px;
  color: var(--text-secondary);
  min-width: 100px;
}
.sync-settings input[type="password"],
.sync-settings input[type="text"] {
  flex: 1;
  min-width: 200px;
  padding: 10px 14px;
  background: rgba(0, 0, 0, 0.3);
  border: 1px solid var(--glass-border);
  border-radius: 8px;
  color: var(--text-primary);
  font-family: var(--font-mono);
  font-size: 13px;
  outline: none;
  transition: border-color 0.2s;
}
.sync-settings input:focus {
  border-color: var(--gradient-start);
}
.sync-panels {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px;
  margin-bottom: 32px;
}
.sync-panel {
  background: var(--glass-bg);
  border: 1px solid var(--glass-border);
  border-radius: var(--card-radius);
  padding: 24px;
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
}
.sync-panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}
.sync-panel-title {
  font-family: var(--font-mono);
  font-size: 15px;
  font-weight: 600;
  color: var(--gradient-start);
}
.sync-panel-actions { display: flex; gap: 8px; }
.sync-file-list { display: flex; flex-direction: column; gap: 10px; }
.sync-file-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  background: rgba(0, 0, 0, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 10px;
  transition: border-color 0.2s;
}
.sync-file-item:hover {
  border-color: rgba(102, 126, 234, 0.2);
}
.sync-file-info { flex: 1; min-width: 0; }
.sync-file-name {
  font-family: var(--font-mono);
  font-size: 13px;
  font-weight: 500;
  color: var(--text-primary);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.sync-file-desc {
  font-size: 12px;
  color: var(--text-muted);
  margin-top: 2px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.sync-file-actions { display: flex; gap: 6px; flex-shrink: 0; }
.sync-empty {
  text-align: center;
  padding: 32px;
  color: var(--text-muted);
  font-size: 14px;
}
.sync-editor {
  background: var(--glass-bg);
  border: 1px solid var(--glass-border);
  border-radius: var(--card-radius);
  padding: 24px;
  margin-bottom: 32px;
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  display: none;
}
.sync-editor.active { display: block; }
.sync-editor-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}
.sync-editor-title {
  font-family: var(--font-mono);
  font-size: 14px;
  font-weight: 600;
  color: var(--text-primary);
}
.sync-editor textarea {
  width: 100%;
  min-height: 300px;
  padding: 16px;
  background: rgba(0, 0, 0, 0.4);
  border: 1px solid var(--glass-border);
  border-radius: 10px;
  color: var(--text-primary);
  font-family: var(--font-mono);
  font-size: 13px;
  line-height: 1.6;
  resize: vertical;
  outline: none;
  transition: border-color 0.2s;
}
.sync-editor textarea:focus {
  border-color: var(--gradient-start);
}
.sync-editor-footer {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-top: 16px;
}
.sync-download-bar {
  text-align: center;
  margin-bottom: 32px;
}
.sync-status {
  font-family: var(--font-mono);
  font-size: 12px;
  color: var(--text-muted);
  margin-top: 8px;
  min-height: 18px;
}
.sync-status.success { color: #4ade80; }
.sync-status.error { color: #f87171; }
.sync-hidden { display: none; }

@media (max-width: 768px) {
  .sync-panels { grid-template-columns: 1fr; }
  .sync-settings-row { flex-direction: column; align-items: stretch; }
  .sync-settings label { min-width: auto; }
}
```

- [ ] **Step 2: Verify styles parse correctly**

Open `index.html` in browser, scroll to bottom. Page should look unchanged (no visual regression). Styles are dormant until HTML is added.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add Claude Sync section CSS styles"
```

---

### Task 3: Add HTML Section Structure

Add the Claude Sync section HTML after the Experience section, and add "Sync" nav link.

**Files:**
- Modify: `index.html` — add nav link in `.nav-links` (after line 477)
- Modify: `index.html` — add section HTML after `</section>` closing tag of Experience (after line 660)

- [ ] **Step 1: Add navigation link**

In the `.nav-links` div, add a new link after the "Experience" link:

```html
<a href="#sync">Sync</a>
```

The nav-links div should now contain:
```html
<a href="#skills">Skills</a>
<a href="#projects">Projects</a>
<a href="#experience">Experience</a>
<a href="#sync">Sync</a>
```

- [ ] **Step 2: Add Claude Sync section HTML**

Insert the following HTML after the closing `</section>` tag of the Experience section (after line 660):

```html
<!-- Claude Sync -->
<section id="sync">
  <div class="container">
    <p class="section-label reveal">Claude Code</p>
    <h2 class="section-title reveal">Claude Sync</h2>
    <p class="section-desc reveal">Manage your Claude Code memory and skills. Upload, edit, and download for quick setup on new devices.</p>

    <!-- Settings -->
    <div class="sync-settings reveal">
      <div class="sync-settings-row">
        <label for="sync-token">GitHub Token</label>
        <input type="password" id="sync-token" placeholder="ghp_xxxxxxxxxxxx" autocomplete="off">
        <button class="btn-glass" id="sync-token-save">Save</button>
        <button class="btn-glass" id="sync-token-clear">Clear</button>
      </div>
      <div class="sync-settings-row" style="margin-top: 12px;">
        <label>Repository</label>
        <input type="text" id="sync-repo" placeholder="owner/repo" value="wlopyst/wlopyst.github.io">
      </div>
      <p class="sync-status" id="sync-settings-status"></p>
    </div>

    <!-- Panels -->
    <div class="sync-panels">
      <!-- Memory Panel -->
      <div class="sync-panel reveal">
        <div class="sync-panel-header">
          <span class="sync-panel-title">Memory</span>
          <div class="sync-panel-actions">
            <button class="btn-glass" id="sync-memory-upload">+ Upload</button>
            <input type="file" id="sync-memory-input" class="sync-hidden" multiple accept=".md">
          </div>
        </div>
        <div class="sync-file-list" id="sync-memory-list">
          <div class="sync-empty">No memory files yet. Click "+ Upload" to add.</div>
        </div>
      </div>

      <!-- Skills Panel -->
      <div class="sync-panel reveal">
        <div class="sync-panel-header">
          <span class="sync-panel-title">Skills</span>
          <div class="sync-panel-actions">
            <button class="btn-glass" id="sync-skills-upload">+ Upload</button>
            <input type="file" id="sync-skills-input" class="sync-hidden" multiple accept=".md" webkitdirectory>
          </div>
        </div>
        <div class="sync-file-list" id="sync-skills-list">
          <div class="sync-empty">No skills yet. Click "+ Upload" to add.</div>
        </div>
      </div>
    </div>

    <!-- Download Bar -->
    <div class="sync-download-bar reveal">
      <button class="btn-glass btn-primary" id="sync-download-all">Download All as ZIP</button>
      <p class="sync-status" id="sync-download-status"></p>
    </div>

    <!-- Editor -->
    <div class="sync-editor" id="sync-editor">
      <div class="sync-editor-header">
        <span class="sync-editor-title" id="sync-editor-filename">editing file...</span>
        <button class="btn-glass" id="sync-editor-close">&times;</button>
      </div>
      <textarea id="sync-editor-content" spellcheck="false"></textarea>
      <div class="sync-editor-footer">
        <button class="btn-glass" id="sync-editor-cancel">Cancel</button>
        <button class="btn-glass btn-primary" id="sync-editor-save">Save &amp; Push</button>
      </div>
      <p class="sync-status" id="sync-editor-status"></p>
    </div>
  </div>
</section>
```

- [ ] **Step 3: Verify HTML renders correctly**

Open `index.html` in browser. Scroll down — the Claude Sync section should appear after Experience with the correct glass-morphism styling. Nav should show "Sync" link.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add Claude Sync section HTML and nav link"
```

---

### Task 4: Add JSZip CDN and Implement ClaudeSync API Class

Add JSZip script tag and implement the core GitHub API interaction class.

**Files:**
- Modify: `index.html` — add JSZip CDN `<script>` before the closing `</body>` tag
- Modify: `index.html` — add ClaudeSync class in the existing `<script>` block

- [ ] **Step 1: Add JSZip CDN**

Add before the closing `</body>` tag, right before the existing `<script>` block:

```html
<script src="https://cdn.jsdelivr.net/npm/jszip@3/dist/jszip.min.js"></script>
```

- [ ] **Step 2: Implement ClaudeSync class**

Add the following class at the top of the existing `<script>` block (after line 669):

```javascript
// ── Claude Sync ──
class ClaudeSync {
  constructor() {
    this.token = localStorage.getItem('claude-sync-token') || '';
    this.repo = localStorage.getItem('claude-sync-repo') || 'wlopyst/wlopyst.github.io';
    this.baseUrl = 'https://api.github.com';
    this.manifestPath = 'claude-sync/manifest.json';
    this.manifest = null;
    this.currentSha = null; // SHA of manifest for updates
  }

  get headers() {
    const h = { 'Accept': 'application/vnd.github.v3+json' };
    if (this.token) h['Authorization'] = `token ${this.token}`;
    return h;
  }

  saveToken(token) {
    this.token = token;
    localStorage.setItem('claude-sync-token', token);
  }

  saveRepo(repo) {
    this.repo = repo;
    localStorage.setItem('claude-sync-repo', repo);
  }

  clearToken() {
    this.token = '';
    localStorage.removeItem('claude-sync-token');
  }

  async loadManifest() {
    const url = `${this.baseUrl}/repos/${this.repo}/contents/${this.manifestPath}`;
    const resp = await fetch(url, { headers: this.headers });
    if (!resp.ok) {
      if (resp.status === 404) {
        this.manifest = { version: 1, updatedAt: new Date().toISOString(), memory: [], skills: [] };
        this.currentSha = null;
        return this.manifest;
      }
      throw new Error(`Failed to load manifest: ${resp.status}`);
    }
    const data = await resp.json();
    this.currentSha = data.sha;
    this.manifest = JSON.parse(atob(data.content));
    return this.manifest;
  }

  async saveManifest(message = 'Update manifest') {
    this.manifest.updatedAt = new Date().toISOString();
    const content = btoa(unescape(encodeURIComponent(JSON.stringify(this.manifest, null, 2))));
    const url = `${this.baseUrl}/repos/${this.repo}/contents/${this.manifestPath}`;
    const body = { message, content, sha: this.currentSha };
    const resp = await fetch(url, {
      method: 'PUT',
      headers: { ...this.headers, 'Content-Type': 'application/json' },
      body: JSON.stringify(body)
    });
    if (!resp.ok) throw new Error(`Failed to save manifest: ${resp.status}`);
    const data = await resp.json();
    this.currentSha = data.content.sha;
    return data;
  }

  async getFile(path) {
    const url = `${this.baseUrl}/repos/${this.repo}/contents/${path}`;
    const resp = await fetch(url, { headers: this.headers });
    if (!resp.ok) throw new Error(`Failed to get file ${path}: ${resp.status}`);
    const data = await resp.json();
    return { sha: data.sha, content: decodeURIComponent(escape(atob(data.content))) };
  }

  async uploadFile(path, content, message) {
    // Check if file exists to get SHA for update
    let sha = null;
    try {
      const existing = await this.getFile(path);
      sha = existing.sha;
    } catch (e) { /* file doesn't exist, create new */ }

    const encodedContent = btoa(unescape(encodeURIComponent(content)));
    const url = `${this.baseUrl}/repos/${this.repo}/contents/${path}`;
    const body = { message, content: encodedContent };
    if (sha) body.sha = sha;
    const resp = await fetch(url, {
      method: 'PUT',
      headers: { ...this.headers, 'Content-Type': 'application/json' },
      body: JSON.stringify(body)
    });
    if (!resp.ok) throw new Error(`Failed to upload ${path}: ${resp.status}`);
    return await resp.json();
  }

  async deleteFile(path, message) {
    const { sha } = await this.getFile(path);
    const url = `${this.baseUrl}/repos/${this.repo}/contents/${path}`;
    const resp = await fetch(url, {
      method: 'DELETE',
      headers: { ...this.headers, 'Content-Type': 'application/json' },
      body: JSON.stringify({ message, sha })
    });
    if (!resp.ok) throw new Error(`Failed to delete ${path}: ${resp.status}`);
    return await resp.json();
  }

  parseFrontmatter(content) {
    const match = content.match(/^---\n([\s\S]*?)\n---/);
    if (!match) return {};
    const meta = {};
    match[1].split('\n').forEach(line => {
      const idx = line.indexOf(':');
      if (idx > 0) {
        const key = line.slice(0, idx).trim();
        const val = line.slice(idx + 1).trim();
        meta[key] = val;
      }
    });
    return meta;
  }
}
```

- [ ] **Step 3: Initialize global instance**

At the end of the script block, before the closing `</script>` tag, add:

```javascript
const sync = new ClaudeSync();
```

- [ ] **Step 4: Verify no JS errors**

Open `index.html` in browser. Open DevTools console — no errors should appear. The `sync` object should be available in console (`sync.token` should return `""`).

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add JSZip CDN and ClaudeSync API class"
```

---

### Task 5: Implement Settings UI Logic

Wire up the token save/clear buttons and repo input.

**Files:**
- Modify: `index.html` — add settings logic in the `<script>` block

- [ ] **Step 1: Add settings initialization and event handlers**

Add the following after the `const sync = new ClaudeSync();` line:

```javascript
// ── Settings UI ──
const tokenInput = document.getElementById('sync-token');
const repoInput = document.getElementById('sync-repo');
const settingsStatus = document.getElementById('sync-settings-status');

// Load saved values
tokenInput.value = sync.token ? '••••••••••••' : '';
repoInput.value = sync.repo;

document.getElementById('sync-token-save').addEventListener('click', () => {
  const val = tokenInput.value.trim();
  if (!val || val.startsWith('••••')) {
    settingsStatus.textContent = 'Enter a valid token';
    settingsStatus.className = 'sync-status error';
    return;
  }
  sync.saveToken(val);
  tokenInput.value = '••••••••••••';
  sync.saveRepo(repoInput.value.trim() || 'wlopyst/wlopyst.github.io');
  settingsStatus.textContent = 'Token saved';
  settingsStatus.className = 'sync-status success';
  setTimeout(() => { settingsStatus.textContent = ''; }, 3000);
  loadSyncData();
});

document.getElementById('sync-token-clear').addEventListener('click', () => {
  sync.clearToken();
  tokenInput.value = '';
  settingsStatus.textContent = 'Token cleared';
  settingsStatus.className = 'sync-status success';
  setTimeout(() => { settingsStatus.textContent = ''; }, 3000);
});
```

- [ ] **Step 2: Verify settings work**

Open in browser. Enter a dummy token, click Save — should show "Token saved". Click Clear — should clear. Check localStorage in DevTools — `claude-sync-token` should be set/cleared.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: implement Claude Sync settings UI logic"
```

---

### Task 6: Implement File List Rendering

Display memory and skills files from the manifest, with Edit and Delete buttons.

**Files:**
- Modify: `index.html` — add list rendering functions in the `<script>` block

- [ ] **Step 1: Add list rendering functions**

Add the following after the settings UI code:

```javascript
// ── File List Rendering ──
const memoryListEl = document.getElementById('sync-memory-list');
const skillsListEl = document.getElementById('sync-skills-list');

function renderFileList(container, items, type) {
  if (!items || items.length === 0) {
    container.innerHTML = `<div class="sync-empty">No ${type} files yet. Click "+ Upload" to add.</div>`;
    return;
  }
  container.innerHTML = items.map(item => `
    <div class="sync-file-item" data-path="${item.path}" data-type="${type}">
      <div class="sync-file-info">
        <div class="sync-file-name">${item.name || item.id}</div>
        <div class="sync-file-desc">${item.description || ''}</div>
      </div>
      <div class="sync-file-actions">
        <button class="btn-glass sync-edit-btn" data-path="${item.path}">Edit</button>
        <button class="btn-glass sync-delete-btn" data-path="${item.path}" data-id="${item.id}">Delete</button>
      </div>
    </div>
  `).join('');

  // Bind edit buttons
  container.querySelectorAll('.sync-edit-btn').forEach(btn => {
    btn.addEventListener('click', () => openEditor(btn.dataset.path));
  });

  // Bind delete buttons
  container.querySelectorAll('.sync-delete-btn').forEach(btn => {
    btn.addEventListener('click', () => deleteFile(btn.dataset.path, btn.dataset.id, type));
  });
}

async function loadSyncData() {
  if (!sync.token) return;
  try {
    const manifest = await sync.loadManifest();
    renderFileList(memoryListEl, manifest.memory, 'memory');
    renderFileList(skillsListEl, manifest.skills, 'skills');
  } catch (e) {
    settingsStatus.textContent = `Error: ${e.message}`;
    settingsStatus.className = 'sync-status error';
  }
}

// Auto-load if token exists
if (sync.token) loadSyncData();
```

- [ ] **Step 2: Verify rendering**

With a valid token and the `claude-sync/manifest.json` in the repo, open in browser. Files should appear in both panels. With no files, "No files yet" message should show.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: implement Claude Sync file list rendering"
```

---

### Task 7: Implement File Upload

Handle file selection, frontmatter parsing, and GitHub API upload with manifest update.

**Files:**
- Modify: `index.html` — add upload logic in the `<script>` block

- [ ] **Step 1: Add upload event handlers and logic**

Add the following after the file list rendering code:

```javascript
// ── File Upload ──
const memoryInput = document.getElementById('sync-memory-input');
const skillsInput = document.getElementById('sync-skills-input');

document.getElementById('sync-memory-upload').addEventListener('click', () => memoryInput.click());
document.getElementById('sync-skills-upload').addEventListener('click', () => skillsInput.click());

async function handleUpload(files, type) {
  if (!sync.token) {
    alert('Please set your GitHub token first.');
    return;
  }

  for (const file of files) {
    const content = await file.text();
    const meta = sync.parseFrontmatter(content);
    const fileName = file.name;

    let repoPath;
    if (type === 'memory') {
      repoPath = `claude-sync/memory/${fileName}`;
    } else {
      // For skills, preserve directory structure from webkitRelativePath
      const relPath = file.webkitRelativePath || fileName;
      repoPath = `claude-sync/skills/${relPath}`;
    }

    try {
      await sync.uploadFile(repoPath, content, `Upload ${type}: ${fileName}`);

      // Update manifest
      const entry = {
        id: fileName.replace('.md', ''),
        name: meta.name || fileName.replace('.md', ''),
        description: meta.description || '',
        path: repoPath
      };
      if (type === 'memory') entry.type = meta.type || 'custom';

      const existingIdx = sync.manifest[type].findIndex(e => e.path === repoPath);
      if (existingIdx >= 0) {
        sync.manifest[type][existingIdx] = entry;
      } else {
        sync.manifest[type].push(entry);
      }
    } catch (e) {
      alert(`Failed to upload ${fileName}: ${e.message}`);
    }
  }

  await sync.saveManifest(`Upload ${type} files`);
  await loadSyncData();
}

memoryInput.addEventListener('change', (e) => {
  handleUpload(Array.from(e.target.files), 'memory');
  e.target.value = '';
});

skillsInput.addEventListener('change', (e) => {
  handleUpload(Array.from(e.target.files), 'skills');
  e.target.value = '';
});
```

- [ ] **Step 2: Verify upload**

With a valid token, click "+ Upload" on Memory panel, select a `.md` file with frontmatter. It should appear in the list and be committed to the repo.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: implement Claude Sync file upload with frontmatter parsing"
```

---

### Task 8: Implement Editor and Delete

Open files in the inline editor, save changes back to GitHub, and delete files with manifest update.

**Files:**
- Modify: `index.html` — add editor and delete logic in the `<script>` block

- [ ] **Step 1: Add editor logic**

Add the following after the upload code:

```javascript
// ── Editor ──
const editorEl = document.getElementById('sync-editor');
const editorFilename = document.getElementById('sync-editor-filename');
const editorContent = document.getElementById('sync-editor-content');
const editorStatus = document.getElementById('sync-editor-status');
let editingPath = null;

async function openEditor(path) {
  if (!sync.token) {
    alert('Please set your GitHub token first.');
    return;
  }
  try {
    const { content } = await sync.getFile(path);
    editingPath = path;
    editorFilename.textContent = path.replace('claude-sync/', '');
    editorContent.value = content;
    editorEl.classList.add('active');
    editorEl.scrollIntoView({ behavior: 'smooth', block: 'start' });
    editorContent.focus();
  } catch (e) {
    alert(`Failed to load file: ${e.message}`);
  }
}

function closeEditor() {
  editorEl.classList.remove('active');
  editingPath = null;
  editorContent.value = '';
  editorStatus.textContent = '';
}

document.getElementById('sync-editor-close').addEventListener('click', closeEditor);
document.getElementById('sync-editor-cancel').addEventListener('click', closeEditor);

document.getElementById('sync-editor-save').addEventListener('click', async () => {
  if (!editingPath) return;
  const content = editorContent.value;
  editorStatus.textContent = 'Saving...';
  editorStatus.className = 'sync-status';

  try {
    await sync.uploadFile(editingPath, content, `Update ${editingPath.replace('claude-sync/', '')}`);

    // Update manifest metadata if frontmatter changed
    const meta = sync.parseFrontmatter(content);
    const type = editingPath.startsWith('claude-sync/memory/') ? 'memory' : 'skills';
    const entry = sync.manifest[type].find(e => e.path === editingPath);
    if (entry && meta.name) {
      entry.name = meta.name;
      entry.description = meta.description || '';
      if (type === 'memory' && meta.type) entry.type = meta.type;
      await sync.saveManifest(`Update metadata for ${editingPath}`);
    }

    editorStatus.textContent = 'Saved!';
    editorStatus.className = 'sync-status success';
    setTimeout(() => { editorStatus.textContent = ''; }, 3000);
    await loadSyncData();
  } catch (e) {
    editorStatus.textContent = `Error: ${e.message}`;
    editorStatus.className = 'sync-status error';
  }
});
```

- [ ] **Step 2: Add delete logic**

Add the following after the editor code:

```javascript
// ── Delete ──
async function deleteFile(path, id, type) {
  if (!sync.token) {
    alert('Please set your GitHub token first.');
    return;
  }
  if (!confirm(`Delete ${path.replace('claude-sync/', '')}?`)) return;

  try {
    await sync.deleteFile(path, `Delete ${path.replace('claude-sync/', '')}`);
    sync.manifest[type] = sync.manifest[type].filter(e => e.path !== path);
    await sync.saveManifest(`Remove ${id} from ${type}`);
    await loadSyncData();
  } catch (e) {
    alert(`Failed to delete: ${e.message}`);
  }
}
```

- [ ] **Step 3: Verify editor and delete**

Click "Edit" on a file — editor should open with content. Edit and save — changes should appear in repo. Click "Delete" — confirm dialog, file removed from list and repo.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: implement Claude Sync editor and delete functionality"
```

---

### Task 9: Implement ZIP Download

Generate a zip file with all synced data in Claude Code directory structure, including a SETUP.md guide.

**Files:**
- Modify: `index.html` — add download logic in the `<script>` block

- [ ] **Step 1: Add download logic**

Add the following after the delete code:

```javascript
// ── ZIP Download ──
const downloadStatus = document.getElementById('sync-download-status');

document.getElementById('sync-download-all').addEventListener('click', async () => {
  if (!sync.token) {
    alert('Please set your GitHub token first.');
    return;
  }

  downloadStatus.textContent = 'Preparing download...';
  downloadStatus.className = 'sync-status';

  try {
    const manifest = await sync.loadManifest();
    const zip = new JSZip();
    const date = new Date().toISOString().slice(0, 10).replace(/-/g, '');
    const root = zip.folder(`claude-sync-export-${date}`);

    // Add memory files
    for (const item of manifest.memory) {
      try {
        const { content } = await sync.getFile(item.path);
        root.file(item.path.replace('claude-sync/', ''), content);
      } catch (e) {
        console.warn(`Skipping ${item.path}: ${e.message}`);
      }
    }

    // Add skills files
    for (const item of manifest.skills) {
      try {
        if (item.path.endsWith('/')) {
          // It's a directory — list files via GitHub tree API
          const treeUrl = `https://api.github.com/repos/${sync.repo}/git/trees/HEAD?recursive=1`;
          const resp = await fetch(treeUrl, { headers: sync.headers });
          if (resp.ok) {
            const tree = await resp.json();
            const prefix = item.path;
            for (const node of tree.tree) {
              if (node.path.startsWith(prefix) && node.type === 'blob') {
                const fileResp = await fetch(`https://api.github.com/repos/${sync.repo}/contents/${node.path}`, { headers: sync.headers });
                if (fileResp.ok) {
                  const fileData = await fileResp.json();
                  const content = decodeURIComponent(escape(atob(fileData.content)));
                  root.file(node.path.replace('claude-sync/', ''), content);
                }
              }
            }
          }
        } else {
          const { content } = await sync.getFile(item.path);
          root.file(item.path.replace('claude-sync/', ''), content);
        }
      } catch (e) {
        console.warn(`Skipping ${item.path}: ${e.message}`);
      }
    }

    // Add SETUP.md guide
    root.file('SETUP.md', `# Claude Code Quick Setup Guide

## 1. Copy Memory Files
Copy all files from \`memory/\` to:
\`\`\`
~/.claude/projects/<your-project-path>/memory/
\`\`\`

Example:
\`\`\`bash
cp -r memory/* ~/.claude/projects/my-project/memory/
\`\`\`

## 2. Install Skills
Copy the \`skills/\` directory contents to:
\`\`\`
~/.claude/plugins/
\`\`\`

Or install via Claude Code plugin manager if available.

## 3. Verify
Launch Claude Code and check if memory loaded:
- Your MEMORY.md index should appear in context
- Skills should be available via \`/skill-name\` commands

## 4. Customize
Edit the memory files to match your new project context:
- Update \`user_*.md\` with your current role/info
- Update \`project_*.md\` with new project details
- Keep \`feedback_*.md\` as-is (these are workflow preferences)
`);

    // Generate and download
    const blob = await zip.generateAsync({ type: 'blob' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `claude-sync-export-${date}.zip`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);

    downloadStatus.textContent = 'Download started!';
    downloadStatus.className = 'sync-status success';
    setTimeout(() => { downloadStatus.textContent = ''; }, 5000);
  } catch (e) {
    downloadStatus.textContent = `Error: ${e.message}`;
    downloadStatus.className = 'sync-status error';
  }
});
```

- [ ] **Step 2: Verify download**

With a valid token and data in the repo, click "Download All as ZIP". A zip file should download containing memory/ and skills/ directories plus SETUP.md. Extract and verify directory structure matches Claude Code expectations.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: implement Claude Sync ZIP download with SETUP guide"
```

---

### Task 10: Final Polish — Animations, Responsive, Integration

Ensure scroll-reveal animations work on the new section, verify mobile responsive layout, and do a final visual review.

**Files:**
- Modify: `index.html` — add reveal observers for new elements

- [ ] **Step 1: Add reveal observers for sync section elements**

In the existing IntersectionObserver code (around line 686), the observer already targets all `.reveal` elements. Since we used the `reveal` class on the new HTML elements, they should automatically animate. Verify by scrolling to the section.

If elements are not animating, ensure the observer is set up after the DOM is ready. The existing code at line 686-693 should work since it runs at the end of `<body>`.

- [ ] **Step 2: Test mobile responsive**

Open in browser, use DevTools responsive mode (375px width). Verify:
- Settings input fields stack vertically
- Memory/Skills panels stack vertically
- All buttons are tap-friendly (min 44px touch target)
- Editor textarea is usable

- [ ] **Step 3: Full integration test**

End-to-end test with a real GitHub token:
1. Set token → "Token saved"
2. Upload a memory `.md` file → appears in list
3. Click Edit → editor opens with content → save → changes in repo
4. Upload a skill folder → appears in list
5. Download ZIP → verify contents match repo
6. Delete a file → removed from list and repo

- [ ] **Step 4: Final commit**

```bash
git add index.html
git commit -m "feat: Claude Sync complete — upload, edit, delete, download"
```
