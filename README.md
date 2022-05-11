## AWS DevSecOps Pipeline

Kubernetes DevSecOps pipeline using AWS cloudnative services and open source security vulnerability scanning tools.

This DevSecOps pipeline uses AWS DevOps tools AWS CodeCommit, AWS CodeBuild, and AWS CodePipeline along with other AWS services.  It is highly recommended to fully test the pipeline in lower environments and adjust as needed before deploying to production.

### Build and Test: 

The buildspecs and property files for vulnerability scanning using AWS CodeBuild:
* buildspec-gitsecrets.yml: buildspec file to perform secret analysis using open source git-secrets.
* buildspec-anchore.yml: buildspec file to perform SCA/SAST analysis using open source Anchore.
* buildspec-ecr-yml: buildspec file to retrive ECR SCA/SAST analysis results and deploy to staging EKS cluster.
* buildspec-owasp.yml: buildspec file to perform DAST analysis using open source OWASP ZAP.
* buildspec-falco.yml: buildspec file to perform RASP scan on the eks cluster and store the results in Cloudwatch alerts log group,
                       This file also contains Starboard: Kubernetes-native security toolkit to scan the eks workloads.

### Lambda files:

AWS lambda is used to parse the scanning analysis results and post it to AWS Security Hub
* import_findings_security_hub.py: to parse the scanning results, extract the vulnerability details.
* securityhub.py: to post the vulnerability details to AWS Security Hub in ASFF format (AWS Security Finding Format).

### Docker Files:
* Dockerfile contains the instructions of the sampleapplication we are going to build.

### eks_files:
* deployment.yaml and serice.yaml files are to deploy into the eks cluster we have created.

## Prerequisites

1. An EKS cluster environment with your application deployed. In this post, we use NodeJS as a sample application, but you can use any other application.
2. Sysdig Falco installed on an EKS cluster. Sysdig Falco captures events on the EKS cluster and sends those events to CloudWatch using AWS FireLens. For implementation instructions, see Implementing Runtime security in Amazon EKS using CNCF Falco. This step is required only if you need to implement RASP in the software factory.
3. A CodeCommit repo with your application code and a Dockerfile.
4. An Amazon ECR repo to store container images and scan for vulnerabilities. Enable vulnerability scanning on image push in Amazon ECR. You can enable or disable the automatic scanning on image push via the Amazon ECR
5. The provided buildspec-*.yml files for git-secrets, Anchore, Amazon ECR, OWASP ZAP, and your Kubernetes deployment .yml files uploaded to the root of the application code repository. Please update the Kubernetes (kubectl) commands in the buildspec files as needed.
6. The Lambda function uploaded to an S3 bucket. We use this function to parse the scan reports and post the results to Security Hub.
7. An application web URL to run the DAST testing.
8. AWS Config and Security Hub services enabled. For instructions, see Managing the Configuration Recorder and Enabling Security Hub manually, respectively.


## Cleanup

1. Delete the EKS cluster.
2. Delete the S3 bucket.
3. Delete the CodeCommit repo.
4. Delete the Amazon ECR repo.
5. Disable Security Hub.
7. Delete the pipeline.
