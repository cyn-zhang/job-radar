# Job Platform Search Guide

Search patterns are dynamic — substitute values from `config.yaml` using the mappings below.

---

## Config → URL mapping

### work_type → search keyword

| config `work_type` | Keyword to append |
|--------------------|-------------------|
| `internship` | `intern` OR `internship` |
| `graduate` | `graduate` OR `grad program` |
| `contract` | `contract` OR `fixed-term` |
| `permanent` | `full-time` OR `permanent` |
| `any` | *(omit)* |

### industry → GradConnection discipline slug

`industry` is a list — search each discipline slug separately when multiple industries are configured.

| config `industry` | GradConnection slug |
|-------------------|---------------------|
| `tech` | `information-technology` |
| `finance` | `banking-finance` |
| `accounting` | `accounting-finance` |
| `consulting` | `management-consulting` |
| `healthcare` | `healthcare-medical` |
| `government` | `government-defence` |
| `legal` | `legal` |
| `education` | `education-training` |
| `energy` | `engineering` |
| `retail` | `retail-consumer-products` |
| `marketing` | `marketing-communications` |
| `hr` | `human-resources-recruitment` |
| `construction` | `construction` |
| `manufacturing` | `transport-logistics` |
| `any` | *(omit discipline — browse all)* |

### work_type → GradConnection program slug

| config `work_type` | GradConnection slug |
|--------------------|---------------------|
| `internship` | `internships` |
| `graduate` | `graduate-jobs` |
| `contract` / `permanent` / `any` | `jobs` |

### level → LinkedIn experience filter (f_E)

| config `level` | LinkedIn `f_E` |
|----------------|----------------|
| `intern` | `1` |
| `graduate` / `junior` | `1,2` |
| `mid` | `3,4` |
| `senior` / `lead` | `4` |
| `manager` | `5` |
| `director` / `vp` / `executive` | `5,6` |

---

## Seek (seek.com.au)

Best for: volume of Australian listings, built-in work type filters.

Search URL pattern:
```
https://www.seek.com.au/{role-slug}-jobs/in-{location}?daterange=14
```

Query tips:
- Append `{work_type keyword}` to role query (e.g. "software engineer intern")
- Filter: Classification → matches your `industry` config
- Sort by: Date (newest first)
- Date range 14 days avoids stale listings

---

## GradConnection (au.gradconnection.com)

Best for: structured grad/intern programs at big companies (Big 4 tech, banks, consulting).

Search URL pattern:
```
https://au.gradconnection.com/{work_type slug}/{discipline slug}/
```

Example (work_type=internship, industry=tech):
```
https://au.gradconnection.com/internships/information-technology/
```

Example (work_type=graduate, industry=finance):
```
https://au.gradconnection.com/graduate-jobs/banking-finance/
```

Query tips:
- Many programs open March–May for Summer intake (Nov–Feb start)
- Check application deadlines carefully — many close months before start date

---

## LinkedIn Jobs

Best for: networking signals, startup roles, international companies with AU offices.

Search URL pattern:
```
https://www.linkedin.com/jobs/search/?keywords={role}&location={location}&f_E={level filter}
```

Example (level=intern, role=Software Engineer, location=Australia):
```
https://www.linkedin.com/jobs/search/?keywords=Software+Engineer+intern&location=Australia&f_E=1
```

Query tips:
- Set alert for your role + location to get email notifications
- Check company pages directly for roles not surfaced by search
- Connect with recruiters on LinkedIn after applying — mention the role

---

## Indeed Australia (indeed.com.au)

Best for: smaller companies, startups, roles not on Seek.

Search URL pattern:
```
https://au.indeed.com/jobs?q={role}+{work_type keyword}&l={city}&fromage=14
```

Example (work_type=internship, role=Data Engineer, location=Melbourne):
```
https://au.indeed.com/jobs?q=data+engineer+intern&l=Melbourne&fromage=14
```

Query tips:
- `fromage=14` = past 14 days
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
- Search pattern for each company: `"{company_name}" careers {work_type keyword} {year}`

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
