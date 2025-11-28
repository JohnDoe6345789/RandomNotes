
3D Printer abstraction daemon and android app

### 1. What this daemon actually is

Think of it as:

> **“CUPS for 3D printers”**  
> Runs on a Pi / NAS / home server, speaks _many_ printer dialects, exposes **one** API.

- Headless **backend only** (no heavy UI baked in)
    
- Simple **REST + WebSocket (or MQTT)** interface
    
- **Plugins** handle each printer type / ecosystem
    

So any app (web UI, mobile app, CLI, Home Assistant, VS Code extension) just talks to _your_ daemon.

---

### 2. Core concepts / API

Define a **generic model** of a printer and jobs, and force plugins to conform to it.

**Printer:**

- `id`
    
- `name`
    
- `type` (fdm, resin, etc. later)
    
- `state` (idle, printing, paused, error)
    
- `capabilities` (supports_pause, supports_temp_control, webcam, etc.)
    
- `telemetry` (temps, progress, speeds, fan, etc.)
    

**Job:**

- `job_id`
    
- `printer_id`
    
- `status` (queued, running, done, failed)
    
- `source` (uploaded_gcode, url, remote_cloud_id)
    
- `estimated_time`, `elapsed_time`, `progress`
    

**API surface (rough idea):**

- `GET /printers`
    
- `GET /printers/{id}`
    
- `POST /printers/{id}/jobs` (upload / start)
    
- `POST /jobs/{id}/pause`
    
- `POST /jobs/{id}/resume`
    
- `POST /jobs/{id}/cancel`
    
- `GET /jobs/{id}`
    
- `WS /events` (state/telemetry stream)
    

---

### 3. Plugin architecture

Each plugin is a Python package implementing a common interface, e.g.:

`class PrinterBackend(Protocol):     

def discover(self) -> list[DiscoveredPrinter]: ...     

def connect(self, config: PrinterConfig) -> None: ...     

def get_state(self) -> PrinterState: ...     

def submit_job(self, gcode_path: Path) -> JobId: ...     

def pause_job(self, job_id: str) -> None: ...     

def resume_job(self, job_id: str) -> None: ...     

def cancel_job(self, job_id: str) -> None: ...     

def stream_events(self) -> Iterable[PrinterEvent]: ...`

Then:

- **MarlinUSBPlugin** – talks raw serial (MK3, Ender, etc.)
    
- **OctoPrintPlugin** – forwards to an existing OctoPrint instance via its API
    
- **PrusaLinkPlugin** – MK4/XL/Core One via PrusaLink HTTP API
    
- **BambuLANPlugin** – Bambu over LAN API (where possible)
    
- **KlipperPlugin** – talks to Moonraker API
    

Plugins can be discovered via `entry_points` (`setup.cfg/pyproject.toml`), so users can `pip install printerd-prusa` to add support.

---

### 4. Daemon internals (minimal but solid)

I’d do:

- **Python + asyncio**
    
- **uvicorn + FastAPI** (or bare Starlette) for REST/WS
    
- A small internal **job queue** so you can:
    
    - queue multiple prints per printer
        
    - serialize access
        
- A **state manager** that:
    
    - polls each backend or listens to its events
        
    - normalises into your `PrinterState`
        
    - pushes updates via WebSocket/MQTT
        

---

### 5. Why this is actually useful (gap confirmation)

This sits in a nice spot:

- **Above**: vendor-specific stacks (PrusaLink, Bambu Studio, OctoPrint, Klipper, etc.)
    
- **Below**: UIs, automations, and “nice stuff” you or others want to build.
    

So you can:

- Have _one_ app controlling **Prusa Core One + Bambu + random Ender**.
    
- Swap printers without rewriting all your scripts / HA automations.
    
- Build niche frontends (phone app, TUI, “print from file server” tool) without caring about vendor APIs.
    

---

If you like, next step I can:

- Sketch a **minimal repo layout** and plugin interface, or
    
- Write a **tiny working prototype**: one core + a fake “MockPrinter” plugin so you can start hacking.

## Get something published to FDroid
as per title

### **SSH Commands Android App**

- A mobile app with **predefined SSH “recipes”**:
    
    - Docker restart commands
        
    - Systemd shortcuts
        
    - Real-time logs buttons
        
    - Raspberry Pi health checks
        
