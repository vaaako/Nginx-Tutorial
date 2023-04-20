# About
This is a simple tutorial<br>
I will only be explaining how to configure a `Nginx` server, without explaining too much of the technical details

*(This tutorial is a simplified and error-corrected version of [this](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04) tutorial)*
>*(No SSL included)*


# Installing and adjusting the Firewall
Install `Nginx`
```sh
sudo apt install nginx
```

It’s necessary to modify the `Firewall` settings to allow outside access to the default web ports:
By typing:
```sh
sudo ufw app list
```

You must see something like this:
```
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
```

- **Nginx Full:** This profile opens both **port 80** *(normal, unencrypted web traffic) and **port 443** (TLS/SSL encrypted traffic)*
- **Nginx HTTP:** This profile opens only **port 80** *(normal, unencrypted web traffic)*
- **Nginx HTTPS:** This profile opens only **port 443** *(TLS/SSL encrypted traffic)*


It is recommended that you enable the most restrictive profile that will still allow the traffic you’ve configured. Right now, we will only need to allow traffic on **port 80**.
```sh
sudo ufw allow 'Nginx HTTP'
```

Then you can verify:
```sh
sudo ufw status
```

If you get a `Status: inactive` message when running the `ufw` status command, it means that the `Firewall` is not yet enabled on the system

It's very important to enable your `Firewall`<br>
To enable your `Firewall` type:
```sh
sudo ufw enable
```

Then try to verify again:
```sh
sudo ufw status
```

# Checking
By typing
```sh
sudo systemctl status nginx
```

You can check if `Nginx` is running<br>
But let's see the page!

First you need you public IPv4, to get this you can google "Wotz my ip?", or, typing this:
```sh
hostname -I
```
>(*IPv4 is the first one*)

Enter it into your browser’s address bar: `http://your_server_ip`
(*The server runs in Port 80*)
>Alternatively, you can access the page by just typing in your browser `localhost` or `127.0.0.1`



# Setting up Virtual Hosts (cool part)
When using the Nginx web server, server blocks (similar to virtual hosts in Apache) can be used to encapsulate configuration details and host more than one domain from a single server<br>
The default page (the one we saw before) is located in `/var/www/html` directory. But that's too boring (if you want just one simple page, this is fine), let's learn how to make a new one

First we create the domain directory: *(obviously you will replace `your_domain` with your domain name. Without http/s, www., .com, etc.)*
```sh
sudo mkdir -p /var/www/your_domain/html # -p flag to create any necessary parent directories:
```

Next, assign ownership of the directory with the $USER environment variable:
```sh
sudo chown -R $USER:$USER /var/www/your_domain/html
```

To ensure that your permissions are correct and allow the owner to read, write, and execute the files while granting only read and execute permissions to groups and others, you can input the following command:
```sh
sudo chmod -R 755 /var/www/your_domain
```

Next, create a sample index.html page using nano or your favorite editor:
```sh
sudo nano /var/www/your_domain/html/index.html
```

Add this sample inside:
```html
<html>
    <head>
        <title>Welcome to Your_domain!</title>
    </head>
    <body>
        <h1>Success! The your_domain virtual host is working!</h1>
    </body>
</html>
```


In order for `Nginx` to serve this content, it’s necessary to create a server block with the correct directives. Instead of modifying the default configuration file directly, let’s make a new one at `/etc/nginx/sites-available/your_domain`:
```sh
sudo nano /etc/nginx/sites-available/your_domain.conf
```

Paste in the following configuration block, which is similar to the default, but updated for our new directory and domain name:

```nginx
server {
    listen 80;
    listen [::]:80;

    root /var/www/your_domain/html; # it can be just "/var/www/your_domai?" if you want
    index index.html index.htm index.nginx-debian.html;

    server_name your_domain www.your_domain;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

**Notice:** that we’ve updated the `root` to our new directory, and the `server_name` to the domain name

**WARNING:** if you pretend to use `localhost` change or add to `server_name`: `localhost 127.0.0.1;`. **Example:** `server_name your_domain www.your_domain localhost 127.0.0.1;`
>**Tip:** You can have `Apache` running on `localhost` and `Nginx` running on `127.0.0.1` ou vice versa *(But the correct is to run in different ports)*

# Almost done
Let’s enable the file by creating a link from it to the sites-enabled directory, which Nginx reads from during startup:
```sh
sudo ln -s /etc/nginx/sites-available/your_domain.conf /etc/nginx/sites-enabled/
```

>Nginx uses a common practice called **symbolic links**, or **symlinks**, to track which of your server blocks are enabled. Creating a **symlink** is like creating a shortcut on disk, so that you could later delete the shortcut from the `sites-enabled` directory while keeping the server block in sites-available if you wanted to enable it


Two server blocks are now enabled and configured to respond to requests based on their listen and `server_name` directives *(you can read more about how Nginx processes these directives here)*:

- **your_domain:** Will respond to requests for `your_domain` and `www.your_domain`.
- **default:** Will respond to any requests on **port 80** that do not match the other two blocks.

To avoid a possible hash bucket memory problem that can arise from adding additional server names, it is necessary to adjust a single value in the `/etc/nginx/nginx.conf` file. Open the file:

```sh
sudo nano /etc/nginx/nginx.conf
```

Uncomment line `24`:
```
server_names_hash_bucket_size 64;
```

Test to make sure that there are no syntax error in any `Nginx` file:
```sh
sudo nginx -t
```

Finally, restart `Nginx`:
```sh
sudo systemctl restart nginx
```


<!--
# new-nginx.sh
For each new page you need to repeat this whole new process, that's why I made `new-apache.sh`

To use just type:
```sh
sudo bash new-nginx.sh your_domain
```
-->


# Nginx Files and Directories
## Content
- `/var/www/html`: The actual web content, which by default only consists of the default `Nginx` page you saw earlier, is served out of the `/var/www/html` directory. This can be changed by altering `Nginx` configuration files.

## Server Configuration
- `/etc/nginx`: The `Nginx` configuration directory. All of the Apache configuration files reside here.
- `/etc/nginx/nginx.conf`: The main `Nginx` configuration file. This can be modified to make changes to the `Nginx` global configuration. This file is responsible for loading many of the other files in the configuration directory.
- `/etc/nginx/sites-available/`: The directory where per-site virtual hosts can be stored. `Nginx` will not use the configuration files found in this directory unless they are linked to the `sites-enabled` directory.
- `/etc/nginx/sites-enabled/`: The directory where enabled per-site virtual hosts are stored. Typically, these are created by linking to configuration files found in the `sites-available` directory.
- `/etc/nginx/snippets`: This directory contains configuration fragments that can be included elsewhere in the `Nginx` configuration. Potentially repeatable configuration segments are good candidates for refactoring into snippets.

## Server Logs
- `/var/log/nginx/access.log`: Every request to your web server is recorded in this log file unless Nginx is configured to do otherwise.
- `/var/log/nginx/error.log`: Any Nginx errors will be recorded in this log