---
layout: post
title: Setup Web Server on the Cloud (EC2)
subtitle: Server basic html on a ec2 instance
categories: AWS
tags: [unix, web-applications, devops, aws]
---

Setting up a web server is easier than you may think and I'll show you how!

On an AWS instance, I'll show you how to configure apache (httpd) on a CentOS machine. This involves:

1. installing a firewall and opening the HTTP port to accept traffic
2. installing httpd and setting it up
3. updating the security group of your cloud instance
4. serving an index file

I have a Cent-OS EC2 instance running and all the next commands are running to this instance. I'm not gonna get into detail on how to create a EC2 instance. After your EC2 instance is running, ssh into the instance and you'll have your instance terminal ready!

First I'm get check for system update and install the firewall manager tool. Then we'll enable the firewall and start it.

```bash
sudo yum update
sudo yum install firewalld

sudo systemctl enable firewalld
sudo systemctl start firewalld
# check status
sudo systemctl status firewalld
```

Then we'll add a rule that opens port 80 for HTTP.

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

Then you'll need to navigate to your AWS console and setup a security group to enable http on port 80.

Then we'll install our web server.

```bash
sudo yum install httpd
sudo systemctl start httpd
# check status
sudo systemtcl status httpd
```

If you'll navigate to your Public IP Address of the EC2 instance, you'll be greeted with a `Red Hat Enterprise Linux Test page` showing that Apache is working correctly, of-course this isn't our content and we'll show how we can add our content here now.

In case you are wondering what file we just saw, it's here.

```bash
nano /usr/share/httpd/noindex/index.html
```

## How To Server Our Custom Content -

Firstly we'll need to make a few directories and change permission on them. We'll also create our html file and add some html content on it.

```bash
sudo mkdir -p /var/www/mywebsite.com/html
sudo mkdir -p /var/www/mywebsite.com/log
# Change permissions
sudo chown -R $USER:$USER /var/www/mywebsite.com/html/
sudo chmod -R 755 /var/www/
# Create your html file
sudo nano /var/www/mywebsite.com/html/index.html
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    Hello, World
</body>
</html>

```

Now that the content is ready, we will changes some Apache Configuration.

- Create two directories are described below.

```bash
sudo mkdir /etc/httpd/sites-available /etc/httpd/sites-enabled
sudo nano /etc/httpd/conf/httpd.conf
```

- Add the web serve configuration for our new directory.

    ```bash
    ...
    IncludeOptional sites-enabled/*.conf
    ```

- We also need to setup our VirtualHost.

```bash
sudo nano /etc/httpd/sites-available/mywebsite.com.conf
```

```bash
<VirtualHost *:80>
	ServerName www.mywebsite.com
	ServerAlias mywebsite.com
	DocumentRoot /var/www/mywebsite.com/html
	ErrorLog /var/www/mywebsite.com/log/error.log
	CustomLog /var/www/mywebsite.com/log/requests.log combined
</VirtualHost>
```

- Create a symlink between both the configuration files and restart our server.

```bash
sudo ln -s /etc/httpd/sites-available/mywebsite.com.conf /etc/httpd/sites-enabled/mywebsite.com.conf
sudo systemctl restart httpd
```

That's it! Navigate to your EC2 address and you'll see your webpage. This was just a basic introduction of serving content on the cloud and we all know there is much more you can do!

**Author - [Animesh Singh](https://iamanimesh.tech)**
