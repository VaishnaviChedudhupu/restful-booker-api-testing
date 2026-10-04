# Restful-Booker API Testing: Defect Notes and Negative Test Results

API under test: https://restful-booker.herokuapp.com (public practice API)
Tool: Postman (collection "Restful-Booker API Test", folder "Negative tests")

## Negative test results (12 tests: 3 passed, 9 failed, 7 distinct defects)

| Test | Scenario | Expected | Actual | Result | Defect |
|---|---|---|---|---|---|
| NEG-01 | Auth with wrong password | 401 or 403, no token | 200 OK, body {"reason":"Bad credentials"} | Failed | API-DEF-01 |
| NEG-02 | Update without login | 401 or 403 | 403 Forbidden | Passed | |
| NEG-03 | Delete with invalid token | 401 or 403 | 403 Forbidden | Passed | |
| NEG-04 | Create booking, firstname missing | 400 | 500 Internal Server Error | Failed | API-DEF-02 |
| NEG-05 | Create booking, totalprice "abc" | 400 | 200 OK, totalprice stored as null | Failed | API-DEF-03 |
| NEG-06 | Update nonexistent booking (valid auth) | 404 | 405 Method Not Allowed | Failed | API-DEF-04 |
| NEG-07 | Get nonexistent booking | 404 | 404 Not Found | Passed | |
| NEG-08 | Delete nonexistent booking (valid auth) | 404 | 405 Method Not Allowed | Failed | API-DEF-04 |
| NEG-09 | Create booking, checkout before checkin | 400 | 200 OK, booking 5202 stored with reversed dates | Failed | API-DEF-05 |
| NEG-10 | Create booking, totalprice -50 | 400 | 200 OK, booking 2803 stored with totalprice -50 | Failed | API-DEF-06 |
| NEG-11 | Create booking, empty body {} | 400 | 500 Internal Server Error | Failed | API-DEF-02 |
| NEG-12 | Create booking, 5,000-character firstname | 400 or 413 | 200 OK, 5.8 KB response | Failed | API-DEF-07 |

## Defects

### API-DEF-01
- Title: POST /auth returns 200 OK for invalid credentials
- Endpoint: POST /auth
- Steps: Send POST {{base_url}}/auth with username "admin" and password "wrongpassword"
- Expected: 401 Unauthorized (or 403), no token
- Actual: 200 OK with body {"reason":"Bad credentials"}; no token returned
- Severity: Medium
- Impact: Clients that rely on the status code may treat a failed login as a success
- Test: NEG-01
- Evidence: NEG-01_API-DEF-01_test_failed.png, NEG-01_API-DEF-01_response_body.png

### API-DEF-02
- Title: POST /booking returns 500 Internal Server Error for incomplete or empty request bodies
- Endpoint: POST /booking
- Steps: 1. Send POST {{base_url}}/booking with a valid body that has no "firstname" (NEG-04). 2. Send POST {{base_url}}/booking with the body {} (NEG-11)
- Expected: 400 Bad Request with a validation message
- Actual: 500 Internal Server Error, body "Internal Server Error", in both cases
- Severity: Medium
- Impact: Invalid input crashes the request instead of being rejected, and clients get no useful error message
- Tests: NEG-04, NEG-11
- Evidence: NEG-04_API-DEF-02_response_500.png, NEG-04_API-DEF-02_test_failed.png, NEG-11_API-DEF-02_response_500.png

### API-DEF-03
- Title: POST /booking accepts a non-numeric totalprice and stores it as null
- Endpoint: POST /booking
- Steps: Send POST {{base_url}}/booking with a valid body but "totalprice": "abc"
- Expected: 400 Bad Request with a validation message, and no booking created
- Actual: 200 OK, booking created (bookingid 4332) with "totalprice": null
- Severity: Medium
- Impact: Invalid input is silently accepted and corrupts stored data, so bookings can exist with no price
- Test: NEG-05
- Evidence: NEG-05_API-DEF-03_test_failed.png, NEG-05_API-DEF-03_response_body.png

### API-DEF-04
- Title: PUT and DELETE /booking/{id} return 405 Method Not Allowed for a nonexistent booking id
- Endpoint: PUT /booking/{id}, DELETE /booking/{id}
- Steps: With valid Basic Auth (admin), send PUT or DELETE to {{base_url}}/booking/99999999
- Expected: 404 Not Found (the same as GET /booking/99999999, which correctly returns 404 in NEG-07)
- Actual: 405 Method Not Allowed, body "Method Not Allowed", on both methods
- Severity: Low
- Impact: Misleading status code. Clients are told the method is unsupported when the real problem is a missing booking. The API is also inconsistent: GET returns 404 for the same id
- Tests: NEG-06, NEG-08
- Evidence: NEG-06_API-DEF-04_response_405.png, NEG-08_API-DEF-04_response_405.png, NEG-07_pass_404.png (comparison)

### API-DEF-05
- Title: POST /booking accepts a checkout date earlier than the checkin date
- Endpoint: POST /booking
- Steps: Send POST {{base_url}}/booking with a valid body but "checkin": "2026-11-10" and "checkout": "2026-11-05"
- Expected: 400 Bad Request with a validation message, and no booking created
- Actual: 200 OK, booking created (bookingid 5202) with "checkin": "2026-11-10" and "checkout": "2026-11-05"
- Severity: Medium
- Impact: Logically invalid bookings are stored, which can break availability, pricing and reporting
- Test: NEG-09
- Evidence: NEG-09_API-DEF-05_test_failed.png, NEG-09_API-DEF-05_response_body.png

### API-DEF-06
- Title: POST /booking accepts a negative totalprice
- Endpoint: POST /booking
- Steps: Send POST {{base_url}}/booking with a valid body but "totalprice": -50
- Expected: 400 Bad Request with a validation message, and no booking created
- Actual: 200 OK, booking created (bookingid 2803) with "totalprice": -50
- Severity: Medium
- Impact: Bookings with impossible prices can be stored, which corrupts revenue totals and reports
- Test: NEG-10
- Evidence: NEG-10_API-DEF-06_test_failed.png, NEG-10_API-DEF-06_response_body.png

### API-DEF-07
- Title: POST /booking accepts a 5,000-character firstname with no length validation
- Endpoint: POST /booking
- Steps: Send POST {{base_url}}/booking with a valid body but a "firstname" of 5,000 characters (generated in a Pre-request script)
- Expected: 400 Bad Request (or 413 Payload Too Large) for input that exceeds a sensible limit
- Actual: 200 OK, booking created, response size 5.8 KB with the full name echoed back
- Severity: Low
- Impact: No input size limits can lead to bloated data, display problems and possible abuse of storage
- Test: NEG-12
- Evidence: NEG-12_API-DEF-07_test_failed.png
