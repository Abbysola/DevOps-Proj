# Tooling Website Deployment Autoomation with Continuous Integration (Introduction to Jenkins)

In previous Project 8, I used the horizontal scalability concept to allow addition of new Web Servers to the Tooling Website. I successfully deployed a set up with 2 Web Servers and also a Load Balancer to distribute traffic between them. This concept will be very useful when there's a need to deploy dozens or even hundreds of servers.

In this project, I am going to start automating part of our routine tasks with a free and open source automation server known as Jenkins. It is one of the most popular CI/CD tools. It was created by a former Sun Microsystems developer Kohsuke Kawaguchi and the project originally had a named “Hudson”.

Jenkins is an open source continuous integration/continuous delivery and deployment (CI/CD) automation software DevOps tool written in the Java programming language. It is used to implement CI/CD workflows, called pipelines.

According to Circle CI, Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. Developers continually commit code in small increments (at least daily, or even several times a day), which is then automatically built and tested before it is merged with the shared repository.

In this project, I will utilize Jenkins CI capabilities to make sure that every change made to the source code in GitHub https://github.com/<mygithubname>/tooling will be automatically be updated to the Tooling Website. Here, I will choose Git repository, provide the link to the GitHub repository and credentials (user/password) so Jenkins can access files in the repository.

### Here is how the updated architecture will look like upon competition of this project:

![Architecture](https://github.com/Abbysola/DevOps-Proj/blob/62dd732f890f5a75ef3fc5b0ded62063842badc6/Project9/Images/1.Architecture.png)   `

## Install and Configure Jenkins Server

1. Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it “Jenkins”

![Jenkins_server]()

2. Install JDK (since Jenkins is a Java-based application)

```
sudo apt update
sudo apt install default-jdk-headless
```

![Installing_JDK]()

3. Install Jenkins

```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins
```

### Tip
If you encounter an error such as shown in the image below:

![GPG_key_error]()

Here's what to know and do:

The error message you encountered is related to GPG (GNU Privacy Guard) key verification for the Jenkins repository. It indicates that the public key required to verify the authenticity of the repository is not available on your system. Without the public key, APT (Advanced Package Tool) considers the repository to be untrusted and disables updates from that repository by default.

To resolve this issue, you can import the missing public key using the apt-key command.

Run the following command to retrieve the missing GPG key:

```sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 5BA31D57EF5975CA```

Make sure Jenkins is up and running

```sudo systemctl status jenkins```

![Jenkins_status]()

4. By default Jenkins server uses TCP port 8080. I opened it by creating a new Inbound Rule in my EC2 Security Group.

![TCP_port8080]()

5. Performing initial Jenkins setup.
From my browser, I gained acces to Jenkins' interactive console by pasting this on browser:

```http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080```

To retrieve the default admin password, run:

```sudo cat /var/lib/jenkins/secrets/initialAdminPassword```

![Jenkins_console]()

![Customize_Jenkins]()

![Getting_Started]()

![Create_admin_user]()

![]()

## Configure Jenkins to retrieve source codes from GitHub using Webhooks

Here, I would demonstrate how to configure a simple Jenkins job/project. This job will will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

1. Enable webhooks in your GitHub repository settings

![Enabling-webhook]()

2. Go to Jenkins web console, click “New Item” and create a “Freestyle project”

![Freestyle_Jenkins_Project]()

To connect to the GitHub repository, provide its URL.

![Copying_repo_link]()

Configure the Jenkins freestyle job. Provide the link to the GitHub repository and its credentials (user/password) so that Jenkins can access files in the repository.

![Accessing_the_repo]()

Save the configuration. 

## Configuring the build manually
Click “Build Now” button. If you have configured everything correctly, the build will be successfull and you will see it under #1

![Configured_build]()

You can open the build and check in “Console Output” to see that it has run successfully. This is a Jenkins build that was configured manually. Howevere, this build does not produce anything because it is manually configured.

## Configuring Build Triggers

1. Click “Configure” your job/project and add these two configurations

a. Configure triggering the job from GitHub webhook:


b. Configure “Post-build Actions” to archive all the files. Files resulted from a build are called “artifacts”.

2. Make any change to any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.

You will see that a new build has been launched automatically (by webhook) and you can see its results (artifacts), saved on Jenkins server.

Now, an automated Jenkins job that receives files from GitHub by webhook trigger has been configured. This method is known as ‘push’ and the files transfer is initiated by GitHub. 

Other methods used include trigger one job (downstreadm) from another (upstream), poll GitHub periodically etc.

By default, the artifacts are stored on Jenkins server locally. You can check them here:

```ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/```

## Configure Jenkins to Copy Files to NFS Server via SSH

The artifacts are now saved locally on the Jenkins server. Here, we would copy them to our NFS server (/mnt/apps) directory.

To do this, a plugin called “Publish Over SSH” will be used.

1. Install “Publish Over SSH” plugin.

On the main Jenkins dashboard, select “Manage Jenkins” and choose “Manage Plugins” menu item.

On “Available” tab, search for “Publish Over SSH” plugin and install it.

2. Configure the job/project to copy artifacts over to NFS server.

On main dashboard, select “Manage Jenkins” and choose “Configure System” menu item.

Scroll down to 'Publish over SSH' plugin configuration section and configure it to be able to connect to your NFS server.

a. Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty). To copy the file, navigate to your .pem file directory.

```cd /path/to/pem-file-directory/```

Use the 'cat' command to view the contents of the .pem file.

```cat file.pem```

b. Hostname – can be private IP address of your NFS server

c. Username – ec2-user (since NFS server is based on EC2 with RHEL 8)

d. Remote directory – /mnt/apps since your Web Servers use it as a mointing point to retrieve files from the NFS server. 

You can test the configuration and make sure it is successful. Note that TCP port 22 on NFS server must be open to receive SSH connections.

e. Save the configuration 

3. Open your Jenkins job/project configuration page and add another “Post-build Action”.

4. Configure it to send all files from the build into the previously defined remote directory. Here, all files and directories will be copied using **.

5. Save the configuration.

6. Make a change in the README.MD file in your GitHub Tooling repository.

Webhook will trigger a new job and in the “Console Output” of the job, there would be a success maessage at the bottom.

7. To make sure that the files in /mnt/apps have been updated, connect via SSH/Putty to your NFS server and check README.MD file

```cat /mnt/apps/README.md```

If the changes previously made in your README.MD file reflects, it shows that the job works as expected.

































