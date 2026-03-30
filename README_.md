# 🤖 LinkedIn AI Auto Applier

> Fully automated LinkedIn Easy Apply bot with a GUI configuration wizard, AI-powered question answering, stealth Chrome, profile management, and CSV export — built for serious job seekers.

---

## 🎬 Video Demo

> _Paste your video link below (Google Drive, Loom, YouTube, etc.)_

| | |
|---|---|
| **Demo Video** | 🎥 _[Add your video link here]_ |

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Requirements](#requirements)
- [Installation](#installation)
- [Setting Up Portable Chrome](#setting-up-portable-chrome)
- [Configuration Guide](#configuration-guide)
- [Step-by-Step Usage Guide](#step-by-step-usage-guide)
- [AI Integration](#ai-integration)
- [Output Files](#output-files)
- [Stealth Mode](#stealth-mode)
- [How the Bot Works (Internals)](#how-the-bot-works-internals)
- [Building to .exe](#building-to-exe)
- [Notes & Limitations](#notes--limitations)

---

## Overview

**LinkedIn AI Auto Applier** is a Windows desktop automation tool that logs into your LinkedIn account, searches for jobs based on your configured keywords and filters, and applies to them automatically via Easy Apply — all without you lifting a finger. A full-screen Tkinter GUI wizard collects all your personal info, search preferences, and application answers before the bot starts. An optional AI layer (OpenAI, DeepSeek, or Gemini) powers context-aware answers to custom application questions. All results are logged to dated CSV files.

---

## ✨ Features

### GUI & Setup
- **Full configuration wizard** — Tkinter GUI with tabs for personal info, search filters, application preferences, and LinkedIn credentials
- **Profile Save/Load** — Save all settings as a named JSON profile, reload on next run
- **Excel credentials store** — Load multiple LinkedIn accounts from a hidden `.credentials.xlsx` file with autocomplete
- **Config file writer** — On submit, the GUI writes all values directly back into the Python config files (`personals.py`, `search.py`, `questions.py`, etc.)
- **Input validation** — Every field validated at launch via `validator.py` with clear, actionable error messages

### Search & Filtering
- Multiple search terms with configurable switching after N applications
- Randomized search order option
- Full LinkedIn filter support: Sort By, Date Posted, Salary, Experience Level, Job Type, On-site/Remote/Hybrid, Companies, Industry, Job Function, Job Titles, Benefits, Commitments
- Boolean filters: Easy Apply only, Under 10 applicants, In your network, Fair chance employer
- Pause after filters to manually inspect results before applying

### Smart Job Screening
- **Bad word filtering** — Skip jobs whose description contains blacklisted words (e.g. "No C2C", "Clearance", ".NET")
- **Company filtering** — Avoid companies with bad words in their About section, with whitelist exceptions
- **Experience gate** — Automatically skip jobs requiring more experience than your current level
- **Masters degree logic** — Optionally unlocks jobs up to +2 years above your experience if you have a Masters
- **Security clearance skip** — Skip jobs requiring clearance if not applicable
- **Duplicate detection** — Skips jobs already in your applied history CSV

### Application Handling
- **Easy Apply automation** — Fills all standard LinkedIn application fields automatically
- **AI-powered question answering** — Sends unknown questions to your configured LLM for contextual answers
- **Random fallback** — If AI is off, randomly answers unrecognized questions (and logs them for review)
- **Pause before submit** — Optional manual review of each application before submission
- **Pause on failed question** — Optional pause when the bot can't answer a question confidently
- **Salary auto-formatting** — Salary, CTC, and notice period auto-converted to lakhs, monthly, and weekly formats as needed by the form
- **Resume upload** — Uploads your default resume from the configured folder path

### Runtime & Safety
- **Tab crash guard** — Detects and recovers from stale/closed browser tabs
- **Browser alive watcher** — Background thread detects if you close Chrome manually and gracefully stops
- **Run non-stop mode** — Keeps looping through searches until stopped
- **Alternate sort by** — Alternates between "Most recent" and "Most relevant" in non-stop mode
- **Cycle date posted** — Cycles through date filters automatically in non-stop mode
- **Keep screen awake** — Prevents PC from sleeping during a run
- **Daily limit detection** — Stops gracefully if LinkedIn's daily Easy Apply limit is reached

### Output
- Applied jobs saved to `all excels/YYYY/MM/Applied Application DD-MM-YYYY.csv`
- Failed jobs saved to `all excels/YYYY/MM/Failed Application DD-MM-YYYY.csv`
- All activity logged to `logs/log.txt`
- End-of-run summary: Easy Applied, External, Failed, Skipped counts
- Motivational quote on exit 🙂

---

## 🛠️ Tech Stack

| Component | Technology |
|---|---|
| GUI | Python `tkinter` + `ttk` |
| Browser Automation | Selenium WebDriver |
| Anti-Detection | `undetected-chromedriver` (stealth mode) + standard Selenium (fallback) |
| Chrome | Portable Chrome (bundled, isolated from system Chrome) |
| AI Layer | OpenAI API / DeepSeek API / Google Gemini API (optional) |
| Data Export | Python `csv`, `pandas`, `openpyxl` |
| Resume Generation | `python-docx`, `fpdf` (experimental) |
| Config Storage | Python files + JSON profiles |
| Notifications | `pyautogui` alerts |
| Threading | Python `threading` |

---

## 📁 Project Structure

```
linkedin-auto-applier/
│
├── runAiBot.py                   # Main entry point — orchestrates login, search, apply loop
│
├── modules/
│   ├── open_chrome.py            # Chrome WebDriver factory (stealth + fallback)
│   ├── clickers_and_finders.py   # Selenium helper functions (click, find, scroll, type)
│   ├── helpers.py                # Logging, buffering, date parsing, utility functions
│   ├── validator.py              # Config file validation (types, values, options)
│   ├── extractor.py              # Job detail extraction
│   ├── generator.py              # Resume generator (docx + PDF, experimental)
│   ├── user_input_popup.py       # Main GUI configuration wizard
│   ├── user_input_popup_copy.py  # GUI backup/variant
│   │
│   └── ai/
│       ├── openaiConnections.py  # OpenAI API client
│       ├── deepseekConnections.py# DeepSeek API client
│       └── geminiConnections.py  # Google Gemini API client
│
├── config/
│   ├── personals.py              # Your personal info (name, address, EEO answers)
│   ├── questions.py              # Application answers (salary, visa, experience, etc.)
│   ├── search.py                 # LinkedIn search preferences and filters
│   ├── secrets.py                # LinkedIn credentials + AI API keys
│   ├── settings.py               # Bot behavior settings + file paths
│   ├── resume.py                 # Resume configuration (experimental)
│   └── .credentials.xlsx        # Multi-account Excel credential store (hidden)
│
├── GoogleChromePortable/
│   ├── App/Chrome-bin/chrome.exe # Portable Chrome binary
│   └── Driver/chromedriver.exe   # Matching ChromeDriver binary
│
├── chrome_profile/               # Auto-created — persistent Chrome session
├── profiles/                     # Auto-created — saved JSON configuration profiles
│
├── all resumes/
│   └── default/                  # Place your resume file here
│
├── all excels/                   # Auto-created — dated CSV output files
│   └── YYYY/MM/
│       ├── Applied Application DD-MM-YYYY.csv
│       └── Failed Application DD-MM-YYYY.csv
│
└── logs/
    └── log.txt                   # Full activity log
```

---

## ⚙️ Requirements

- **Windows** (Portable Chrome is a Windows binary)
- **Python 3.9+**

### Python Dependencies

```bash
pip install selenium undetected-chromedriver pandas openpyxl pyautogui python-docx fpdf
```

For AI features (optional — only if `use_AI = True`):

```bash
pip install openai google-generativeai
```

---

## 🚀 Installation

### 1. Clone or Download

```bash
git clone https://github.com/YOUR_USERNAME/linkedin-auto-applier.git
cd linkedin-auto-applier
```

### 2. Install Dependencies

```bash
pip install selenium undetected-chromedriver pandas openpyxl pyautogui python-docx fpdf
```

### 3. Set Up Portable Chrome

See [Setting Up Portable Chrome](#setting-up-portable-chrome) below.

### 4. Add Your Resume

Place your resume file inside:
```
all resumes/default/
```

### 5. Run

```bash
python runAiBot.py
```

The GUI wizard opens first. Fill in your details, click **🚀 SUBMIT & START APPLYING**, and the bot takes over.

---

## 🌐 Setting Up Portable Chrome

The bot uses an isolated Portable Chrome instance to avoid interfering with your system Chrome profile.

### Step 1 — Download Portable Chrome

1. Go to [portableapps.com/apps/internet/google_chrome_portable](https://portableapps.com/apps/internet/google_chrome_portable)
2. Download and extract it so the binary is at:
   ```
   GoogleChromePortable\App\Chrome-bin\chrome.exe
   ```

### Step 2 — Download Matching ChromeDriver

1. Open Portable Chrome and go to `chrome://version` to find the version number.
2. Download the matching `chromedriver` from [googlechromelabs.github.io/chrome-for-testing](https://googlechromelabs.github.io/chrome-for-testing/)
3. Place it at:
   ```
   GoogleChromePortable\Driver\chromedriver.exe
   ```

> ⚠️ Chrome and ChromeDriver versions **must match exactly** (same major version).

---

## 🔧 Configuration Guide

All config files live in the `config/` folder. The GUI wizard populates these automatically — you don't need to edit them by hand unless you prefer to.

### `config/secrets.py` — LinkedIn Credentials & AI

| Variable | Description |
|---|---|
| `username` | Your LinkedIn email |
| `password` | Your LinkedIn password |
| `use_AI` | `True` to enable AI question answering |
| `ai_provider` | `"openai"`, `"deepseek"`, or `"gemini"` |
| `llm_api_url` | API base URL (e.g. `https://api.openai.com/v1/`) |
| `llm_api_key` | Your API key (`"not-needed"` for local Ollama) |
| `llm_model` | Model name (e.g. `"gpt-4o"`, `"deepseek-chat"`) |
| `stream_output` | `True` to stream AI responses to console |

### `config/personals.py` — Personal Info

Name, phone, address, city, state, zipcode, country, and EEO fields (ethnicity, gender, disability, veteran status).

### `config/questions.py` — Application Answers

| Variable | Description |
|---|---|
| `default_resume_path` | Path to your resume folder |
| `years_of_experience` | Years of experience to answer experience questions |
| `require_visa` | `"Yes"` or `"No"` |
| `us_citizenship` | Citizenship status (exact string from options list) |
| `desired_salary` | Number (auto-converted to monthly/lakhs as needed) |
| `current_ctc` | Current salary number |
| `notice_period` | Notice period in days (auto-converted to weeks/months) |
| `linkedin_headline` | Your LinkedIn headline text |
| `linkedin_summary` | Your summary text |
| `cover_letter` | Your cover letter text |
| `confidence_level` | A number 1–10 for experience confidence questions |
| `pause_before_submit` | `True` to review each application before submitting |
| `pause_at_failed_question` | `True` to pause when a question can't be answered |
| `overwrite_previous_answers` | `True` to overwrite previously saved answers |

### `config/search.py` — Search Preferences

| Variable | Description |
|---|---|
| `search_terms` | List of job titles to search (e.g. `["Data Engineer", "Senior DE"]`) |
| `search_location` | Location string (e.g. `"United States"`) |
| `switch_number` | Switch to next search term after N applications |
| `sort_by` | `"Most recent"` or `"Most relevant"` |
| `date_posted` | `"Past 24 hours"`, `"Past week"`, `"Past month"`, `"Any time"` |
| `easy_apply_only` | `True` to only show Easy Apply jobs |
| `experience_level` | List of levels (e.g. `["Mid-Senior level", "Director"]`) |
| `job_type` | List of types (e.g. `["Full-time", "Contract"]`) |
| `on_site` | List of settings (e.g. `["Remote", "Hybrid"]`) |
| `bad_words` | Job description keywords that trigger a skip |
| `about_company_bad_words` | Company section keywords that trigger a skip |
| `current_experience` | Your experience in years (used for experience gate logic) |
| `security_clearance` | `True` if you have an active clearance |
| `did_masters` | `True` if you have a Masters degree |

### `config/settings.py` — Bot Behavior

| Variable | Description |
|---|---|
| `run_in_background` | `True` for headless (no visible browser) |
| `stealth_mode` | `True` to use `undetected-chromedriver` |
| `close_tabs` | `True` to close external application tabs |
| `follow_companies` | `True` to follow companies after applying |
| `run_non_stop` | `True` to keep looping searches |
| `alternate_sortby` | Alternates sort order in non-stop mode |
| `cycle_date_posted` | Cycles date filter in non-stop mode |
| `click_gap` | Max delay between clicks in seconds |
| `keep_screen_awake` | Prevents PC sleep during run |
| `smooth_scroll` | Smooth vs instant scrolling |
| `pause_after_filters` | Pause after applying search filters |

---

## 📖 Step-by-Step Usage Guide

### Step 1 — Launch the App

```bash
python runAiBot.py
```

The **GUI configuration wizard** opens automatically.

### Step 2 — Fill in Your Details

The GUI has multiple scrollable sections:

**LinkedIn Account**
- Select your email from the autocomplete dropdown (loaded from `.credentials.xlsx`) or type it manually
- Enter your password

**Personal Information**
- Full name, phone number, current city, address, state, zip, country
- EEO fields: ethnicity, gender, disability status, veteran status

**Profile Details**
- LinkedIn headline, summary, cover letter
- Years of experience, notice period, current CTC, desired salary
- LinkedIn URL, website/portfolio URL
- Visa requirement, US citizenship status

**Search Settings**
- Job titles to search (multi-select from a list)
- Search location, switch number
- Sort by, date posted, experience level, job type, on-site preference
- Industry, Easy Apply only, under 10 applicants toggle

**Bot Settings**
- Pause after filters, security clearance, Masters degree flag, current experience

### Step 3 — (Optional) Load a Saved Profile

Click **📂 Load Profile** to load a previously saved JSON profile — all fields populate instantly.

### Step 4 — Save Your Profile

Click **💾 Save Profile** before starting so you can reload next time without re-entering everything.

### Step 5 — Submit & Start

Click **🚀 SUBMIT & START APPLYING**. The GUI:
1. Validates all fields
2. Writes all values into the Python config files
3. Closes the wizard and launches the bot

### Step 6 — Bot Runs Automatically

Watch the console or log file as the bot:
1. Validates all config files
2. Launches Portable Chrome (stealth or standard)
3. Logs into LinkedIn with your credentials
4. Searches for jobs using your first search term with all filters
5. Optionally pauses after filters for you to review
6. Iterates through job cards, screening each one for bad words and experience requirements
7. Clicks into each qualifying job and attempts Easy Apply
8. Handles all standard form fields, uploads resume, answers questions
9. If `pause_before_submit = True`, pauses for your review before each submit
10. Saves result to CSV, moves to next job

### Step 7 — Review Results

When the bot finishes (or you press Stop), a summary popup shows:
- Jobs Easy Applied
- External job links collected
- Failed jobs
- Skipped jobs

CSV files are saved in `all excels/YYYY/MM/`.

---

## 🧠 AI Integration

When `use_AI = True` in `secrets.py`, the bot sends custom/unrecognized application questions to your configured LLM:

| Provider | Setup |
|---|---|
| **OpenAI** | Set `ai_provider = "openai"`, add your `llm_api_key`, set `llm_model = "gpt-4o"` |
| **DeepSeek** | Set `ai_provider = "deepseek"`, add your DeepSeek API key, set `llm_model = "deepseek-chat"` |
| **Google Gemini** | Set `ai_provider = "gemini"`, add your Gemini API key |
| **Local LLM (Ollama, LM Studio)** | Set `ai_provider = "openai"`, set `llm_api_url` to your local server, set `llm_api_key = "not-needed"` |

The AI extracts relevant skills and generates context-aware answers based on your `user_information_all` field and the question text. If AI is disabled, the bot answers randomly and logs those questions for your post-run review.

---

## 📊 Output Files

### Applied Jobs CSV — `Applied Application DD-MM-YYYY.csv`

Each row is one Easy Applied job. Columns include Job ID, Job Title, Company, Location, Work Type, Posted Date, Application Date, Salary, and more.

### Failed Jobs CSV — `Failed Application DD-MM-YYYY.csv`

Jobs where the application could not be completed — includes the error reason.

### Log File — `logs/log.txt`

Complete activity log with timestamps: every search, every click, every skip reason, every error.

---

## 🥷 Stealth Mode

Set `stealth_mode = True` in `settings.py` to use `undetected-chromedriver` instead of standard Selenium. This patches Chrome at the binary level to remove automation signatures that LinkedIn's anti-bot system looks for. If `undetected-chromedriver` fails to launch (e.g. version mismatch), the bot automatically falls back to standard Selenium Chrome.

Both modes use a **single persistent Chrome profile** stored in `chrome_profile/` — so your LinkedIn login session is preserved across runs.

---

## 🔍 How the Bot Works (Internals)

```
runAiBot.py
│
├── validate_config()          → Check all config files for type/value errors
├── get_user_inputs()          → Launch GUI, collect and write all settings
├── open_chrome.py             → Launch Portable Chrome (stealth or standard)
├── login_LN()                 → Log into LinkedIn, fallback to manual if needed
│
└── run() [main apply loop]
    ├── For each search_term:
    │   ├── set_search_location()
    │   ├── search_jobs()          → Build LinkedIn search URL with all filters
    │   ├── pause_after_filters    → Optional manual inspection
    │   │
    │   └── For each job card on each page:
    │       ├── get_applied_job_ids()     → Skip already-applied jobs
    │       ├── Screen for bad_words      → Skip if match found
    │       ├── Screen for experience     → Skip if above threshold
    │       ├── Screen about_company      → Skip if bad company match
    │       │
    │       └── apply_to_job()
    │           ├── Click Easy Apply
    │           ├── fill_up()             → Fill all form steps
    │           │   ├── personal fields (name, phone, address, EEO)
    │           │   ├── resume upload
    │           │   ├── standard questions (salary, visa, experience, etc.)
    │           │   └── unknown questions → AI or random fallback
    │           ├── pause_before_submit   → Optional review
    │           └── Submit → log to CSV
│
└── Final summary + motivational quote + browser close
```

---

## 📦 Building to .exe

```bash
pip install pyinstaller
```

```bash
pyinstaller --onefile --windowed ^
  --add-data "GoogleChromePortable;GoogleChromePortable" ^
  --add-data "config;config" ^
  --add-data "modules;modules" ^
  --add-data "all resumes;all resumes" ^
  runAiBot.py
```

> At runtime, the exe detects `sys.frozen` and uses `sys._MEIPASS` for bundled assets, while keeping `all excels/`, `logs/`, `profiles/`, and `chrome_profile/` next to the `.exe` (writable, not in temp).

---

## ⚠️ Notes & Limitations

- **Windows only** — Portable Chrome binaries are Windows `.exe` files.
- **LinkedIn account required** — You must have an active LinkedIn account.
- **Easy Apply only** — The bot cannot fill external third-party application portals.
- **Daily LinkedIn limits** — LinkedIn enforces a daily Easy Apply cap. The bot detects this and stops gracefully.
- **Form variations** — Some custom Easy Apply forms may not be handled. Failed applications are logged to the failed CSV.
- **ChromeDriver must match Chrome** — Update both together when upgrading Portable Chrome.
- **Credentials security** — `secrets.py` and `.credentials.xlsx` contain your LinkedIn password. Do not commit these to a public repository.
- **Stealth mode is experimental** — `undetected-chromedriver` may break on Chrome updates. If it fails, the bot falls back to standard Selenium automatically.
- **AI costs money** — If using OpenAI or DeepSeek with a paid API key, each question answered costs tokens. Monitor your API usage.

---

## 📄 License

Copyright © 2026 Jay Verma. All rights reserved.

---

*Built for the US IT consulting and staffing space*
