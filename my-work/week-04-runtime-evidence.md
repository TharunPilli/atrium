# Week 4 – Runtime Security Testing and Evidence

**Your name:** Tharun Pilli  
**Application:** Atrium  
**Application URL:** `http://localhost:9090`  
**Testing tool:** OWASP ZAP  
**Account referenced in the testing:** `alice.nolan`

## 1. Application Details

For this lab, the Atrium application was used as the target for runtime security testing. The application was accessed locally through the browser, and OWASP ZAP was used as the security testing tool.

The testing focused mainly on the sign-in process, dashboard access and Staff Directory functionality.

The account used in the shared testing observations was Alice Nolan (`alice.nolan`).

## 2. Signed-in Application Testing

The signed-in testing observations recorded in the shared report included the following requests:

| Page or request | HTTP status | Observation |
|---|---|---|
| `/login` (POST) | 302 Found | Login request redirected after submission |
| `/dashboard` | 200 OK | Dashboard page was accessible |
| `/directory` | 200 OK | Staff Directory page loaded |
| `/directory?q=alice` | 200 OK | Directory search returned a matching result |

The directory search response reportedly displayed Alice Nolan's information, including:

- Name: Alice Nolan
- Department: Operations
- Email: alice.nolan@atrium.example

These observations demonstrate the expected application navigation and directory search behaviour in the shared testing record.

**Evidence source:** These HTTP status codes and response observations were provided in a friend's testing report and have not been independently reproduced or verified by me.

## 3. OWASP ZAP Scan Observations

A shared OWASP ZAP scan report contained the following results:

- Total requests sent: 1,157
- New alerts identified: 51
- Scan scope: Whole Atrium website

One important limitation was mentioned in the report. The scan was performed against the general application scope and was not configured using the required authenticated Alice Context and a targeted directory-search scope.

Because of this, the scan results should be treated as preliminary observations rather than proof that the required authenticated active scan was completed.

**Evidence source:** The request count and alert count are peer-provided results, not independently generated results from my own saved ZAP session.

## 4. Important Security Alerts

The shared scan report identified two alerts that were considered important for further investigation.

### 4.1 Reflected Cross-Site Scripting (XSS)

- **Affected page:** `/directory`
- **Parameter:** `q`
- **Reported risk:** High
- **Reported confidence:** Medium

The scan reportedly detected script-like input being reflected in the directory search response.

This may indicate that user-provided search text is not being handled safely before it is displayed in the browser.

If confirmed, an attacker could potentially supply malicious input that affects users viewing the search results.

However, the report did not include independent browser evidence proving that the payload executed successfully. Therefore, this should be treated as a potential reflected XSS issue until manually verified.

### 4.2 SQL Injection

- **Affected page:** `/directory`
- **Parameter:** `q`
- **Reported risk:** High
- **Reported confidence:** Low

The shared report described a specially modified search value that caused the application to return a `500 Internal Server Error`.

The response reportedly contained a SQLite syntax error.

This suggests that the supplied search value may have caused an unexpected database-query error.

However, an HTTP 500 response and a database syntax error alone do not prove that SQL injection was successfully exploited or that unauthorized database access occurred.

Further testing is required to confirm the actual cause and impact.

**Evidence source for both alerts:** Peer-provided ZAP observations. The original alert records, requests and responses were not available for independent verification.

## 5. Evidence and Limitations

The available shared observations included:

- A normal directory search for `alice` reportedly returned HTTP 200 OK.
- The response reportedly contained Alice Nolan's directory details.
- ZAP reported possible reflected XSS and SQL injection issues involving the `q` parameter.
- The SQL-related observation involved a SQLite syntax error and an HTTP 500 response.

There are also important limitations:

1. The original ZAP History entries and alert screenshots are unavailable.
2. The modified requests have not been independently repeated.
3. Browser execution of the reported XSS payload has not been confirmed.
4. The reported SQL injection behaviour has not been independently reproduced.
5. Signed-out route coverage has not been verified from saved evidence.
6. The required authenticated Context-based directory scan was not completed in the shared scan configuration.

During my own work, I also attempted authenticated spidering, but ZAP returned zero URLs. This means that the authenticated spider coverage remains incomplete.

## 6. Possible Security Impact

If the reported issues are confirmed, they could affect the security and reliability of the Staff Directory.

A reflected XSS vulnerability could potentially allow malicious content to execute in another user's browser when they open a specially crafted search URL.

A SQL injection vulnerability could potentially allow specially constructed search input to interfere with database queries.

However, these impacts are conditional. The available observations are not sufficient to confirm successful exploitation or unauthorized access.

## 7. Suggested Remediation

The following improvements could help address the reported issues:

1. **Output encoding:** Encode user-supplied search text before displaying it in HTML.
2. **Parameterized queries:** Use prepared statements and parameterized SQL instead of directly combining search input with SQL commands.
3. **Input handling:** Handle unusual and invalid search values safely.
4. **Error handling:** Avoid displaying internal database errors to application users.
5. **Security verification:** Repeat the tests after applying the changes to confirm that the unwanted behaviour has stopped.

These recommendations should be followed by appropriate functional and security testing.

## 8. Conclusion

The Week 4 exercise focuses on investigating the runtime security of the Atrium application, particularly its Staff Directory search functionality.

The available peer-provided ZAP observations identify possible reflected XSS and SQL injection issues involving the directory search parameter. The report also describes normal directory access and a database syntax error produced by modified input.

However, the alerts still require independent manual verification before they can be treated as confirmed vulnerabilities.

My own authenticated spider attempt returned zero URLs, and the original ZAP alert evidence is unavailable. Therefore, complete route coverage and the required authenticated active-scan verification remain outstanding.

Overall, the available observations provide useful areas for further security investigation, but they are not sufficient to establish that either reported vulnerability was successfully exploited.