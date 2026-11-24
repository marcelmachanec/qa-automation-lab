# QA Automation Lab

[![Bug Check](https://github.com/marcelmachanec/qa-automation-lab/actions/workflows/bugcheck.yml/badge.svg)](https://github.com/marcelmachanec/qa-automation-lab/actions/workflows/bugcheck.yml)
[![Checks](https://github.com/marcelmachanec/qa-automation-lab/actions/workflows/checks.yml/badge.svg)](https://github.com/marcelmachanec/qa-automation-lab/actions/workflows/checks.yml)

**Robot Framework test suite that hunts real, seeded bugs in a full web shop — and scores itself.**

The target is [Practice Software Testing](https://github.com/testsmith-io/practice-software-testing)
(Laravel API + Angular UI). Its `sprint5-with-bugs` build ships with 94 documented bugs.
Every week CI starts that build in Docker, runs this suite against it, and publishes a
scorecard of which known bugs the tests caught.

## Latest scorecard (CI run, 2026-11-23)

| Suite | Targeted bugs | Caught |
|---|---|---|
| API (security / OWASP) | 11 | **10** |
| UI (storefront) | 6 | **6** |
| **Total** | **17** | **16** |

Examples of what the suite catches:

- **SQL injection** on `users/login` that signs you in as another user
- **Broken object-level authorization**: reading another customer's profile or invoices
- **Missing admin check**: a customer token can delete a brand
- **Data exposure**: internal fields in API responses, `laravel.log` served over the web
- **Weak tokens**: access token valid for 260,000 minutes
- **UI defects**: reversed sort order, wrong nav links, typos in labels

Still missed: lockout after one failed login (bug 21). That is the next test to write.

## What this project shows

- **API testing with a custom keyword library**: `PstApi.py` (one HTTP session, bearer tokens)
  wrapped in Robot keywords, plus offline tests of that library against a mock API.
- **Security-minded test design** based on the OWASP API Top 10. Every bug test is tagged
  `bug:<id>`, so a failing test maps straight to a known defect.
- **UI automation** with the Browser library (Playwright engine) and page objects.
- **Test effectiveness, not only pass/fail**: `scripts/bug_scorecard.py` reads Robot's
  `output.xml` and the app's bug list and reports caught / missed / not run.
- **CI on GitHub Actions**: lint + offline suite on every PR and push to `main`, smoke tests on PRs, and a
  weekly full bug check against the Dockerised app.
- **AI-assisted, human-reviewed workflow**: most changes were drafted by coding agents
  (Claude Code, Codex) in PRs and reviewed before merge. Agent rules live in `AGENTS.md`.

## Run it

```bash
pip install -r requirements.txt
rfbrowser init                     # once, for UI tests

# start the app with seeded bugs (Docker, first run takes a few minutes)
SPRINT=sprint5-with-bugs WAIT_FOR_UI=true scripts/start_app.sh

# API + security suite
robot -d output -i smoke --nostatusrc \
  --variable API_BASE_URL:http://localhost:8091 --variable REQUIRE_API:true tests

# UI suite
robot -d output --variable BASE_URL:http://localhost:4200 \
  --variable API_BASE_URL:http://localhost:8091 tests/ui

# scorecard
python scripts/bug_scorecard.py output/output.xml \
  app/practice-software-testing/sprint5-with-bugs/testSessions/listOfBugs.md
```

Offline only (no app needed): `robot tests/helpers` · Lint: `ruff check .`

## Layout

| Path | Contents |
|---|---|
| `tests/smoke/` | API suites: login, login security, authorization, data exposure |
| `tests/ui/` | UI suites: login, storefront bugs |
| `tests/helpers/` | Offline tests of the keyword library against a mock API |
| `resources/api/` | `PstApi.py` keyword library and shared login keywords |
| `resources/pages/` | Page objects and shared browser keywords |
| `scripts/` | App start-up, bug scorecard, Windows run scripts |
| `app/` | The application under test (git submodule) |

Czech version: [README.cs.md](README.cs.md)

## License

MIT
