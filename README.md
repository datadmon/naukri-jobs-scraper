# Naukri Jobs Scraper (Python): Naukri.com jobs to JSON and CSV

[![Run on Apify](https://img.shields.io/badge/Run%20on-Apify-00A67E?logo=apify&logoColor=white)](https://apify.com/data_daemon/naukri-jobs-scraper)
![Output](https://img.shields.io/badge/output-JSON%20%7C%20CSV%20%7C%20Excel-blue)
![License: MIT](https://img.shields.io/badge/license-MIT-green)

Scrape **Naukri.com jobs** into JSON or CSV from Python: title, company, city, work mode, experience, **salary in rupees and LPA**, skills, applicants, views, role, industry, AmbitionBox rating and the **full job description**. Search by keyword and city, or export every job Naukri posted in the last N days.

[Naukri](https://www.naukri.com) is India's largest job board. This repository is a small, tested Python client for the hosted
**[Naukri Jobs Scraper – India Jobs API with Salary (LPA)](https://apify.com/data_daemon/naukri-jobs-scraper)** on Apify: no browser, proxy, login or scraper maintenance on your side. Price:
**$0.80 per 1,000 jobs, full description included, plus $0.001 per run**; Apify's free plan ($5 of credit every month) covers about 6,000 jobs.

## Quick start

```bash
pip install apify-client
export APIFY_TOKEN=your_token      # Apify Console > Settings > Integrations
python scrape.py --keyword "data analyst" --city "Bangalore" --max 50
```

`scrape.py` starts the Actor, waits for it and writes `naukri-jobs.json` (every field) and `naukri-jobs.csv`
(the main columns). For any other option, put the Actor input in a file and pass `--input`:

```bash
python scrape.py --input examples/input.json
```

```json
{
  "queries": [
    "data analyst"
  ],
  "locations": [
    "Bangalore"
  ],
  "postedWithinDays": 3,
  "maxJobs": 5
}
```

## Sample output

A real run of [`examples/input.json`](examples/input.json) on 5 November 2026
([full JSON](examples/sample-output.json), [CSV](examples/sample-output.csv); descriptions shortened):

| postedAt | title | company | locationText | experienceText | salaryText |
|---|---|---|---|---|---|
| 2026-11-05 | Data Analyst | Thyrocare | Bengaluru | 1-3 Yrs | 7-9 Lacs |
| 2026-11-04 | ISC Data Analyst | nVent | Bengaluru | 3-8 Yrs | Not Disclosed |
| 2026-11-04 | Data Analyst | Moveinsync Technology Solutions | Bengaluru | 1-3 Yrs | Not Disclosed |
| 2026-11-04 | Data Analyst | Quess IT Solutions | Bengaluru | 7-12 Yrs | Not Disclosed |
| 2026-11-05 | Data Analyst | Jobaaj Com | Pune, Gurugram, Bengaluru | 0-2 Yrs | 3-6.5 Lacs |

## Output fields

| Field | Description |
|---|---|
| `id` | Naukri job ID (e.g. 041126017601). |
| `dedupKey` | naukri:<id>, stable across runs. |
| `url` | Job page on naukri.com. |
| `title` | Job title. |
| `company` | Company name, Naukri company and group IDs, logo, website and address (from the job page), AmbitionBox rating, review count and URL. |
| `experienceMinYears` | Minimum years of experience. |
| `experienceMaxYears` | Maximum years of experience. |
| `experienceText` | Experience as shown, e.g. "3-6 Yrs". |
| `salaryMinInr` | Minimum salary in INR per year; null when the employer hides it. |
| `salaryMaxInr` | Maximum salary in INR per year. |
| `salaryMinLpa` | Minimum salary in lakhs per annum (LPA). |
| `salaryMaxLpa` | Maximum salary in LPA. |
| `salaryHidden` | true when the employer did not disclose the salary. |
| `salaryText` | Salary as shown, e.g. "3-8 Lacs PA" or "Not Disclosed". |
| `locations` | Cities of the job. |
| `locationText` | Location as shown on the search card, e.g. "Hybrid - Pune". |
| `workMode` | office, hybrid or remote. |
| `skills` | Key skills. |
| `roleCategory` | Naukri role category (job page). |
| `role` | Naukri role (job page). |
| `functionalArea` | Department / function (job page). |
| `industry` | Industry (job page). |
| `employmentType` | E.g. "Full Time, Permanent" (job page). |
| `education` | Required UG / PG / doctorate degrees (job page). |
| `openings` | Number of openings. |
| `applicants` | Applications so far (job page). |
| `views` | Job page views (job page). |
| `postedByConsultant` | true when posted by a recruitment consultant (job page). |
| `walkIn` | true for walk-in interviews. |
| `postedAt` | When the job was posted (ISO 8601, UTC). |
| `summary` | Short summary from Naukri. |
| `description` | Full description as plain text (with descriptions on). |
| `query` | The keyword that found the job (null for all-jobs runs). |
| `scrapedAt` | When the job was scraped (ISO 8601). |

## Daily feed of only new jobs

```bash
python scrape.py --keyword "data analyst" --only-new naukri-daily --max 0
```

With `--only-new <name>` the Actor remembers which jobs it has delivered under that name and returns only new ones;
skipped jobs are not charged. Run it from cron, a GitHub Actions schedule or an Apify schedule:

```yaml
# .github/workflows/naukri-daily.yml
on:
  schedule: [{cron: "30 2 * * *"}]
  workflow_dispatch:
jobs:
  scrape:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install apify-client && python scrape.py --keyword "data analyst" --only-new naukri-daily --max 0
        env: {APIFY_TOKEN: "${{ secrets.APIFY_TOKEN }}"}
      - uses: actions/upload-artifact@v4
        with: {name: naukri-jobs, path: "naukri-jobs.*"}
```

## Other ways to call it

**curl**, one request that runs the Actor and returns the rows:

```bash
curl -X POST "https://api.apify.com/v2/acts/data_daemon~naukri-jobs-scraper/run-sync-get-dataset-items" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"queries": ["data analyst"], "locations": ["Bangalore"], "postedWithinDays": 3, "maxJobs": 5}'
```

**Apify CLI:** `apify call data_daemon/naukri-jobs-scraper --input-file examples/input.json`

**No code:** open the [Actor page](https://apify.com/data_daemon/naukri-jobs-scraper), fill the form and export to CSV/Excel, or connect it to Google Sheets,
Zapier, Make or n8n. AI agents can call it through the Apify MCP server (`https://mcp.apify.com`).

## FAQ

### Does Naukri have a public jobs API?

No. Naukri offers no public job-search API for third parties; its recruiter products (Resdex, job posting) are paid and employer-side. This repo calls a hosted Apify Actor that reads the public job search and job pages, so you get an API without running a scraper yourself.

### Can I export every job on Naukri, not just a keyword?

Yes. Set `"allJobs": true` with `postedWithinDays` (for example 1 for a daily feed). Naukri's search stops paging at about 15,000 results, so the Actor splits large exports by job function and city and merges them without duplicates.

### How do I get only new Naukri jobs every day?

Run `scrape.py --only-new my-feed` once a day (cron, GitHub Actions or an Apify schedule). Jobs already delivered to that feed name are skipped and not charged.

### Is it legal to scrape Naukri?

The Actor reads only public job listings, without logging in, at a polite request rate, and returns job and company
data, not personal profiles. Check the rules that apply to your use of the data in your jurisdiction.

## Related

- [IIMJobs Jobs Scraper – MBA, Finance & Management Jobs India](https://apify.com/data_daemon/iimjobs-jobs-scraper)
- [Hirist.tech Jobs Scraper – India IT & Tech Jobs, Full JD](https://apify.com/data_daemon/hirist-jobs-scraper)
- [Foundit Jobs Scraper | foundit.in (Monster India) Jobs API](https://apify.com/data_daemon/foundit-jobs-scraper)
- [India Jobs Feed | Naukri + 9 Indian Job Boards in One API](https://apify.com/data_daemon/india-jobs-feed)
- [India Job Alerts](https://github.com/datadmon/india-job-alerts): new jobs from seven Indian boards every morning via GitHub Actions
- [All Daemon Data Actors](https://apify.com/data_daemon)

Unofficial project, not affiliated with Naukri. Issues and questions: open an issue here or on the
[Actor's Issues tab](https://apify.com/data_daemon/naukri-jobs-scraper/issues).
