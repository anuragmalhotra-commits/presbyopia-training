# Presbyopia Screening Training — Pre/Post Test

Knowledge assessment forms used before and after presbyopia screening training
for community health workers (ASHAs). Built for The Nudge Institute.

**Live:** https://<org>.github.io/<repo>/

| Page | Path | Purpose |
|---|---|---|
| Landing | `/` | Choose pre-test or post-test |
| Pre-test | `/pre/` | Filled **before** the training. Shows no score and no answers. |
| Post-test | `/post/` | Filled **after** the training. Shows score and an answer review. |

---

## How it works

Three static HTML files. No build step, no framework, no server. Each file is
completely self-contained — all CSS and JavaScript is inline, so a page works
as soon as it loads and keeps working if the connection drops mid-test.

On submission the form POSTs JSON to a Google Apps Script Web App, which writes
a row to a Google Sheet. Hosting is therefore interchangeable: the forms don't
care whether they're served from GitHub Pages, Cloudflare, or a USB stick.

```
Participant's phone  ──POST JSON──>  Apps Script  ──appendRow──>  Google Sheet
     (this repo)                     (Web App URL)                (Pre Test / Post Test tabs)
```

---

## Languages

Five: Hindi, English, Marathi, Odia, Khasi. Selecting a state sets the language
automatically (Maharashtra → Marathi, Odisha → Odia, Meghalaya → Khasi,
Haryana → Hindi), and it can be changed manually at any point before the test
starts.

District and block names are translated too, but the Google Sheet always
records the **English** name. This is deliberate: if the sheet stored whatever
the participant saw, the same block would arrive as several different strings
and the analysis would fragment.

> **Open item:** the Khasi translations have not been reviewed by a native
> speaker. This should happen before Meghalaya goes live.

---

## Pre/post matching

The join key between the two tests is the participant's mobile number,
normalised to 10 digits (`+91`, leading `0`, spaces and dashes stripped). It is
written to the sheet as `Participant ID`.

A number is used rather than a name because names don't survive being typed
twice in two languages, days apart. To pair the tests, match `Participant ID`
across the `Pre Test` and `Post Test` tabs.

---

## Making changes

Everything intended for editing sits at the top of each HTML file, inside the
`<script>` block, in three clearly marked sections:

1. **Geography** — `STATES`, `DISTRICTS`, `BLOCKS`
2. **Interface text** — the `UI` object, one entry per language
3. **Questions** — the `QUESTIONS` array, with `correct` holding the index of
   the right answer

`pre/index.html` and `post/index.html` share the same code and differ only in
the `QUIZ_TYPE` constant. **A change to the questions must be made in both
files** or the two tests will diverge and the scores won't be comparable.

Each page shows a small build stamp in the footer (e.g. `v2026-08-26 08:09`).
After deploying, check that stamp on a phone to confirm the new version is
actually live rather than a cached copy.

### Adding blocks for a new district

```js
const BLOCKS = {
  "MH|Wardha": [
    { en: "Arvi", hi: "आर्वी", mr: "आर्वी", or: "ଆର୍ଭି", kha: "Arvi" },
    // ...
  ],
};
```

The key is `"STATECODE|<English district name>"` and must match the district's
`en` value exactly. A district with no entry here shows a free-text box instead
of a dropdown, so nobody ever gets stuck in the field.

---

## Deploying

GitHub Pages serves this repo directly. Push to `main` and the site updates
within a minute or two. Settings → Pages → Deploy from a branch → `main` →
`/ (root)`.

---

## Backend

The Apps Script lives in the Google Sheet (Extensions → Apps Script), not in
this repo. After editing it you must redeploy: **Deploy → Manage deployments →
edit → Version: New version**. Skipping this leaves the old code running at the
same URL.

Sheet columns:

```
Timestamp | State | District | Block | Language | Name | Phone |
Participant ID | Education | Score | Total | Score % |
Q1 … Q15 | A1 … A15 | Submission ID
```

- **Q1–Q15** — `1` correct, `0` incorrect
- **A1–A15** — which option was chosen (`a`, `b`, `c`…), comma-separated for
  multi-select questions. This is what makes distractor analysis possible.
- **Submission ID** — used to reject duplicate retries

### Offline submissions

Every submission is written to the device's local storage before any network
call is attempted, with a unique `submissionId`. If the send fails the item
stays queued and is retried on page load, when the browser regains
connectivity, and every 30 seconds while anything is pending.

While something is queued the page says so plainly — it does **not** claim the
response was recorded — and shows a pending count so a trainer knows before
everyone leaves the room.

The `submissionId` lets the Apps Script ignore duplicates, so a retry after an
ambiguous send can never create a second row. If local storage is unavailable
(private browsing) the form sends directly instead and reports failure
honestly.

---

## Open items

- [ ] Khasi translations reviewed by a native speaker
- [ ] Question 2 — confirm whether the correct answer is 35 or 40 years
- [ ] Block lists for Odisha and Meghalaya programme districts
- [ ] Marathi/Hindi renderings of Odisha district names are unreviewed
