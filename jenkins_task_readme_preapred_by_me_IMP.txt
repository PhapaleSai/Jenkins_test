This is my Jenkins cicd pratical steps which i have done 
EC2 instance — launch & security group

AMI: Ubuntu Server 22.04 (or 24.04).

Instance type: t2.micro (Free Tier).

Key pair: create and download .pem (you’ll use this for SSH / MobaXterm).

Security Group inbound rules (example):

SSH (TCP 22) → Your IP only (use My IP in console)

HTTP (TCP 80) → 0.0.0.0/0 (so Apache can be reached)

Custom TCP (TCP 8080) → 0.0.0.0/0 (so GitHub can reach Jenkins UI webhook endpoint)
IMP Please download the key which you have needed for creating ec2 instance eg my key name is new_account.pem 

# update
    2  sudo apt update -y
    3  sudo apt upgrade -y
    4  # install apache & git
    5  sudo apt install -y apache2 git
    6  # (optional) install Node.js 18 LTS if your project uses npm
    7  curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
    8  sudo apt install -y nodejs build-essential
    9  # install Java (OpenJDK) required by Jenkins
   10  sudo apt install -y fontconfig openjdk-21-jre
   11  java -version   # verify
   12  # Add Jenkins apt repository (LTS) and install Jenkins
   13  sudo mkdir -p /etc/apt/keyrings
   14  sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
   15  echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
   16  sudo apt update
   17  sudo apt install -y jenkins
   18  # start and enable Jenkins
   19  sudo systemctl enable --now jenkins
   20  sudo systemctl status jenkins  # quick check
   21  # make deploy directory for web app
   22  sudo mkdir -p /var/www/html/webdirectory
   23  # give Jenkins ownership so it can write there (no sudo from builds needed)
   24  sudo chown -R jenkins:jenkins /var/www/html/webdirectory
   25  sudo chmod -R 755 /var/www/html/webdirectory
   26  java --version
   27  jenkins --version
	git --help or version to check if git is install or not if you choosing ubuntu server it will have git installed by default
   28  sudo cat /var/lib/jenkins/secrets/initialAdminPassword
	enter
	credentials like 
	name:- sai
	password:1
	confirm password : 1
	full name : sai
	email :- sai@gmail.com
   29  history after that i have created a new repo in my github with public access and inserted a simple hi page in it <!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Hi Page</title>
</head>
<body>
  <h1>👋 Hi, Welcome!</h1>
  <p>This is a simple HTML page created by me.</p>
</body>
</html> 

Go in the aws ec2 and open the public ip where you will see in my case is the ubuntu page
after that go in the Jenkins browser 
enter creadentials or u are already logged in 
then go in manage jenkisn 
in available plugins search for "publish over ssh" and install it

after installing go to dashboard and then in manage Jenkins
click system configuration:-
then go in last you will find
"Publish over SSH"
A configuration to use to connect to a SSH server
Jenkins SSH Key

Open the new_account.pem its the key, which you have needed for creating ec2 instance eg my key name is new_account.pem 

copy its content 
it will start like 
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEA2REio81P.....
..............MIIEogIBAAKCAQEA
------END RSA PRIVATE KEY-----

then add the key content to Key here

then click on ADD below it which shows this:-
SSH Servers
Add

then after clicking add Enter details like :-
Name :- DemoServer

Hostname : is your ec2 instance public ip go in ec2 dashboard for getting this select instance and scroll below you will see
something like 
Instance Summary :-
Public IPv4 address
3.110.102.25
copy ip and then paste it in the hostname.

Username:- i have used ubuntu so i am entering username ubuntu if you have used Amazon Linux then enter username as ec2-user

Remote Directory :- /var/www/html/webdirectory/ it is the like index.html site which will show when page/website is loaded

after filling details press Test Configuration 
it will show  Success if everthing is good that is in my case 

then click on apply and then click on save

