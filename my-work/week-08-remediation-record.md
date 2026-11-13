# Week 8 remediation record

**Your name:** Tharun Pilli  
**Date:** 13 November 2026  
**Atrium version or copy used:** Local Atrium project

## 1. Original finding and before evidence

- **Page and route:** Staff Directory – `/directory`
- **Account used:** Alice Nolan
- **Earlier finding or source I started from:** While checking the directory search, I found that the search input was being directly added to the SQL query. This could allow a user to change how the query works by entering specially crafted input.

**Ordinary search:**
- Search text: `Keane`
- URL: `/directory?q=Keane`
- Result: Bob Keane was displayed with his department as Finance.
- Number of results: 1 colleague.
- HTTP status: Not separately recorded.

**No-match search:**
- Search text: `Zawadzki`
- URL: `/directory?q=Zawadzki`
- Result: The page displayed "No colleagues found."
- Number of results: 0.
- HTTP status: Not separately recorded.

**Comparison search:**
- Search text: `' OR '1'='1`
- URL: `/directory?q=%27+OR+%271%27%3D%271`
- Result before repair: All three colleagues, Alice Nolan, Bob Keane and Morgan Doyle, were displayed.
- Number of results: 3 colleagues.
- HTTP status: Not separately recorded.

**What the comparison shows, and what it does not show:**

When I entered the injection string, the directory displayed all three colleagues instead of treating it as a normal search term. This showed that the search input could change the SQL query. It helped me identify the SQL injection problem in the directory search. However, this test does not mean that every database query in the application has the same problem.

## 2. Root cause

- **Source of the search text:** `req.query.q` in `src/handlers/directory.js`, inside the `showDirectory()` function.
- **Path through the handler:** The application reads the search text from the URL and passes it to the `searchDirectory(db, q)` function.
- **Database operation:** `db.prepare(sql)` inside `searchDirectory()` in `src/handlers/directory.js`.
- **Control missing on that path:** The search value was directly joined with the SQL query instead of being passed separately as a parameter.
- **Difference between the visible result and the root cause:** The visible problem was that all colleagues appeared when I entered the injection string. The actual reason was that the application was treating the search input as part of the SQL command.

After checking the code, I understood that the main problem was not the search box itself