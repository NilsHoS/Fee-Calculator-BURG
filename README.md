# BURG Tools & Apps

Internal tooling for the BURG QHSSE recruitment and sales team. All tools run as a client-side web app — no backend, no login required.

**Live:** [https://nilshos.github.io/BURG-Apps/](https://nilshos.github.io/BURG-Apps/)

---

## Tools overview

| Tool | Function | For |
|---|---|---|
| Fee Checker | Estimate the recruitment fee before a client call | Sales consultants |
| Sales Overdracht | Hand off a vacancy from sales to a consultant via email | Sales consultants |
| Definitief Honorarium | Calculate the final fee at confirmed placement | Recruiters, sales |
| Pipeline | Manage approved vacancies and assign them to recruiters | Team leads |
| Sales Dashboard | Track leads and manage sales status per vacancy | Sales consultants |
| Dashboard | Overview of vacancies, sales and conversion | Management |
| Verdeling Plaatsing | Split the placement fee between sales officer, recruiter and consultant | Team leads |
| Proeftijd Tracker | Track the probationary period of newly placed candidates | Recruiters |

---

## Tool details

### Fee Checker (`calculator.html`)

Estimates the recruitment fee before a client conversation. Uses sliders and toggle buttons — no typing required.

**Inputs**
- Monthly gross salary (slider: €2,000 – €12,000)
- Number of salary periods (12, 13 or 14)
- Holiday pay percentage (slider: 0 – 8%)
- Company car (toggle: adds €6,000 to annual salary)
- Bonus percentage (slider: 0 – 10%, calculated over base salary × periods, holiday pay excluded)
- Fee percentage (buttons: 21 – 26%)

**Output**
- Estimated monthly salary, gross annual salary, fee amount and fee percentage
- Go / No-go verdict: fee below €13,500 = no-go

**Formula**
```
annual = (monthly × periods × (1 + holiday%)) + (monthly × periods × bonus%) + car
fee    = annual × fee%
```

---

### Sales Overdracht (`jobpull.html`)

Structured handoff form from sales to a consultant. On submit, the form is sent as an email via EmailJS to the configured recipients.

**Inputs**
- Betrokkenen: sales name, company name, job title, estimated seniority
- Gesprekspartners: contact person 1 and 2 (name + role)
- Afspraak: date, time (drum picker), type (Teams / on-site), address and car reservation if on-site
- Info over de job: job description, how long they have been searching, other agencies involved, candidates in process, salary indication
- Voorwaarden: fee percentage and exclusivity (shown conditionally)
- Optional notes

**Output**
Email sent to all recipients in `ONTVANGERS[]`. The template is configured in EmailJS.

**Configuration** — edit the constants at the top of `jobpull.html`:
```js
const EMAILJS_PUBLIC_KEY  = '...';
const EMAILJS_SERVICE_ID  = '...';
const EMAILJS_TEMPLATE_ID = '...';
const ONTVANGERS = ['email@example.com'];
```

---

### Definitief Honorarium (`plaatsing.html`)

Calculates the final placement fee once salary and agreed fee percentage are known. Requires typed input for both mandatory fields.

**Inputs**
- Monthly gross salary (text field, Dutch notation: `5.250` or `5250`)
- Number of salary periods (12, 13 or 14)
- Holiday pay percentage (slider: 0 – 8%)
- Company car (toggle: adds €6,000)
- Bonus percentage (slider: 0 – 10%)
- Fee percentage (text field, e.g. `23`)

**Output**
- Gross annual salary, fee percentage, final honorarium
- Go / No-go verdict against the €13,500 minimum

**Formula** — identical to Fee Checker, with exact typed inputs instead of sliders.

---

### Verdeling Plaatsing (`plaatsing-verdeling.html`)

Splits the deal value between the three parties involved in a placement.

**Fixed split**
| Role | Share |
|---|---|
| Sales officer | 30% |
| Recruiter | 30% |
| Consultant | 40% |

**Onboarding rule** — if a person is in onboarding, 20% of their share is redirected:
- Sales officer in onboarding → 20% to Head of Sales
- Recruiter or consultant in onboarding → 20% to Head of Recruitment

**Advance payment (aanbetaling)** — optional, choose €2,500 / €5,000 / €10,000. Split 50% sales officer, 50% consultant. The recruiter receives nothing from the advance.

**Inputs**
- Deal value (excl. VAT)
- Advance payment toggle + amount selection
- Names of sales officer, recruiter and consultant
- Onboarding toggle per person

**Output**
Per person: gross share, net share after onboarding deduction, amount already received via advance payment, amount still to be paid. Plus totals.

---

### Proeftijd Tracker (`proeftijd.html`)

Tracks the probationary period for up to 6 placed candidates simultaneously. State is saved in `localStorage` — data persists after closing the browser.

**Inputs (per candidate via modal)**
- Candidate name
- Start date of probationary period (day / month / year selects)
- Duration: 1, 2 or 3 months

**Output**
- Split-flap countdown display per candidate showing days (D), hours (U) and minutes (M) remaining
- Progress bar from start to end date
- "GESLAAGD" (passed) state when the period ends
- Updates every 10 seconds

---

### Pipeline (`pipeline.html`)

Manage approved vacancies and assign them to recruiters. Data is stored locally.

---

### Sales Dashboard (`sales.html`)

Track leads and manage the sales status per vacancy.

---

### Dashboard (`dashboard.html`)

Overview of vacancies, sales pipeline and conversion statistics.

---

## Technical

- **Stack:** plain HTML, CSS, vanilla JavaScript — no frameworks, no build step
- **Hosting:** GitHub Pages (`gh-pages` branch or `main` depending on repo settings)
- **Storage:** `localStorage` only (Proeftijd Tracker). All other tools are stateless.
- **Email:** Sales Overdracht uses [EmailJS](https://www.emailjs.com) (client-side, no server needed)
- **PWA:** `manifest.json` + Apple meta tags — can be installed as a home screen app on iOS/Android
- **Offline:** no service worker; requires an internet connection for the EmailJS call in Sales Overdracht
- **Dark mode:** all tools support `prefers-color-scheme: dark` automatically

### File structure

```
index.html              — tool menu
calculator.html         — Fee Checker
jobpull.html            — Sales Overdracht
plaatsing.html          — Definitief Honorarium
plaatsing-verdeling.html — Verdeling Plaatsing
proeftijd.html          — Proeftijd Tracker
pipeline.html           — Pipeline
sales.html              — Sales Dashboard
dashboard.html          — Dashboard
styles.css              — shared design system
manifest.json           — PWA manifest
handleiding.html        — user guide (Dutch)
```

### Adding a recipient to Sales Overdracht

Open `jobpull.html` and add an email address to the `ONTVANGERS` array:

```js
const ONTVANGERS = [
  'existing@burgqhsse.nl',
  'new@burgqhsse.nl',
];
```

Commit and push — GitHub Pages will deploy within a minute.

---

BURG QHSSE · 2026
