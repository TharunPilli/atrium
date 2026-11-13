# Week 9 Verification Record

**Your name:** Tharun Pilli  
**Date:** 13 November 2026  
**Atrium copy used for the before result:** Week 8 vulnerable version (before applying the repair)  
**Atrium copy used for the after result:** Current Atrium working copy with the Week 8 repair applied

## 1. The claim I am checking

The main purpose of this verification is to check whether the SQL injection problem in the Staff Directory search has been fixed.

The search input entered by a user should be treated only as a value. It should not be able to change or modify the SQL query executed by the database.

In simple words, even if someone enters SQL-related characters in the search box, the application should treat them as normal search text instead of executing them as part of an SQL command.

## 2. Original evidence and change

- **Account and starting data:** I signed in to Atrium using Alice Nolan's account. The database contained three colleagues: Alice Nolan from Operations, Bob Keane from Finance and Morgan Doyle from IT Services.
- **Page or request:** Staff Directory – `/directory`
- **Original search input:** `' OR '1'='1`
- **Expected result:** The application should search for the entered text and display only matching colleagues.
- **Original result before the repair:** In my Week 8 testing, this input returned all three colleagues, even though it was not a normal search term.

To fix this problem, I modified the file:

`src/handlers/directory.js`

The query was changed to use parameterized SQL with placeholders (`?`). The search term is passed separately using:

`db.prepare(sql).all(term, term);`

This prevents the search input from becoming part of the SQL instruction. Instead, it is treated as a value that the database should search for.

## 3. Repeating the original check after the repair

After completing the Week 8 repair, I repeated the same test using the same account and the same search input.

- **Account used:** Alice Nolan
- **Page:** `/directory`
- **Search input:** `' OR '1'='1`
- **Expected result:** No matching colleagues should be displayed.
- **Actual result:** The page displayed "No colleagues found" and showed 0 colleagues.

When I compared this with my Week 8 before-repair result, I observed a clear difference.

Before the repair, the same input displayed all three colleagues. After the repair, it displayed zero colleagues.

This indicates that the original SQL injection behaviour was stopped in the directory search query.

## 4. Targeted verification tests

I performed the following tests through the Staff Directory page while signed in as Alice Nolan.

| Test input | Expected result | Actual result | Status |
|---|---|---|---|
| `' OR '1'='1` | No colleagues | No colleagues found | Pass |
| `Keane` | Bob Keane | Bob Keane | Pass |
| `Finance` | Bob Keane | Bob Keane | Pass |
| `Zawadzki` | No colleagues | No colleagues found | Pass |

All four tests gave the expected results.

The original SQL injection input no longer returned all the colleagues, and normal searches continued to work correctly.

## 5. Security scan

After completing the manual verification, I ran the security scan using the following command:

`npm.cmd run scan`

The scan completed and reported three findings under the rule:

`scan-rules.sql-built-by-joining-text`

The reported files were:

1. `src/db/reset.js`
2. `src/handlers/directory.js`
3. `src/handlers/resources.js`

The directory handler was still reported because the SQL query is created by joining two strings using the `+` operator.

However, after checking the actual code, I confirmed that the query uses placeholders and passes the search values separately through `.all(term, term)`.

Therefore, the scanner finding alone does not prove that the repaired directory query is vulnerable. The scan is based on a pattern and needs to be considered along with the actual code.

The scan reported three findings, which is the same count as the Week 8 scan. I have not treated the scan result alone as proof that the application is secure.

## 6. Checking normal directory behaviour and related paths

I also checked whether the Staff Directory continued to work normally after the repair.

I performed the following searches:

| Search input | Actual result |
|---|---|
| Empty search | All three colleagues were displayed |
| `Keane` | Bob Keane was displayed |
| `Finance` | Bob Keane was displayed |
| `Zawadzki` | No colleagues found |
| `Morgan Doyle'` | No colleagues found |

The empty search displayed all three seeded colleagues:

- Alice Nolan – Operations
- Bob Keane – Finance
- Morgan Doyle – IT Services

I also tested `Morgan Doyle'` with an additional apostrophe. The application displayed no matching colleagues, which was the expected result.

**Automated testing:**

I ran the following command:

`npm.cmd test`

The test results were:

- Total tests: 22
- Passed: 22
- Failed: 0
- Skipped: 0

All existing automated tests passed successfully.

**Related route checked:**

I inspected the file `src/routes/directory.js`.

The route is:

`directoryRoutes.get("/directory", requireAuth, showDirectory);`

From this, I understood that a request to `/directory` first passes through the authentication middleware and then reaches the `showDirectory` handler, where the directory search is processed.

This helped me confirm the route connected to the handler that I repaired in Week 8.

## 7. Final judgement

**My conclusion: Repaired for the checked directory database-query property.**

Based on the tests I performed, I believe the SQL injection issue in the Staff Directory database query has been fixed.

The strongest evidence is the comparison of the same SQL injection input before and after the repair.

Before the repair, the input `' OR '1'='1` displayed all three colleagues. After the repair, it displayed no colleagues.

I also checked normal name searches, department searches, empty searches, searches with no matching results and an input containing an apostrophe. All these checks produced the expected results.

The automated test suite also passed all 22 tests without any failures.

However, the existing automated tests mainly check normal application behaviour. They do not independently prove that the SQL injection issue is fixed. The manual before-and-after comparison provides the direct evidence for that security property.

My conclusion is limited to the directory database-query behaviour that I tested. It does not mean that the entire Atrium application is completely free from security issues.

## 8. Residual risk and limits

Although the directory query has been repaired, there may still be other areas that need attention.

One area identified in the Week 9 learning material is the search-text summary displayed on the directory page. The search text is also displayed in the page, and this is a separate output-handling boundary from the database query.

The Week 9 slides identify a possible issue where HTML-like search text may be interpreted by the browser instead of being displayed as plain text. I did not independently verify this behaviour during my manual tests.

Also, the security scan reported three findings. These should be reviewed individually to understand whether they represent actual vulnerabilities or pattern-based warnings.

The `npm.cmd run check` command mentioned in the Week 9 slides was not available in my project because no `check` script was defined in `package.json`. Therefore, I did not record that check as completed.

**What should be checked next?**

The directory page summary should be reviewed to ensure that user-provided search text is safely escaped before being displayed. The remaining scan findings should also be investigated separately.

Overall, the Week 8 directory SQL injection repair passed the manual verification and normal regression checks that I performed.