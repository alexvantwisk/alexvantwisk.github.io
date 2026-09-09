# Personal website upgrade

Spec: docs/superpowers/specs/2026-09-06-personal-website-design.md
Plan: docs/superpowers/plans/2026-09-06-personal-website.md

- [x] Step 0: docs (PR #10)
- [x] Step 1: foundation (Tasks 1–4, PR #11)
- [x] Step 2: consulting moved (Tasks 5–8, PR #12)
- [x] Step 3: home and About (Tasks 9–10, PR #13)
- [x] Step 4: CV (Tasks 11–12, PR #14)
- [x] Step 5: blog (Tasks 13–16, PR #15)
- [x] Step 6a: social card and Netlify parity (Tasks 17–18, PR #16)
- [x] Step 6b: domain cutover (Task 19 A4–A5, Task 20; PR #21 after the draft #17 was closed; live at https://avantwisk.com on 2026-09-07)

## Review (2026-09-06, updated 2026-09-07)

**What shipped.** Eight stacked pull requests into `main`, to be merged in order #10 → #16, with #17 held as a draft until the domain exists. Every PR passed CI (Quarto 1.9.36 render plus an offline lychee link check). The site is now Alexander's personal site: home, About, hand-written CV, a blog with categories, full-text RSS and freeze-based R execution (two starter posts), the consulting pages moved as a unit under `/consulting/` with alias redirects and a subnav, a site-wide privacy notice covering POPIA and UK GDPR, a five-file stylesheet, a personal share card, and Netlify building with the same Quarto version as CI.

**Verified.** Each task was implemented by a fresh subagent and reviewed against its brief before the next started; a final whole-branch review found one Important defect (the site-level share image resolved per page directory), fixed in PR #16 by making the path root-absolute. Alias pages, the RSS feed, search indexing, and the freeze round-trip were all checked in the rendered output. CI on PR #15 proved the blog builds on a runner with no R installed.

**Deliberately left out.** CV PDF download (Alexander's choice; README explains how to add it), dark mode, comments, a Projects page, a separate Talks page, and any link between this repo and the Personal-CV project.

**Deviations from the plan.** Step PRs target `main` and stack on each other rather than waiting for merges; the 404 page is excluded from the link check until cutover; the 404 page's contact link was fixed in Step 2; Netlify config is kept until DNS moves because Netlify is live; Step 6 was split into a mergeable PR and a draft cutover PR.

**Follow-ups completed 2026-09-07.** Repo renamed to `alexvantwisk.github.io` (PR #18 set the root site URL and dropped the 404 link-check exclusion); Netlify unlinked and its config removed (PR #19); the two-business-day reply promise dropped (PR #20); avantwisk.com registered, DNS set at Porkbun, custom domain and HTTPS enforcement set on Pages, CNAME and site URL merged (PR #21). The github.io address 301-redirects to the domain.

**Open for Alexander.** Optionally align "MSc Biostatistics (Cum Laude)" on the consulting landing with "Passed with Distinction" on the CV; both are his own wording.

## Incident 2026-09-09: site served the README instead of the rendered pages

**Symptom.** https://avantwisk.com/ showed the contents of `README.md` — no navbar, no styling, no pages.

**Cause.** The GitHub Pages source had reverted to the legacy branch builder (`build_type: legacy`, source `main` / `/`). That runs Jekyll over the repo root, where there is no `index.html` — only `index.qmd` — so Jekyll rendered `README.md` as the homepage. A `pages-build-deployment` run at 14:44 UTC published it over the good Actions deploy. The `publish.yml` workflow itself was healthy the whole time; its uploaded `_site` artifact was simply being ignored.

**Fix.** Set the source back to GitHub Actions and redeployed:

```
gh api -X PUT repos/alexvantwisk/alexvantwisk.github.io/pages -f build_type=workflow
gh workflow run publish.yml --ref main
```

The custom domain and the HTTPS certificate survived the change untouched.

**Verified.** Run 34366446987 succeeded (render, offline link check, upload, deploy). `/` serves the styled homepage and `/about.html` — a path the Jekyll build cannot produce — resolves.

**Guard.** This breaks silently: `publish.yml` still goes green while the wrong content is served. If the homepage ever shows the README again, check `gh api repos/alexvantwisk/alexvantwisk.github.io/pages` first and confirm `build_type` is `workflow`. Changing the Pages source in the web UI is what flips it.
