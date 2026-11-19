# Security Assessment Report

**VoyageCalc — Staging Environment**  
**Prepared:** 19 November 2026  
**Classification:** Confidential / sanitized copy

## 1. Summary

I tested the VoyageCalc staging application at `https://staging.nauticalmove.com` from a user's point of view. I did not have access to the source code. The checks were carried out through the browser and read-only API requests using one USER-role account.

The main issue I could confirm was that the main page's fresh HTTP 200 response did not include a Strict-Transport-Security (HSTS) header. I have rated this Low because the browser did move from an HTTP address to HTTPS during my check, but I did not capture the actual redirect response.

I also saw several good security controls. MFA was required when I signed in, the Cargo API asked for a Bearer token when accessed without authentication, and the API responses included several protective security headers. After logging out, using the browser Back button showed the sign-in page rather than the Cargo List data.

I did not confirm a critical or high-severity vulnerability during the checks I was able to complete. Some areas still need further testing, especially access between different organizations and roles.

## 2. Scope and How I Tested

- Target: `https://staging.nauticalmove.com` (staging environment only).
- This was a black-box assessment. I used the application in a browser and made a small number of read-only GET requests. No source code was provided.
- I had access to one USER-role account only. I did not have a second account, another organization, or an administrator account to compare permissions.
- I avoided destructive requests, brute-force attempts, service disruption, and changes to sensitive data.
- I explored Overview, Voyage Estimator, Fixtures, Received Fixtures, Cargo List, Oil Flows, Organization Users, Fleet in Service, Scheduled Deliveries, Demolitions, Valuations, and Security.

## 3. Main Finding: HSTS Header Missing

**Finding F-01 | Severity: Low**

**What I observed:** I checked a fresh HTTP 200 response for the main application page. It included Content-Security-Policy, Permissions-Policy, Referrer-Policy, X-Content-Type-Options, and X-Frame-Options. I did not see a Strict-Transport-Security header.

**Why it matters:** HSTS tells browsers to use HTTPS for a site. Without it, a browser that has not previously learned the policy may be more exposed to downgrade or SSL-stripping attacks during an initial connection. This check does not show that the application served sensitive information over HTTP.

**What I recommend:** Add the Strict-Transport-Security header to HTTPS responses at the edge or reverse proxy. Choose the max-age after checking that HTTPS works across the relevant domains. Only add includeSubDomains or request preload after reviewing all affected subdomains.

**Evidence note:** When I entered the HTTP address in the browser, it ended up on HTTPS. I did not capture the raw HTTP response, redirect status, or Location header, so I cannot say from this test alone exactly how the redirect was handled.

## 4. Other Security Checks

- **API authentication:** An unauthenticated Cargo API request returned a message saying a Bearer Authorization token was required. The status code was not recorded, so I have not labelled it as a confirmed 401 or 403.
- **API response headers:** The API responses I inspected included `Cache-Control: no-store`, a restrictive Content-Security-Policy, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, and `Referrer-Policy: no-referrer`.
- **Login messages:** An unknown but correctly formatted email address and the known test account with an incorrect password produced the same `Invalid credentials` message. I did not observe account enumeration in these checks.
- **MFA:** Authenticator-based MFA was enabled and was required during a fresh login. I did not test MFA bypass or account recovery.
- **Logout and browser Back:** After logging out, I pressed the browser Back button. The sign-in page appeared instead of the protected Cargo List content. This does not prove that the server invalidated the token.
- **Search input:** I tried a few special characters and an XSS marker in the Valuations search field. I did not see script execution. These limited checks do not prove that all SQL injection or XSS risks are absent.

## 5. Access to Business Data: Needs Further Checking

While using the USER account, I could view several business datasets, including vessel, cargo, operator, owner, charterer, valuation, and operational information.

I could not determine whether this level of access was correct because I was not given the expected organization or role access rules. I also had no second account from another organization to compare.

