# Job Platform Search Guide

## Seek (seek.com.au)
Best for: volume of Australian listings, Seek intern/grad filter

Search URL pattern:
```
https://www.seek.com.au/{role}-jobs/in-{location}?daterange=14&workarrangement=remote,hybrid
```

Query tips:
- Use "graduate" OR "intern" OR "junior" — companies use these interchangeably
- Filter: Classification → Information & Communication Technology
- Sort by: Date (newest first)
- Set date range to last 14 days to avoid stale listings

Sample searches:
- `software engineer internship Melbourne`
- `data engineer intern Sydney`
- `devops intern Australia remote`
- `machine learning intern graduate`

---

## GradConnection (gradconnection.com.au)
Best for: structured grad/intern programs at big companies (Big 4 tech, banks, consulting)

Search URL pattern:
```
https://au.gradconnection.com/internships/information-technology/
```

Query tips:
- Filter by: Discipline → Engineering / IT / Computer Science
- Filter by: Program type → Internship, Vacation Program
- Many programs open March–May for Summer intake (Nov–Feb start)
- Check application deadlines carefully — many close months before start date

Key companies to check directly:
- Atlassian, Canva, Afterpay/Block, REA Group, Seek (the company), Xero, WiseTech Global
- Big 4: Deloitte, PwC, KPMG, EY (tech/digital arms)
- Big banks: ANZ, NAB, CBA, Westpac (tech divisions)

---

## LinkedIn Jobs
Best for: networking signals, startup roles, international companies with AU offices

Search URL pattern:
```
https://www.linkedin.com/jobs/search/?keywords={role}+intern&location=Australia&f_E=1
```
(f_E=1 = Entry level / Internship)

Query tips:
- Use "Easy Apply" filter to find quick applications
- Set alert for "Software Engineer Intern Australia" to get email notifications
- Check company pages directly for CS internship postings not on job boards
- Connect with recruiters on LinkedIn after applying — mention the role

---

## Indeed Australia (indeed.com.au)
Best for: smaller companies, startups, roles not on Seek

Search URL pattern:
```
https://au.indeed.com/jobs?q={role}+intern&l={city}&fromage=14
```

Query tips:
- `fromage=14` = past 14 days
- Use advanced search: "intern" OR "internship" OR "graduate" in title
- Indeed aggregates from company sites — click through to original posting to apply

---

## Aus Internship Finder
URL: https://aus-internship-finder.vercel.app/
Raw data: https://raw.githubusercontent.com/YangS1718/aus-internship-finder/main/asset.json

**Company discovery only — not a source of truth for timing or deadlines.**
169 structured programs at top Australian firms. NOT a live job board. NOT indexed (single-page app).

How to use:
- Use only to discover which companies run intern/grad programs
- Do NOT rely on it for application dates — fetch those live from Seek, GradConnection, LinkedIn, or the company's career page directly
- Search pattern for each company: "{company_name}" careers internship {year}

Do NOT search: site:aus-internship-finder.vercel.app — not indexed.

---

## Company Career Pages (direct)
Check these directly — many don't post everywhere:

| Company | Career Page |
|---------|-------------|
| Atlassian | atlassian.com/company/careers |
| Canva | canva.com/careers |
| Afterpay/Block | careers.block.xyz |
| REA Group | rea-group.com/careers |
| Xero | xero.com/about/careers |
| WiseTech Global | wisetechglobal.com/careers |
| Envato | careers.envato.com |
| MYOB | myob.com/au/about/careers |
| Seek (company) | seek.com.au/work-for-seek |
| SafetyCulture | safetyculture.com/about/careers |
| Airtasker | airtasker.com/careers |
| Buildkite | buildkite.com/careers |
| Rokt | rokt.com/careers |
| Deputy | deputy.com/careers |

**Government / Research:**
- data61.csiro.au (CSIRO Data61 internships)
- defence.gov.au (DST Group)
- ato.gov.au/careers
- service.nsw.gov.au/careers

---

## Timing Guide (Australian Internship Cycle)

| Intake | Program Period | Applications Open | Applications Close |
|--------|---------------|-------------------|-------------------|
| Summer (main) | Nov – Feb | March – June | June – August |
| Winter | Jun – Jul | March – April | April – May |
| Semester-based | Flexible | Rolling | Rolling |

> ⚠️ Big company summer programs (Atlassian, Canva, Big 4) often close 4–6 months before the start date. Check early.
