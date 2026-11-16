# Greenhouse, Lever, Ashby, Workable & Workday jobs API: search 1M+ open jobs in one call

How to search the open jobs of 20,600 companies (1,093,665 jobs on 2026-11-16) that hire through **Workday, Greenhouse, Lever, Ashby, Workable, SmartRecruiters, Personio, Recruitee, Teamtailor and Breezy HR**, without crawling each career page yourself.

This repo is a short guide with real output. The search runs on the Apify Actor [Greenhouse, Lever, Ashby & Workable Jobs API](https://apify.com/renzomacar/ats-jobs-search). It reads a prebuilt index (rebuilt twice a day from the public job boards), so a search answers in a few seconds.

## Example: remote data engineer jobs with a published salary, posted this week

Input:

```json
{
  "keywords": ["data engineer"],
  "remoteOnly": true,
  "postedWithinDays": 7,
  "onlyWithSalary": true,
  "maxItems": 25
}
```

Run on 2026-11-16: 17 jobs in 3.9 seconds. First ten (full file: [sample-output.json](sample-output.json)):

| Company | Title | Location | Salary | Posted | ATS |
|---|---|---|---|---|---|
| Airwallex | Staff Data Platform Engineer, Knowledge Platform | US - Seattle | 200000-260000 USD | 2026-11-16 | ashby |
| The Voleon Group | Staff Software Engineer, Data Infrastructure | Remote, United States; New York; Berkele | 295000-400000 USD | 2026-11-13 | ashby |
| Tessera Labs | Data Forward Deployed Engineer – Applied Data & ML Engineering | Remote in the U.S. | 180000-233000 USD | 2026-11-13 | ashby |
| Logicalis Spain | Data Engineer Sr dbt | Madrid, ES | 45000-50000 EUR | 2026-11-13 | teamtailor |
| Abnormal | Software Engineer I, Data Platforms | Remote - USA | 122400-176000 USD | 2026-11-13 | greenhouse |
| Natera | Sr. Forward Deployed Data Engineer | US Remote | 125000-156300 USD | 2026-11-12 | greenhouse |
| Ditto | Senior Software Engineer, Data Sync | Remote (Atlanta, Washington DC, San Fran | 188000-259000 USD | 2026-11-12 | ashby |
| Clera | Senior Data Engineer | remote | 120000-160000 USD | 2026-11-12 | ashby |
| Clera | Senior Data Engineer | remote | 80000-120000 USD | 2026-11-12 | ashby |
| Clera | Senior Data Engineer | remote | 100000-140000 USD | 2026-11-12 | ashby |

One record:

```json
{
  "company": "Airwallex",
  "companySlug": "airwallex",
  "ats": "ashby",
  "jobId": "9ef670e4-5767-42d0-91d1-6addace2cb67",
  "title": "Staff Data Platform Engineer, Knowledge Platform",
  "department": "Engineering",
  "team": "Knowledge Platform",
  "location": "US - Seattle",
  "remote": true,
  "workplaceType": "remote",
  "employmentType": "Full-time",
  "postedAt": "2026-11-16T02:38:52.604Z",
  "updatedAt": null,
  "url": "https://jobs.ashbyhq.com/airwallex/9ef670e4-5767-42d0-91d1-6addace2cb67",
  "applyUrl": "https://jobs.ashbyhq.com/airwallex/9ef670e4-5767-42d0-91d1-6addace2cb67/application",
  "salary": {
    "min": 200000,
    "max": 260000,
    "currency": "USD",
    "interval": "year",
    "text": "$200K – $260K • Offers Equity • Offers Bonus"
  },
  "descriptionText": null,
  "indexedAt": "2026-11-16T05:32:56.390Z",
  "scrapedAt": "2026-11-16T09:43:49.042Z"
}
```

## Call it from code

You need an Apify account and its API token (`APIFY_TOKEN`).

curl, results back in the same request:

```bash
curl -X POST "https://api.apify.com/v2/acts/renzomacar~ats-jobs-search/run-sync-get-dataset-items?token=$APIFY_TOKEN"   -H "content-type: application/json"   -d '{"keywords":["data engineer"],"remoteOnly":true,"postedWithinDays":7,"maxItems":25}'
```

Python:

```python
from apify_client import ApifyClient

client = ApifyClient("YOUR_APIFY_TOKEN")
run = client.actor("renzomacar/ats-jobs-search").call(run_input={
    "keywords": ["data engineer"],
    "remoteOnly": True,
    "postedWithinDays": 7,
    "maxItems": 25,
})
for job in client.dataset(run["defaultDatasetId"]).iterate_items():
    print(job["company"], "|", job["title"], "|", job["url"])
```

JavaScript:

```js
import { ApifyClient } from "apify-client";

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });
const run = await client.actor("renzomacar/ats-jobs-search").call({
  companies: ["stripe", "airwallex"],
  maxItems: 200,
});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
console.log(items.length);
```

## Filters

| Field | What it does |
|---|---|
| `keywords` | Words that must all be in the job title. Quote a phrase for an exact match; end a word with `*` to match its beginning. Several entries mean any of them. |
| `excludeKeywords` | Titles to leave out. |
| `locations` | Text that must appear in the job location. |
| `companies` / `excludeCompanies` | Limit to, or skip, the companies you name. |
| `ats` | Limit to some systems: `greenhouse`, `lever`, `ashby`, `workable`, `smartrecruiters`, `personio`, `recruitee`, `teamtailor`, `breezy`, `workday`. |
| `remoteOnly` | Only jobs marked remote. |
| `postedWithinDays` / `postedAfter` | Only jobs published since then. |
| `minSalary`, `salaryCurrency`, `onlyWithSalary` | Filter on the published pay. |
| `onlyNew` | Only jobs not returned by earlier runs of the same search, for a daily new-jobs feed. |
| `includeDescription` | Add the full description text. |
| `maxItems` | 1 to 10,000. |

## What it does not cover

- Companies on iCIMS, BambooHR or a custom careers site. Workday is covered for 880 employers; for any other Workday career site see the [Workday jobs API guide](https://github.com/Perufitlife/workday-jobs-api).
- Pay is only there when the company published it: a minority of jobs. Workday, Workable, SmartRecruiters and Personio publish no pay field.
- The index is rebuilt twice a day, so a job opened an hour ago may not be in yet.

## Price

Pay per result on Apify: $3 per 1,000 jobs returned. A search that returns nothing costs nothing.
