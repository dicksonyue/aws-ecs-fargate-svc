## Launch the stack (10 mins)
-	Launch the Cloudformation template cf.yaml, I tested in us-west-2, it should work in other region, play safe launch in us-west-2
-	You should find the ALB link in the output

## Config  Jenkins (1 hr)
1.	Launch a t2.medium ec2
2.	Bootstrap like here https://d1.awsstatic.com/Projects/P5505030/aws-project_Jenkins-build-server.pdf
a.	[ec2-user ~]$ sudo yum update –y
b.	Add the Jenkins repo using the following command:
[ec2-user ~]$ sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkinsci.org/redhat/jenkins.repo
c.	3Import a key file from Jenkins-CI to enable installation from the package:
[ec2-user ~]$ sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
d.	Install Jenkins:
[ec2-user ~]$ sudo yum install jenkins -y
e.	Start Jenkins as a service:
[ec2-user ~]$ sudo service jenkins start
3.	Install a few plugin
a.	CloudBees Docker Build and Publish plugin
b.	GitHub Authentication plugin
c.	GitHub Integration Plugin
d.	GitHub plugin
e.	Slack Notification Plugin (optional)
4.	Config
a.	Follow here Step 7: Configure Jenkins
http://docs.aws.amazon.com/AWSGettingStartedContinuousDeliveryPipeline/latest/GettingStarted/CICD_Jenkins_Pipeline.html , plus my sreen cap
./jenkins-config
                                                              i.      Source Code Management – git
                                                            ii.      Build trigger – Github hook
                                                          iii.      Build      
1.	#!/bin/bash
2.	DOCKER_LOGIN=`aws ecr get-login --region us-west-2`
3.	${DOCKER_LOGIN}
4.	echo $GIT_COMMIT
                                                           iv.      (ADD)Docker Build and Publish
1.	Repository Name
2.	Docker registry URL (create your own ECR under ECS)
3.	Advanced (click)
a.	 Build context set “./svc-10001/”
b.	Repeat this for 10002 and 10003
                                                             v.      (ADD) Execute shell
1.	#!/bin/bash
2.	AWSREGION=us-west-2
3.	CFFILE=cf.yaml
4.	CFSTACKNAME=ecsv1
5.	ImgSvc10001={ecr-url}/aws-voting-app-svc-10001:$GIT_COMMIT
6.	ImgSvc10002={ecr-url}/aws-voting-app-svc-10002:$GIT_COMMIT
7.	aws cloudformation update-stack --template-body file://$CFFILE  --parameters ParameterKey=SubnetId,ParameterValue=subnet-6712a002\\,subnet-6c50801b ParameterKey=VpcId,ParameterValue=vpc-953ae1f0 ParameterKey=ImgSvc10001,ParameterValue=$ImgSvc10001 ParameterKey=ImgSvc10002,ParameterValue=$ImgSvc10002 --capabilities CAPABILITY_IAM --stack-name $CFSTACKNAME --region $AWSREGION
