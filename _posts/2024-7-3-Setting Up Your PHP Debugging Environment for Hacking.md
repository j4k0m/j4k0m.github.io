---
layout: post
title: Setting Up Your PHP Debugging Environment for Hacking
---

# Setting Up Your PHP Debugging Environment for Hacking

Due to popular demand, I've decided to create a blog post on how I set up my PHP debugging environment for hacking PHP applications.

In this guide, I will walk you through my setup, which includes using an Ubuntu VPS for the web server and Xdebug. Additionally, I use VSCode as my debugging tool and Burp Suite for testing. Let's get started!

You can get a $200 free credit on DigitalOcean by using this link. Alternatively, you can set up your own Ubuntu server: https://try.digitalocean.com/freetrialoffer/

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/8df080cc-6c9e-4da9-a0f1-fc85191daf10)

We then create an Ubuntu server and set up our SSH public key.

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/92f05567-93e4-4390-9fa2-e831fae5a2e2)

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/0de347c3-aedf-4b4c-9818-940f6e3af052)

## Installing PHP, Apache2 and Xdebug

We will need to install PHP and the Apache2 web server to host our testing application.

```bash
sudo apt update
sudo apt upgrade
sudo apt install apache2 php libapache2-mod-php php-xdebug
```

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/83209a22-84c9-4985-b322-440246ae4658)

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/3bfaf14b-6f88-4f60-95f3-fcb514b07169)

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/85db032a-dae3-4f3a-8640-1382e446ab91)

Next, we create our testing PHP file in the `/var/www/html/` directory.

```php
<?php

echo "test";
```

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/d9bc647a-753e-41a4-abbe-e22d6ebaa419)

Verify that everything is installed by running `php -v`.

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/967b2b8b-9cf2-4235-9a51-0057a5dc448e)

Next, we need to set up our Xdebug configuration in `/etc/php/8.3/mods-available/xdebug.ini`.

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/54d1274f-49be-4298-825a-b37aa8fe063a)

```
zend_extension=xdebug.so
xdebug.mode = debug
xdebug.start_with_request = yes
xdebug.client_port = 9000
xdebug.client_host = 164.90.218.170
```

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/0f5f4731-a337-4306-83ba-53f5e9b4e531)

Be sure to replace `192.168.1.52` with the IP address of your Linux machine.

## Bringing it all together..

In your VSCode, install the following plugin. This will allow us to connect to our Linux server and control it via VSCode.

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/c8b96a8b-5acb-45bd-8896-4bc223d3d276)

Once it is installed it is the time to add our VPS server:

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/e6b1d663-be24-4470-98ba-6433563237b8)

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/24b21b50-1f37-4523-92c7-98e86a25d635)

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/b95edd9b-a428-4b74-bd73-79b816244997)

Once the plugin is added, you will find your server listed in the plugin menu. If it doesn't appear, try refreshing the menu. Go ahead and connect to your server.

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/4300b04b-2c99-4959-8b34-39d13650c0d0)

Once you are connected, install the following plugin to add support for Xdebug in our VSCode.

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/72f3bbdf-f149-4552-ad60-d15ef815cea9)

Now open the application path. In my case, it is `/var/www/html` since that is where we created the testing PHP file.

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/20ad6f6a-2030-40e9-8c1f-6b6ecabf6c72)

Next, navigate to the Debug menu and create the `launch.json` file, which will be stored in `/var/www/html/.vscode`.

This file will contain our VSCode debug configuration, which will be used to connect to our Xdebug.

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/4d142835-d15d-447b-af71-eeb7969aa5e9)

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/48389c90-5a6f-4bbd-9141-31b2a3b018b5)

```json
{
    "version": "0.2.0",
    "configurations": [
      {
        "name": "Listen for XDebug",
        "type": "php",
        "request": "launch",
        "hostname": "164.90.218.170",
        "port": 9000,
        "pathMappings": {
          "/var/www/html/": "${workspaceFolder}/",
        }
      }
    ]
}
```

Make sure to replace `hostname` with your machine's IP address and add the PHP application path to `PathMappings`.

Now that everything is set up, all that's left is to set our breakpoints and start the debugger.

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/2082ebe9-95a4-46f2-a815-32da5c1fd003)

We set the breakpoint at line 3 where the `echo`  is. Once we visit the page, we can see that the breakpoint is triggered.

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/085d3c04-23fa-4f77-8d2e-4def216e8b46)

![image](https://github.com/j4k0m/j4k0m.github.io/assets/48088579/df0d8b4e-28c1-4140-9e2f-d0fac8ffdc80)

Thanks for reading, and happy hacking!









