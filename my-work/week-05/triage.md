# Week 5 Cross-Tool Triage

**Your name:** Tharun Pilli  
**Date:** 11/11/2026  
**Atrium address and port:** http://localhost:9090  
**Atrium version or date of local copy:** Local copy, 11/11/2026

Fill this in as you work. If a check could not be completed, record what stopped you and what evidence you used instead. Do not paste a full session cookie value.

## 1. Scan coverage and sources

- **Static evidence:** My `npm run scan` output: Semgrep reported SQL query construction in `directory.js` and `resources.js`.
- **Runtime evidence:** My ZAP run, Week 4 record, and browser checks for `/directory` and `/dashboard`.
- **Signed-in account used for protected routes:** `alice.nolan`.
- **Evidence that a protected route returned its page rather than `/login`:** `/dashboard` and `/directory` returned HTTP 200 while signed in.
- **Active scan target and how login and logout stayed outside it:** The previous ZAP scan covered the whole site. It was not limited to the required target, so login and logout were not confirmed to be excluded.
- **An entry my scan did not reproduce, and how I checked it instead:** The Semgrep SQL query warning was checked manually in `src/handlers/directory.js`, lines 8–13. Runtime verification is still pending.

## 2. The queue

| ID | Status | Evidence or missing check | Overlap with another ID? | Investigate now, later, or close? |
|---|---|---|---|---|
| S1 | Needs review | Semgrep found SQL query string concatenation in `directory.js`. | Z1 | Investigate now |
| S2 | Ruled out | `src/handlers/resources.js`, lines 1–5, uses a `?` placeholder and passes `req.params.id` separately. | None | Close |
| S3 | Ruled out | `reset.js`, lines 12–15, uses `?` placeholders for INSERT values. | None | Close |
| Z1 | Needs review | `GET /directory?q=alice%27` returned HTTP 500 with a SQLite error. SQL execution or data access is not proven. | S1 — same search input path, different runtime evidence | Investigate now |
| Z2 | Needs review | ZAP reported the search value in the directory response; browser script execution has not been confirmed. | Z3 — same route, different HTML location | Investigate now |
| Z3 | Needs review | ZAP flagged the search field value attribute; direct HTML inspection is still needed. | Z2 — same route, different HTML location | Investigate later |
| Z4 | Needs review | ZAP reported no Content-Security-Policy header on `GET /dashboard`; response headers still need verification. | None | Investigate later |
| Z5 | Needs review | ZAP reported that the `POST /login` Set-Cookie for `connect.sid` lacked HttpOnly; verify the response header in ZAP History. | None | Investigate later |

## 3. First three matters to investigate

### 1. First: SQL query construction and runtime error

- **Queue ID or IDs:** S1 and Z1
- **Account and place:** `alice.nolan`; `GET /directory?q=`
- **Why first:** The source code joins the search value into SQL text, and the changed search returned HTTP 500. This needs investigation, but SQL injection is not yet proven.
- **Next check:** Compare a normal search with a search containing one apostrophe and inspect the response and database query handling.

### 2. Second: Reflected search value

- **Queue ID or IDs:** Z2 and Z3
- **Account and place:** `alice.nolan`; `GET /directory?q=`
- **Why second:** ZAP found the search value reflected in the response and flagged the search field attribute. Actual browser script execution has not been confirmed.
- **Next check:** Inspect the returned HTML and test with harmless text to see where the search value appears.

### 3. Third: Security headers and session cookie

- **Queue ID or IDs:** Z4 and Z5
- **Account and place:** `alice.nolan`; `/dashboard` and `POST /login`
- **Why third:** ZAP reported missing security headers and a missing HttpOnly cookie flag. These are configuration concerns that need response-header verification.
- **Next check:** Inspect the dashboard response headers and the login Set-Cookie header in ZAP History.

## 4. One exclusion or combination

- **Entry ID:** S2
- **Close, exclude from active work, or combine with another entry?** Close.
- **Evidence for this decision:** `src/handlers/resources.js`, lines 1–5, uses a `?` placeholder and passes `req.params.id` separately to the database query.
- **What change would make me reopen it:** Reopen if the query is changed to concatenate untrusted input into the SQL statement.

## 5. One static and runtime comparison

- **Semgrep entry and ZAP entry:** S1 and Z1.
- **What the code observation shows:** `src/handlers/directory.js`, lines 8–11, concatenate the search value `q` into the SQL query.
- **What the request and response show:** `GET /directory?q=alice%27` returned HTTP 500 with a SQLite error.
- **Do they concern the same underlying matter? Why?** Yes. Both concern unsafe SQL query construction in the directory search function.
- **One claim they do not yet prove:** They do not prove successful SQL injection, database extraction, or access to other users' data.

## 6. One finding for a colleague

**Finding:**

The directory search at `/directory?q=` has a potential SQL injection issue for `alice.nolan`. In `src/handlers/directory.js`, lines 8–11, the search value is concatenated directly into the SQL query. A search containing an apostrophe returned HTTP 500 with a SQLite error. This may affect the security and reliability of directory searches, but successful SQL injection or data extraction has not been proven. The recommended repair is to use a parameterized SQL query and pass the search value separately.

**Evidence references:**

- **Static:** `src/handlers/directory.js`, lines 8–11 (SQL query construction).
- **Static:** Semgrep scan output showing SQL query string concatenation.
- **Runtime:** ZAP History, `GET /directory?q=alice%27`, HTTP 500 with SQLite error.
- **Account:** `alice.nolan` (signed-in test session).

---