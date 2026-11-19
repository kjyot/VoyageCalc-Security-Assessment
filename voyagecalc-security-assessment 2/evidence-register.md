# Evidence Register

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
