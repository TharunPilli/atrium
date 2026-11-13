# Week 7 API investigation

**Your name:** Tharun Pilli  
**Date:** 13 November 2026  
**Atrium address and port:** http://localhost:9090  
**Atrium version or date of local copy:** Local Atrium copy used on 13 November 2026.

## 1. Original profile request

- **Signed-in account:** Alice Nolan (`alice.nolan`), ID 17.
- **Page that made the request:** My Profile page (`/profile`).
- **Method and full API path:** `GET http://localhost:9090/api/profile/17`
- **Was a session cookie present? (yes/no, value hidden):** I was using Alice's signed-in browser session, but I did not separately confirm the cookie header in the captured request.
- **First response status and content type:** At first, I got `304 Not Modified` because the browser was using a cached response. After enabling Disable cache and refreshing the page, I got `200 OK`. The content type was `application/json`.
- **Returned ID, username and field names:** The returned ID was `17` and the username was `alice.nolan`. The response contained these fields: `id`, `username`, `displayName`, `email`, `department`, `role` and `bio`.
- **What did the request ask for, in your own words?**

  The request was asking Atrium's API to give the profile details of the user with ID 17. The response showed Alice Nolan's information, including her email, department, role and bio.

## 2. Replayed and changed profile requests

| Request | Account whose session I used | First status | Returned ID and name, or error | Evidence reference |
|---|---|---|---|---|
| Repeat of the original path: `/api/profile/17` | Alice Nolan | 200 OK | ID 17 – Alice Nolan | Browser response |
| Changed path: `/api/profile/18` | Alice Nolan | 200 OK | ID 18 – Bob Keane | Browser response after changing the ID |

- **What I changed:** I changed only the ID at the end of the URL from `17` to `18`.
- **What stayed the same:** I used the same browser, the same Alice login session and the same GET request. Only the profile ID was changed.
- **Whose record the changed ID names, and how I checked:** ID 18 belongs to Bob Keane. I checked his name and details in the API response and compared them with the information available in the directory.
- **What the comparison shows:**

  I was able to get Bob's profile details even though I was signed in as Alice. The API returned `200 OK` instead of stopping me from accessing another user's profile. This suggests that the API checks whether someone is signed in, but does not properly check whether that person is allowed to view the particular profile.

I also tried `/api/profile/99` while signed in as Alice. It returned Morgan Doyle's administrator profile with `200 OK`, which gave me another example of the same issue.

## 3. Without sign-in

- **How I made sure the window had no Atrium session:** I opened a fresh InPrivate browser window so that it would not use my normal browser's Alice login session.
- **`GET /api/profile/17`: first status and response body:** I received `401 Unauthorized`. The response was:

  `{"error":"You need to sign in to use this part of Atrium."}`

- **`GET /profile`: first status, redirect if any, and final page:** When I opened `/profile`, Atrium redirected me to the login page. I did not record the exact HTTP status of the redirect.
- **What this establishes about authentication:** This shows that Atrium checks whether a user is signed in before allowing access to the profile API. When I tried it without signing in, the API returned a 401 error.
- **What it does not establish about authorisation:** This does not mean that every signed-in user is allowed to access every profile. My earlier test showed that Alice could still access Bob's and Morgan's profile records by changing the ID.

## 4. Administration page and API

| Account | Request path | First status | Relevant response text or JSON fields | Evidence reference |
|---|---|---|---|---|
| Alice | `/admin/users` | 403 Forbidden | `Administrators only.` | Alice browser Network capture |
| Alice | `/api/admin/users` | 200 OK | Returned a list of three accounts with fields such as `id`, `displayName`, `username`, `department` and `role` | Alice browser Network capture and JSON response |
| Morgan | `/admin/users` | Page opened successfully; exact HTTP status not recorded | Administration page displayed the staff accounts | Morgan's signed-in InPrivate browser |
| Morgan | `/api/admin/users` | 200 OK | Returned the three staff accounts with fields such as `id`, `displayName`, `username`, `department` and `role` | Morgan browser Network capture and JSON response |

- **Which request did Morgan's page make after loading?**

  After Morgan opened the administration page, I checked the Network tab and found a `GET /api/admin/users` request.

- **What role decision should the server make for each route?**

  Both the administration page and its API should allow only administrators. Alice is a normal member, so she should get `403 Forbidden` from both routes. Morgan has the admin role, so she should be able to access both.

- **Where did the observed decisions agree or differ?**

  When I tried `/admin/users` as Alice, I got `403 Forbidden` with the message `Administrators only.` But when I tried `/api/admin/users` using the same Alice session, I got `200 OK` and could see the staff account list.

  When I signed in as Morgan, who is an administrator, the administration page opened and the API also returned `200 OK`. So the page was checking the admin role correctly, but the API was not applying the same restriction.

## 5. Security conclusions

**Profile request.**

During my testing, I was signed in as Alice Nolan (ID 17). I changed the request from `GET /api/profile/17` to `GET /api/profile/18` and received `200 OK` with Bob Keane's profile details. I also tried ID 99 and received Morgan Doyle's profile. From these tests, I found that the API was allowing Alice to view other users' profiles just by changing the ID in the URL.

The server should check whether the signed-in user is actually allowed to access the requested profile. It should not allow access just because the user has logged in. This looks like a broken object-level authorisation issue. I only tested viewing profile information, not changing anyone