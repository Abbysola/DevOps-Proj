# Load Balancer Solution with NGINX and SSL

When data is being transferred between a client (browser) and a Web Server over the Internet, it passes through multiple network devices and can be easily intercepted and compromised by cybercriminals if the data is not encrypted. This type of information security threat is called Man-In-The-Middle (MIMT) attack.

To avoid this, SSL and its newer version, TSL – is a security technology that protects connection from these attacks by creating an encrypted session between client and Web server. 

SSL/TLS uses digital certificates to identify and validate a Website. A browser reads the certificate issued by a Certificate Authority (CA) to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.


In this project, we will:
1. configure an NGINX Balancer solution
1. register a new domain name and configure secured connection using SSL/TLS certificates

Here is what our target architecture looks like:
![Architecture]()

# Configure NGINX as a load balancer

1. Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx_LB. Open TCP port 80 for HTTP connections and TCP port 443 for secured HTTPS connections.

![Nginx_LB]()

![Opening ports]()

2. Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses

![ETC:host update]()

3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

4. Update the instance and Install Nginx

```
sudo apt update
sudo apt install nginx
```

5. Configure Nginx LB using the Web Servers’ names defined in /etc/hosts

1. Open the default nginx configuration file by running:

```sudo vi /etc/nginx/nginx.conf```

2. Insert the following configuration into http section

```
 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

![Editing_nginx_file]()

3. Comment out the line below in the code
#       include /etc/nginx/sites-enabled/*;

4. Restart Nginx and make sure the service is up and running

```
sudo systemctl restart nginx
sudo systemctl status nginx
```

![Nginx_status]()


### Register a New Domain Name and Configure Secured Connection using SSL/TLS Certificates.

1. Register a new domain name with any registrar in any domain zone 

2. Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP

Click Allocate new address in the Elastic IPs page.

Then, click Allocate in the next page.

![Associate_elastic_IP]()

Right-click the row of the newly created elastic IP, and click Associate address.
![Associate_elastic_IP2]()

Choose the EC2 instance you are integrating.

![Associate_elastic_IP3]()

![Associate_elastic_IP4]()

![Associate_elastic_IP5]()

![Elastic_IP_associated]()

![Elastic_IP_associated2]()

Note that elastic IP is the solution to having a static IP address that does not change after reboot.

3. Update A record in your registrar to point to Nginx LB using Elastic IP address

4. Update your nginx.conf with server_name www.<your-domain-name.com> instead of server_name www.domain.com

![Updating_the_conf]()

5. Create a record on Route 53 for the domain name

![NS_Update1]()

![NS_Update2]()

![NS_Update3]()

![NS_Update4]()

![NS_Update5]()

![NS_Update6]()

![NS_Update7]()

![NS_Update8]()

![NS_Update9]()

![NS_Update10]()

6. Check that your Web Server can be reached from your browser using new domain name using HTTP protocol – http://<your-domain-name.com>

![Domain_name_reachable]()

7. Install certbot and request for an SSL/TLS certificate
Make sure snapd service is active and running

```sudo systemctl status snapd```

![Snapd_status]()

Install certbot

```sudo snap install --classic certbot```

Request your certificate (just follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file so make sure you have updated it on step 4).

```sudo ln -s /snap/bin/certbot /usr/bin/certbot```

sudo certbot --nginx -d <domain-name> -d <domain-name>

![Creating_certificate]()

Test secured access to your Web Solution by trying to reach https://<your-domain-name.com>

![Testing_domain_name_with_https]()

We can now access our website by using HTTPS protocol (that uses TCP port 443) and a padlock pictogram will be present in the browser’s search string.

![Chacking_certificate1]()

![Checking_certificate2]()

Click on the padlock icon and you can see the details of the certificate issued for your website.     

![Checking_certificate3]()

Testing renewal command in dry-run mode

![Testing_renewal]()

```sudo certbot renew --dry-run```

Scheduling a job that to run renew command periodically. Here, we configure a cronjob to run the command twice a day.

To do so, edit the crontab file with the following command:

```crontab -e```

Add following line:

* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1

![Scheduling_cron_job]()

To change the interval, the schedule expression can be adjusted.
