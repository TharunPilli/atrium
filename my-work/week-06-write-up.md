# B9CY104 – Week 6 Lab Write-up
## Preparing Evidence for a Security Report

**Your name:** Tharun Pilli  
**Date:** 13 November 2026  
**Atrium version or date of local copy:** Local project copy tested on 13 November 2026

---

# Task 1 – Audit the Evidence

In this task, I reviewed the static scan results and the authenticated ZAP alert. My aim was to understand what the tools reported and then identify what I could actually verify through the source code and browser testing.

## 1.1 Static scan finding

The static scan produced three code findings using the rule:

`scan-rules.sql-built-by-joining-text`

One of the findings was reported in `src/handlers/directory.js`, around lines 8–11.

**What the scanner reported:**

The scanner identified SQL query construction using joined text. This suggested that user input might be included directly in an SQL statement.

**What I verified:**

I opened the directory handler and found that the search value `q` was directly concatenated into the SQL query inside the `searchDirectory()` function.

**What was still missing:**

The scanner result alone could not prove that the application was vulnerable. I needed to trace where the input came from, understand how it reached the database, and test the behaviour using an authenticated account.

## 1.2 Dynamic scan finding

I also reviewed the ZAP alert related to the directory search page.

**Alert name:** User Controllable HTML Element Attribute (Potential XSS)  
**ZAP rule:** 10031  
**Risk:** Informational  
**Confidence:** Low  
**Affected URL:** `http://localhost:9090/directory?q=Alice`  
**Parameter:** `q`  
**Account used:** Alice Nolan (`alice.nolan`)

The alert was present in my ZAP report. I checked the authenticated request in ZAP History and confirmed that the directory page returned successfully.

The recorded request used the ordinary search value `Alice` and returned HTTP 200 OK.

However, the alert itself was not enough to confirm that JavaScript could execute. I needed to repeat the test using a different search value and inspect the actual response.

**What was still missing:**

I needed to compare the ordinary request with an altered request, inspect the response body, and check how the search value was displayed on the page.

---

# Task 2 – Validate the Static Code Finding

## Finding: User input is directly concatenated into the directory SQL query

### Affected files and location

- `src/routes/directory.js`
- `src/handlers/directory.js`
- Function: `searchDirectory(db, q)`
- Relevant query construction: lines 8–11 of `src/handlers/directory.js`

### Account and conditions

I performed the testing while signed in to Atrium as Alice Nolan (`alice.nolan`).

The directory route uses the `requireAuth` middleware, so the user must be authenticated to access this functionality.

### What I found in the source code

I first checked `src/routes/directory.js` and confirmed that the `/directory` route uses authentication before calling the directory handler.

I then inspected `src/handlers/directory.js`.

The handler reads the search value from `req.query.q` and passes it to the `searchDirectory()` function.

Inside this function, the search value is directly joined to the SQL query using string concatenation.

This means the user-controlled value becomes part of the SQL statement itself instead of being passed separately as a parameter.

### Testing with an ordinary input

I searched for:

`GET /directory?q=Alice`

**Result:** HTTP 200 OK.

The directory page loaded successfully and displayed one matching colleague.

This gave me a normal result to compare against the changed input.

### Testing with a changed input

I then tested:

`GET /directory?q=Alice%27`

The decoded search value was `Alice'`.

**Result:** HTTP 500 Internal Server Error.

The application displayed an SQLite syntax error. The stack trace pointed to the `searchDirectory()` function in `src/handlers/directory.js`, at line 13.

### My conclusion

After comparing the source code with the browser results, I confirmed that the directory search builds its SQL query using user-controlled input.

The ordinary search worked, but the changed input caused a database syntax error. This supports the finding that the query is not safely parameterised.

The test demonstrated an error condition, but I did not demonstrate successful extraction or modification of database records. Therefore, I am not claiming that the entire database can be compromised based on this test alone.

### Possible impact

A specially crafted search value can interfere with the directory query and cause the application to return a server error.

The error response also reveals database-related information that should not normally be displayed to users.

### Recommended fix

The application should use parameterised SQL queries instead of directly joining user input into the query string.

For example, placeholders can be used for both search conditions, and the search pattern can be passed separately to the database.

