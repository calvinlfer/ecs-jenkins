# ECS Jenkins

A cloud formation template which runs Jenkins as an ECS service on an existing ECS Cluster.

This template depends on the creation of an ECS cluster using [ECSCluster](https://github.com/ukayani/ecs-cluster#ecsclustertemplate)
 
Once an ECS cluster has been created with the linked template, the stack name should be provided as input to this `ECSJenkins.template`

The ECS cluster should also be using an EFS mount [EFS Mount Template](https://github.com/ukayani/ecs-cluster#efswithmounttargettemplate)

### ECR ###
 
If you plan on using Docker images in conjunction with Jenkins, AWS ECR is an excellent choice as a Container Registry
where you can store private Docker images. A common practice is to have Jenkins slaves build Docker images and push 
these images to ECR as part of the Continuous Delivery process. In order to do this, you need to have the correct IAM 
roles and permissions set up for this to work. 

#### ECR resides in a different account from Jenkins ####

The [ECRCrossAccountRole](ECRCrossAccountRole.yml) should be used when 
your AWS ECR repository is in another account. Execute [ECRCrossAccountRole](ECRCrossAccountRole.yml) on the account
where ECR is hosted (call this Account X) and specify the AWS Account ID which will be able to push images to 
(call this Account Y). This will generate an IAM Role ARN that can be used in Account Y to publish ECR images to 
Account X.

The Jenkins slaves (run as ECS tasks) will push Docker images to ECR. The Jenkins slaves need the proper permissions so
that they can assume the correct IAM role which has the right IAM policies to publish to ECR in another account. You 
can assign the correct permissions that the Jenkins slave ECS tasks will use by executing the
[CrossAccountSlaveRoles](CrossAccountSlaveRoles.yml) CloudFormation template. 

#### ECR resides in the same account as Jenkins ####

The Jenkins slaves (run as ECS tasks) will push Docker images to ECR and need the proper permissions in order to do 
this. You can assign the correct permissions that the Jenkins slave ECS tasks will use by executing the 
[LocalSlaveRoles](LocalSlaveRoles.yml) CloudFormation template.
 
## What is included
 
- Creation of a task/service to run Jenkins
- An ELB which fronts the Jenkins service, allowing communication on port 50000 (to do master/slave) and 80
- Jenkins instance is backed by an EFS volume to store Jenkins data
- Logging to CloudWatch for the Jenkins service 