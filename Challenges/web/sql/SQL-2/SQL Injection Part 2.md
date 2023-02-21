# SQL Injection Part 2

Category: Web
Difficulty: Medium
Platform: TryHackMe
Status: Rooted/Finished
Tags: SQL-Injection, web

***TABLE OF CONTENTS:***

---

# **SQL Injection Attack on an UPDATE Statement**

- If a SQL injection occurs on an UPDATE statement, the damage can be much more severe as it allows one to change records within the database. In the employee management application

![profile.png](SQL%20Injection%20Part%202%20823ada8f8d6c4dcca401769441cc5b85/profile.png)

- This edit page allows the employees to update their information, but they do not have access to all the available fields, and the user can only change their information. If the form is vulnerable to SQL injection, an attacker can bypass the implemented logic and update fields they are not supposed to, or for other users.
- We will now enumerate the database via the UPDATE statement on the profile page. We will assume we have no prior knowledge of the database. By looking at the web page's source code, we can identify potential column names by looking at the name attribute. The columns don't necessarily need to be named this, but there is a good chance of it, and column names such as "email" and "password" are not uncommon and can easily be guessed.

![sql.png](SQL%20Injection%20Part%202%20823ada8f8d6c4dcca401769441cc5b85/sql.png)

- To confirm that the form is vulnerable and that we have working column names, we can try to inject something similar to the code below into the nickName and email field:

```sql
asd',nickName='test',email='hacked
```

- When injecting the malicious payload into the nickName field, only the nickName is updated. When injected into the email field, both fields are updated
- The first test confirmed that the application is vulnerable and that we have the correct column names. If we had the wrong column names, then non of the fields would have been updated. Since both fields are updated after injecting the malicious payload, the original SQL statement likely looks something similar to the following code:

```sql
UPDATE <table_name> SET nickName='name', email='email' WHERE <condition>
```

- With this knowledge, we can try to identify what database is in use. There are a few ways to do this, but the easiest way is to ask the database to identify itself. The following queries can be used to identify MySQL, MSSQL, Oracle, and SQLite:

```sql
# MySQL and MSSQL
',nickName=@@version,email='
# For Oracle
',nickName=(SELECT banner FROM v$version),email='
# For SQLite
',nickName=sqlite_version(),email='
```

- Injecting the line with "`sqlite_version()`" into the nickName field shows that we are dealing with SQLite and that the version number is 3.27.2
- Knowing what database we are dealing with makes it easier to understand how to construct our malicious queries. We can proceed to enumerate the database by extracting all the tables. In the code below, we perform a sub-query to fetch all the tables from database and place them into the nickName field. The sub-query is enclosed inside parenthesis. The `group_concat()` function is used to dump all the tables simultaneously.
    - The `group_concat()` function returns a string which is the concatenation of all non-NULL values of X. If parameter Y is present then it is used as the separator between instances of X. A comma (",") is used as the separator if Y is omitted. The order of the concatenated elements is arbitrary.
        
        ```sql
        ',nickName=(SELECT group_concat(tbl_name) FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%'),email='
        ```
        
    - We can then continue by extract all the column names from the usertable:
        
        ```sql
        ',nickName=(SELECT sql FROM sqlite_master WHERE type!='meta' AND sql NOT NULL AND name ='usertable'),email='
        ```
        
    - And as can be seen below, the usertable contains the columns: UID, name, profileID, salary, passportNr, email, nickName, and password:
- By knowing the names of the columns, we can extract the data we want from the database. For example, the query below will extract profileID, name, and passwords from usertable. The subquery is using the `group_concat()` function to dump all the information simultaneously, and the [||](https://sqlite.org/lang_expr.html#operators) operator is "concatenate" - it joins together the strings of its operands (`sqlite.org`).

```sql
',nickName=(SELECT group_concat(profileID || "," || name || "," || password || ":") from usertable),email='
```

- After having dumped the data from the database, we can see that the password is hashed. This means that we will need to identify the correct hash type used if we want to update the password for a user. Using a hash identifier such as hash-identifier, we can identify the hash as SHA256

## **Tasks**

- Log in to the "SQL Injection 5: UPDATE Statement" challenge and exploit the vulnerable profile page to find the flag. The credentials that can be used are:
    - profileID: `10`
    - password: `toor`
        - `CREATE TABLE secrets ( id integer primary key, author integer not null, secret text not null )`
            
            ```sql
            ',nickName=(SELECT group_concat(profileID || "," || author || "," || secret || ":") from secrets),email='
            ```
            
        - Log in with admin account ⇒ `99:Password123`
        - Flag ⇒ `THM{b3a540515dbd9847c29cffa1bef1edfb}`