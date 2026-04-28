# Reptalytics — Setup Guide

Step-by-step instructions for getting the project running locally in RStudio
and publishing to GitHub Pages.

---

## 1  Prerequisites

| Tool | Version | Download |
|------|---------|----------|
| R | ≥ 4.3 | https://cran.r-project.org |
| RStudio | ≥ 2023.12 ("Ocean Storm") | https://posit.co/download/rstudio-desktop |
| Quarto CLI | ≥ 1.4 | https://quarto.org/docs/get-started |
| Python | ≥ 3.10 | https://www.python.org/downloads |
| Git | any recent | https://git-scm.com |

> Quarto ships bundled with recent RStudio versions.  
> Run `quarto check` in a terminal to confirm it is on your PATH.

---

## 2  Clone / Create the GitHub Repo

### Option A — start fresh on GitHub (recommended)

1. Go to https://github.com/new
2. Repository name: `reptalytics`
3. Set visibility (Public required for free GitHub Pages)
4. ✅ Add a README
5. Click **Create repository**
6. Copy the HTTPS clone URL

```bash
git clone https://github.com/<your-username>/reptalytics.git
cd reptalytics
```

### Option B — push this folder to a new repo

```bash
cd reptalytics          # the folder you downloaded
git init
git add .
git commit -m "Initial commit — Reptalytics Quarto report"
git branch -M main
git remote add origin https://github.com/<your-username>/reptalytics.git
git push -u origin main
```

---

## 3  Open the R Project in RStudio

1. In RStudio: **File → Open Project…**
2. Navigate to the `reptalytics/` folder
3. Select `reptalytics.Rproj` → **Open**

RStudio will now set the working directory to the project root automatically.

---

## 4  Set Up the Python Environment

Quarto uses **reticulate** to run Python chunks. You need a Python environment
with the project's dependencies installed.

### 4a  Create a virtual environment (recommended)

Open the **Terminal** tab in RStudio (not the R Console) and run:

```bash
# Windows
python -m venv .venv
.venv\Scripts\activate

# macOS / Linux
python3 -m venv .venv
source .venv/bin/activate
```

### 4b  Install Python packages

```bash
pip install -r requirements.txt
```

### 4c  Tell Quarto / reticulate which Python to use

Create (or edit) `_quarto.yml` in the project root — it is already included in
this repo and points to the virtual environment automatically via the
`python` key. If you need to override it, set the environment variable before
rendering:

**Windows (PowerShell):**
```powershell
$env:QUARTO_PYTHON = ".\.venv\Scripts\python.exe"
```

**macOS / Linux:**
```bash
export QUARTO_PYTHON=".venv/bin/python"
```

Alternatively, add this block to the top of `reptalytics_report.qmd`:

```yaml
jupyter:
  kernelspec:
    name: python3
    display_name: Python 3
    language: python
```

And point reticulate from R before rendering:

```r
Sys.setenv(RETICULATE_PYTHON = ".venv/bin/python")
```

---

## 5  Render the Report

### From RStudio (GUI)

Open `reptalytics_report.qmd` → click the **Render** button in the toolbar.

### From the Terminal

```bash
quarto render reptalytics_report.qmd
```

The output file `reptalytics_report.html` will appear in the project root.
Open it in any browser — it is fully self-contained (no server needed).

> **Note:** The first render will pull live data from the NYC Open Data API
> (several hundred thousand rows). Expect 2–5 minutes depending on your
> connection. Subsequent renders are faster if you enable Quarto caching by
> adding `cache: true` to the relevant code chunks.

---

## 6  Publish to GitHub Pages

### 6a  Configure `_quarto.yml` output directory

The included `_quarto.yml` already sets `output-dir: docs` so GitHub Pages
can serve from the `docs/` folder on the `main` branch.

### 6b  Render to `docs/`

```bash
quarto render reptalytics_report.qmd --output-dir docs
```

### 6c  Commit & push

```bash
git add docs/
git commit -m "Publish rendered report to docs/"
git push
```

### 6d  Enable GitHub Pages

1. Go to your repo on GitHub
2. **Settings → Pages**
3. Source: **Deploy from a branch**
4. Branch: `main` / folder: `/docs`
5. Click **Save**

Your report will be live at:
`https://<your-username>.github.io/reptalytics/reptalytics_report.html`

---

## 7  Project Folder Structure

```
reptalytics/
├── reptalytics.Rproj          # RStudio project file
├── reptalytics_report.qmd     # ← main Quarto document
├── _quarto.yml                # Quarto project settings
├── requirements.txt           # Python dependencies
├── .gitignore
├── docs/
│   └── custom.css             # custom HTML theme
├── data/                      # (gitignored) place raw data here if needed
└── scripts/                   # optional: standalone .py helper scripts
```

---

## 8  Troubleshooting

| Problem | Fix |
|---------|-----|
| `ModuleNotFoundError: plotly` | Activate venv, run `pip install -r requirements.txt` |
| Quarto can't find Python | Set `QUARTO_PYTHON` env var (see §4c) |
| API timeout during render | Add `cache: true` to `#\| label: load-data` chunk |
| HTML renders but Plotly charts blank | Open in Chrome/Firefox, not Safari's built-in previewer |
| `git push` rejected | Run `git pull --rebase origin main` first |

---

## 9  Optional: NYC Open Data App Token

The API works without a token but is rate-limited (~1000 req/hr per IP).
For faster / more reliable fetching:

1. Create a free account at https://data.cityofnewyork.us
2. Go to **Profile → Developer Settings → Create New App Token**
3. Copy the token and paste it into `reptalytics_report.qmd`:

```python
APP_TOKEN = "your_token_here"   # line ~30 in the setup chunk
```

Keep your token out of version control — use a `.env` file or an environment
variable instead.

---

*Reptalytics · Baruch College CIS 9650*
