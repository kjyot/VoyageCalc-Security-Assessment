# VoyageCalc Security Assessment

This repository contains the sanitized report and evidence summary from a black-box security assessment of the VoyageCalc staging application.

## Files

- `REPORT.md` — the assessment report in Markdown
- `evidence-register.md` — a short list of the evidence reviewed

## Assessment scope

- Target: `https://staging.nauticalmove.com`
- Testing type: black-box; no source-code access
- Account coverage: one USER-role staging account
- Testing was limited to authorized staging activity and read-only requests.

## Before sharing

This package does not include passwords, Bearer tokens, cookies, MFA setup QR codes or secrets, customer data, or raw browser Network exports. Check any additional screenshots or evidence separately before adding them.

The results describe the tests completed on the staging environment. They are not a full penetration test or a security certification.
