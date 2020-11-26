## My Project

# Overview
Through steps discussed in the sections below, you can automate the creation of infrastructure that includes a pipeline connected to a CodeCommit source repository, picks up new / changed CloudFormation (CF) template from the repository and deploys the template into the environment.
The solution is composed of 2 templates:
1.	SetupDeploymentPipeline.json is the solution CF template to automate the build / setup of the deployment pipeline discussed above. Click [here](https://github.com/aws-samples/containers-fargateprofile-devops-via-codepipeline/blob/main/SetupDeploymentPipeline.json) to download the file.
2.	FargateProfile.json is example CF template to demo / test the deployment of Fargate profile (FP) in your EKS environment. You should be able to replace this with any other CF template of your choice based on your needs.  With little to no adjustment to the solution CF template, the solution can be used to support deployment of template for other AWS resources as well.  Click [here](https://github.com/aws-samples/containers-fargateprofile-devops-via-codepipeline/blob/main/FargateProfile.json) to download the file.  
The AWS services and capabilities in play to realize this solution are: AWS Code Pipeline, AWS CloudWatch Event Rule, AWS S3, AWS Code Commit and AWS CloudFormation. 

# Deployment Pipeline Solution Design
The solution template SetupDeploymentPipeline.json file when used, will create CF resource stack, which includes your artifact store, pipeline, and change-detection resources, such as your Amazon CloudWatch Events rule. After you create your resource stack in AWS CloudFormation, you can view your pipeline in the AWS CodePipeline console. The pipeline is a two-stage pipeline with a CodeCommit source stage and a CodeDeploy deployment stage as depicted in the diagram below:

![](https://github.com/aws-samples/containers-fargateprofile-devops-via-codepipeline/blob/main/Diagram.png)

Flow:
1.	DevOps Admin commits Fargate Profile CF Template
2.	Fargate Profile is uploaded to CodeCommit Repository.
3.	AWS Code Pipeline gets triggered to kick off the execution.
4.	Code Pipeline invokes the Source and Deploy stages part of the solution, where Source stage pulls the FP template from the repository and passes it to Deploy stage. Deploy stage using CloudFormation provider feature, create CF stack to deploy the template.
5.	EKS Cluster is updated to reflect the Fargate profile based on the definition in the template.

Below are the steps to follow to deploy the code pipeline solution discussed above using the solution template. 

# Deployment Steps
There are three main steps in launching this solution: preparing an AWS account and pre-requisite setup, launching the stack, and testing the deployment. Each is described in more detail in this section.

## Step 1. Prepare an AWS Account and Pre-requisite Setup
1.	If you don’t already have an AWS account, create one at http://aws.amazon.com by following the on-screen instructions. Part of the sign-up process involves receiving a phone call and entering a PIN using the phone keypad. Be sure you’ve signed up for the CloudFormation service.
2.	Use the region selector in the navigation bar of the console to choose the desired AWS region
3.	Create a key pair. This is optional, since we’ll be using AWS Management Console for deployment. However, if you prefer to use CLI from your workstation, this is required step. 
4.	You must complete following pre-requisites to deploy this solution: 
> 1.	Create an S3 bucket in the AWS region targeted for deployment. S3 bucket will be used for 2 purposes:
> > 1.	To house zipped file containing initial version of FargateProfile.json CF template to be used in the pipeline.
> > 2.	To store pipeline event outputs upon pipeline jobs execution.
> 2.	Review the provided FargateProfile.json file and make sure the resource definition is valid, i.e. “ClusterName”, ”FargateProfileName” and “PodExecutionRoleArn” are set per your environment, where ClusterName and PodExecutionRoleArn in the template should be updated to point to resources existing in your environment. 
  If don’t exist already, created these resources before moving any further with this solution.  
> 3.	Create a zip called “FargateProfile.json.zip” containing updated FargateProfile.json, and upload it into the S3 bucket created step 1 above.

## Step 2. Launch the Stack

Create CF stack using AWS CloudFormation template file SetupDeploymentPipeline.json. 
You can create the Stack by uploading the template using AWS Management Console (or CLI, if you prefer). Refer [CF document](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) for steps on how to create the stack.

Before you launch the stack, review the architecture, resources configuration. The template includes default settings that you can further customize to your environment, if you like. 

Make sure to supply desired value for the following parameters when launching the stack:
1. RepositoryName: Name of the CodeCommit repository you’d like created as part of the stack creation. This repo will house CF template to be used by Code pipeline to deploy resources. Default is “FargateRepo”. 
1. BranchName: Branch Name for the repository. Default is “master”.
1. CodePipelineArtifactStoreBucket: It is the S3 bucket name created in Step 1 above. Default is “fargatepipelinebucket”
1. FPTemplateName: This is the initial Fargate profile CloudFormation template name uploaded to S3 bucket. Default is “FargateProfile.json”

**Create Details**
Here’s a listing of the key AWS resources that are created when this stack is launched:
* IAM – Policy and Role
* CodeCommit Repository – Hosts Fargate Profile templates for deployment
* CloudWatchEventRule – To trigger the execution of Code pipeline
* CodePipeline – deployment pipeline with CodeCommit Integration and CodeDeploy stage.

CLI Example
Alternatively, you can launch the same stack from the command line. For details, refer [Create Stack via CLI](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-cli-creating-stack.html ) 

## Step 3. Review the Deployment
Upon successfully execution of deployment steps, you should see the Fargate profile added to EKS Cluster. 

**To verify profile addition to your cluster via the console**
1.	Open the Amazon EKS console at [EKS Console](https://console.aws.amazon.com/eks/home#/clusters.)
2.	Click on the EKS Cluster in question.
3.	Go to compute tab and verify that Fargate profile is added.

**To review code pipeline and history via the console**
Follow steps outlined
[here](https://docs.aws.amazon.com/codepipeline/latest/userguide/pipelines-view-console.html#pipelines-list-console)

# Step 4. Cleanup the Environment
When done using the solution, make sure that you delete the infrastructure and resources that were created part of the deployment steps above. So that, you don't continue to incur any AWS charges.  To delete the resource:
1. Delete the CloudFormation stack for the FP profile template first. Stack name "FPDeployment", this is the stack that was created coz of successful execution of the Deploy stage in the piepline. For steps to delete the stack from AWS Management Console, click
[here](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-delete-stack.html)
2. Delete the CloudFormation stack for the solution CF template. This is the stack you created during "Step 2. Launch the Stack" section above. 
3. Delete the S3 bucket created in "Step 1. Prepare an AWS Account and Pre-requisite Setup" above.

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