After making the change, the ordinary and altered inputs should be tested again to confirm that they are handled safely.

### Comparison with another static finding

The scanner also reported a similar match in `src/db/reset.js`, around lines 12–13.

After checking that code, I found that the insert statement uses placeholders and passes the values separately through the `.run()` method.

This showed me that two scanner results can match the same rule but still behave differently. It is important to inspect the actual code before deciding whether a finding is a genuine vulnerability.

---

# Task 3 – Validate the Dynamic Finding

## Finding: Directory search input is rendered as unescaped HTML

### Affected file and location

- URL: `GET /directory?q=...`
- View: `views/directory/index.ejs`
- Relevant lines: 13 and 20
- Handler: `src/handlers/directory.js`

### Account and conditions

I performed this test while logged in as Alice Nolan (`alice.nolan`).

The directory page requires authentication.

### ZAP alert details

ZAP reported the following alert:

- **Alert:** User Controllable HTML Element Attribute (Potential XSS)
- **Rule:** 10031
- **Risk:** Informational
- **Confidence:** Low
- **Parameter:** `q`

The original alert request used the ordinary value `Alice`. I therefore carried out an additional test to check how the application handled HTML markup in the search value.

### Testing with an ordinary input

I first searched for:

`GET /directory?q=Alice`

**Result:** HTTP 200 OK.

The page loaded normally, and the result summary displayed the search term as ordinary text.

### Testing with HTML input

I then searched using:

`GET /directory?q=%3Cb%3EAlice%3C%2Fb%3E`

This represents the input:

`<b>Alice</b>`

**Result:** HTTP 200 OK.

The page returned zero matching colleagues, but I noticed something important in the response.

The search field displayed the submitted tags as escaped text, while the result summary interpreted the `<b>` tags as HTML and displayed Alice in bold.

The relevant response content was:

```html
<input type="text" id="directory-search" name="q"
       value="&lt;b&gt;Alice&lt;/b&gt;">

<p class="result-summary">
  Showing colleagues matching <strong><b>Alice</b></strong>.
</p>
```

### What I found in the source code

I opened `views/directory/index.ejs` and compared the two output locations.

At line 13, the search field uses:

`<%= q %>`

This escapes the submitted value before displaying it.

At line 20, the result summary uses:

`<%- q %>`

This displays the value without escaping it.

The handler obtains the search term from the request and passes it to the view. This explains why the same input is handled differently in the two places.

### My conclusion

I was able to reproduce the HTML injection in the directory result summary.

The submitted `<b>Alice</b>` markup was interpreted by the browser instead of being displayed as plain text.

This confirms that the application renders user-controlled HTML in this location. However, I did not demonstrate JavaScript execution, so I am describing this as HTML injection and potential XSS rather than confirmed script execution.

### Possible impact

A user can influence the HTML displayed in the authenticated directory page by submitting specially formatted search input.

This could affect how the page is displayed. More serious consequences, such as script execution or session theft, were not demonstrated during my test.

### Recommended fix

The unescaped output at line 20 should be replaced with escaped EJS output.

For example:

```ejs
<p class="result-summary">
  Showing colleagues matching <strong><%= q %></strong>.
</p>
```

After applying the fix, the same altered input should be tested again. The browser should display the tags as text rather than interpreting them as HTML.

---

# Task 4 – Cross-check Both Findings and Review

In this task, I compared the results from the source code with the behaviour observed in the running application.

## 4.1 Cross-checking the static finding

The source code showed that the directory search value was directly concatenated into the SQL query.

I compared this with my browser tests:

- Searching for `Alice` returned HTTP 200 OK.
- Searching for `Alice'` returned HTTP 500 Internal Server Error.
- The error pointed to the directory search function.

The runtime result supported what I had found in the source code. It increased my confidence that the SQL construction issue was real.

However, the test did not prove that database records could be extracted or modified.

## 4.2 Cross-checking the dynamic finding

For the dynamic finding, I checked the EJS template and compared it with the browser response.

The source code showed that:

- Line 13 used escaped output.
- Line 20 used unescaped output.

The browser test showed exactly this difference. The search field displayed the tags as text, but the result summary interpreted them as HTML.

This helped me understand the reason behind the ZAP alert and confirmed the behaviour I observed.

