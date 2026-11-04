## Lab-01: Unprotected Admin Functionality

# Lab Information
- **Platform:** PortSwigger Web Security Academy
- **Category:** Broken Access Control
- **Difficulty:** Apprentice
- **Status:** Solved

# Objective
Locate an unprotected admin panel that is not linked from anywhere in the application, and use it to delete another user (`carlos`), demonstrating a missing access control check.

# Vulnerability
The application exposes an administration panel at a predictable or discoverable URL (e.g., `/admin`), but does not enforce any authentication or authorization check on that page. Access is only protected by obscurity, relying on the URL not being advertised or linked anywhere in the site, rather than by verifying the requester is actually an authenticated administrator.

# Methodology
1. Go to the lab and view robots.txt by appending "/robots.txt" to the lab URL. Notice that the Disallow line discloses the path to the admin panel.
2. In the URL bar, replace /robots.txt with /administrator-panel to load the admin panel.
3. Delete the user carlos. And you see the lab is solved.

# Evidence

![robots.txt disclosing the hidden admin path](images/lab-01-payload.png)

![Admin panel accessed without authentication, lab solved](images/lab-01-solved.png)

# Root Cause
The application assumes that because the admin panel's URL is not linked anywhere in the UI, it is effectively hidden and therefore safe. This is "security through obscurity": the page itself performs no server-side check to confirm the requester is an authenticated administrator, so anyone who discovers or guesses the URL has full access.

# Remediation
- Enforce server-side authentication and authorization checks on every sensitive endpoint, not just ones linked from the UI.
- Never rely on an unpredictable or unlinked URL as the sole protection for sensitive functionality.
- Apply role-based access control (RBAC) so admin actions are only available to users with the administrator role, verified on every request.
- Avoid listing sensitive paths in `robots.txt`, since it is publicly readable and often checked by attackers; use authentication instead of obscurity.

# Key Learnings
- Hiding a URL is not the same as securing it. Any page without a proper access control check can be found through files like `robots.txt`, sitemap files, JavaScript source, or simple guessing.
- Access control decisions must always be enforced on the server, for every request to a sensitive resource, regardless of whether a link to that resource is visible in the UI.
- `robots.txt` is a common and low-effort reconnaissance source during access control testing, since it often lists exactly the paths an organization wants hidden from search engines, and therefore from casual users.
