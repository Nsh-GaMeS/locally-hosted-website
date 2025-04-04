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


Ok, in the mean time, on the cloudflare dashboard, we need to set up a DNS record for our domain name. This will allow us to point our domain name to our server's IP address. Since were running a local server, we can use our public IP address. You can find your public IP address by running the following command in your terminal:
```bash
curl ifconfig.me
```
This will return your public IP address. Copy this address and save it for now. Don't worry about the IP address changing, we will set up a dynamic DNS later on in this guide. For now, just copy your public IP address and save it for later.

Once you have your public IP address, on the DNS settings page, you should see a button that says "Add record". Click on this button and select "A" from the dropdown menu. In the "Name" field, enter your domain name (e.g. example.com). In the "IPv4 address" field, enter your public IP address. Leave the TTL as "Auto" and click on the "Save" button. This will add an "A" record for your domain name that points to your server's IP address.

You should now see the "A" record in the list of DNS records for your domain name. If you don't see it, don't worry, it may take a few minutes to propagate.

__Note:__ If you want to use a subdomain (e.g. hello.website.com), you can enter the subdomain in the "Name" field (e.g. hello). This will create an "A" record for the subdomain that points to your server's IP address.

OK, now that we have our DNS record set up, we can check if it's working by running the following command in your terminal:
```bash
ping yourdomain.com
```
This should show you requests being sent to your server's IP address. If you see requests being sent to your server's IP address, congratulations! Your DNS record is working and pointing to your server's IP address.

Another quick thing to do is to port forward your server to your local machine. You can do this by going to your router's settings and forwarding port 80 (HTTP) and port 443 (HTTPS) to your server's IP address. This will allow you to access your server from anywhere using your domain name.
You can find your router's IP address by running the following command in your terminal:
```bash
ip route | grep default
```
or checking your defualt gateway in your network settings. This will return your router's IP address. Copy this address and save it for now. Once you have your router's IP address, you can go to your router's settings and forward port 80 (HTTP) and port 443 (HTTPS) to your server's IP address. This will allow you to access your server from anywhere using your domain name.

### Bout time, server setup ಥ_ಥ
___

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

Now what it will do now is the server know what computer we use and it will remember it. So we can SSH into our server with this key. Back your key up since this is the  way to access your server.

Once we're in our server, we can start installing all the necessary packages.

Lets configure our server to only use SSH keys to connect to our server. This is a good security practice and will help keep your server secure. You can do this by running the following command in your terminal:
```bash
sudo nano /etc/ssh/sshd_config
```

This will open the sshd_config file in the nano text editor. We need to change(sometimes uncomment/add) the following lines in the file:
```bash
passwordAuthentication no
UsePAM no
ChallengeResponseAuthentication no
```
This means that you will only be able to connect to your server using SSH keys. Once you've made these changes, save the file by pressing `CTRL + X`, then `Y`, then `Enter`.
Now, we need to restart the SSH service for the changes to take effect. You can do this by running the following command in your terminal:
```bash
sudo systemctl restart sshd
```
This will restart the SSH service and apply the changes we made to the sshd_config file.

### Packages
We will be using the following packages:
- nginx
- certbot
- python3-certbot-nginx

First things first, update your system:
```bash
sudo apt update && sudo apt upgrade -y
```

Then will install nginx and certbot:
```bash
sudo apt install nginx certbot python-certbot-nginx -y
```
This will install nginx and certbot on your server.

### Configure nginx
Now that we have nginx installed, we'll set up our configuration file. 
``` bash
cp /etc/nginx/sites-available/default /etc/nginx/sites-available/yourdomain
```
This will create a copy of the default configuration file and name it yourdomain. You can name it whatever you want, but for this guide, we'll use yourdomain.

Lets open the configuration file in nano:
```bash
sudo nano /etc/nginx/sites-available/yourdomain
```
This will open the configuration file in the nano text editor. You should see something like this:
```nginx
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                try_files $uri $uri/ =404;
        }
}
```

We need to change the following lines in the file:
```nginx
server {
        listen 80;
        listen [::]:80;

        root /var/www/domain;
        index index.html index.htm index.nginx-debian.html;

        server_name yourdomain.com www.yourdomain.com; # Change this to your domain name

        location / {
                try_files $uri $uri/ =404;
        }
}
```
Save and exit the file by pressing `CTRL + X`, then `Y`, then `Enter`.

Cool, now thats that done, we'll make a symbolic link to the configuration file in the sites-enabled directory. This will enable the configuration file and make it active. You can do this by running the following command in your terminal:
```bash
sudo ln -s /etc/nginx/sites-available/yourdomain /etc/nginx/sites-enabled/yourdomain
```
This will create a symbolic link to the configuration file in the sites-enabled directory.

