## Lab explanation
* Follow the lab explanation from:  
  https://docs.google.com/document/d/1EqHDDQq3PEMahgqfHdYd_dKz73W56KGeeW7Pmn_BQPA/edit  

## Jenkins required plugins:
* Workspace Cleanup
* Pipleline Utility Steps
* Pipleline: Stage View
* Discard Old Build

## Creating AWS credentials
1. Create an IAM group, as explained on:  
 https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups_create.html
2. Attach the policy **"AmazonEC2ReadOnlyAccess"**, as explained on:  
  https://docs.aws.amazon.com/IAM/latest/UserGuide/id_groups_manage_attach-policy.html
3. Create an IAM user for programmatic access only, as explained on:  
  https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html  
  Note 1: Add the IAM user to the previously created group.  
  Note 2: Save the **"credentials.csv"** in a secured location  
4. Create **credentials** file on your local workstation, with the following content:  
  **[default]**  
  **aws_access_key_id = <AWS_ACCESS_KEY>**  
  **aws_secret_access_key = <AWS_SECREST_ACCESS_KEY_ID>**  
  Note: Replace **<AWS_ACCESS_KEY>** and **<AWS_SECREST_ACCESS_KEY_ID>** with the values from the **"credentials.csv"**  

## Uploading AWS credentials file to Jenkins
1. Login to Jenkins console
2. Follow the instructions below and add a new global credential:  
   https://www.jenkins.io/doc/book/using/using-credentials/#adding-new-global-credentials  
   Credential type: **Secret file**  
   When asked to choose a file, select the previously created **credentials** file.  
   ID: **credentials**
3. Click OK

## Uploading Docker Hub credentials to Jenkins
1. Create Docker Hub access token, as instructed below:  
   https://docs.docker.com/docker-hub/access-tokens/#create-an-access-token
2. Login to Jenkins console
3. Follow the instructions below and add a new global credential:  
   https://www.jenkins.io/doc/book/using/using-credentials/#adding-new-global-credentials  
   Credential type: **Username and password**  
   Username: Specify your Docker Hub username.  
   Password: Specify your Docker Hub access token.  
   ID: **dockerhub_id**
4. Click OK

## Updating DockerHub repository
1. Edit the **Jenkinsfile** on the GitHub repository
2. Update the value **DOCKER_REGISTRY** with your target repository

## Create a Jenkins pipeline
1. Login to Jenkins console
2. Follow the instructions in the video below to configure Jenkins pipeline from GitHub repository:  
   https://www.youtube.com/watch?v=56jtwSrNvrs
