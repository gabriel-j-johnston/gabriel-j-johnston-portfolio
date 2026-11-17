# Portfolio A/B test: portfolio_redesign_v1

Version A is the original site. Version B is the field-guide redesign. Both send data to the same
Google Analytics 4 property (G-EEBJ189DWL), and every event is tagged with the version the visitor saw.

## Files

| Path | What it is |
|---|---|
| `index.html` | The split page. Sends each new visitor to `a/` or `b/` at random (50/50), and keeps them on the same version on later visits from that browser. |
| `a/index.html` | Your original site, unchanged apart from the tracking block in `<head>`. |
| `b/index.html` | The redesign, unchanged apart from the tracking block in `<head>`. |

Put all three in the root of the GitHub Pages repo, replacing the current `index.html`. Old links such as
`https://gabriel-j-johnston.github.io/gabriel-j-johnston-portfolio/#case-equinix` still work: the split page passes the `#case-…` part through.

## Links to use

- `https://gabriel-j-johnston.github.io/gabriel-j-johnston-portfolio/` is the normal link for applications, LinkedIn, and your email signature. It is randomized.
- `https://gabriel-j-johnston.github.io/gabriel-j-johnston-portfolio/?me=1` should be opened once in every browser you use. It marks your own visits as internal so they
  can be filtered out. `?me=0` undoes it.
- `https://gabriel-j-johnston.github.io/gabriel-j-johnston-portfolio/?v=a` or `?v=b` forces a version, for showing someone a specific design. These visits are tagged
  `exp_assignment = forced`; leave them out of the analysis.
- UTM tags work as usual, for example `https://gabriel-j-johnston.github.io/gabriel-j-johnston-portfolio/?utm_source=linkedin&utm_medium=post`.

## What gets recorded

GA4 already records users, sessions, engagement time, and engagement rate. The tracking block adds these events:

| Event | When it fires | Parameters |
|---|---|---|
| `case_open` | A case study is opened | `case_id` (equinix, nvidia, ano) |
| `route_time` | The visitor leaves a view or hides the tab. Counts visible time only. | `route` (home or case-…), `seconds` |
| `section_view` | A section reaches the middle of the screen, once per view | `route`, `section` (work, about, question, findings, q-result…) |
| `contact_action` | Email copied, LinkedIn opened, resume opened, or resume PDF opened | `method` |
| `outbound_click` | The DOI link is clicked | `target` |
| `case_mode` | Quick read or full case study is chosen (both sites already sent this) | `mode`, `case_id` |
| `page_view` | Each case study and the resume (both sites already sent this) | |

Every event also carries `exp_variant` (a or b) and `exp_assignment` (random, forced, direct_link).

## One-time setup in Google Analytics (about 10 minutes)

1. **Custom dimensions** (Admin → Custom definitions). Add an event-scoped dimension for each of
   `exp_variant`, `exp_assignment`, `route`, `section`, `method`, `case_id`, and `mode`. Add one user-scoped dimension
   for `exp_variant`. Then add an event-scoped custom metric for `seconds`, with the unit set to seconds.
2. **Key events** (Admin → Key events). Mark `contact_action` and `case_open` as key events.
3. **Exclude yourself** (Admin → Data filters). Set the "Internal traffic" filter to Active. It drops events
   tagged `traffic_type = internal`, which is what `?me=1` adds.
4. **Avoid double page views** (Admin → Data streams → your stream → Enhanced measurement → Page views →
   Advanced). Turn off "Page changes based on browser history events". Both sites already send their own page
   views for case studies and the resume.
5. **Optional, recommended for the stats:** link GA4 to BigQuery (Admin → BigQuery links) with daily export.
   The GA4 interface only shows totals. BigQuery gives one row per event, which lets you run significance tests
   per visitor.

Custom dimensions only collect data from the day you create them, so set them up before you switch the
split on.

## Suggested test plan

Pick the primary metric before you launch, and don't stop the test early because one version looks ahead.

- **Primary metric.** Case-study open rate: the share of sessions that open at least one case study. This is
  the most direct measure of whether the design gets people to the work.
- **Secondary metrics.** Contact-action rate. Engaged time per session (sum of `route_time` seconds). Reading
  depth: the share of case views that reach `findings`, `impact`, or `q-result`. The quick-read vs. full split.
- **Guardrail.** Engagement rate. Neither version should lose visitors at the door.
- **Sample size.** At 95% confidence and 80% power, detecting a lift in the primary metric from 30% to 40%
  needs about 350 sessions per version. A bigger lift, 30% to 45%, needs about 160 per version. Contact actions
  are rarer, so a 10% to 20% change needs about 200 per version.
- **Analysis.**
  - Rates (opens, contact actions): a two-proportion z-test or chi-square.
  - Time (skewed): a Mann-Whitney U test, or a t-test on log seconds.
  - Report effect sizes with confidence intervals, not only p-values.
  - Use only `exp_assignment = random` sessions from non-internal traffic.
