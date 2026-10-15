# Week 3 record
Name: Tharun Pilli
Date: 15 October 2026


 1. The finding I chose

- **File:** `src/handlers/directory.js`
- **Line:** 8–13 (SQL construction and execution; the user input is read on line 19).
- **What the tool said about it:** Semgrep reported `scan-rules.sql-built-by-joining-text` with blocking severity. It warned that building an SQL statement by joining text can be dangerous if any of that text comes from outside the application.

2. What an assistant told me

- **Assistant used:** ChatGPT.
- **Its explanation, in my own words:** The assistant explained that the directory search takes the user's search input and joins it directly into an SQL query. This may allow SQL injection because the input can change how the database interprets the query.
- **One thing it asserted that I had not verified at that point:** I had not yet checked whether a specially crafted search input actually changed the results returned by the running application.

 3. What the code shows

**Where does the data come from?**

The data comes from the `q` query parameter in the HTTP request. In `src/handlers/directory.js`, line 19, the application reads `req.query.q` and assigns it to `q` if it is a string.

**What happens to it on the way?**

The value of `q` is passed to `searchDirectory(db, q)`. Inside this function, the application concatenates `q` into the SQL query used to search display names and departments.

**Where does it become dangerous?**

The risk occurs in lines 10–11, where `q` is inserted directly into the SQL text. The query is then prepared and executed using `db.prepare(sql).all()` on line 13. Preparing the query after inserting the input does not separate that input from the SQL statement.

**What stands in the way?**

The code checks that `req.query.q` is a string, but this does not prevent SQL injection. I could not identify an effective parameterized query or other control that prevents the search input from changing the SQL statement in this function.

4. What the running application shows

**Ordinary case**

- **What I entered:** `Nolan`
- **What came back:** One colleague, Alice Nolan, was displayed.

**The case I was testing for**

- **What I entered:** `' OR '1'='1`
- **What came back:** Three colleagues were displayed: Alice Nolan, Bob Keane and Morgan Doyle.
- **How this differs from the ordinary case:** The normal search returned one colleague, whereas the specially crafted input returned all three colleagues shown in the directory. This result is consistent with the input changing the SQL query's behaviour.

 5. My answer


**Why, in two or three sentences.**

The finding is real because the code inserts user-controlled search input directly into the SQL query without using parameterized placeholders for that input. In my local test, searching for `Nolan` returned one colleague, while the test payload returned three colleagues. Together, the code and observed behaviour support the conclusion that the directory search is vulnerable to SQL injection.

**What would change my mind.**

If further investigation showed that the observed results came from a different code path or that the running application was not executing the displayed query, I would reconsider the conclusion. A code change that safely parameterizes the search input, followed by testing the updated version, would also change the assessment of that version.

**How far this answer reaches.**

This conclusion applies to the directory search in the version of Atrium I inspected and tested locally, using the data available in that lab environment. I established that the normal search returned one colleague and the test input returned three, consistent with SQL injection. I have not established that every page or query in Atrium is vulnerable, or that the same issue exists in any other application.

 6. Back to the assistant

- **Did I check it?** Yes. I inspected `src/handlers/directory.js` and tested the normal and specially crafted search inputs against the local application.
- **Was it right?** Yes. The direct concatenation in the source code and the difference between the observed search results support the assistant's initial explanation.

