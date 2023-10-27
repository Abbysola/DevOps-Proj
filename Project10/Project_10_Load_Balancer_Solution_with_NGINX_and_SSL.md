# Load Balancer Solution with NGINX and SSL

When data is being transferred between a client (browser) and a Web Server over the Internet, it passes through multiple network devices and can be easily intercepted and compromised by cybercriminals if the data is not encrypted. This type of information security threat is called Man-In-The-Middle (MIMT) attack.

To avoid this, SSL and its newer version, TSL – is a security technology that protects connection from these attacks by creating an encrypted session between client and Web server. 

SSL/TLS uses digital certificates to identify and validate a Website. A browser reads the certificate issued by a Certificate Authority (CA) to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.


In this project, we will:
1. configure an NGINX Balancer solution
1. register a new domain name and configure secured connection using SSL/TLS certificates

Here is what our target architecture looks like:
![Architecture](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/1.Architecture.png)

# Configure NGINX as a load balancer

1. Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it Nginx_LB. Open TCP port 80 for HTTP connections and TCP port 443 for secured HTTPS connections.

![Nginx_LB](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/2.Nginx_LB.png)

![Opening ports](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/3.Opening_necessary_ports.png)

2. Update /etc/hosts file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses

![ETC:host update](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/4.Etc%3Ahost_update_with_webservers.png)

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
```

![Editing_nginx_file](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/5.Editing_nginx_config_file.png)

3. Comment out the line below in the code
#       include /etc/nginx/sites-enabled/*;

4. Restart Nginx and make sure the service is up and running

```
sudo systemctl restart nginx
sudo systemctl status nginx
```

![Nginx_status](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/5.Nginx_status.png)


### Register a New Domain Name and Configure Secured Connection using SSL/TLS Certificates.

1. Register a new domain name with any registrar in any domain zone 

2. Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP

Click Allocate new address in the Elastic IPs page.

Then, click Allocate in the next page.

![Associate_elastic_IP](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/6.Associate_elastic_IP.png)

Right-click the row of the newly created elastic IP, and click Associate address.
![Associate_elastic_IP2](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/7.Associate_elastic_IP2.png)

Choose the EC2 instance you are integrating.

![Associate_elastic_IP3](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/8.Associate_elastic_IP3.png)

![Associate_elastic_IP4](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/9.Associate_elastic_IP4.png)

![Associate_elastic_IP5](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/10.Associate_elastic_IP4.png)

![Elastic_IP_associated](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/11.Elastic_IP_associated.png)

![Elastic_IP_associated2](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/12.Elastic_IP_associated2.png)

Note that elastic IP is the solution to having a static IP address that does not change after reboot.

3. Update A record in your registrar to point to Nginx LB using Elastic IP address

4. Update your nginx.conf with server_name www.<your-domain-name.com> instead of server_name www.domain.com

![Updating_the_conf](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/13.Updating_the_conf.png)

5. Create a record on Route 53 for the domain name

![NS_Update1](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/14.NS_Update.png)

![NS_Update2](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/15.NS_Update2.png)

![NS_Update3](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/17.NS_Update3.png)

![NS_Update4](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/18.NS_Update4.png)

![NS_Update5](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/19.NS_Update5.png)

![NS_Update6](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/20.NS_Update6.png)

![NS_Update7](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/21.NS_Update7.png)

![NS_Update8](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/22.NS_Update8.png)

![NS_Update9](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/23.NS_Update9.png)

![NS_Update10](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/24.NS_Update10.png)

![NS_Update11](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/25.NS_Update11.png)

6. Check that your Web Server can be reached from your browser using new domain name using HTTP protocol – http://<your-domain-name.com>

![Domain_name_reachable](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/26.developerabby.com.ng_success.png)

7. Install certbot and request for an SSL/TLS certificate
Make sure snapd service is active and running

```sudo systemctl status snapd```

![Snapd_status](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/27.Snapd_status.png)

Install certbot

```sudo snap install --classic certbot```

Request your certificate (just follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file so make sure you have updated it on step 4).

```sudo ln -s /snap/bin/certbot /usr/bin/certbot```

sudo certbot --nginx -d <domain-name> -d <domain-name>

![Creating_certificate](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/28.Creating_certificate.png)

Test secured access to your Web Solution by trying to reach https://<your-domain-name.com>

![Testing_domain_name_with_https](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/29.Testing_domain_with_https.png)

We can now access our website by using HTTPS protocol (that uses TCP port 443) and a padlock pictogram will be present in the browser’s search string.

![Checking_certificate1](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/30.Checking_certificate1.png)

![Checking_certificate2](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/31.Checking_certificate2.png)

Click on the padlock icon and you can see the details of the certificate issued for your website.     

![Checking_certificate3](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/32.Checking_certificate3.png)

Testing renewal command in dry-run mode

![Testing_renewal](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/33.Testing_renew_with_dry_run.png)

```sudo certbot renew --dry-run```

Scheduling a job that to run renew command periodically. Here, we configure a cronjob to run the command twice a day.

To do so, edit the crontab file with the following command:

```crontab -e```

Add following line:

* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1

![Scheduling_cron_job](https://github.com/Abbysola/DevOps-Proj/blob/main/Project10/Images/34.Scheduling_cron_job.png)

To change the interval, the schedule expression can be adjusted.
