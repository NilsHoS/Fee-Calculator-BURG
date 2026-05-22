# BURG-Apps — Projectcontext voor Claude Code

## Wat is dit project?
Multi-tool recruitment portal voor BURG. Gebouwd door Max (data/backend) en Nils (frontend/app).
Gehost op: https://nilshos.github.io/BURG-Apps/
Lokale repo: C:\Users\MaxvanLeeuwenBURGBed\BURG-Apps

## Stack
- **Frontend**: GitHub Pages (HTML/CSS/JS)
- **Database**: Supabase (project: burg-jobs, id: ziwqshuabwcthqjspuso, regio: eu-west-1)
- **Automatisering**: n8n (mvl1009.app.n8n.cloud)
- **Scraping**: Apify (LinkedIn, Indeed, Werkzoeken.nl, bedrijfswebsites)
- **Branch workflow**: Max werkt op `feature/database-koppeling`, Nils reviewt en merget via PR

## Pagina's in de app
- `vacatures.html` — overzicht nieuwe vacatures uit Supabase
- `pipeline.html` — goedgekeurde vacatures met recruiter assignment en statusbeheer
- `swipe` — swipe interface per medewerker (achter inlogscherm per medewerker)
- `fee-calculator` — fee berekening tool
- `placement-distribution` — plaatsing distributie tool

---

## Supabase structuur

### Tabel: `jobs` (hoofdtabel, ~165 rijen)
| Kolom | Type | Toelichting |
|---|---|---|
| id | uuid | primary key |
| job_title | text | |
| company_name | text | |
| job_location | text | |
| job_url | text | |
| job_description | text | volledige vacaturetekst |
| data_source | text | 'linkedin' / 'indeed' / 'werkzoeken.nl' / 'bedrijfswebsite' |
| review_status | text | default 'pending' — swipe beslissing |
| review_notes | text | |
| reviewed_by | text | welke recruiter |
| reviewed_at | timestamptz | |
| nogo_reason | text | reden van no-go |
| seniority_level | text | nu altijd 'medior' — nog niet via LLM bepaald |
| seniority_reviewed_at | timestamptz | |
| sales_status | text | |
| sales_notes | text | |
| assigned_to | text | |
| contact_name | text | |
| contact_email | text | |
| contact_phone | text | |
| contact_title | text | |
| contact_linkedin | text | |
| contact_raw | text | |
| recruiter_name | text | LinkedIn recruiter |
| recruiter_linkedin | text | LinkedIn recruiter URL |
| recruiter_headline | text | |
| company_website | text | |
| company_linkedin | text | |
| company_address | text | |
| company_industry | text | |
| company_name_display | text | |
| enriched_at | timestamptz | Apollo enrichment timestamp |
| employment_type | text | |
| industry | text | |
| salary | text | |
| posted_at | timestamptz | |
| date_scraped | timestamptz | |
| created_at | timestamptz | default now() |
| company_name_normalized | text | voor dedup |
| job_title_normalized | text | voor dedup |

### Tabel: `geziene_vacatures` (dedup tabel)
Kolommen: `job_title_normalized`, `company_name_normalized`, `scraped_at`
Wordt gecheckt via "If row does not exist" node in n8n vóór elke insert in jobs.

### Tabel: `employees` (6 rijen)
Kolommen: `id`, `name`, `email`, `fte_hours`, `seniority_levels`, `is_present`

---

## n8n Workflows

### Workflow 1: "Apify Workflow 1" (id: R3Ky6mByr4CRooNC) ✅ ACTIEF
**Flow:**
```
Apify Trigger (LinkedIn/Indeed/Werkzoeken) 
  → Get dataset items 
  → normalisatie_en_filtering (Code node) 
  → If row does not exist (geziene_vacatures check) 
  → Create a row (jobs tabel) 
  → Insert row (geziene_vacatures)
```

**Apify actors:**
- LinkedIn: `gdbRh93zn42kBYDyS` (curious_coder/linkedin-jobs-search-scraper)
- Indeed: `qA8rz8tR61HdkfTBL` (curious_coder/indeed-scraper)
- Werkzoeken.nl: `j4z8q58J839Zr6uJx` (blackfalcondata/werkzoeken-scraper)
- Bedrijfswebsites: **nog toe te voegen** (Website Content Crawler, maxCrawlDepth: 1)