## 4.3 Review using an AI assistant

I asked ChatGPT to review both findings and identify any statements that were not fully supported by the evidence.

The review highlighted two important points.

**First criticism – Static finding:**

The SQL syntax error showed that the input affected the database query, but it did not prove successful data extraction or complete database compromise.

**Change made:**

I limited the impact statement to the behaviour I actually observed: unsafe SQL construction, a server error and database error information being displayed.

**Second criticism – Dynamic finding:**

The ZAP alert was informational and had low confidence. Although the browser interpreted the submitted HTML tags, I had not demonstrated JavaScript execution.

**Change made:**

I described the issue as HTML injection and potential XSS. I avoided claiming confirmed script execution, session theft or account compromise.

## 4.4 Final review decision

I decided to keep both findings because they were supported by source-code inspection and browser testing.

The review helped me make the conclusions more accurate and avoid claiming impacts that I had not demonstrated.

---

# Task 5 – Procrastimate Setup for CA1

For the final task, I downloaded the Procrastimate application ZIP file and the CA1 brief from Moodle.

I extracted the application and renamed the outer folder using my student number.

### Folder name

`procrastimate-20102025`

### Project folder location

`C:\Users\HaKa\Downloads\procrastimate-20102025\procrastimate`

### Port configuration

I created a `.env` file in the project root and configured the port using the last four digits of my student number.

```env
PORT=2025
```

### Node.js version

My installed Node.js version was:

`v24.21.0`

This meets the Node.js version requirement mentioned in the application README.

### Installing dependencies

I ran:

```powershell
npm.cmd ci
```

The installation completed successfully.

- 99 packages were added.
- 100 packages were audited.
- npm reported 2 vulnerabilities: 1 moderate and 1 high.

I did not run an automatic fix because I wanted to keep the provided project dependencies unchanged.

### Running the doctor command

I ran:

```powershell
npm.cmd run doctor
```

The command returned the following output:

```text
ok   Node version - found v24.21.0, Procrastimate needs Node 22 (22.12 or later) or Node 24
ok   SQLite library - better-sqlite3 loaded and ran a query
ok   Data folder - C:\Users\HaKa\Downloads\procrastimate-20102025\procrastimate\data is writable
ok   Port 2025 (from .env) - free to use; open http://127.0.0.1:2025
Everything Procrastimate needs is in place.
```

All the checks passed successfully.

### Starting the application

I started the application using:

```powershell
npm.cmd start
```

The terminal displayed:

```text
Procrastimate is running at http://127.0.0.1:2025
Press Ctrl and C together to stop it.
```

I opened the application in my browser using:

`http://127.0.0.1:2025`

I signed in using the `maya` account, and the application successfully loaded the dashboard showing Maya Farrell's bookings.

After confirming that the application was working, I stopped the server using Ctrl+C.

### Final result

The Procrastimate setup was completed successfully. The application passed all doctor checks, ran on my configured port 2025 and allowed me to sign in as Maya.

---

# Evidence Checklist

The following evidence was collected during the practical and should be kept with the report:

- [ ] Static scan output showing the directory handler finding.
- [ ] Source-code evidence from `src/routes/directory.js` and `src/handlers/directory.js`.
- [ ] Ordinary directory search request and response.
- [ ] Changed SQL input request and 500 error response.
- [ ] ZAP alert screenshot showing the alert name, risk and confidence.
- [ ] Authenticated ZAP History request and response.
- [ ] Altered HTML input request and response.
- [ ] EJS template evidence showing lines 13 and 20.
- [ ] AI review and the changes made after reviewing the findings.
- [ ] Procrastimate installation and doctor output.
- [ ] Screenshot of the Procrastimate dashboard after signing in as Maya.
- [ ] Screenshot or terminal evidence showing that the application was stopped.

**Note:** Passwords and full session-cookie values have not been included in this report.

---

## Overall Reflection

This practical helped me understand why a scanner alert should not immediately be treated as a confirmed vulnerability. I learned to check the source code, reproduce the behaviour in the browser and compare ordinary requests with changed inputs.

I also understood the importance of explaining what the evidence proves and what it does not prove. Finally, I completed the Procrastimate setup for CA1 and confirmed that the application runs successfully on my own configured port.