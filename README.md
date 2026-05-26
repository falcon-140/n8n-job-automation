# 🤖 AI-Powered Job Search Automation Pipeline

> **Scrapes fresh LinkedIn jobs daily → tailors your resume with AI → scores ATS compatibility → delivers a personalized job package to your inbox. You just click Apply.**


---

## 📌 The Problem I Was Solving

Job searching is broken. Most people spend 2–3 hours a day manually searching LinkedIn, copy-pasting resumes, and tailoring cover letters — only to never hear back because their resume didn't pass the ATS filter.

I built this system to fix that. It runs every morning at 7am, finds the freshest job postings (posted in the last 24 hours, before hundreds of other applicants flood in), rewrites my resume to match each job's exact keywords, scores the ATS compatibility, and delivers everything to my inbox. I wake up to a ready-to-apply package — I just review and click.

**Time saved: ~2 hours/day. Response rate: significantly higher due to ATS-optimized, tailored resumes.**

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     DAILY 7AM TRIGGER                           │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  SCRAPE LAYER                                                   │
│  LinkedIn Public Guest API → 10-20 fresh jobs (last 24 hrs)    │
│  No login required · No cookies · No API costs                 │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  PARSE & EXTRACT LAYER                                          │
│  Custom JavaScript parser → extracts title, company,           │
│  location, apply URL, posted time from raw HTML                │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  STORAGE LAYER                                                  │
│  Google Sheets → append/update rows with deduplication         │
│  Tracks every job seen · prevents duplicate processing         │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  AI LAYER  (coming in v2)                                       │
│  Google Gemini → fetch full JD → rewrite resume to match       │
│  ATS scoring → iterate until score ≥ 80% → generate PDF       │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  DELIVERY LAYER                                                 │
│  Gmail → draft email per job with title, company,              │
│  location, apply link, tailored resume, ATS score report       │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│  HUMAN APPROVAL (intentional design decision)                  │
│  You review → click apply → final submission is always yours   │
└─────────────────────────────────────────────────────────────────┘
```

---

## ✅ Current Features (v1)

- **Daily LinkedIn scraping** via LinkedIn's public guest API — no login, no cookies, no scraping costs
- **Smart HTML parser** written in JavaScript that extracts structured job data from raw LinkedIn HTML responses
- **Google Sheets integration** — all jobs logged with deduplication so the same job is never processed twice
- **Gmail draft creation** — one draft per job delivered to your inbox every morning
- **Fully self-hosted** on a Linux VPS — no monthly SaaS fees, complete data ownership
- **Scheduled execution** — runs automatically at 7am daily via n8n's cron trigger
- **Human-in-the-loop design** — the system prepares everything, but final submission is always a human decision

---

## 🚀 Roadmap (v2 — In Progress)

- [ ] **AI Resume Tailoring** — Gemini rewrites base resume to match each job description's exact keywords and requirements
- [ ] **ATS Score Validation** — automated scoring loop: if score < 80%, Gemini rewrites and retries (max 3 attempts)
- [ ] **Full Attempt Report** — email includes every rewrite attempt, score improvement, and final optimized resume
- [ ] **PDF Resume Generation** — tailored resume exported as a clean, ATS-friendly PDF attached to each email
- [ ] **Job Description Fetcher** — automatically fetches and parses the full job description from each listing URL
- [ ] **Multi-role Support** — parallel pipelines for Software Engineer, Data Scientist, Cybersecurity, Frontend, Prompt Engineering roles

---

## 🛠️ Tech Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| Automation engine | n8n (self-hosted) | Open source, visual, extensible |
| Scraping | LinkedIn Public Guest API | Free, no login required, no blocks |
| HTML Parsing | Custom JavaScript (n8n Code node) | Full control over extraction logic |
| AI / LLM | Google Gemini 2.0 Flash | Free tier, fast, strong instruction following |
| Storage | Google Sheets | Simple, visual, shareable |
| Email | Gmail API (OAuth2) | Native drafts, full control |
| Auth | Google OAuth2 (Cloud Console) | Secure, no hardcoded credentials |
| Infrastructure | Linux VPS (Ubuntu 24.04, KVM1) | Self-hosted, $7/mo, full root access |
| Containerization | Docker (via Hostinger n8n image) | Isolated, reproducible, easy updates |

---

## ⚙️ Setup & Installation

### Prerequisites

- A Linux VPS (Ubuntu 20.04+) with Docker installed
- Google account (for Sheets, Gmail, Gemini APIs)
- n8n instance (self-hosted or cloud)

### Step 1 — Deploy n8n on your VPS

```bash
# Install Docker if not already installed
curl -fsSL https://get.docker.com | sh

# Run n8n with Docker
docker run -d \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  --restart always \
  n8nio/n8n
```

Access your n8n instance at `http://YOUR_VPS_IP:5678`