It will redirect you to dashboard then 
click on new item.
give it name :- demo_project
and click on freestyle project.

Choose GitHub Project 
Project Url:-
"https://github.com/PhapaleSai/Jenkins_test/blob/main/index.html" this the project file which you want to show

then go in Source Code Management
choose Git
Repositories
Repository URL :- https://github.com/PhapaleSai/Jenkins_test.git :- your https link of repo from GitHub
Credentials keep none as we are using a public repository only so no need to give credentials 
Branch Specifier (blank for 'any')
*/main : it is main branch in my case so i have written here main branch

Then in the Triggers choose or click the 
" GitHub hook trigger for GITScm polling"

its important change here 
go in GitHub
to your public reposirory for which you have entered path 
click settings 
go in Code and Automations
click on Webhooks
add wehook 
Payload URL *
:- http://3.110.102.251:8080/github-webhook/ :-3.110.102.251:8080 this is my Jenkins browser/server ip paste it here .

go below 
Which events would you like to trigger this webhook?
choose Just the push event.

just add the webhook in my case 

Now again back to Jenkins server 
you will see Environment
choose : Send files or execute commands over SSH after the build run 
then you will see
SSH Server
Name : DemoServer which we configured before so use this 
then below it you will find
Transfer Set
	Source files
	- **/*<extensions> :-base syntax
	- **/*html,**/*css  :-these are the file which i am having in my project so please use only html and css file in my case i have only a single html file so **/*html this can work for me basically these are extension of files which you are having in the GitHub repository 

	Remove prefix
		keep blank

	Remote directory
	 we have already setup /var/www/html/webdirectory/ inside the publish over ssh so just
	enter " / " forward slash here.


Go in Post-build Actions
Choose :- Send build artifacts over SSH
Name : DemoServer which we configured before so use this 
then below it you will find
Transfer Set
	Source files
	- **/* :to take all files
	Remove prefix
		keep blank

	Remote directory
	 we have already setup /var/www/html/webdirectory/ inside the publish over ssh so just
	enter " / " forward slash here.


go in last click on apply and save


Hurray our pipeline is created 


for testing go in GitHub
inside your index.html
change some conetent and commit it 

go in your ssh instance that is your ec2instace
enter commands like
cd /var/www/
sudo chmod 777 html 
sudo chown -R ubuntu:ubuntu /var/www/html/webdirectory :- this will give permission to ubuntu server
sudo chmod -R 755 /var/www/html/webdirectory

Go in 
/var/www/html/webdirectory
to check files are availbe are not if file is available then pipeline is succesfull build

i had got errors 
sudo vim /etc/apache2/sites-enabled/000-default.conf
press i to go in insert mode
Change :-
   DocumentRoot /var/www/html
to this :-
	DocumentRoot /var/www/html/webdirectory

:wq!:-to exit vim

sudo systemctl restart apache2 
Now you will see GitHub conetent on hitting ec2instance ip :- 
 ec2 instance public ip go in ec2 dashboard for getting this select instance and scroll below you will see
something like 
Instance Summary :-
Public IPv4 address
3.110.102.25

so like whenever i do changes in my GitHub index.html or any file it will reflect on the ec2-instancepage
This is pratical 

so like when you visit after some time .
DO this following changes 
9) WHEN YOU START/STOP THE INSTANCE CHANGE THE IP IN THE FOLLOWING :

10) CHANGE THE HOSTNAME (UPDATED IP)(PAYLOAD URL) IN GITHUB WEBHOOK (GO INSIDE OF THE REPO->SETTINGS->WEBHOOK'S)
11) IN JENKINS -> MANAGE JENKINS -> SYSTEM -> LOOK FOR PUBLISH OVER SSH
12) GO IN THE JOB, CLICK ON APPLY -> SAVE
13) GO TO YOUR PIPEPLINE AND THEN CLICK ON BUILD NOW
----------------------------------















