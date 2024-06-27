---
layout: post
title: Technical Analysis of GLPi Blind SQL Injection
---

This article provides an in-depth technical analysis of CVE-2022-31061, a blind SQL injection vulnerability discovered in GLPI during LDAP authentication. This vulnerability was tested on GLPI version 10.0.1. We explore the exploitation process, pinpointing the exact location and underlying causes of the vulnerability.

You can download the vulnerable version of GLPI from the following URL: https://github.com/glpi-project/glpi/releases/download/10.0.1/glpi-10.0.1.tgz

First, we need to activate LDAP authentication from the settings so that it appears as a login source in the menu, as shown below:

![Pasted image 20240626220558 - Copy](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/3cb74c6b-91aa-4c93-8cb2-5d91b383911d)

LDAP authentication can be easily activated by navigating to **Home > Setup > Authentication > LDAP directories**, and adding a new directory.

![Pasted image 20240626220628 - Copy](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/e9c2dd9f-e7ee-49f0-b068-5a0215fa4ece)

Set the directory to active by selecting 'Yes' and provide an appropriate name for the directory. The name itself doesn't affect the functionality.

![Pasted image 20240626220643 - Copy](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/6eb00d16-3872-4065-b0d9-6deac5d006a5)

We entered a test username and password, and selected `ldap-auth` as the login source we created earlier.

![Pasted image 20240626220547](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/049d7934-2854-45bc-9c4f-d3feac733402)

By capturing the request, we can see how our data is being sent. The most important piece of information is the `auth` parameter.

![Pasted image 20240626220933 - Copy](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/ab484310-abd2-4dcb-8509-5b9710c7358c)

The code snippet shows part of the login process in `/glpi/front/login.php`. Here, we can see that the `auth` parameter is being checked around line 68. Then, on line 88, it’s passed to the `login()` function. The values of the parameters being passed are visible in the ‘Locals’ variables area.

![Pasted image 20240626221255 - Copy](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/62f5c7d1-ddb6-4928-b3d4-a1698b5423bf)

The `login()` function in the `glpi/src/Auth.php` file appears to handle login requests. When the function receives an `auth` parameter with a value like `ldap-1`, it splits the value into an array containing two elements: `"ldap"` and `"1"`. The first element, `"ldap"`, represents the authentication method, which is checked around line 742. If the method is LDAP, the code jumps to another function to proceed with LDAP authentication.

![Pasted image 20240626221517 - Copy](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/47a7c6a3-3c39-4cf4-8d6e-c99925c17f00)

The code creates a new array named `search_params`. This array will contain three elements used in the SQL query.

The three elements in the `search_params` array are:

- `'name'`:  Represents the username entered by the user during login.
- `'authtype'`:  `3 ` indicate that LDAP authentication is being used.
- `'auths_id'`: This element represents the directory ID associated with the LDAP authentication. It is retrieving this value from the `$this->user->fields["auths_id"]` variable.

![Pasted image 20240626221750 - Copy](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/ce84bf4f-8855-490c-b831-05be617e878f)

In the file `glpi/src/CommonDBTM.php`, the function `getFromDBByCrit()` constructs an SQL query using the `crit` parameter. This query is executed directly **without any sanitization**. This means user input might be directly included in the query, making the application vulnerable to SQL injection attacks.

The execution occurs on line 385, with the result being stored in the `iter` variable. The complete SQL query can be observed in the "Local" variables section.

```sql
SELECT `id` FROM `glpi_users` WHERE `name` = 'test' AND `authtype` = '3' AND `auths_id` = '1'
```

![Pasted image 20240626222053 - Copy](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/531dd21c-8539-40cb-8e1b-763619c36020)

We attempted a SQL injection attack using the following payload:

```sql
ldap-1'+UNION+SELECT+SLEEP(5) #+
```

This payload causes the application to pause for 5 seconds, demonstrating that it is vulnerable to SQL injection.

![Pasted image 20240626222253 - Copy](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/83541e34-de1f-4184-be1e-09fdde1edc6f)

![Pasted image 20240626222314 - Copy](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/8e604deb-1943-479a-9bdc-e1f8dfff5193)

Our payload was successfully injected into the SQL query, causing the application to hang for 5 seconds.

![Pasted image 20240626222408 - Copy](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/d731f3dd-fd37-48a6-8a2d-49cf6679110a)

I did not attempt to write a complete exploitation due to time constraints. However, for a fully functional proof of concept that can actually extract data, you can refer to this PoC: [https://github.com/Feals-404/GLPIAnarchy/blob/main/exploits/CVE202231061.go](https://github.com/Feals-404/GLPIAnarchy/blob/main/exploits/CVE202231061.go)

Thank you for reading!.

