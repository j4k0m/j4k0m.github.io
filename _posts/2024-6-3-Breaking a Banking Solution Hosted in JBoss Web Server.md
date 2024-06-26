---
layout: post
title: Breaking a Banking Solution Hosted on a JBoss Web Server
---

I was tasked with testing a banking application. Despite numerous prior penetration tests, I was surprised by the extent of critical vulnerabilities and the insecure implementation of authorization. The mission extended over four days, yet I breached the server within the first few hours. I proceeded with the pentest in white-box mode.

This blog won't delve deeply into technical details. Instead, it aims to narrate stories and highlight intriguing discoveries made during my missions. You may glean a few insights along the way.

## Good functionality, bad implementation

While exploring the application, I typically examine interesting functions and consider potential back-end code vulnerabilities. During my navigation, I discovered a function that generates a PDF report and saves it on the local server at a specific path defined in the application's configuration.

Once the report is generated, a new button appears, allowing users to download the report. Clicking this button sends a request to an endpoint, which responds with an array of bytes representing the PDF file's data. This data is then converted client-side into a downloadable and readable PDF file.

The request contained two notable parameters: `generated_filename` and `report_location`. The `report_location` parameter specifies the folder where reports are stored, and the `generated_filename` is the name of the file within that folder.

This function had a simple path traversal vulnerability. By changing `report_location` to `/etc` and `generated_filename` to `passwd`, I received a longer and different response in the form of an array of bytes.

I then created a Python script to automate the entire process, sending the request and converting the bytes into readable text.

Upon reviewing the source code, I found that the function concatenates both parameters and passes the resulting value to the `FileInputStream` function.

Since I could read local files, the first thing I did was check all the users on the machine by reading the `passwd` file. Next, I examined each user's `.bash_history` in hopes of finding interesting data. I successfully read the bash history of the JBoss user, who installed and managed the JBoss web server. Normally, I don't find much critical information in these files, but this time the system administrator had used `jboss-cli` and included both the username and password for the JBoss console in the command. This allowed me to log into the JBoss console as a superuser.

Quickly, I deployed my own JSP webshell in WAR format and executed commands as the JBoss user. Then, I downloaded both the API and application source code for further analysis.

## The JBoss console password is so weak!!

I couldn't ignore the JBoss console password; it was too weak. I kept thinking that an attacker, even from a black-box position, could potentially break into the application.

The issue is that the password complies with the current password policy. If an attacker knows this policy, they could use tools like `crunch` to generate the password or simply brute force it manually. In this case, the password included "jboss" followed by just three integers, making it especially vulnerable.

## HashMap based authorization but no different between JWTs

While analyzing the application's source code, I discovered a concerning pattern. After each request, the application retrieves the JWT from the `Authorization` header and performs a `find` operation in a hashmap. Strangely, the application does not parse or verify the JWT; it merely checks for its existence in the hashmap.

The application doesn't cryptographically verify the validity of the JWT and does not ensure the JWT is tied to a specific user. Instead, it checks if the JWT is present in the hashmap. If the JWT exists in the hashmap, it grants authorization; if not, it displays a "token invalid" error.

This means there is no role-based implementation. If you possess a JWT that exists in the hashmap, you can access all endpoints.

I searched for `PostMapping`, which maps POST request endpoints to backend functions. Using a simple Bash script with curl, I sprayed my JWT to these endpoints and looked for responses indicating successful authorization by checking for a certain response.

## What else would I have done?

If I had more time, I might have attempted to bypass the authorization altogether. Perhaps I could have tried to force the application to leak the hashmap contents or manipulate the web server into generating a heap dump from which I could extract the authorization tokens. However, this is purely speculative, and we will never know for sure.