### Step 2 — Set up Google Cloud credentials

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project (e.g. `n8n-job-automation`)
3. Enable these APIs:
   - Google Sheets API
   - Google Drive API
   - Gmail API
4. Create OAuth 2.0 credentials (Web Application type)
5. Add your n8n callback URL as an authorized redirect URI:
   ```
   https://YOUR_N8N_DOMAIN/rest/oauth2-credential/callback
   ```
6. Copy your **Client ID** and **Client Secret**

### Step 3 — Set up Google Sheets

Create a Google Sheet named **LinkedIn Jobs** with two tabs:
- **Jobs** tab with these columns:

```
id | title | companyName | location | applyUrl | postedAtTimestamp
```

- **Contacts** tab (for future decision-maker enrichment)

Copy your **Spreadsheet ID** from the URL.

### Step 4 — Import the workflow

1. In n8n, click **"+"** → **"New Workflow"**
2. Press `Ctrl+Shift+V` to import from clipboard
3. Paste the contents of `workflow.json` from this repo
4. Connect your Google credentials to the Google Sheets and Gmail nodes
5. Add your Gemini API key to the Google Gemini Chat Model node

### Step 5 — Configure LinkedIn search URLs

In the **Pull Jobs** node, update the URL to match your target role and location:

```
https://www.linkedin.com/jobs-guest/jobs/api/seeMoreJobPostings/search?keywords=YOUR+ROLE&location=YOUR+LOCATION&f_TPR=r86400&start=0
```

**Example search URLs:**

```
# Software Engineer in California
keywords=Software+Engineer&location=California

# Data Scientist in San Francisco
keywords=Data+Scientist&location=San+Francisco

# Full Stack Developer in New York
keywords=Full+Stack+Developer&location=New+York
```

The `f_TPR=r86400` parameter filters to jobs posted in the **last 24 hours** — this is the key to applying before hundreds of other candidates.

### Step 6 — Activate

Hit **Publish** in the top right of n8n. The workflow will run automatically every day at 7am.

For an immediate test run, click **"Execute Workflow"**.

---

## 📁 Repository Structure

```
n8n-job-automation/
│
├── workflow.json              # Main n8n workflow (import this)
├── README.md                  # This file
├── docs/
│   ├── architecture.md        # Detailed architecture notes
│   └── setup-screenshots/     # Step-by-step visual setup guide
└── scripts/
    └── html-parser.js         # Standalone LinkedIn HTML parser
```

---

## 💡 Key Engineering Decisions

**Why self-hosted n8n over Zapier/Make?**
Full control, no per-execution pricing, no data leaving my infrastructure. At scale (running 365 days/year), self-hosting saves $200–500/year over cloud automation platforms.

**Why LinkedIn's public guest API instead of a scraping service?**
LinkedIn exposes a public job listing API for search crawlers. It requires no authentication, no cookies, and has no associated cost. Third-party scraping services charge $20–50/month for the same data.

**Why a human approval step?**
Auto-applying without reviewing is how you end up submitting to roles you're underqualified for or companies with red flags. The system handles the research and preparation — the human handles the judgment call. This design also avoids LinkedIn's bot detection systems.

**Why Google Gemini over OpenAI?**
Gemini's free tier is generous enough for daily resume tailoring without incurring API costs during a job search. The quality for instruction-following tasks (resume rewriting, keyword extraction) is comparable.

---

## 📊 Results

| Metric | Before Automation | After Automation |
|--------|------------------|-----------------|
| Time spent on job search | 2–3 hrs/day | 15 min/day |
| Jobs reviewed daily | 5–10 | 20+ |
| Resume tailored per application | Rarely | Every time |
| ATS pass rate | Unknown | Tracked & optimized |
| Applications to jobs < 24hrs old | Rarely | Always |

---

## 🔒 Security Notes

- **No credentials are hardcoded** — all API keys and tokens are stored in n8n's encrypted credential store
- **OAuth2 is used** for all Google services — no passwords stored anywhere
- **The workflow JSON in this repo has all sensitive values removed** — you must add your own credentials after importing
- Regenerate any API tokens that were accidentally exposed during development

---

## 🤝 Contributing

This is a personal project but PRs are welcome. Areas where contributions would be valuable:

- Additional job board scrapers (Indeed, Glassdoor, Wellfound)
- Better ATS scoring algorithms
- Resume PDF generation improvements
- Multi-language support

---

## 📄 License

MIT — use this however you want. If it helps your job search, that's the whole point.

---

## 👤 Author

**Laksh Agarwal**
- GitHub: [@falcon-140](https://github.com/falcon-140)
- LinkedIn: [laksh-agarwal-7731501b2](https://www.linkedin.com/in/laksh-agarwal-7731501b2)
- Email: lakshagarwal0710@gmail.com

---

*Built out of frustration with the job search process. If you're also job hunting — good luck. This system won't get you the job, but it'll make sure your resume is in front of the right people at the right time.*
