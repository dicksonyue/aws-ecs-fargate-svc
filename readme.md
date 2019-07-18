## Launch the stack (10 mins)
-	Launch the Cloudformation template cf.yaml, I tested in us-west-2, it should work in other region, play safe launch in us-west-2
-	You should find the ALB link in the output

## Setup Codecommit

### a new repo
1. create a new repo in codecommit
2. create a IAM user for git user, add codecommitfull access for git push, pull....  https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-gc.html?icmpid=docs_acc_console_connect_np
3. under IAM user, create Git Credentials , this is the login during the git clone

### push a new version to repo
1. git config --global credential.helper '!aws --profile HERE_YOUR_CLI_PROFILE_NAME codecommit credential-helper $@'
2. git init
3. git add *
4. git commit -m "first commit"
5. git remote add origin {git-url}
6. git push -u origin master
7. Login with the Git Credential

## Setup Jenkins EC2

1. Launch a t3 small ec2
2. Associate a EIP to the EC2
3. Security group - inbound 8080 port 0.0.0.0/0
4. sudo yum update –y
5. sudo yum install java
6. sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkinsci.org/redhat/jenkins.repo
7.  sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
8. sudo yum install jenkins -y
9. sudo service jenkins start
10. Visit to the Jenkins {EIP}:8080
11. Follow the setup guide
12. Install Plugins
13. CloudBees Docker Build and Publish plugin, Slack Notification Plugin
14. sudo yum install git -y
15. sudo amazon-linux-extras install docker
16. sudo usermod -a -G docker ec2-user
17. sudo usermod -a -G docker jenkins
18. sudo service jenkins restart

## Setup Jenkins project
### API
1. New project, freestyle
2. Source Code Management , set the repo url and credential
3. Build step 1, docker image build push throght "Docker build and push plugin"
4. Build step 2, deployment. update the ecs task definition with the latest built docker image. update through cloudformation.

