# Week 2 security requirements

Your name: Tharun Pilli
Date: 05-10-2026

---

## 1. What this application is

Atrium is a small local staff-workspace application used to manage staff information and shared workplace resources. It provides a staff directory, shared documents/resources, a personal profile page, and account administration for administrators. Staff members can search for colleagues, view and update their own profile information, and view shared resources. Atrium recognises two roles, member and admin, with administrators having additional access to staff account management.

**Where the assistant's explanation did not match the application.**

Nothing found in the pages I checked. The explanation matched the Staff Directory, My Profile and Documents/Resources pages that I checked in Atrium.

---

## 2. What is worth protecting

| Asset | What it costs if this goes wrong |
|---|---|
| Staff personal and contact information | If the wrong person can see or change staff email addresses, departments or biographies, private staff information could be exposed or made inaccurate, which could cause unwanted contact or confusion. |
| Shared resource information | If the wrong person can change or damage resource records such as the expense claim form, supplier register or new starter checklist, staff could rely on incorrect information when carrying out their work. |
| User accounts and roles | If an unauthorised person can use another person's account or gain administrator access, they could access information or account functions that should be restricted to them. |
| Atrium availability and stored information | If Atrium or its stored information is unavailable when staff need it, users may not be able to find colleagues, view shared resources or access their profile information when required. |

---

## 3. Security requirements

### Requirement 1 - Staff information

Only signed-in Atrium users may access the staff directory and view colleague information. A user who is not signed in must not be able to open the directory and view staff details.

### Requirement 2 - Shared resources

Only signed-in Atrium users may view the shared resources in the Documents area. A user who is not signed in must not be able to open the resources page or view resource details.

### Requirement 3 - Administrator accounts

Only users with the admin role may access the Staff Accounts administration page. A member account must not be allowed to access the administrator-only staff account information.

### Requirement 4 - User profiles

A signed-in user may edit their own profile information, including their email address, department and short biography, but must not be able to edit another user's profile through the normal profile function.

---

## 4. Requirement rejected or substantially rewritten

### Original sentence

"Atrium should keep user information secure and only allow appropriate users to access it."

### My version

"Only users with the admin role may access the Staff Accounts administration page. A member account must not be allowed to access the administrator-only staff account information."

### Why I rejected the original

The original requirement fails the **specificity test** because the words "secure" and "appropriate users" do not clearly say what is allowed or who is restricted.

It also fails the **checkability test** because another person would not know exactly what they should test to decide whether the requirement has been met.

The rewritten requirement is specific about the user role and the exact Atrium page that should be checked.

---

## 5. How somebody would check one of these

### The requirement

Only users with the admin role may access the Staff Accounts administration page. A member account must not be allowed to access the administrator-only staff account information.

### Sign in as

Sign in to Atrium using the member account:

Username: `alice.nolan`

### Steps

1. Open Atrium at `http://localhost:9090`.
2. Sign in using the `alice.nolan` member account.
3. After signing in, try to open the Staff Accounts administration page.
4. Check whether the member account can access the staff account information.
5. Record what happens when the member account attempts to access the page.

### What result would mean the requirement is met

The requirement is met if the `alice.nolan` member account cannot access the Staff Accounts administration page or its staff account information, and access is denied or the user is redirected to a permitted page.

### What result would mean the requirement is not met

The requirement is not met if the `alice.nolan` member account can open the Staff Accounts administration page or can view staff account information that should be restricted to administrators.

---

## Optional - if time is available

### Order of importance

1. User accounts and roles - Incorrect access to accounts or administrator functions could give an unauthorised user access to other protected information.
2. Staff personal and contact information - Staff information should not be exposed or changed by unauthorised users.
3. Shared resource information - Incorrect resource information could cause staff to use outdated or incorrect workplace information.
4. Atrium availability and stored information - Staff need Atrium to be available when they need to access the directory, resources and profiles.