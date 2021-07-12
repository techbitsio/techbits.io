<!--- META
title=How to compile NGINX and ModSecurity on Ubuntu server 21.04
publish_date=20210712
description=ModSecurity is a powerful Web Application Firewall, but requires compiling to run with NGINX. Here's how to get it running on Ubuntu 21.04.
author=techbitsio
tags=security, nginx
header_image=red-backlit-keyboard.jpg
comments=9
-->

ModSecurity is an open-source web application firewall, and although it was originally developed for Apache, it can now sit in front of NGINX and even IIS web servers.

Where a software firewall will sit on a host and protect against network intrusions, a web application firewall exists only to filter HTTP traffic and protect against specific threats.

This post will cover how to compile ModSecurity and NGINX, and configure both to run on Ubuntu 21.04. Previous versions of ModSecurity still required Apache components to compile, however these dependencies are no longer required.

To follow along, you'll need a user account with sudo privileges. If you're using the root account, you can run the commands as they are, or remove `sudo`. Up to you.

There are a few points where specific versions of packages are downloaded - make sure you check for current versions and substitute the commands accordingly.

You can also jump to the end to find the complete build script.

## Compile libmodsecurity
Unfortunately, we can't just install everything here. We need to compile it first, but before that we need to install some pre-requisites:

```shell
sudo apt-get update
sudo apt-get install g++ flex bison curl doxygen libyajl-dev libgeoip-dev libtool dh-autoreconf libcurl4-gnutls-dev libxml2 libpcre++-dev libxml2-dev make -y
```

Now we download ModSecurity from the GitHub repo:

```shell
cd /opt/
sudo git clone https://github.com/SpiderLabs/ModSecurity
cd ModSecurity/
sudo ./build.sh
sudo git submodule init
sudo git submodule update
```

Finally, we configure and compile (make) ModSecurity:

```shell
sudo ./configure
sudo make ## This step can take 10+ minutes to run!
sudo make install 
```

## Compile the ModSecurity NGINX module
Next, we need to download and compile the NGINX ModSec module, but before doing this, make sure you have NGINX installed. If you're not sure, run `nginx -v`. You should see a version number if it's installed:

```shell
$ nginx -v
nginx version: nginx/1.18.0 (Ubuntu)
```

If not, install it with:

```shell
sudo apt-get install nginx -y
```

*You can also download and compile NGINX yourself, but that's a how-to for another day*

Let's install the pre-reqs and download the module:

```shell
sudo apt-get install libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev -y
cd /opt/
sudo git clone --depth 1 https://github.com/SpiderLabs/ModSecurity-nginx.git
```

At this point we need to download the NGINX source, but the version you download needs to match the version you have installed. Again, `nginx -v` will give you the version, so replace the number in the commands below if required:

```shell
sudo wget http://nginx.org/download/nginx-1.18.0.tar.gz
sudo tar zxvf nginx-1.18.0.tar.gz
sudo rm nginx-1.18.0.tar.gz
```

Now we can compile the module and copy it to the right directory:

```shell
cd nginx-1.18.0
sudo ./configure --with-compat --add-dynamic-module=/opt/ModSecurity-nginx
sudo make modules

sudo cp objs/ngx_http_modsecurity_module.so /usr/share/nginx/modules
cd ~/
```

## Add modsecurity to the NGINX config
To tell NGINX to use the ModSec addon, add `load_module modules/ngx_http_modsecurity_module.so;` to `/etc/nginx/nginx.conf`, within the `events` section. You can do this manually, or run the command:

```shell
sudo sed -i 's/events {/load_module modules\/ngx_http_modsecurity_module.so;\n\nevents {/' /etc/nginx/nginx.conf
```

## Create a ModSec config
We'll use the recommended config from the SpiderLabs/ModSecurity repo, then we'll move it into the `/etc/nginx/modsec` directory as `modsecurity.conf`:

```shell
sudo mkdir /etc/nginx/modsec
sudo wget -P /etc/nginx/modsec/ https://raw.githubusercontent.com/SpiderLabs/ModSecurity/v3/master/modsecurity.conf-recommended
sudo mv /etc/nginx/modsec/modsecurity.conf-recommended /etc/nginx/modsec/modsecurity.conf
```

The unicode.mapping file seems to sometimes get missed by the OS, so we'll copy it into the same directory as the config file:

```shell
sudo cp /opt/ModSecurity/unicode.mapping /etc/nginx/modsec
```

