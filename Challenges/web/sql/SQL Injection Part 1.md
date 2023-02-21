# SQL Injection Part 1

Category: Web
Difficulty: Medium
Platform: TryHackMe
Status: Rooted/Finished
Tags: SQL-Injection, web

***TABLE OF CONTENTS:***

---

# Description

- **Understand how SQL injection attacks work and how to exploit this vulnerability.**
    - This room is meant as an introduction to SQL injection and demonstrates various SQL injection attacks. It is not meant as a way to learn the SQL language itself. Some previous knowledge of the SQL language is highly recommended.
    - It is possible to display the SQL queries performed by the 
    application on the challenges by enabling "Show Query" in the top-right 
    menu on `http://MACHINE_IP:5000`. It is also possible to display tutorial for the challenges by enabling "Guidance".
    - All the scripts mentioned in the tasks can be viewed and downloaded by visiting the "Downloads" page in the top-left corner on the landing page or visiting `http://MACHINE_IP:5000/downloads/`

## SQL Injection: Part 1

- SQL injection is a technique used by attackers to execute malicious SQL statements, allowing them to steal info and make changes to databases. Our web app has SQL injection vulnerabilities due to developer mistakes.
- Applications will often need dynamic SQL queries to be able to display content based on different conditions set by the user. To allow for dynamic SQL queries, developers often concatenate user input directly into the SQL statement. Without checks on the received input, string concatenation becomes the most common mistake that leads to SQL injection vulnerability.
    - Example: A web application allows users to search for products by entering a keyword. The application then dynamically builds a SQL query using the user input and executes it. Without proper checks, the user input can be manipulated to inject malicious SQL code, potentially allowing an attacker to access sensitive data.
- Without input sanitization, the user can make the database interpret the user input as a SQL statement instead of as data. In other words, the attacker must have access to a parameter that they can control, which goes into the SQL statement. With control of a parameter, the attacker can inject a malicious query, which will be executed by the database. If the application does not sanitize the given input from the attacker-controlled parameter, the query will be vulnerable to SQL injection attack.
    - The following PHP code demonstrates a dynamic SQL query in a login from. The user and password variables from the POST request is concatenated directly into the SQL statement.
        
        ```sql
        $query = "SELECT * FROM users WHERE username='" + $_POST["user"] + "' AND password= '" + $_POST["password"]$ + '";"
        ```
        
        - If the attacker supplies the value `' OR 1=1-- -` inside the username parameter, the query might return more than one user. Most applications will process the first user returned, meaning that the attacker can exploit this and log in as the first user the query returned.
        - The double-dash (--) sequence is a comment indicator in SQL and causes the rest of the query to be commented out. In SQL, a string is enclosed within either a single quote (') or a double quote ("). The single quote (') in the input is used to close the string literal. If the attacker enters `' OR 1=1-- -` in the username parameter and leaves the password blank, the query above will result in the following SQL statement.
        
        ```sql
        SELECT * FROM users WHERE username = '' OR 1=1-- -' AND password = ''
        ```
        
        - If the database executes the SQL statement above, all the users in the users table are returned. Consequently, the attacker bypasses the application's authentication mechanism and is logged in as the first user returned by the query.
            - The reason for using  **-- -** instead of **--** is primarily because of how MySQL handles the double-dash comment style.
            - From a --  sequence to the end of the line. In MySQL, the -- (double-dash) comment style requires the second dash to be followed by at least one white space or control character (such as a space, tab, newline, and so on).

## **SQL Injection 1: Input Box Non-String**

- When a user logs in, the application performs the following query:

```sql
SELECT uid, name, profileID, salary, passportNr, email, nickName, password FROM usertable WHERE profileID=10 AND password = 'ce5ca67...'
```

- When logging in, the user supplies input to the profileID parameter. For this challenge, the parameter accepts an integer, as can be seen here:

```sql
profileID=10
```

- Since there is no input sanitization, it is possible to bypass the login by using any **True** **condition** such as the one below as the ProfileID

```sql
1 or 1=1-- -
```

- Bypass the login and retrieve the flag.
    - Flag ⇒ `THM{dccea429d73d4a6b4f117ac64724f460}`

## **SQL Injection 2: Input Box String**

- This challenge uses the same query as in the previous challenge. However, the parameter expects a string instead of an integer, as can be seen here:

```sql
profileID='10'
```

- Since it expects a string, we need to modify our payload to bypass the login slightly. The following line will let us in:

```sql
1' or '1'='1'-- - || (in programming every quoted thing called string.)
```

- Bypass the login and retrieve the flag.
    - Flag ⇒ `THM{356e9de6016b9ac34e02df99a5f755ba}`

## **SQL Injection 3 and 4: URL and POST Injection**

- Here, the SQL query is the same as the previous one:

```sql
SELECT uid, name, profileID, salary, passportNr, email, nickName, password FROM usertable WHERE profileID='10' AND password='ce5ca67...'
```

- But in this case, the malicious user input cannot be injected directly into the application via the login form because some client-side controls have been implemented:

```jsx
function validateform() {
    var profileID = document.inputForm.profileID.value;
    var password = document.inputForm.password.value;

    if (/^[a-zA-Z0-9]*$/.test(profileID) == false || /^[a-zA-Z0-9]*$/.test(password) == false) {
        alert("The input fields cannot contain special characters");
        return false;
    }
    if (profileID == null || password == null) {
        alert("The input fields cannot be empty.");
        return false;
    }
}
```

- The JavaScript code above requires that both the profileID and the password only contains characters between a-z, A-Z, and 0-9. Client-side controls are only there to improve the user experience and is in no way a security feature as the user has full control over the client and the data it submits. For example, a proxy tool such as Burp Suite can be used to bypass the client side JavaScript validation `https://portswigger.net/support/using-burp-to-bypass-client-side-javascript-validatio`)

### **SQL Injection 3: URL Injection**

- This challenge uses a GET request when submitting the login form
- The login and the client-side validation can then easily be bypassed by going directly to this URL:

```
http://10.10.27.117:5000/sesqli3/login?profileID=-1' or 1=1-- -&password=a
```

- The browser will automatically urlencode this for us. Urlencoding is needed since the HTTP protocol does not support all characters in the request. When urlencoded, the URL looks as follows:

```
http://10.10.27.117:5000/sesqli3/login?profileID=-1%27%20or%201=1--%20-&password=a
```

- The %27 becomes the single quote (') character and %20 becomes a blank space.
    - Flag ⇒ `THM{645eab5d34f81981f5705de54e8a9c36}`

### **SQL Injection 4: POST Injection**

- When submitting the login form for this challenge, it uses the HTTP POST method. It is possible to either remove/disable the JavaScript validating the login form or submit a valid request and intercept it with a proxy tool such as Burp Suite and modify it:
    - Flag ⇒ `THM{727334fd0f0ea1b836a8d443f09dc8eb}`

---