- Possibly with exportable YAML command packs.

### **5. AI Chocolatey package generator** - integrate with github

- A Python script that outputs a full multi-file Chocolatey package folder tree:
    
    - `.nuspec`
        
    - `chocolateyinstall.ps1`
        
    - checksums auto-downloaded

### **Code-gen tool that creates entire project skeletons**

- Based on your usual preference:
    
    - PEP8
        
    - mypy strict
        
    - Functions ≤10 lines
        
    - Tests in `tests/`
        
    - README + scripts
        
    - Bash + Batch for execution
    - Just use a GUI to install the thing
        
- Could generate:
    
    - Python
        
    - Node
        
    - Rust
        
    - Local/LAN microservices
### **. Local LLM “Codex clone” backend**

- A small server that wraps llama.cpp or Ollama and exposes a stable API for IDE extensions.
    
- Essentially your own “local GitHub Copilot server”.


## Docker backup and restore script

As per title


### **9. Pi OS migration assistant**

- Automate switching Raspberry Pi from Raspbian → Ubuntu Server:
    
    - Preserve Docker
        
    - Preserve configs
        
    - Auto reconfigure network
        
    - Reinstall packages
        
- Think “Mac Migration Assistant but for Raspberry Pi”.


## WSL2 GUI bootstrap script

as per title

## QT6 does Android + iOS + Mac Win Lin Desktop!

Just wow :)


## Wonder if I can write a Steam GUI in Qt6?

