
## Lab-04: SQL Injection Attack, Querying the Database Type and Version on MySQL and Microsoft

# Lab Information
- **Platform:** PortSwigger Web Security Academy
- **Category:** SQL Injection
- **Difficulty:** Practitioner
- **Status:** Solved

# Objective
Perform a UNION based SQL injection attack against a MySQL or Microsoft SQL Server database to retrieve its type and version, and display this information on the page.

# Vulnerability
The applications product category filter builds its SQL query by directly concatenating the category URL parameter into the statement, without sanitization or parameterization. The backend query looks like this 
    "SELECT * FROM products WHERE category = 'Gifts' AND released = 1"

Because user input is placed inside the query, an attacker can append a "UNION SELECT" clause to retrieve data that is not normally returned, such as the database version.

# Methodology
1. Accessed the lab and connected Burp to the browser.
2. Navigated to the Accessories page and captured the request in Burp's HTTP history. Sent it to Repeater.
3. Determined the number of columns using ORDER BY, injected into the GET request's category parameter:
- 'ORDER BY 1--  → 200 OK
- 'ORDER BY 2--  → 200 OK
- 'ORDER BY 3--  → 500 Internal Server Error
-  confirms the query returns exactly 2 columns.
5. Tested which columns accept string data, injecting into the  GET request headers:
   'UNION SELECT 'a',NULL#
   → 200 OK response, confirming column 1 accepts text.
6. Retrieved the database version:
   'UNION SELECT @@version,NULL#
   →200 ok response with version as 8.0.42 in burp.
   And you see the lab was solved.
   
# Evidence
![UNION SELECT payload retrieving database version](images/lab-04-payload.png)
![Lab solved](images/lab-04-solved.png)

# Root Cause
User input is concatenated directly into the SQL query as a string, allowing an attacker to inject a "UNION SELECT" clause that queries the databases built-in version variable instead of the intended "products" table. The application also returned the query output directly to the page, to make the extracted data visible.

# Remediation
- Use parameterized queries/prepared statements so user input is always treated as data, never as SQL query.
- Apply least privileges to the database account used by the application.
- Disable detailed database error messages in production.
- Avoid returning raw query output directly to users wherever possible.

# Key Learnings
- MySQL and Microsoft SQL Server both expose version information through "@@version", to make database fingerprinting straightforward once a UNION injection point is found.
- Comment syntax differs across databases, MySQL requires a space after "--" or an alternative like "#", where Microsoft SQL Server accepts "--" without a trailing space.
- Database fingerprinting is an early reconnaissance step in SQL injection attacks, since it determines which syntax and system tables/variables to target next.
