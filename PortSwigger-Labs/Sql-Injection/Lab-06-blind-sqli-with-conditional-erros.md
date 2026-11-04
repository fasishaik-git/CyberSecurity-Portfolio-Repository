# Lab-06: Blind SQL Injection with Conditional Errors

# Lab Information
- **Platform:** PortSwigger Web Security Academy
- **Category:** SQL Injection (Blind)
- **Difficulty:** Practitioner
- **Status:** Solved

# Objective
Exploit a blind SQL injection vulnerability by triggering conditional database errors, using this to determine the password of the "administrator" user one character at a time.

# Vulnerability
The application uses a "TrackingId" whose value is inserted directly into a SQL query on the backend, without sanitization or parameterization. The response does not return any query results or error messages directly, so the injection is "blind." However, the application behaves differently depending on whether the injected SQL condition is true or false, a "500 Internal Server Error" is returned when the condition causes the query to throw a database error, while a normal "200 OK" response is returned otherwise. This difference in behavior is enough to infer true/false answers about the data, one bit at a time.

# Methodology
1) Access the lab by clicking on ACCESS THE LAB button. And connect the burp to your browser.
2) Now make the interceptor on, in browser click on Home, now texts you will see the requests and responses captured under burp.Choose "GET /" URL and send it to repeater. Make interceptor off.
3) Prove that parameter is vulnerable, You see a tracking Id header with its value in burp request tab, you have to inject everything at the end of trackingid value of the entire lab.
- single quote (') -> gives you internal error
- double quote (") -> gives you 200 OK response.
- Payload "||(select '') ||'" -> gives you internal server error because we are not dealing with MySQL db.
- Payload "||(select '' from dual) ||'" -> gives 200 OK response because here you confirm it is Oracle db.
- Payload "||(select '' from dualnoexist) ||'" -> gives you internal server error, which again confirms we are dealing with non-mysql/oracle db.
4) Confirm that the users table exists in the database
- Use payload "||(Select '' From users where rownum=1)||'" gives 200OK response, it means users table exist.
- In payload "rownum=1" it is used to test by only 1 row of users table, if not included which will going to hit for all rows in users table and breaks the query.
5) Confirm that the administrator user exists in the users table.
- Use payload "||(Select case when (1=1) then to_char(1/0) ELSE '' END from users where username='administrator')||'" it will gives you internal server error, where the above query payload intentionally made to give error for if administrator user exist.
- If you try with non-exist user, you will get 200 OK response, it means user does not exist.
6) Determine length of the Password.
- Use payload "||(Select case when (1=1) then to_char(1/0) ELSE '' END from users where username='administrator' and LENGTH(password)>1) ||'" gives you internal
server error.
- Try with >50, it gives 200 OK it means password length is less than 50.
- Right click in burp, send it to intruder. Navigate to intruder > positions > click once on clear § > select "50". Using mouse > click once on Add § > 50 is highlighter > Change it to 30 to 1.
- Navigate to Payloads tab > Payload set 1 > Payload type > Numbers > Type sequential > From 1 > To 30 > Step 1 and click on Start attack button.
-You will see the result as 1-19 500, 20-30 200 OK, it means the password length is not greater than 20 & not less than 19, it is exact 20 size.
7) Output the administrator password.
- Use payload "||(Select case when (1=1) then to_char(1/0) THEN TO_CHAR(1/0) ELSE "" END FROM users where username='administrator' and SUBSTR(password,1,1)='a') ||'", it will gives you 200 OK because the first character of password is not 'a'. Send to Intruder by right click where we will automate.
- Intruder>positions>click clear §>select '1' beside of password and 'a' and click Add §, do separat each. Payloads> payload set 1 > payload type numbers > sequential 1> 20> 1.
-> which will try for 1, to 20 place of character Position> attack type cluster bomb, which has to do first. Payloads> payload set 2 > payload type brute force> min length 1 max length 1 and click on start attack button. Wait until completed fully.
- Look for 500 status code, which it the exact password value at 1-20 positions and payload 1 place & payload 2 the value of password.
- Go to browser's lab, click on my account, login as administrator you see the lab was solved.

# Evidence
![Conditional error payload triggering a 500 response](images/lab-blind-conditional-errors-payload.png)
![Lab solved, logged in as administrator](images/lab-blind-conditional-errors-solved.png)

# Root Cause
User input from the `TrackingId` cookie is concatenated directly into a SQL query, and the application's error handling leaks information: it responds differently (`500` vs `200`) depending on whether an injected condition is true or false. This difference in behavior is sufficient for an attacker to reconstruct arbitrary data from the database without ever seeing direct query output.

# Remediation
- Use parameterized queries (prepared statements) so user input is always treated as data, never as SQL code.
- Handle database errors gracefully and consistently, so application behavior does not differ based on whether a query succeeded or failed.
- Avoid using user-controlled input (including cookies) in SQL queries without validation.
- Apply least privilege to the database account used by the application.
- Implement rate limiting or anomaly detection to slow down automated blind injection attempts.

# Key Learnings
- Blind SQL injection does not require visible query output. Any observable difference in application behavior, such as an HTTP status code, response time, or response length, can serve as a true/false oracle.
- Conditional error-based techniques (`CASE WHEN ... THEN (error) ELSE (no error) END`) are a reliable way to turn a blind injection point into a binary signal.
- Data can be extracted character by character using functions like `SUBSTR()` combined with conditional logic, though this is slow manually and is typically automated with tools like Burp Intruder.
- Cookies and other non-obvious input points (not just URL parameters and form fields) must also be treated as untrusted and validated.