i found this [Felgo: Native Cross-Platform App Development and Professional Services](https://felgo.com/)

## Submit app

After much thought a Fiverr user could publish any app (assuming they have good connections)

## Local LAN "cloud" GUI portal

Little GUI app that can be a portal to local cloud

Stores usernames, passwords, urls, opens the browser for you

Nmap + fuzzy logic could generate some of this

## Windows 11 ISO - custom build

Python GUI that does a custom build of Windows 11 - it could auto download the windows 11 iso (if possible)

[JohnDoe6345789/UnattendedWinstall: Personalized Unattended Answer Files that helps automatically debloat and customize Windows 10 & 11 during the installation process.](https://github.com/JohnDoe6345789/UnattendedWinstall)

Just copy to usb before boot

## Make a site that had a variety of jobs in life and ballpark costs

as per title

```html


<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Life Jobs & Ballpark Costs</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root {
      --bg: #0f172a;
      --bg-alt: #020617;
      --card: #020617;
      --accent: #38bdf8;
      --accent-soft: rgba(56, 189, 248, 0.1);
      --text: #e5e7eb;
      --muted: #9ca3af;
      --danger: #fb7185;
      --border: #1e293b;
      --radius-lg: 18px;
      --radius-xl: 24px;
      --shadow-soft: 0 18px 40px rgba(0, 0, 0, 0.45);
    }

    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
        sans-serif;
      background: radial-gradient(circle at top, #0b1120 0, #020617 60%);
      color: var(--text);
      min-height: 100vh;
    }

    .app-shell {
      max-width: 1100px;
      margin: 0 auto;
      padding: 24px 16px 40px;
    }

    header {
      display: flex;
      flex-direction: column;
      gap: 12px;
      margin-bottom: 24px;
    }

    @media (min-width: 768px) {
      header {
        flex-direction: row;
        align-items: center;
        justify-content: space-between;
      }
    }

    .title-block h1 {
      font-size: 1.8rem;
      margin: 0 0 6px;
      letter-spacing: 0.03em;
    }

    .title-pill {
      display: inline-flex;
      align-items: center;
      gap: 8px;
      font-size: 0.8rem;
      color: var(--muted);
      background: rgba(15, 23, 42, 0.9);
      border-radius: 999px;
      padding: 4px 10px;
      border: 1px solid rgba(148, 163, 184, 0.3);
    }

    .title-pill-dot {
      width: 7px;
      height: 7px;
      border-radius: 999px;
      background: radial-gradient(circle, #22c55e 0, #16a34a 40%, #166534 100%);
      box-shadow: 0 0 8px rgba(34, 197, 94, 0.8);
    }

    .subtitle {
      font-size: 0.92rem;
      color: var(--muted);
      max-width: 520px;
    }

    .controls {
      display: flex;
      flex-direction: column;
      gap: 8px;
      margin-top: 14px;
    }

    @media (min-width: 768px) {
      .controls {
        align-items: flex-end;
        justify-content: flex-end;
      }
    }

    .controls-row {
      display: flex;
      flex-wrap: wrap;
      gap: 8px;
    }

    .pill-select,
    .search-input {
      background: rgba(15, 23, 42, 0.9);
      border-radius: 999px;
      border: 1px solid rgba(148, 163, 184, 0.4);
      font-size: 0.85rem;
      color: var(--text);
      padding: 7px 12px;
      outline: none;
    }

    .pill-select:focus,
    .search-input:focus {
      border-color: var(--accent);
      box-shadow: 0 0 0 1px rgba(56, 189, 248, 0.6);
    }

    .search-input {
      min-width: 180px;
    }

    .tag-chip {
      font-size: 0.78rem;
      padding: 5px 10px;
      border-radius: 999px;
      border: 1px solid rgba(148, 163, 184, 0.4);
      color: var(--muted);
      background: rgba(15, 23, 42, 0.9);
    }

    .layout {
      display: grid;
      grid-template-columns: minmax(0, 1.9fr) minmax(0, 1.1fr);
      gap: 16px;
      align-items: flex-start;
    }

    @media (max-width: 900px) {
      .layout {
        grid-template-columns: minmax(0, 1fr);
      }
    }

    .cards {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
      gap: 14px;
    }

    .card {
      background: radial-gradient(circle at top left, #1f2937 0, #020617 55%);
      border-radius: var(--radius-lg);
      border: 1px solid rgba(15, 23, 42, 1);
      box-shadow: var(--shadow-soft);
      padding: 14px 14px 12px;
      position: relative;
      overflow: hidden;
      transform: translateY(0);
      transition:
        transform 0.18s ease-out,
        box-shadow 0.18s ease-out,
        border-color 0.18s ease-out,
        background 0.18s ease-out;
    }

    .card::before {
      content: "";
      position: absolute;
      inset: 0;
      background: radial-gradient(
        circle at top right,
        rgba(56, 189, 248, 0.18),
        transparent 55%
      );
      opacity: 0;
      pointer-events: none;
      transition: opacity 0.2s ease-out;
    }

    .card:hover {
      transform: translateY(-3px);
      box-shadow: 0 20px 45px rgba(0, 0, 0, 0.7);
      border-color: rgba(148, 163, 184, 0.7);
      background: radial-gradient(circle at top, #111827 0, #020617 60%);
    }

    .card:hover::before {
      opacity: 1;
    }

    .card-header {
      display: flex;
      justify-content: space-between;
      gap: 8px;
      margin-bottom: 6px;
    }

    .card-title {
      font-size: 0.98rem;
      font-weight: 600;
    }

    .card-category {
      font-size: 0.75rem;
      padding: 3px 8px;
      border-radius: 999px;
      background: rgba(15, 23, 42, 0.8);
      border: 1px solid rgba(148, 163, 184, 0.45);
      color: var(--muted);
      white-space: nowrap;
    }

    .card-row {
      display: flex;
      justify-content: space-between;
      align-items: baseline;
      gap: 6px;
      margin-bottom: 3px;
    }

    .card-label {
      font-size: 0.78rem;
      color: var(--muted);
      text-transform: uppercase;
      letter-spacing: 0.04em;
    }

    .card-value {
      font-size: 0.96rem;
      font-variant-numeric: tabular-nums;
    }

    .card-value-typical {
      color: var(--accent);
      font-weight: 600;
    }

    .card-value-high {
      color: #fbbf24;
    }

    .card-meta {
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 8px;
      margin-top: 4px;
    }

    .card-note {
      font-size: 0.78rem;
      color: var(--muted);
    }

    .card-tag {
      font-size: 0.72rem;
      padding: 2px 7px;
      border-radius: 999px;
      background: rgba(15, 23, 42, 0.8);
      border: 1px solid rgba(148, 163, 184, 0.35);
      color: var(--muted);
      white-space: nowrap;
    }

    .aside-panel {
      background: linear-gradient(
        145deg,
        rgba(15, 23, 42, 0.95),
        rgba(15, 23, 42, 0.98),
        #020617
      );
      border-radius: var(--radius-xl);
      border: 1px solid rgba(148, 163, 184, 0.35);
      padding: 16px 16px 14px;
      box-shadow: var(--shadow-soft);
      position: sticky;
      top: 16px;
    }

    .aside-title {
      font-size: 0.96rem;
      font-weight: 600;
      margin-bottom: 6px;
    }

    .aside-grid {
      display: grid;
      grid-template-columns: repeat(2, minmax(0, 1fr));
      gap: 10px;
      margin: 10px 0 12px;
    }

    .stat {
      padding: 8px 9px;
      border-radius: 14px;
      background: rgba(15, 23, 42, 0.9);
      border: 1px solid rgba(148, 163, 184, 0.4);
    }

    .stat-label {
      font-size: 0.72rem;
      color: var(--muted);
      margin-bottom: 3px;
    }

    .stat-value {
      font-size: 0.9rem;
      font-weight: 600;
      font-variant-numeric: tabular-nums;
    }

    .legend {
      font-size: 0.78rem;
      color: var(--muted);
      display: flex;
      flex-direction: column;
      gap: 4px;
    }

    .legend-row {
      display: flex;
      align-items: center;
      gap: 6px;
    }

    .legend-swatch {
      width: 9px;
      height: 9px;
      border-radius: 999px;
    }

    .legend-low {
      background: var(--muted);
    }
    .legend-typical {
      background: var(--accent);
    }
    .legend-high {
      background: #fbbf24;
    }

    .aside-footer {
      font-size: 0.72rem;
      color: var(--muted);
      border-top: 1px dashed rgba(148, 163, 184, 0.35);
      padding-top: 8px;
      margin-top: 8px;
    }

    .aside-badge {
      display: inline-flex;
      align-items: center;
      gap: 6px;
      font-size: 0.72rem;
      border-radius: 999px;
      border: 1px solid rgba(148, 163, 184, 0.45);
      padding: 3px 8px 3px 4px;
      margin-bottom: 4px;
    }

    .badge-flag {
      font-size: 0.82rem;
    }

    .pill-danger {
      color: var(--danger);
      border-color: rgba(248, 113, 113, 0.7);
    }

    .no-results {
      font-size: 0.9rem;
      color: var(--muted);
      padding: 20px 2px;
    }

    footer {
      margin-top: 28px;
      font-size: 0.75rem;
      color: var(--muted);
      text-align: center;
      opacity: 0.85;
    }
  </style>
</head>
<body>
  <div class="app-shell">
    <header>
      <div class="title-block">
        <div class="title-pill">
          <span class="title-pill-dot"></span>
          Life cost browser · UK · rough estimates
        </div>
        <h1>“Jobs in Life” &amp; Ballpark Costs</h1>
        <p class="subtitle">
          Big life milestones and recurring expenses, with very rough
          UK ballpark figures. Use this as a sanity check, not a quote.
        </p>
      </div>
      <div class="controls">
        <div class="controls-row">
          <select id="filter-category" class="pill-select">
            <option value="all">All categories</option>
          </select>
          <select id="filter-type" class="pill-select">
            <option value="all">All types</option>
            <option value="one-off">One-off</option>
            <option value="recurring">Recurring</option>
          </select>
          <input
            id="search"
            class="search-input"
            type="search"
            placeholder="Search (e.g. wedding, car, rent)…"
          />
        </div>
        <div class="controls-row">
          <span class="tag-chip">£ = rough UK 2024–2025</span>
          <span class="tag-chip">Typical = mid-range ballpark</span>
        </div>
      </div>
    </header>

    <main class="layout">
      <section>
        <div id="cards" class="cards"></div>
        <div id="no-results" class="no-results" style="display:none;">
          No matches. Try a different search or reset filters.
        </div>
      </section>

      <aside class="aside-panel">
        <div class="aside-title">Quick reference</div>

        <div class="aside-grid">
          <div class="stat">
            <div class="stat-label">Typical UK house price</div>
            <div class="stat-value">~£270k</div>
          </div>
          <div class="stat">
            <div class="stat-label">Typical wedding (full day)</div>
            <div class="stat-value">~£20–23k</div>
          </div>
          <div class="stat">
            <div class="stat-label">Basic funeral (from)</div>
            <div class="stat-value">~£4k</div>
          </div>
          <div class="stat">
            <div class="stat-label">Baby’s first year</div>
            <div class="stat-value">~£7–11k</div>
          </div>
        </div>

        <div class="legend">
          <div class="legend-row">
            <span class="legend-swatch legend-low"></span>
            <span>Low = bare-bones / budget option</span>
          </div>
          <div class="legend-row">
            <span class="legend-swatch legend-typical"></span>
            <span>Typical = “middle of the road” ballpark</span>
          </div>
          <div class="legend-row">
            <span class="legend-swatch legend-high"></span>
            <span>High = premium / big-city end</span>
          </div>
        </div>

        <div class="aside-footer">
          <div class="aside-badge">
            <span class="badge-flag">⚠️</span>
            <span>Not financial advice</span>
          </div>
          <div>
            Numbers are rough UK estimates for 2024–2025 and can be far
            off for your area or lifestyle. Always check real quotes
            before making decisions.
          </div>
        </div>
      </aside>
    </main>

    <footer>
      Built as a simple static page. You can tweak the data inside
      <code>LIFE_JOBS</code> below to match your situation / country.
    </footer>
  </div>

  <script>
    /**
     * Minimal “jobs in life” dataset.
     * All figures are super rough UK ballparks in GBP.
     * low/typical/high are for quick comparison, not precise quotes.
     */
    const LIFE_JOBS = [
      // HOUSING
      {
        id: "buy-home",
        category: "Housing",
        name: "Buy a home (nationwide avg.)",
        type: "one-off",
        low: 180000,
        typical: 270000,
        high: 450000,
        timeframe: "one-off purchase",
        notes: "Huge regional variation, London can be much higher.",
      },
      {
        id: "first-time-buyer",
        category: "Housing",
        name: "First-time buyer deposit (10%)",
        type: "one-off",
        low: 20000,
        typical: 27000,
        high: 40000,
        timeframe: "saved over years",
        notes: "Assumes 10% deposit on an average home.",
      },
      {
        id: "rent-flat",
        category: "Housing",
        name: "Rent 1-bed flat (outside London)",
        type: "recurring",
        low: 700,
        typical: 900,
        high: 1200,
        timeframe: "per month",
        notes: "Cheaper in smaller towns, more in hotspots.",
      },
      {
        id: "rent-flat-london",
        category: "Housing",
        name: "Rent small flat (London)",
        type: "recurring",
        low: 1300,
        typical: 1700,
        high: 2300,
        timeframe: "per month",
        notes: "Central or trendy areas can be higher.",
      },

      // TRANSPORT
      {
        id: "used-car",
        category: "Transport",
        name: "Buy used car (typical petrol)",
        type: "one-off",
        low: 5000,
        typical: 15000,
        high: 25000,
        timeframe: "one-off purchase",
        notes: "Budget hatchback at low end, newer SUV at high end.",
      },
      {
        id: "car-running",
        category: "Transport",
        name: "Running costs for a car",
        type: "recurring",
        low: 200,
        typical: 300,
        high: 500,
        timeframe: "per month",
        notes: "Includes insurance, fuel, tax, basic maintenance.",
      },
      {
        id: "ebike",
        category: "Transport",
        name: "Electric bike for commuting",
        type: "one-off",
        low: 800,
        typical: 2000,
        high: 4000,
        timeframe: "one-off purchase",
        notes: "Budget hub motors to high-end cargo / mid-drive.",
      },

      // FAMILY & KIDS
      {
        id: "baby-first-year",
        category: "Family & kids",
        name: "Baby – first year of life",
        type: "one-off",
        low: 6000,
        typical: 9000,
        high: 12000,
        timeframe: "over first year",
        notes: "Depends heavily on childcare, gear and lifestyle.",
      },
      {
        id: "childcare-full",
        category: "Family & kids",
        name: "Full-time nursery / childcare",
        type: "recurring",
        low: 800,
        typical: 1200,
        high: 1800,
        timeframe: "per month",
        notes: "Varies by region, age and hours per week.",
      },
      {
        id: "school-uniform",
        category: "Family & kids",
        name: "School uniform & basics",
        type: "recurring",
        low: 150,
        typical: 250,
        high: 400,
        timeframe: "per year",
        notes: "State school; private / specialist schools cost more.",
      },

      // EDUCATION & CAREER
      {
        id: "uni-fees",
        category: "Education & career",
        name: "Undergrad tuition (home student)",
        type: "recurring",
        low: 9000,
        typical: 9535,
        high: 10000,
        timeframe: "per academic year",
        notes: "Capped for home students; loans usually cover this.",
      },
      {
        id: "uni-living",
        category: "Education & career",
        name: "Student living costs",
        type: "recurring",
        low: 800,
        typical: 1100,
        high: 1400,
        timeframe: "per month",
        notes: "Includes rent, food, bills, study costs.",
      },
      {
        id: "career-switch",
        category: "Education & career",
        name: "Career switch bootcamp / course",
        type: "one-off",
        low: 1000,
        typical: 4000,
        high: 8000,
        timeframe: "per course",
        notes: "Coding bootcamps and similar intensive programmes.",
      },

      // BIG LIFE EVENTS
      {
        id: "wedding",
        category: "Big life events",
        name: "UK wedding (full day, mid-range)",
        type: "one-off",
        low: 12000,
        typical: 22000,
        high: 30000,
        timeframe: "one-off event",
        notes: "Includes venue, food, photography, dress, etc.",
      },
      {
        id: "wedding-budget",
        category: "Big life events",
        name: "Budget wedding / micro-wedding",
        type: "one-off",
        low: 2000,
        typical: 6000,
        high: 10000,
        timeframe: "one-off event",
        notes: "Registry office, pub meal, small guest list.",
      },
      {
        id: "honeymoon",
        category: "Big life events",
        name: "Honeymoon",
        type: "one-off",
        low: 2000,
        typical: 5000,
        high: 10000,
        timeframe: "one-off trip",
        notes: "Short European trip to long-haul luxury.",
      },
      {
        id: "funeral-basic",
        category: "Big life events",
        name: "Basic funeral",
        type: "one-off",
        low: 3500,
        typical: 4300,
        high: 5500,
        timeframe: "one-off event",
        notes: "No wake included; cremation often cheaper than burial.",
      },
      {
        id: "funeral-full",
        category: "Big life events",
        name: "Funeral + wake & extras",
        type: "one-off",
        low: 6000,
        typical: 9000,
        high: 11000,
        timeframe: "one-off event",
        notes: "Includes venue, catering, flowers, printing, cars.",
      },

      // HEALTH & WELLBEING
      {
        id: "dentist",
        category: "Health & wellbeing",
        name: "Dental check-ups & minor work (private)",
        type: "recurring",
        low: 100,
        typical: 250,
        high: 600,
        timeframe: "per year",
        notes: "Basic check-up, clean and the odd filling.",
      },
      {
        id: "gym",
        category: "Health & wellbeing",
        name: "Gym membership",
        type: "recurring",
        low: 20,
        typical: 40,
        high: 80,
        timeframe: "per month",
        notes: "Budget chain to full-service gym / health club.",
      },
      {
        id: "therapy",
        category: "Health & wellbeing",
        name: "Private therapy / counselling",
        type: "recurring",
        low: 40,
        typical: 65,
        high: 100,
        timeframe: "per session",
        notes: "Weekly sessions add up quickly over a year.",
      },

      // DAY-TO-DAY LIFE
      {
        id: "groceries",
        category: "Day-to-day",
        name: "Groceries (single adult)",
        type: "recurring",
        low: 120,
        typical: 180,
        high: 250,
        timeframe: "per month",
        notes: "Cooking at home; takeaways push this up.",
      },
      {
        id: "utilities",
        category: "Day-to-day",
        name: "Energy + water + broadband",
        type: "recurring",
        low: 140,
        typical: 220,
        high: 320,
        timeframe: "per month",
        notes: "Depends on property size and insulation.",
      },
      {
        id: "phone",
        category: "Day-to-day",
        name: "Mobile phone (SIM-only)",
        type: "recurring",
        low: 10,
        typical: 20,
        high: 40,
        timeframe: "per month",
        notes: "Handset finance would be extra.",
      },
      {
        id: "holiday",
        category: "Day-to-day",
        name: "Holiday budget (per person)",
        type: "one-off",
        low: 500,
        typical: 1000,
        high: 2500,
        timeframe: "per trip",
        notes: "Short-haul vs. long-haul and hotel choice.",
      },
    ];

    function formatMoney(amount) {
      if (amount >= 1000) {
        const thousands = Math.round(amount / 100) / 10;
        return "£" + thousands.toFixed(thousands % 1 === 0 ? 0 : 1) + "k";
      }
      return "£" + amount.toString();
    }

    function buildCard(job) {
      const div = document.createElement("article");
      div.className = "card";

      const header = document.createElement("div");
      header.className = "card-header";

      const title = document.createElement("div");
      title.className = "card-title";
      title.textContent = job.name;

      const cat = document.createElement("div");
      cat.className = "card-category";
      cat.textContent = job.category;

      header.appendChild(title);
      header.appendChild(cat);
      div.appendChild(header);

      const lowRow = document.createElement("div");
      lowRow.className = "card-row";
      lowRow.innerHTML =
        '<span class="card-label">Low</span>' +
        '<span class="card-value">' +
        formatMoney(job.low) +
        "</span>";
      div.appendChild(lowRow);

      const typicalRow = document.createElement("div");
      typicalRow.className = "card-row";
      typicalRow.innerHTML =
        '<span class="card-label">Typical</span>' +
        '<span class="card-value card-value-typical">' +
        formatMoney(job.typical) +
        "</span>";
      div.appendChild(typicalRow);

      const highRow = document.createElement("div");
      highRow.className = "card-row";
      highRow.innerHTML =
        '<span class="card-label">High</span>' +
        '<span class="card-value card-value-high">' +
        formatMoney(job.high) +
        "</span>";
      div.appendChild(highRow);

      const meta = document.createElement("div");
      meta.className = "card-meta";

      const note = document.createElement("div");
      note.className = "card-note";
      note.textContent = job.notes;

      const tag = document.createElement("div");
      tag.className = "card-tag";
      tag.textContent = job.type + " · " + job.timeframe;

      meta.appendChild(note);
      meta.appendChild(tag);
      div.appendChild(meta);

      return div;
    }

    function getActiveFilters() {
      const catEl = document.getElementById("filter-category");
      const typeEl = document.getElementById("filter-type");
      const searchEl = document.getElementById("search");

      return {
        category: catEl.value,
        type: typeEl.value,
        search: searchEl.value.trim().toLowerCase(),
      };
    }

    function jobMatchesFilters(job, filters) {
      if (filters.category !== "all" && job.category !== filters.category) {
        return false;
      }
      if (filters.type !== "all" && job.type !== filters.type) {
        return false;
      }
      if (filters.search) {
        const haystack =
          (job.name + " " + job.category + " " + job.notes).toLowerCase();
        if (!haystack.includes(filters.search)) {
          return false;
        }
      }
      return true;
    }

    function renderCards() {
      const filters = getActiveFilters();
      const container = document.getElementById("cards");
      const noResults = document.getElementById("no-results");

      container.innerHTML = "";

      const filtered = LIFE_JOBS.filter((job) =>
        jobMatchesFilters(job, filters)
      );

      if (!filtered.length) {
        noResults.style.display = "block";
        return;
      }

      noResults.style.display = "none";

      filtered
        .slice()
        .sort((a, b) => a.category.localeCompare(b.category))
        .forEach((job) => {
          container.appendChild(buildCard(job));
        });
    }

    function populateCategoryFilter() {
      const select = document.getElementById("filter-category");
      const categories = Array.from(
        new Set(LIFE_JOBS.map((job) => job.category))
      ).sort((a, b) => a.localeCompare(b));

      categories.forEach((cat) => {
        const opt = document.createElement("option");
        opt.value = cat;
        opt.textContent = cat;
        select.appendChild(opt);
      });
    }

    function init() {
      populateCategoryFilter();
      renderCards();

      document
        .getElementById("filter-category")
        .addEventListener("change", renderCards);
      document
        .getElementById("filter-type")
        .addEventListener("change", renderCards);
      document.getElementById("search").addEventListener("input", renderCards);
    }

    document.addEventListener("DOMContentLoaded", init);
  </script>
</body>
</html>



```

## Spend some time publishing models on Printables

as per title

Wifi mount

etc