## Optional: Turn SecRuleEngine On
By default, ModSec will detect and log broken rules, but it won't block any traffic. To change this, change `SecRuleEngine DetectionOnly` to `SecRuleEngine On` in `/etc/nginx/modsec/modsecurity.conf`. Or run this command:

```shell
sudo sed -i 's/SecRuleEngine DetectionOnly/SecRuleEngine On/' /etc/nginx/modsec/modsecurity.conf
```

## Add ModSec rules
At this point, although ModSec is installed, we don't have any rules. Fortunately, the OWASP Foundation releases a ruleset that anyone can use as a starting point, so for the purposes of this guide, that's what we'll add.

You'll need to find the current version number [here](https://github.com/coreruleset/coreruleset/releases/latest) and add it into the commands below. In this example, it's v3.3.2.

```shell
cd ~/
wget https://github.com/coreruleset/coreruleset/archive/refs/tags/v3.3.2.tar.gz

tar -xzvf v3.3.2.tar.gz
rm v3.3.2.tar.gz

sudo mv coreruleset-3.3.2 /usr/local
sudo cp /usr/local/coreruleset-3.3.2/crs-setup.conf.example /usr/local/coreruleset-3.3.2/crs-setup.conf
```

## Enable modsecurity and add rules file to default site
For this guide, we're adding the rules to the `default` site, so change the commands to reflect the path to your site if you're using something different.

Use a text editor to edit `/etc/nginx/sites-enabled/default` and under `server_name _;` add:

```shell
modsecurity on;
modsecurity_rules_file /etc/nginx/modsec/main.conf;
```

Or run this command:

```shell
sudo sed -i 's/server_name _;/server_name _;\n\tmodsecurity on;\n\tmodsecurity_rules_file \/etc\/nginx\/modsec\/main.conf;/' /etc/nginx/sites-enabled/default
```

We've told NGINX that `/etc/nginx/modsec/main.conf` is where our rules are, but we've not created that file yet.

We'll create it and include the modsec config file, the crs-setup config file and all of the rules files by using a line with a wildcard: `*.conf`.

```shell
sudo touch /etc/nginx/modsec/main.conf
echo "Include /etc/nginx/modsec/modsecurity.conf" | sudo tee -a /etc/nginx/modsec/main.conf
echo "Include /usr/local/coreruleset-3.3.2/crs-setup.conf" | sudo tee -a /etc/nginx/modsec/main.conf
echo "Include /usr/local/coreruleset-3.3.2/rules/*.conf" | sudo tee -a /etc/nginx/modsec/main.conf
```

## Restart NGINX and Test

It's easy to do all of the above and find it doesn't work because you've not reloaded the NGINX config. To do that, run:

```shell
sudo nginx -s reload
```

To test that the new rules are being enforced, we can use the following curl command:

```shell
curl -H "User-Agent: nessustest" http://localhost/
```

The above uses a User-Agent which is included in the `rules/scanners-urls.data` file, so ModSec blocks it, and we get the following output:

```html
<html>
    <head>
        <title>403 Forbidden</title>
    </head>
    <body>
        <center><h1>403 Forbidden</h1></center>
        <hr>
        <center>nginx/1.18.0 (Ubuntu)</center>
    </body>
</html>
```

## Wrapping up

Adding a web application firewall to your site can add an important extra security layer, especially for those running custom and less well known applications, where security may not have been a consideration during development. Even for Wordpress, Joomla or Drupal, new vulnerabilities get found periodically so a WAF might prevent an attack while you're waiting for a patch.

Even static sites can benefit by blocking certain user-agents from sniffing around, rate limiting against DOS attacks (although not a real substitution for DDoS protection), or even just by logging suspicious traffic to enable monitoring from a different application.

If you've made it this far, thanks for sticking with it! I've tested everything numerous times, but as always, please leave a comment or submit an edit if there's anything missing or there are any errors.

Below is a script that will automate everything covered above (plus it'll automatically get the right version of the Modsec NGINX module, and will work out the latest OWASP CRS download). For errors/suggestions with that, please add a comment directly to the [gist](https://gist.github.com/techbitsio/a564f7b7951de7c5963d78f0ffd08f69).

<script src="https://gist.github.com/techbitsio/a564f7b7951de7c5963d78f0ffd08f69.js"></script>

*Header image by [Artem Smus](https://unsplash.com/photos/T6sqjmY094M)*