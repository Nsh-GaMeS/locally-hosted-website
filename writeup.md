# Intro
This is a guide on how to configure and run an nginx web server locally to host a webserver on your local machine/server. This guide will cover the installation of nginx, configuration of the server, and how to test it locally. The goal is to have a basic understanding of how to set up a website using nginx and how to access it from anywhere!

# Prerequisites
- A local machine or server running a Linux-based operating system (I will be using my server running a virtual machine with Ubuntu).
- A domain name (many registrars, many options, I will be using Namecheap for this guide).
- update your system (sudo apt update && sudo apt upgrade -y)

# Lets begin!
I'm going to assume that you've already spun up a virtual machine with Ubuntu on it. If you haven't, you can do so using VirtualBox or any other hypervisor of your choice. I'll try and add a guide for that later :).

### Working the domain

First things first, Lets change our DNS provider to Cloudflare. This is optional, but I find it easier to manage my Domains with Cloudflare. You can sign up for a free account at [Cloudflare](https://www.cloudflare.com/). Once you've signed up, you can add your domain name to Cloudflare and change your nameservers to the ones provided by Cloudflare.

This will allow you to manage your DNS records from Cloudflare's dashboard. Which is much easier than managing them from your registrar's dashboard.

Once logged in, cloudflare will ask you to add your domain name. Once you've added your domain name, Cloudflare will provide you with two nameservers. You will need to go to your registrar's dashboard and change your nameservers to the ones provided by Cloudflare. This may take a few minutes to reconfigure, so be patient.

Once you've changed your nameservers, you can go back to Cloudflare's dashboard and click on your domain name. This will take you to the DNS settings for your domain name. You should see a list of DNS records for your domain name. If you don't see any records, don't worry, we'll add them later.


Ok, in the mean time, on the cloudflare dashboard, we need to set up a DNS record for our domain name. This will allow us to point our domain name to our server's IP address. You can do this by going to the DNS settings for your domain name and adding an "A" record. The "A" record should point to your server's IP address. Since were running a local server, we can use our public IP address. You can find your public IP address by running the following command in your terminal:
```bash
curl ifconfig.me
```
This will return your public IP address. Copy this address and save. Don't worry about the IP address changing, we will set up a dynamic DNS later on in this guide. For now, just copy your public IP address and save it for later.

Once you have your public IP address, on the DNS settings page, you should see a button that says "Add record". Click on this button and select "A" from the dropdown menu. In the "Name" field, enter your domain name (e.g. example.com). In the "IPv4 address" field, enter your public IP address. Leave the TTL as "Auto" and click on the "Save" button. This will add an "A" record for your domain name that points to your server's IP address.

You should now see the "A" record in the list of DNS records for your domain name. If you don't see it, don't worry, it may take a few minutes to propagate.

__Note:__ If you want to use a subdomain (e.g. hello.website.com), you can enter the subdomain in the "Name" field (e.g. hello). This will create an "A" record for the subdomain that points to your server's IP address.

OK, now that we have our DNS record set up, we can check if it's working by running the following command in your terminal:
```bash
ping yourdomain.com
```
This should show you requests being sent to your server's IP address. If you see requests being sent to your server's IP address, congratulations! Your DNS record is working and pointing to your server's IP address.

### Finally, server setup

#### Connecting to your server. 
Alright, so first things first, Nobody likes putting in their username and password for everything, so let's generate some SSH keys for our server. This will allow us to connect to our server without having to enter a password every time. You can do this by running the following command in your terminal:
```bash
ssh-keygen
```
after running this command, and hitting enter a few times, you should see some beautiful Cryptographic ✨art✨ representing your SSH keys. 

Now, that thats done, lets make sure that our virtual machine works. We can do this by SSHing into our server. You can do this by running the following command in your terminal:
```bash
ssh username@yourserverip
```
Replace username with your username and yourserverip with your server's IP address.
 
If you see a message saying "Are you sure you want to continue connecting (yes/no)?" type yes and hit enter. This will add your server's IP address to your known hosts file. You should now be connected to your server. If you see a message saying "Permission denied", make sure that you're using the correct username and IP address.

Now, exit the SSH session by running the following command:
```bash
exit
```
This will take you back to your local machine.
Now, we need to copy our SSH keys to our server. You can do this by running the following command in your terminal:
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub username@yourserverip
```
Replace username with your username and yourserverip with your server's IP address.

You will be prompted to enter your password. Once you've entered your password, your SSH keys will be copied to your server. You'll see a message saying "Now try logging into the machine, with "ssh 'username@yourserverip'". Try it out! You should now be able to SSH into your server without having to enter a password. If you see a message saying "Permission denied", make sure that you're using the correct username and IP address.

Now what it will do now is the server know what computer we use and it will remeber it. So we can SSH into our server with this key. Back your key up since this is the  way to access your server.