So now we need to create the root directory for our website. You can do this by running the following command in your terminal:
```bash
sudo mkdir /var/www/domain
```
Go into the directory:
```bash
cd /var/www/domain
```
Now, we need to create an index.html file in the root directory. You can do this by running the following command in your terminal:
```bash
sudo nano index.html
```
This will open the index.html file in the nano text editor. You can add any HTML code you want to this file. For this guide, we'll just add a simple HTML page that says "Hello World!". You can do this by adding the following code to the index.html file:
```html
<h1>Hello World!</h1>
<p>This is a test page for yourdomain.com</p>
```
Save and exit the file by pressing `CTRL + X`, then `Y`, then `Enter`.


Lets restart nginx to apply the changes.
```bash
sudo systemctl restart nginx
```

### Certbot
Now that we have nginx installed and configured, we'll set up SSL for our website using certbot. This will allow us to use HTTPS for our website and encrypt the traffic between the server and the client.

```bash
certbot --nginx 
```

You'll be prompted to enter your email address and agree to the terms of service. Once you've done that, certbot will automatically configure nginx for SSL and obtain a free SSL certificate from Let's Encrypt.

Now that we have SSL set up, we can test our website. Go to your domain in your browser.
You should see the "Hello World!" page that we created earlier. If you see this page, congratulations! You've successfully set up a web server using nginx and SSL.

Since your SSL certificate will expire after a while, we'll need to renew it every so often. You can do this by running the following command in your terminal:
```bash
sudo certbot renew
```
This will renew your SSL certificate and update the nginx configuration file. But this is long and boring, so we'll set up a cron job to do this for us. You can do this by running the following command in your terminal:
```bash
sudo crontab -e
```
This will open the crontab file in the nano text editor. Add the following line to the end of the file:
```bash
1 1 1 * * certbot renew
```
This will run the certbot renew command every month to renew your SSL certificate. You can change the time and date to whatever you want, but for this guide, we'll run it every month.

### Dynamic DNS

Now that we have our web server set up, we need to set up dynamic DNS. This will allow us to access our server using our domain name even if our public IP address changes. Which it enevitably will. We can do this with many different services, but for this guide, we'll use Cloudflare's dynamic DNS service.

For this guide I'll use this DDNS updater script. [Github](https://github.com/K0p1-Git/cloudflare-ddns-updater/)

In the script, you will need to enter the following:
- Your Cloudflare email
- choose which domain you want to update (if you have multiple domains)
- Your API Token or Global API Key
- Zone ID (you can find this in the Cloudflare dashboard in the overview tab)
- Your record name (e.g. yourdomain.com or hello.yourdomain.com)
- Is proxied (true or false, by default it is true) 

Once you fill out the script, save it and exit.

now lets make that script executable:
```bash
chmod +x cloudflare-ddns-updater.sh
```
Now we need to set up a cron job to run the script every 5 minutes. You can do this by running the following command in your terminal:
```bash
crontab -e
```
This will open the crontab file in the nano text editor. Add the following line to the end of the file:
```bash
*/5 * * * * /path/to/cloudflare-ddns-updater.sh > /dev/null 2>&1
```
Replace /path/to/cloudflare-ddns-updater.sh with the path to the script. This will run the script every 5 minutes and update your DNS record with your public IP address.

That's it! You've successfully set up a web server using nginx and SSL, and set up dynamic DNS using Cloudflare. You can now access your server using your domain name from anywhere!
You can also access your server using HTTPS. If you have any questions or comments, feel free to leave them below. Thanks for reading!

# Conclusion
When making this guide, I was thinking about how to make it as simple as possible. I hope this guide was helpful and easy to follow. 

I also made my own site following the steps from this guide. 
You can check it out at <center>[nsh-games-den.me](https://server.nsh-games-den.me)</center> I will be adding more content to the site in the future, so stay tuned!

__Note:__ To edit the site content, like the html css and js files, you'll need to SSH into your server and edit the files in the /var/www/domain directory. You can do this by running the following command in your terminal:
```bash
ssh username@yourserverip
```
Replace username with your username and yourserverip with your server's IP address.

Once you're in your server, you can navigate to the /var/www/domain directory and edit the files using nano or any other text editor of your choice. You can also use SFTP to transfer files to your server. You can do this by using an SFTP client like FileZilla or WinSCP. Just enter your server's IP address, username, and password, and you should be able to access your server's files.

