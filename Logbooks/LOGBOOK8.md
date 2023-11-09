# CTF - Control The Flag

When entering the website we are presented with a login page. To unlock the vault we need to provide an valid username and password combination.
The goal for this CTF is to log in as valid user. To achieve that, we can use SQL Injection, since the user inputs are not verified, nor used as parameters for the query.

This is the SQL query executed:
```php
    $query = "SELECT username FROM user WHERE username = '".$username."' AND password = '".$password."'";
```

The input used is: `' or 1=1;#`
This input is always true and comments the part where the password is verified.
![login](/Logbooks/img/Week8/login.png)

This is the SQL query that is actually executed:
```php
    $query = "SELECT username FROM user WHERE username = '"' or 1=1;#"' AND password = '".$password."'";
```



![flag](/Logbooks/img/Week8/flag.png)