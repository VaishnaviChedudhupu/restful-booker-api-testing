Restful-Booker API Testing (Postman)

Functional and negative API tests for the public Restful-Booker practice API, written as a Postman collection with automated assertions. The negative tests found 7 distinct defects, each documented with steps to reproduce, expected and actual results, and screenshots.

Results
Measure	Result
Test cases	20 (8 functional, 12 negative and boundary)
Passed / failed	11 passed, 9 failed
Postman assertions	40 (27 in the functional run, 13 in the negative tests)
Defects logged	7 (5 Medium, 2 Low)

The functional run passed all 27 assertions. The negative tests are written against the correct behaviour (for example, invalid input should return 400), so a failed test means the API behaves differently from that expectation.

Defects found
ID	Summary	Severity
API-DEF-01	POST /auth returns 200 OK for invalid credentials	Medium
API-DEF-02	POST /booking returns 500 for a missing field or an empty body	Medium
API-DEF-03	POST /booking accepts a text totalprice and stores it as null	Medium
API-DEF-04	PUT and DELETE return 405 instead of 404 for a nonexistent booking	Low
API-DEF-05	POST /booking accepts a checkout date earlier than the checkin date	Medium
API-DEF-06	POST /booking accepts a negative totalprice	Medium
API-DEF-07	POST /booking accepts a 5,000-character firstname	Low

Full details are in reports/Restful_Booker_API_Test_Pack.xlsx and docs/API_Defect_Notes.md.

What is tested
Health check, authentication, create, read (list and by id), full update, partial update and delete
Status codes, response time, required fields, data types and returned values
Authorization: update and delete without a valid login or token
Invalid input: missing field, wrong data type, empty body, negative value, reversed dates, oversized value
Nonexistent ids on GET, PUT and DELETE
Repository contents
postman/
  restful-booker-collection.json      Postman collection (functional requests and "Negative tests" folder)
  restful-booker-environment.json     Environment with the base_url variable
reports/
  Restful_Booker_API_Test_Pack.xlsx   Summary, test cases, defect log, traceability matrix
docs/
  API_Defect_Notes.md                 Test results and defect write-ups
  screenshots/                        Evidence for each test and defect
How to run
Open Postman and click Import. Import both files from postman/.
Select the Restful-Booker environment (top right). Check that base_url is https://restful-booker.herokuapp.com.
Run the 8 functional requests with the Collection Runner. Untick the "Negative tests" folder. All 27 assertions should pass.
Run the Negative tests folder as a separate run. Run Create booking first, because NEG-02 and NEG-03 use the booking_id it saves.
Known limitations
Restful-Booker is a public practice API and some of these defects may be deliberate. The defect status is Open because I do not control the API.
The API resets its data about every 10 minutes, so booking ids in the screenshots will not exist when you re-run the tests. Create booking saves a fresh id on each run.
Response times vary on the free public server, so the response time assertions use a 5-second limit.
The requirements in the traceability matrix are derived from the API documentation and standard REST conventions. They are not an official specification.
Tools

Postman (collections, environments, Collection Runner, JavaScript test scripts), Excel