**normalisatie_en_filtering code node — wat hij doet:**
1. `detectSource(j)` — bepaalt linkedin / indeed / werkzoeken.nl op basis van veldnamen
2. `normalizeItem(j)` — mapt ruwe Apify output naar Supabase velden per source
3. Filtert op QHSSE keywords in jobtitel (uitgebreide lijst aanwezig)
4. Filtert op excludedCompanies (recruitmentbureaus, uitzendbureaus — grote lijst aanwezig)
5. Filtert op excludedIndustries (staffing, HR, executive search)
6. Filtert op excludedTitleWords (stage, intern, stagiair)
7. Deduplicatie binnen dezelfde batch op job_url of titel+bedrijf combo
8. Output: genormaliseerde items met job_title_normalized en company_name_normalized

**NOG TOE TE VOEGEN aan normalisatie_en_filtering:**
- Source detectie: `if (j.crawl && j.metadata) return 'bedrijfswebsite';`
- Normalisatie blok voor bedrijfswebsite:
  - Sla depth 0 items over (overzichtspagina's)
  - Jobtitel: `metadata.title` splitsen op ' - ', laatste deel = bedrijfsnaam
  - Locatie: `metadata.jsonLd[0].address.addressLocality`
  - Beschrijving: `text` veld
  - Contactgegevens: via bestaande `extractContact()` functie op `text` veld

### Workflow 2: "Apollo Enrichment" (id: z297XGfx0k31sB5a) ✅ ACTIEF
**Flow:**
```
Webhook (POST) 
  → Get a row (jobs tabel op job_id) 
  → If (recruiter_linkedin niet leeg) 
  → HTTP Request (Apollo API: people/match op linkedin_url) 
  → Update a row (contact_email, contact_phone, enriched_at)
```
Apollo gebruikt `recruiter_linkedin` als lookup key.
Webhook URL: `https://mvl1009.app.n8n.cloud/webhook/d9c398f5-d635-4fd2-b4ce-231b3bc087fe`

---

## Wat werkt al ✅
- LinkedIn, Indeed, Werkzoeken.nl scraping actief en dagelijks
- Normalisatie + QHSSE keyword filter + excludedCompanies filter + dedup
- Supabase insert via n8n
- Swipe interface per medewerker (achter inlogscherm)
- Pipeline.html met recruiter assignment en statusbeheer
- Apollo enrichment workflow via webhook
- Volledige Supabase structuur aanwezig

## Wat nog moet gebeuren 🔧

### 1. Bedrijfswebsite scraping toevoegen (prioriteit)
- Nieuwe Apify Trigger node in Workflow 1 voor Website Content Crawler
- 207 gewone website URLs klaar om als startUrls in te voeren
- 26 ATS URLs (Workday x4, Recruitee x4, etc.) apart aanpakken later
- normalisatie_en_filtering code node uitbreiden met bedrijfswebsite blok
- Schedule: 2x per maand (niet dagelijks)

### 2. Anthropic API koppelen aan n8n (via bedrijfsaccount)
- LLM-based QHSSE classificatie ter vervanging van keyword filter
- Automatische seniority bepaling (nu krijgt alles 'medior')
- Model: claude-haiku voor kostenefficiëntie (~$0.001 per vacature)

### 3. Volledig automatische go/no-go (langetermijndoel)
- LLM beoordeelt vacatures op basis van historische swipe data in jobs tabel
- Elke swipe nu = trainingsdata (review_status + nogo_reason zijn al aanwezig)
- Swipe scherm wordt optioneel zodra model goed genoeg is

---

## Werkwijze
- Max werkt in `C:\Users\MaxvanLeeuwenBURGBed\BURG-Apps`
- Branch: `feature/database-koppeling`
- Wijzigingen via PR naar main — Nils reviewt en merget
- Claude Code starten: `cd C:\Users\MaxvanLeeuwenBURGBed\BURG-Apps` → `claude`
- Nils werkt aan frontend/app features via zijn eigen multi-agent Claude Code setup

---

## Instructie voor Claude Code: CLAUDE.md bijwerken

Werk aan het einde van elke werksessie de CLAUDE.md bij op basis van wat er gedaan is:
- Zet ✅ achter voltooide stappen
- Voeg nieuwe Supabase kolommen toe als die zijn aangemaakt
- Voeg nieuwe n8n workflows of nodes toe als die zijn gebouwd
- Verplaats afgeronde taken van "Wat nog moet gebeuren" naar "Wat werkt al"
- Commit de bijgewerkte CLAUDE.md mee met je andere wijzigingen

Doe dit altijd zonder dat Max erom hoeft te vragen.
