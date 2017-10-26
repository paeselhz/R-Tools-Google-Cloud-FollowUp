# Google Cloud Platform - Configuring R Tools with NGINX as Front-End Server
The follow up guide of [Google Cloud Platform - How-To deploy Shiny Server and RStudio Server](https://github.com/paeselhz/RStudio-Shiny-Server-on-GCP), introducing NGINX tools to consolidate a website hosting RStudio Server and Shiny Server on the web.

# Introduction

Differently from the previous version of the tutorial, which can be acessed [here](https://github.com/paeselhz/RStudio-Shiny-Server-on-GCP), this tutorial is more complex, and requires an intermediate knowledge of linux based code, and how permissions work. Also, this guide requires the user to already be the owner of a public domain on the web, which can be bought from multiple sources, as Google Domains, or GoDaddy, or any other of your choice.

This tutorial was developed using code run at a Linux Trusty (14.04) Virtual Machine.

### Disclaimer

The actions of this tutorial imply knowledge from the user, and are more complicated to perform. Take care not to brick your Virtual Machine by running code unadvertedly!

# The First Steps

To complete this tutorial, you'll need to have the following:
- Static IP connected to your VM Instance (which can be done viewing the tutorial [here](/docs/vm_external_ip.md)).
- A domain in your name, and connected to the external IP of your VM Instance.
- Time to understand the way things are done to execute them easily.

Now, open the shell of your VM instance, to install a few programs in order to enable a secure firewall, install the nginx to run as a front-end server, and clone a git to help create an SSL certificate for your website, giving the user the `https://` authentication.

The programs needed will be downloaded with the following lines of code:

```
sudo apt-get install ufw nginx git
```

Then, clone the git that helps with the creation of the SSL certificate.

```
cd /
sudo git clone https://github.com/letsencrypt/letsencrypt
```

Now, let's set the three main ports used by this VM instance by allowing them into ufw: 22 (SFTP), 443 (Secure Connections), 80 (Default HTTP Connections)

```
sudo ufw allow https
sudo ufw allow http
sudo ufw allow 22/tcp
```

In order to start the configuration and the creation of the SSL certificate, you need to stop any NGINX process running at your vm instance and enabling the firewall by running: 

```
sudo service nginx stop
sudo ufw enable
```

Now that you have all setup, you can go to your domain administrator page of your choice, and set the A argument of your domain with the Host `@` and the Set pointing to your `EXTERNAL_IP_ADDRESS` of your machine. Also, now is a good time to set a subdomain to connect to the RStudio directly like`rstudio.yourdomain.com`.

# SSL certificates for both yourdomain.com and rstudio.yourdomain.com

You have set both, domain and subdomain, pointing to the EXTERNAL_IP_ADDRESS of your VM instance. Now, you need to create and download the SSL certificates for both, in order to guarantee a secure connection from user to server. To do so, you need to execute the following code:

```
cd /letsencrypt
sudo ./letsencrypt-auto certonly --standalone -d yourdomain.com -d www.yourdomain.com
sudo ./letsencrypt-auto certonly --standalone -d rstudio.yourdomain.com
```

If all went correctly, both executions should have returned successfully, with the keys stored at `/etc/letsencrypt/live/yourdomain.com` and `/etc/letsencrypt/live/rstudio.yourdomain.com`. Now it's time to reverse proxy the connection to allow both RStudio and Shiny Server to be connected securely via HTTPS.

## Configuring NGINX reverse proxy

To allow the HTTP upgrades and Connection upgrades required to perform the revese proxy, you will need to edit the configuration file of the nginx server. To do that you need to run `sudo nano /etc/nginx/nginx.conf`. And near the end of the configuration file (around line 60), you'll insert the snippet below **(beware to insert that before the Virtual Host Configs)**.

```
##
# Map reverse proxy settings for RStudio and Shiny Server
##
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}
 
##
# Virtual Host Configs
##
```

# IN DEVELOPMENT!