For that reason, I am recording this as an area that needs an authorization review. I am not claiming that I found an IDOR, cross-organization data leak, or privilege escalation.

**Recommended follow-up:** The application owner should define which roles can see which records. The team should then test list and individual-record API access with approved test accounts from at least two different organizations.

## 6. Checks That Remain Inconclusive or Were Not Completed

- **Pagination page=0:** The available evidence did not establish how the authenticated API handles this value. One attempt was unauthenticated and returned a message asking for a Bearer token. Another attempt showed HTTP 200, but I did not capture the response body. The result is inconclusive.
- **Page size limit=100:** I observed HTTP 200 for a request using `limit=100`. I did not capture the response body values needed to confirm the effective limit or number of returned records. Maximum page-size enforcement remains unverified.
- **Session handling:** I did not independently verify whether the server invalidates a token immediately on logout. I also did not measure idle or absolute session timeouts.
- Login and MFA rate limits, password lockout, account recovery, MFA bypass, IDOR, cross-organization access, privilege escalation, and comprehensive SQL injection or XSS testing were not completed.
- **HTTP redirect:** The browser ended up on HTTPS after I entered an HTTP address, but I did not capture the raw redirect response.

## 7. Credential and Evidence Handling

During the assessment, Bearer tokens and an MFA enrollment QR/setup secret were accidentally visible in shared screenshots or messages. I later confirmed that the exposed tokens had been revoked or rotated. If the MFA setup secret was active when it was exposed, it should also be reset by re-enrolling MFA.

This repository does not include passwords, Bearer tokens, cookies, MFA setup secrets, or customer data. These items should not be added to a public repository or shared screenshots.

## 8. Recommended Next Steps

1. Add HSTS to HTTPS responses and verify it by checking the raw response headers.
2. Write down the expected role and organization access rules, then test them with approved accounts from different organizations.
3. Verify server-side token invalidation on logout, token rotation, idle timeout, and absolute timeout.
4. Confirm accepted pagination values and enforce a maximum page size on the server.
5. Review login and MFA rate limits, recovery controls, and monitoring using approved, low-impact tests.
6. Carry out further SQL injection and XSS checks using approved test data, and verify parameterized database queries and safe output encoding.

## 9. Conclusion

This assessment found one Low-severity security-hardening issue: the HSTS header was missing from the fresh main-page response I inspected. I also observed several positive controls, including MFA at login, API authentication requirements, and protective response headers.

I did not confirm a critical or high-severity exploitable vulnerability in the checks completed. The biggest remaining question is whether the USER role should be able to access all of the business data visible in the application. That cannot be answered without the intended access policy and additional test accounts.

The results apply only to the staging environment and the tests described in this report. This was not a full penetration test, and it should not be treated as a security certification.

## Appendix: Evidence List

- E-01: Fresh main-page HTTP 200 response headers; HSTS was not present.
- E-02: Unauthenticated Cargo API request returned a message requiring a Bearer Authorization token; status was not recorded.
- E-03: Authenticated Cargo API request for `page=1` and `limit=20` returned HTTP 200; API response headers were inspected.
- E-04: `page=0` attempts; response evidence was incomplete, so pagination behavior remains inconclusive.
- E-05: `limit=100` request returned HTTP 200; the effective limit and record count were not captured.
- E-06: Login error comparison; both test cases displayed the same invalid-credentials message.
- E-07: MFA was enabled and required during a fresh login.
- E-08: After logout, browser Back showed the sign-in page.
- E-09: Browser navigation from an HTTP address ended at HTTPS; raw redirect details were not captured.
- E-10: Basic search special-character and XSS-marker checks showed no visible script execution.

## Limitations

Testing was limited to the staging environment and one USER-role account. No source code, server logs, raw HAR export, or second test account was available. The report uses qualitative severity and does not assign a CVSS score because the available evidence was not enough to establish exploitability and the affected security boundary.
