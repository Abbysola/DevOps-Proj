# Tooling Website Deployment Autoomation with Continuous Integration (Introduction to Jenkins)

In previous Project 8, I used the horizontal scalability concept to allow addition of new Web Servers to the Tooling Website. I successfully deployed a set up with 2 Web Servers and also a Load Balancer to distribute traffic between them. This concept will be very useful when there's a need to deploy dozens or even hundreds of servers.

In this project, I am going to start automating part of our routine tasks with a free and open source automation server known as Jenkins. It is one of the most popular CI/CD tools. It was created by a former Sun Microsystems developer Kohsuke Kawaguchi and the project originally had a named “Hudson”.

Jenkins is an open source continuous integration/continuous delivery and deployment (CI/CD) automation software DevOps tool written in the Java programming language. It is used to implement CI/CD workflows, called pipelines.

According to Circle CI, Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. Developers continually commit code in small increments (at least daily, or even several times a day), which is then automatically built and tested before it is merged with the shared repository.

In this project, I will utilize Jenkins CI capabilities to make sure that every change made to the source code in GitHub https://github.com/<mygithubname>/tooling will be automatically be updated to the Tooling Website. Here, I will choose Git repository, provide the link to the GitHub repository and credentials (user/password) so Jenkins can access files in the repository.

### Here is how the updated architecture will look like upon competition of this project:

![Architecture]()   `

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
1. Click “Build Now” button. If you have configured everything correctly, the build will be successfull and you will see it under #1































